# Unity Editor Automation Patterns

Use these patterns when a VRChat avatar modding task benefits from deterministic Unity automation. Keep scripts project-local, usually under `Assets/Editor/`, and write logs into `Logs/`.

## Editor Script Shape

```csharp
using System;
using System.IO;
using System.Linq;
using System.Text;
using nadena.dev.modular_avatar.core;
using nadena.dev.ndmf;
using UnityEditor;
using UnityEditor.Animations;
using UnityEditor.SceneManagement;
using UnityEngine;
using UnityEngine.SceneManagement;
using VRC.SDK3.Avatars.Components;
using VRC.SDK3.Avatars.ScriptableObjects;

public static class AvatarWorkUtility
{
    [MenuItem("Tools/Avatar Work/Rebuild Workspace")]
    public static void RebuildWorkspace()
    {
        var log = new StringBuilder();
        // Discover or load assets by path after they have been identified.
        // Instantiate base avatar, attach clothing, configure MA, clone menus, save scenes/prefab.
        File.WriteAllText("Logs/avatar-work-report.txt", log.ToString(), Encoding.UTF8);
    }
}
```

For long operations, add marker-file autorun only when useful:

```csharp
[InitializeOnLoadMethod]
private static void AutoRunWhenRequested()
{
    var marker = Path.Combine(Directory.GetParent(Application.dataPath).FullName, "Temp", "run_avatar_work");
    if (!File.Exists(marker)) return;
    if (EditorApplication.isPlaying)
    {
        EditorApplication.isPlaying = false;
        return;
    }
    File.Delete(marker);
    EditorApplication.delayCall += RebuildWorkspace;
}
```

## Descriptor Configuration

```csharp
private static void ApplyDescriptor(
    VRCAvatarDescriptor descriptor,
    VRCExpressionsMenu menu,
    VRCExpressionParameters parameters,
    RuntimeAnimatorController fx)
{
    descriptor.expressionsMenu = menu;
    descriptor.expressionParameters = parameters;

    for (var i = 0; i < descriptor.baseAnimationLayers.Length; i++)
    {
        var layer = descriptor.baseAnimationLayers[i];
        if (layer.type != VRCAvatarDescriptor.AnimLayerType.FX) continue;
        layer.isDefault = false;
        layer.animatorController = fx;
        descriptor.baseAnimationLayers[i] = layer;
        break;
    }

    EditorUtility.SetDirty(descriptor);
}
```

Clear upload blueprint IDs on generated work prefabs:

```csharp
private static void ClearBlueprintId(GameObject avatarRoot)
{
    foreach (var component in avatarRoot.GetComponents<Component>())
    {
        if (component == null || component.GetType().Name != "PipelineManager") continue;
        var field = component.GetType().GetField("blueprintId");
        if (field == null) continue;
        if (!string.IsNullOrEmpty(field.GetValue(component) as string))
        {
            field.SetValue(component, string.Empty);
            EditorUtility.SetDirty(component);
        }
    }
}
```

## Modular Avatar Configuration

```csharp
private static void ConfigureClothing(Transform avatarRoot, Transform clothingRoot, VRCExpressionsMenu outfitMenu)
{
    var armature = avatarRoot.Find("Armature");
    if (!armature) throw new InvalidOperationException("Avatar Armature not found.");

    foreach (var mergeArmature in clothingRoot.GetComponentsInChildren<ModularAvatarMergeArmature>(true))
    {
        mergeArmature.mergeTarget.Set(armature.gameObject);
        EditorUtility.SetDirty(mergeArmature);
    }

    foreach (var installer in clothingRoot.GetComponentsInChildren<ModularAvatarMenuInstaller>(true))
    {
        installer.menuToAppend = outfitMenu;
        installer.installTargetMenu = null;
        EditorUtility.SetDirty(installer);
    }
}
```

If the clothing has no `ModularAvatarMergeArmature` but is clearly adapted and bone names match, prefer adding the component at the clothing armature root over manually reparenting bones.

## Menu Cloning

Clone menus into work assets before changing labels or submenu structure:

