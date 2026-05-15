---
name: vrchat-avatar-modding
description: Set up and automate VRChat avatar modding workspaces in Unity. Use when Codex needs to install or verify the VRChat-compatible Unity/VCC/VPM environment, import arbitrary avatar and clothing packages, attach adapted clothing to an avatar, merge armatures/bones with Modular Avatar or NDMF, preserve and combine FX controllers, expression menus, and parameters, localize menu labels, validate synced parameter cost, debug clothing/tail/body toggles, preview with Gesture Manager, or produce a reusable VRChat-ready scene/prefab for any avatar plus any compatible outfit.
---

# VRChat Avatar Modding

Use this workflow to turn a base VRChat avatar and an adapted outfit into a working Unity project with menus, parameters, bone binding, and local preview. Never assume asset names, folders, body variants, or menu structure; discover them from the imported project each time.

For reusable Unity editor automation patterns, read `references/unity-editor-automation.md` only when you need to write or patch editor scripts.

## Core Rules

- Work from discovered facts: find `VRCAvatarDescriptor`, humanoid `Animator`, armature root, clothing prefab roots, FX controllers, expression menus, expression parameters, and Modular Avatar components before editing.
- Keep source assets intact. Create a work scene, prefab, generated menus, and editor scripts under a clearly named work folder such as `Assets/VRCWork`, `Assets/AvatarWork`, or the repo's existing work folder.
- Use the current VRChat-supported Unity version and SDK/VPM package set. If the current supported Unity version or package version matters, verify from official VRChat/Unity sources before installing or changing versions.
- Prefer established VRChat tools over manual rig surgery: VCC/VPM, VRChat SDK Avatars, NDMF, Modular Avatar, lilToon/Poiyomi as required by materials, Gesture Manager for local menu testing, and Avatar Optimizer only when optimization is requested.
- Treat a clothing package as "adapted" only when it was made for the avatar or its armature and body proportions. If the mesh does not fit, weights are wrong, or bone names do not correspond, stop and explain that Blender/weight-painting or a dedicated fitting tool is required.
- Never upload, publish, delete source assets, or overwrite user packages unless explicitly asked.

## Environment Setup

1. Confirm the project target:
   - Avatar package/archive path.
   - Clothing package/archive path(s).
   - Desired Unity install location and project directory.
   - Whether to preserve original outfit menus, add new outfit menus, translate labels, or preserve vendor labels.

2. Verify or install tooling:
   - Unity Hub and the VRChat-supported Unity editor version.
   - VCC/VPM or equivalent package management.
   - VRChat SDK Avatars and SDK Base.
   - NDMF and Modular Avatar for non-destructive merges.
   - Shader dependencies used by the packages, usually lilToon or Poiyomi.
   - Gesture Manager for local expression/radial menu testing.

3. Create or open a clean Unity project:
   - Keep it separate from raw downloads.
   - Import avatar first, then shaders/dependencies, then outfit.
   - Let Unity finish compiling/importing before running editor scripts.
   - Save logs and generated reports under the project or workspace `Logs/` folder.

## Asset Discovery

Use `rg --files`, Unity YAML inspection, and editor scripts to identify:

- Base avatar prefab or scene object:
  - Contains `VRCAvatarDescriptor`.
  - Has a humanoid `Animator` and an `Armature`.
  - Uses the intended expression menu, expression parameters, and FX controller.

- Clothing prefab candidates:
  - Usually named by avatar compatibility, body size, breast size, or variant.
  - May include `ModularAvatarMergeArmature`, `ModularAvatarMergeAnimator`, `ModularAvatarMenuInstaller`, `ModularAvatarParameters`, `MA Shape Changer`, or custom installer scripts.
  - May have multiple variants such as S/M/L, flat/medium/large, default/body-slider variants, or separate accessory roots.

- Original avatar toggles:
  - Original clothing, hair, body, tail, ear, breast/body-size, face, and other toggles.
  - Record control labels, parameter names, parameter types, default values, and FX layers.

- Outfit toggles:
  - Outfit pieces, tail, accessories, masks, weapons, body tubes, toys, or props.
  - Preserve vendor labels unless the user explicitly asks for translation.

## Build The Workspace

1. Create work assets:
   - `Assets/<WorkRoot>/Scenes/<avatar>_<outfit>.unity`
   - `Assets/<WorkRoot>/Scenes/<avatar>_<outfit>_Test.unity`
   - `Assets/<WorkRoot>/Prefabs/<avatar>_<outfit>.prefab`
   - `Assets/<WorkRoot>/Expressions/` or `ExpressionsFixed/`
   - `Assets/Editor/` utility scripts if the repo already uses editor automation.

2. Instantiate the correct base avatar:
   - Prefer the original full avatar prefab if the user wants original clothing and body toggles preserved.
   - Avoid using a vendor "kisekae", "naked", "test", or clothing demo prefab if it omits original outfit objects or uses a reduced FX/menu, unless that is explicitly desired.
   - Clear blueprint IDs on generated work prefabs/scenes.

3. Attach the clothing:
   - Instantiate the chosen clothing prefab as a child of the avatar root unless the package requires a different installer location.
   - Reset local transform to zero position, identity rotation, and one scale.
   - Configure each `ModularAvatarMergeArmature` target to the avatar's armature.
   - Keep accessory or weapon local bones if the outfit expects them; body clothing bones should resolve to the avatar armature after NDMF processing.

