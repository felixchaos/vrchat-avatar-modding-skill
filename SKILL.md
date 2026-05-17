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
- Never upload, publish, delete source assets, or overwrite user packages unless explicitly asked. Prefer saved SDK sessions or a user hand-off for login, 2FA, and account authorization. If the user explicitly provides credentials in the current conversation and explicitly asks Codex to type them, use Computer Use only as a one-time visible UI input path; never place passwords, OTPs, cookies, or tokens in shell commands, scripts, logs, reports, skills, git commits, or chat summaries, and never enable password saving unless the user explicitly asks.

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
   - Clear blueprint IDs on generated work prefabs/scenes when the user wants a new avatar container. Re-check the serialized scene/prefab for `blueprintId` immediately before opening the SDK upload panel.

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
   - When choosing a body-size variant, synchronize the visible renderer blendshape weights, expression parameter defaults, Modular Avatar shape/parameter defaults, and any size-specific PhysBone roots. Do not leave multiple body-size variants visible or active unless the outfit explicitly supports that.

## Toggles And Body Variants

- For original clothing toggles, verify the target objects named in the original animations exist in the chosen base avatar.
- For base tail versus outfit tail, keep separate parameters such as `Tail` and `<Outfit>/Tail` when both exist. Verify both animated paths exist after NDMF processing.
- For breast/body sliders:
  - Choose clothing variants that match the avatar's default body slider.
  - If the avatar's slider changes body blendshapes but the clothing lacks matching blendshapes, document the limitation or add supported shape-change components.
  - Disable or reset toy/prop animations that drive breast or body bones when testing alignment.
- For body-shape reveal/hide links that depend on clothing coverage:
  - Drive the reveal from VRChat expression parameters and FX/Modular Avatar animator conditions, not from Unity Hierarchy active states or Scene Visibility eye icons. Manual editor hiding does not prove the in-game radial parameter chain works.
  - Require every relevant cover-off/open parameter before showing protruding or exposed body shape keys, and add immediate fallback transitions that reset those keys to zero when any covering clothing or accessory is enabled again.
  - Keep source body defaults conservative; use a small appended FX layer through Modular Avatar Merge Animator so the reveal is runtime-only and can be audited independently.
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
   - If the outfit uses `ModularAvatarMenuInstaller` or similar installers, the source descriptor may show only the base avatar menu. Audit the NDMF-baked avatar as the source of truth for final menu entries and merged parameters.

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
   - Record the baked menu tree and baked expression parameter cost, not only the pre-bake descriptor. This catches menu installers that work only after NDMF and menu assets that look incomplete before baking.

4. PhysBone budget and duplicate audit:
   - Count `VRCPhysBone` components separately from armature bones. A large SDK number such as `1338` is often component count, not bone count.
   - Break the count down by top-level avatar root, clothing root, accessory root, and source prefab so the cause is visible.
   - Detect multiple `VRCPhysBone` components on the same GameObject and repeated exact component signatures. Imported source packages can already contain duplicates; do not assume Codex caused them.
   - Repair only generated work scenes and work prefabs unless the user explicitly asks to modify vendor/source packages. Keep one exact duplicate component and remove only truly identical duplicates.
   - Re-run the audit after repair. A valid result has no duplicate component paths, no repeated same-node signatures, no null roots caused by the repair, and total PhysBones within the current VRChat limit.
   - Do not strip add-on PhysBones from upload builds just to silence a warning when the repaired count is under the limit; preserve clothing dynamics unless there is a real SDK block or the user accepts the loss.

5. Default/radial sync audit:
   - Confirm the default scene view is the desired avatar state, usually the original base outfit unless the user chose otherwise.
   - Confirm add-on outfits, props, DPS/interaction systems, and alternate bodies are off or hidden by default unless explicitly requested.
   - Confirm original and outfit parameters have matching defaults in source parameters, Modular Avatar parameter components, and the NDMF-baked descriptor.
   - Confirm every important menu control has a matching expression parameter and that the baked FX controller contains conditions for it.

6. Runtime Gesture Manager preview:
   - Enter Play Mode.
   - Attach Gesture Manager to the avatar.
   - Open `Expressions`.
   - Test root menu, base clothing menu, base body/tail menu, outfit menu, and critical submenus.
   - Toggle each main outfit piece off and on, including both base tail and outfit tail.
   - Use a walk preview or VRChat proxy walk animation to catch bad bone binding, clipping, or offsets.

7. Capture evidence:
   - Save a short text report with parameter cost, menu tree, FX checks, and bone checks.
   - Save at least one screenshot from the test scene when the user wants visual confirmation.

