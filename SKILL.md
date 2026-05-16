---
name: vrchat-avatar-modding
description: Set up and automate VRChat avatar modding workspaces in Unity. Use when Codex needs to install or verify the VRChat-compatible Unity/VCC/VPM environment, import arbitrary avatar and clothing packages, attach adapted clothing to an avatar, merge armatures/bones with Modular Avatar or NDMF, preserve and combine FX controllers, expression menus, and parameters, localize menu labels, validate synced parameter cost, debug clothing/tail/body toggles, preview with Gesture Manager, prepare private SDK upload builds, or produce a reusable VRChat-ready scene/prefab for any avatar plus any compatible outfit.
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
- Never upload, publish, delete source assets, or overwrite user packages unless explicitly asked. When upload is requested, treat login, 2FA, and account authorization as a user hand-off in the Unity SDK or browser; do not read, infer, or store credentials.

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
   - Do not infer toggle polarity from names alone. Inspect animation curves, FX conditions, original expression parameter defaults, and scene active states before changing defaults.
   - If a Modular Avatar add-on must stay active for installers or merge components, keep its root active and hide only renderer-bearing child objects by default.
   - Synchronize defaults across the scene/prefab state, `VRCExpressionParameters`, `ModularAvatarParameters`, and the NDMF-baked descriptor.
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

## Menus And Toggle Semantics

- Preserve vendor control labels unless the user asks for translation. If labels are explicit state controls such as `On`/`Off`, first determine whether they control object visibility, animation playback, mode selection, or a lock; do not merge unrelated controls just because their names look paired.
- VRChat expression pages have eight slots. Flatten only when it improves actual usability and keeps the page within the limit; otherwise keep clear submenus.
- For toggle buttons, verify the menu control writes the intended parameter value and that the FX layer or Modular Avatar generated layer consumes that same parameter.
- For mutually exclusive options, prefer a single `Int` parameter and option controls that set distinct values when the vendor asset is already structured that way. Do not convert booleans into ints unless you can also update every FX condition and menu control.

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

4. Default/radial sync audit:
   - Confirm the default scene view is the desired avatar state, usually the original base outfit unless the user chose otherwise.
   - Confirm add-on outfits, props, DPS/interaction systems, and alternate bodies are off or hidden by default unless explicitly requested.
   - Confirm original and outfit parameters have matching defaults in source parameters, Modular Avatar parameter components, and the NDMF-baked descriptor.
   - Confirm every important menu control has a matching expression parameter and that the baked FX controller contains conditions for it.

5. Runtime Gesture Manager preview:
   - Enter Play Mode.
   - Attach Gesture Manager to the avatar.
   - Open `Expressions`.
   - Test root menu, base clothing menu, base body/tail menu, outfit menu, and critical submenus.
   - Toggle each main outfit piece off and on, including both base tail and outfit tail.
   - Use a walk preview or VRChat proxy walk animation to catch bad bone binding, clipping, or offsets.

6. Capture evidence:
   - Save a short text report with parameter cost, menu tree, FX checks, and bone checks.
   - Save at least one screenshot from the test scene when the user wants visual confirmation.

## SDK Uploads And Versioning

Only perform SDK upload when the user explicitly requests it.

- Before uploading, run static, baked, default/radial, and Gesture Manager checks. Do not upload a build that fails the default/menu sync audit unless the user explicitly accepts the issue.
- Prefer the visible VRChat SDK Control Panel for the final upload flow because it handles login, 2FA, account permissions, ownership prompts, thumbnails, content warnings, and publish buttons in the supported path.
- Unity editor scripts may call the public SDK builder API for repeatable builds or uploads, but they still require a valid SDK login session. If a headless or `-executeMethod` upload cannot load `APIUser.CurrentUser`, open the SDK Authentication panel and let the user log in.
- For a first upload, make or select a thumbnail, set `releaseStatus` to `private` unless the user asks otherwise, and let the SDK assign the `PipelineManager.blueprintId`. Keep that blueprint ID in the work scene/prefab for later version updates.
- For adult, suggestive, violent, horror, or otherwise sensitive avatar content, set the appropriate VRChat content warning tags in the SDK upload metadata. Do not publish public content by default.
- Use a simple project build version such as `0.1.0` for the first real in-game test, then increment the patch or minor version after user-tested changes.
- After upload, save the scene, prefab, project assets, and a text report containing avatar name, version, blueprint ID if visible, validation results, SDK warnings, and upload time.
- If the user asks for a backup, copy the Unity project source to the requested external drive or backup directory. Exclude generated caches such as `Library`, `Temp`, `Obj`, and crash dumps; include `Assets`, `Packages`, `ProjectSettings`, `UserSettings`, and `Logs`.

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

- Unity process exists but no window is visible:
  - Check running Unity processes and the project lockfile owner.
  - Quit the stale Unity process gracefully and reopen the exact editor version directly with `-projectPath`.
  - Remove `Temp/UnityLockfile` only when no Unity process owns it.

- SDK upload is blocked at login:
  - Open `VRChat SDK > Show Control Panel > Authentication`.
  - Ask the user to complete username/password/2FA or browser authorization.
  - Do not scrape browser cookies or ask the user to paste passwords.

- SDK upload fails after a headless build script:
  - Read both the generated upload report and `~/Library/Logs/Unity/Editor.log`.
  - Retry through the visible SDK panel when account state, ownership confirmation, content warning UI, or thumbnail UI is involved.

## Completion Criteria

Finish with:

- Saved work scene and test scene.
- Saved reusable prefab.
- Preserved source packages and source assets.
- Merged menus and parameters under work assets.
- Passing parameter budget report.
- Passing baked bone report.
- Passing default/radial sync report.
- Gesture Manager preview ready for user testing, or Unity closed if requested.
- Optional private SDK upload completed, saved, versioned, and backed up only when explicitly requested.