4. Preserve and merge animation controllers:
   - Keep the base avatar's FX controller as the descriptor FX layer when original avatar toggles must work.
   - Merge outfit FX controllers through Modular Avatar or NDMF. Do not replace the base FX with an outfit test controller if that removes original clothing, tail, or body layers.
   - If a clothing package has multiple merge animators, preserve their ordering and write-default settings unless there is a demonstrated conflict.

5. Build expression menus:
   - Keep the avatar's root menu and append the outfit under a clear top-level submenu such as the outfit name.
   - Clone generated menus into the work folder instead of editing vendor menu assets.
   - Preserve parameter names exactly.
   - Preserve vendor menu labels unless asked to translate them.
   - If translating labels, translate labels only, not parameter names or asset paths.
   - Remove invalid submenu references from non-submenu controls if copied menus contain stale references.

6. Build expression parameters:
   - Start from the base avatar's expression parameters.
   - Merge outfit parameters from Modular Avatar components or vendor parameter assets.
   - Preserve parameter type, saved/networkSynced behavior, explicit defaults, and local-only settings where meaningful.
   - Compute VRChat synced parameter cost and keep it at or below `256`.
   - If over budget, propose reducing network sync on cosmetic-only parameters, grouping mutually exclusive states into ints, or removing unused toggles.

7. Set safe defaults:
   - Avoid default overlap between base clothing and new outfit.
   - If base menu controls are semantically `*_OFF` toggles, defaulting those bools to `true` usually hides the original outfit while preserving the menu.
   - Keep the new outfit defaults from the vendor package unless they conflict with the base avatar.
   - Align body-size defaults with the chosen clothing variant.

## Toggles And Body Variants

- For original clothing toggles, verify the target objects named in the original animations exist in the chosen base avatar.
- For base tail versus outfit tail, keep separate parameters such as `Tail` and `<Outfit>/Tail` when both exist. Verify both animated paths exist after NDMF processing.
- For breast/body sliders:
  - Choose clothing variants that match the avatar's default body slider.
  - If the avatar's slider changes body blendshapes but the clothing lacks matching blendshapes, document the limitation or add supported shape-change components.
  - Disable or reset toy/prop animations that drive breast or body bones when testing alignment.
- For props and weapons, accept local prop bones when intended, but ensure body clothing renderers use avatar bones.

## Validation

Run validation after every substantial rebuild:

1. Static menu and parameter audit:
   - Root menu includes expected base avatar entries and outfit entry.
   - Every menu parameter exists in expression parameters.
   - Every page has no more than 8 controls.
   - Parameter cost is `<= 256`.

2. FX compatibility audit:
   - Base avatar FX still contains original clothing, body, tail, hair, and face conditions or blend-tree parameters.
   - Outfit FX conditions are present after merge.
   - Animated binding paths refer to objects that exist in the baked avatar.

3. NDMF/Modular Avatar baked audit:
   - Duplicate the avatar and process it with NDMF.
   - Check every clothing `SkinnedMeshRenderer`.
   - Body clothing bones should be children of the baked avatar armature.
   - Missing/null bones must be `0`.
   - Local bones should be limited to accessories, weapons, and props that are intentionally self-contained.

4. Runtime Gesture Manager preview:
   - Enter Play Mode.
   - Attach Gesture Manager to the avatar.
   - Open `Expressions`.
   - Test root menu, base clothing menu, base body/tail menu, outfit menu, and critical submenus.
   - Toggle each main outfit piece off and on, including both base tail and outfit tail.
   - Use a walk preview or VRChat proxy walk animation to catch bad bone binding, clipping, or offsets.

5. Capture evidence:
   - Save a short text report with parameter cost, menu tree, FX checks, and bone checks.
   - Save at least one screenshot from the test scene when the user wants visual confirmation.

## Troubleshooting

- Menu exists but toggle has no effect:
  - The parameter may be missing, wrong type, or not used by the FX controller.
  - The FX animation may target a path absent from the current avatar.
  - The outfit merge animator may not have been merged or may target the wrong layer.

- Original clothing disappeared or cannot toggle:
  - The base prefab may be a reduced/naked/change-clothes variant.
  - The base FX/menu may have been replaced by an outfit demo/test FX.
  - Rebuild from the original full avatar prefab and merge the outfit on top.

- Tail does not hide/show:
  - Confirm whether the user means base avatar tail or outfit tail.
  - Check that `Tail` and outfit tail parameters are distinct.
  - Check animation paths for `Other_tail`, `Tail`, or outfit-specific tail roots in the baked avatar.

- Chest/breast offset:
  - Check clothing size variant against body/breast slider default.
  - Reset breast/toy/plug parameters that drive breast bones.
  - Verify clothing breast renderers bind to the avatar armature after NDMF.
  - If the clothing lacks matching blendshapes for the avatar's slider, report the limitation.

- Preview looks blurry:
  - Set Game View scale to `1x`.
  - Use an orthographic camera with stable framing.
  - Avoid judging mesh quality from a zoomed-out screenshot.

- Unity appears stuck:
  - Check whether it is compiling/importing.
  - Inspect `~/Library/Logs/Unity/Editor.log`.
  - Do not interrupt imports unless clearly deadlocked.

## Completion Criteria

Finish with:

- Saved work scene and test scene.
- Saved reusable prefab.
- Preserved source packages and source assets.
- Merged menus and parameters under work assets.
- Passing parameter budget report.
- Passing baked bone report.
- Gesture Manager preview ready for user testing, or Unity closed if requested.
