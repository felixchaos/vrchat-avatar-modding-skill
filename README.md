# VRChat Avatar Modding Skill

A Codex skill for setting up and automating VRChat avatar modding workflows in Unity.

It teaches Codex how to:

- Configure or verify a VRChat-compatible Unity/VPM workspace.
- Import arbitrary avatar and adapted clothing packages.
- Attach clothing to avatars with Modular Avatar and NDMF.
- Preserve original avatar FX, expression menus, parameters, and toggles.
- Merge outfit menus and parameters without hardcoding specific assets.
- Validate VRChat synced parameter usage.
- Audit and repair duplicated PhysBone components in generated work assets.
- Audit default state synchronization across scene objects, expression parameters, Modular Avatar defaults, and the baked radial menu.
- Test original clothing, outfit pieces, tails, body sliders, and common toggle failures.
- Preview and debug the result with Gesture Manager.
- Prepare private SDK upload builds, version reports, and project backups when explicitly requested, with credential handling guidance that never stores passwords or tokens.

The skill is intentionally generic. It does not depend on any specific avatar, outfit, shader, or vendor package.

## Install

Clone this repository into your Codex skills directory:

```bash
git clone https://github.com/felixchaos/vrchat-avatar-modding-skill.git ~/.codex/skills/vrchat-avatar-modding
```

Restart Codex or reload skills after cloning.

## Use

Ask Codex to use the skill when working on VRChat avatar setup or clothing installation, for example:

```text
Use the VRChat avatar modding skill to set up this avatar and adapted clothing package in Unity, merge bones and menus, then open a Gesture Manager preview.
```

Provide:

- The avatar package or archive path.
- The clothing package or archive path.
- The target Unity project location.
- Whether original avatar clothing and labels should be preserved or translated.

## Contents

- `SKILL.md`: the main workflow Codex loads when the skill triggers.
- `references/unity-editor-automation.md`: Unity editor automation patterns for rebuilding scenes, merging Modular Avatar settings, cloning menus, auditing parameters/defaults, checking baked bones, and preparing SDK upload automation.
- `agents/openai.yaml`: UI metadata for Codex.

## Notes

This skill does not perform Blender mesh fitting or weight painting. If an outfit is not actually adapted to the avatar's body, armature, or proportions, Codex should identify that limitation instead of pretending Unity-side merging can fix it.

## License

MIT