```csharp
private static VRCExpressionsMenu CloneMenuTree(
    VRCExpressionsMenu source,
    string outputDir,
    Func<string, string> labelTransform,
    Dictionary<VRCExpressionsMenu, VRCExpressionsMenu> map)
{
    if (!source) return null;
    if (map.TryGetValue(source, out var existing)) return existing;

    var clone = UnityEngine.Object.Instantiate(source);
    clone.name = labelTransform(source.name);
    map[source] = clone;

    var path = AssetDatabase.GenerateUniqueAssetPath(outputDir + "/" + clone.name + ".asset");
    AssetDatabase.CreateAsset(clone, path);

    for (var i = 0; i < clone.controls.Count; i++)
    {
        var control = clone.controls[i];
        control.name = labelTransform(control.name);
        if (control.type == VRCExpressionsMenu.Control.ControlType.SubMenu && control.subMenu)
        {
            control.subMenu = CloneMenuTree(control.subMenu, outputDir, labelTransform, map);
        }
        else if (control.subMenu)
        {
            control.subMenu = null;
        }
        clone.controls[i] = control;
    }

    EditorUtility.SetDirty(clone);
    return clone;
}
```

Create a top-level outfit wrapper when appending an outfit menu:

```csharp
private static VRCExpressionsMenu CreateWrapper(string label, VRCExpressionsMenu child, string path)
{
    var wrapper = ScriptableObject.CreateInstance<VRCExpressionsMenu>();
    wrapper.name = label + "入口";
    wrapper.controls.Add(new VRCExpressionsMenu.Control
    {
        name = label,
        type = VRCExpressionsMenu.Control.ControlType.SubMenu,
        subMenu = child,
        parameter = new VRCExpressionsMenu.Control.Parameter { name = "" }
    });
    AssetDatabase.CreateAsset(wrapper, path);
    EditorUtility.SetDirty(wrapper);
    return wrapper;
}
```

## Parameter Audit

```csharp
private static int EstimateParameterCost(VRCExpressionParameters.Parameter p)
{
    if (!p.networkSynced) return 0;
    return p.valueType == VRCExpressionParameters.ValueType.Bool ? 1 : 8;
}
```

Use `parameters.CalcTotalCost()` when the VRChat SDK is available. Report both total cost and a list of parameters.

## Baked Bone Audit

```csharp
private static void AuditBaked(GameObject avatarRoot, StringBuilder report)
{
    var duplicate = UnityEngine.Object.Instantiate(avatarRoot);
    duplicate.name = "AvatarWork_BakedAudit";
    try
    {
        AvatarProcessor.ProcessAvatar(duplicate);
        var armature = duplicate.transform.Find("Armature");
        foreach (var smr in duplicate.GetComponentsInChildren<SkinnedMeshRenderer>(true))
        {
            var missing = 0;
            var avatarBones = 0;
            var localBones = 0;
            foreach (var bone in smr.bones ?? Array.Empty<Transform>())
            {
                if (!bone) { missing++; continue; }
                if (armature && bone.IsChildOf(armature)) avatarBones++;
                else localBones++;
            }
            report.AppendLine($"{GetPath(smr.transform)} bones={smr.bones.Length} avatar={avatarBones} local={localBones} missing={missing}");
        }
    }
    finally
    {
        UnityEngine.Object.DestroyImmediate(duplicate);
        AvatarProcessor.CleanTemporaryAssets();
    }
}
```

Interpretation:

- Body clothing renderers should have avatar-armature bones and `missing=0`.
- Weapon/accessory renderers may retain local bones if intentionally self-contained.
- Any missing bone on body clothing requires investigation before delivery.

## Gesture Manager Preview

For a repeatable test scene:

- Open the test scene.
- Ensure a camera and directional light.
- Ensure a Gesture Manager object.
- Enter Play Mode.
- Set avatar locomotion parameters such as `Grounded=1`, `Upright=1`, `VelocityZ > 0`, `VelocityMagnitude > 0` when previewing walk.
- Select the Gesture Manager so the inspector shows the radial menu.

Do not compile new scripts while Play Mode is running unless you intentionally accept a domain reload.

## Useful Log Checks

- Unity editor log: `~/Library/Logs/Unity/Editor.log`
- Generated skill/project logs: `<project>/Logs/*.txt`
- Common Unity import issue on Apple Silicon: optional x86-only native plugin warnings may appear for unused audio plugins; do not confuse these with avatar build failures.