## SDK Uploads And Versioning

Only perform SDK upload when the user explicitly requests it.

- Before uploading, run static, baked, default/radial, and Gesture Manager checks. Do not upload a build that fails the default/menu sync audit unless the user explicitly accepts the issue.
- Prefer the visible VRChat SDK Control Panel for the final upload flow because it handles login, 2FA, account permissions, ownership prompts, thumbnails, content warnings, and publish buttons in the supported path.
- Unity editor scripts may call the public SDK builder API for repeatable builds or uploads, but they still require a valid SDK login session. If a headless or `-executeMethod` upload cannot load `APIUser.CurrentUser`, open the SDK Authentication panel. If the user asked Codex to enter credentials, use Computer Use to type them into the visible SDK login form without logging or persisting them; otherwise hand the login step to the user.
- For a first upload or explicitly requested new container, make or select a thumbnail, set `releaseStatus` to `private` unless the user asks otherwise, verify `PipelineManager.blueprintId` is empty, and let the SDK assign the new ID. After success, save and record the generated `avtr_...` ID in the upload report.
- For later uploads, preserve or explicitly assign the existing `PipelineManager.blueprintId` so the same private avatar is updated instead of creating a new avatar placeholder.
- Generate a unique thumbnail file for each SDK upload attempt, or skip image upload when the thumbnail is unchanged. VRChat may reject an already-uploaded identical file before the bundle update step.
- For adult, suggestive, violent, horror, or otherwise sensitive avatar content, set the appropriate VRChat content warning tags in the SDK upload metadata. Do not publish public content by default.
- Use a simple project build version such as `0.1.0` for the first real in-game test, then increment the patch or minor version after user-tested changes.
- After upload, save the scene, prefab, project assets, and a text report containing avatar name, version, blueprint ID if visible, validation results, SDK warnings, and upload time.
- If the user asks for a backup, copy the Unity project source to the requested external drive or backup directory. Exclude generated caches such as `Library`, `Temp`, `Obj`, and crash dumps; include `Assets`, `Packages`, `ProjectSettings`, `UserSettings`, and `Logs`.
- Treat SDK alerts by severity. Use SDK-provided Auto Fix for red or upload-blocking items such as protected-layer particle collision or unsupported Unity constraints. Performance-tier warnings such as VeryPoor, triangle count, material slots, skinned mesh count, and PhysBone count may still upload when the user accepts the performance tradeoff.
- If an SDK Auto Fix triggers texture or asset reimport, wait for Unity to finish importing before building. Starting upload while imports are active can leave stale validation state.

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

- Outfit root shows few components:
  - Inspect recursively under child installer roots and the baked avatar before concluding the package is missing setup. Many outfits keep the top-level root as a Transform-only container while MergeAnimator, MenuInstaller, Parameters, PhysBone, BoneProxy, and SkinnedMeshRenderer components live on child objects.
  - If automation menus or scripts disappear, fix all unrelated editor compile errors first; one broken editor script prevents Unity from compiling every new menu item and follow-up utility.

- PhysBone count seems impossible:
  - Confirm whether the SDK warning is counting `VRCPhysBone` components rather than transform bones.
  - Audit duplicate components on the same GameObject and compare work assets against the original source prefabs.
  - If an adapted outfit variant has dozens of identical PhysBones per node while sibling variants have one, treat that variant as a duplicated-component import issue.
  - Repair only generated work assets first, then re-run NDMF bone binding and menu/default audits before uploading.

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
  - Prefer a user hand-off for username/password/2FA or browser authorization.
  - If the user explicitly supplied credentials for this one login and asked Codex to type them, use Computer Use on the visible login form only. Do not store the credentials, do not enable password saving, do not write them into scripts or reports, and hand off any 2FA/CAPTCHA unless the user explicitly supplies the one-time code.
  - Do not scrape browser cookies or request persistent access tokens.

- SDK upload fails after a headless build script:
  - Read both the generated upload report and `~/Library/Logs/Unity/Editor.log`.
  - Retry through the visible SDK panel when account state, ownership confirmation, content warning UI, or thumbnail UI is involved.
- Unity crashes on quit after a successful scripted save or upload:
  - Check whether the log reached the save/upload-success lines before the crash.
  - Avoid `-quit` for final SDK uploads in projects that crash during editor shutdown; keep Unity visible, let the SDK finish, save, then quit manually.
  - Do not rerun destructive rebuilds just because Unity crashed after a successful post-save shutdown. First verify the scene, report, and uploaded blueprint ID.

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
