# Using Astra with AI agents

*🏷️ Version 2.3.0+*

> [!WARNING]
> Astra's AI integration is experimental because it depends on the experimental Unity Pipeline package. Its commands and requirements may change as that package evolves.

Astra Framework allows AI agents to create and configure Astra assets directly in the Unity Editor. For example, an agent can create a class, configure its growth formulas, or inspect an entity's setup. Changes go through the same validation rules used by Astra's Inspector tools.

The integration uses the [Unity Pipeline package](https://docs.unity.com/en-us/unity-production-pipeline/local-tools-cli/unity-pipeline-package), which makes twenty `astra_*` commands available through the [Unity CLI](https://docs.unity.com/en-us/unity-cli). The recommended workflow is to install the Astra agent skill and let the agent run these commands directly from the project directory. The skill provides Astra-specific instructions, while the CLI lets the agent inspect and change the project.

Unity also [recommends direct CLI commands for agents that can run shell commands](https://docs.unity.com/en-us/unity-cli/replace-mcp-server-unity-cli#direct-commands-for-the-editor). MCP access remains available as an [optional alternative](#connecting-an-mcp-client), for example when an agent cannot run shell commands.

The integration is optional and runs in the Editor. The usual Inspector workflows remain available with or without it.

## Requirements

The integration requires:

- Unity 6.2 or later with the Astra Framework package installed
- The [Unity CLI](https://docs.unity.com/en-us/unity-cli) available for the project
- `com.unity.pipeline` version `0.7.0-exp.1` through (but excluding) `0.8.0`
- A running Unity Editor instance for commands that inspect or change project content

From the Unity project root, install Pipeline once for the project, then check that the Editor is ready:

```bash
unity pipeline install
unity status
unity command --query astra_ --detail compact
```

The last command should list twenty `astra_*` commands. If they are missing, see [Troubleshooting](#troubleshooting).

## Installing the Astra agent skill

The agent skill teaches an agent how to work with Astra: which concepts and commands to use, how to configure assets, and how to resolve setup issues. The package includes these instructions in `Documentation~/agent/`.

The same skill supports Claude Code, Codex, and OpenCode. Its `SKILL.md` includes the `name` and `description` metadata those clients require. Choose the installation below for the client in use. All relative paths start at the Unity project root (the folder containing `Assets`).

> [!NOTE]
> For the recommended workflow, the agent needs to be able to run Unity CLI commands from the project directory. Install the skill below, then ask the agent to use it with the CLI. MCP configuration is optional.

### Claude Code

For Claude Code, install a project-scoped copy at `.claude/skills/astra-framework/` from the Unity menu:

`Tools > Astra Framework > AI > Install Agent Skill…`

Alternatively, preview the installation and then apply it through the CLI:

```bash
unity command astra_recipe --name install_skill --dry_run true
unity command astra_recipe --name install_skill
```

### Codex

[Codex discovers project skills](https://learn.chatgpt.com/docs/build-skills#where-codex-loads-local-skills) in `.agents/skills/`. Install the Astra skill at `.agents/skills/astra-framework/` by setting the recipe destination:

```bash
unity command astra_recipe --name install_skill --input '{"destination":".agents/skills/astra-framework"}' --dry_run true
unity command astra_recipe --name install_skill --input '{"destination":".agents/skills/astra-framework"}'
```

Open Codex in the Unity project repository. Codex detects skill changes automatically; restart it if the skill does not appear. Ask Codex to use the `astra-framework` skill for Astra authoring tasks.

### OpenCode

[OpenCode discovers project skills](https://docs.opencode.ai/docs/skills/) in `.opencode/skills/`, and also supports `.claude/skills/` and `.agents/skills/`. If the skill is already installed for Claude Code or Codex, OpenCode can use that copy. Keep a single copy in a supported location to avoid duplicate skill names.

For an OpenCode-specific installation, use `.opencode/skills/astra-framework/`:

```bash
unity command astra_recipe --name install_skill --input '{"destination":".opencode/skills/astra-framework"}' --dry_run true
unity command astra_recipe --name install_skill --input '{"destination":".opencode/skills/astra-framework"}'
```

Open OpenCode in the Unity project repository and ask it to load the `astra-framework` skill. If the skill is missing, check that the `skill` tool is enabled and that the client's skill permissions allow it.

## Creating an attribute with an agent

Let's create a `Strength` attribute, like the one in [Create attributes](workflows.md#create-attributes). Keep the Unity Editor open in Edit Mode and open the agent in the Unity project repository, with the Astra skill installed and permission to run Unity CLI commands.

Ask the agent:

> Use the `astra-framework` skill to create a `Elemental Affinity` attribute at `Assets/Astra/Attributes/`. First check whether it already exists. If it does, inspect it and leave it unchanged. Otherwise, show me a dry run and wait for my approval before creating it. I want `Elemental Affinity` to have an upper cap set at 99 as max value. After creation, inspect the asset and report any validation issues.

The agent runs `unity command astra_find` to locate existing attributes and `unity command astra_describe` to inspect them. If the asset does not exist, it can preview creation with `unity command astra_create` and `--dry_run true`. The preview reports the planned changes without writing the asset.

After reviewing the preview, ask the agent to proceed. In Unity's Project window, select `Strength` under `Assets/Astra/Attributes`. Its Inspector shows the same attribute settings as an asset created manually. Review **Has Min Value** and **Has Max Value** to check the limits before using it in an attribute set.

### Running the same commands directly

These are the CLI commands the agent uses. They can also be run manually from the project directory. First, search for the attribute:

```bash
unity command astra_find --type attribute --name Strength
```

If `Assets/Astra/Attributes/Strength.asset` already exists, inspect it with `astra_describe` as shown below. Otherwise, preview its creation:

```bash
unity command astra_create --type attribute --path "Assets/Astra/Attributes/Strength.asset" --dry_run true
```

Review the returned `changes` and `issues`. If the preview matches the intended setup, create the asset:

```bash
unity command astra_create --type attribute --path "Assets/Astra/Attributes/Strength.asset"
```

Missing folders are created automatically. Finally, inspect the saved asset:

```bash
unity command astra_describe --handle "Assets/Astra/Attributes/Strength.asset"
```

Compare the returned `data.fields` with the Inspector settings. Writes run Astra's validation rules, so the returned values show the configuration that was actually kept. See [Safe authoring rules](#safe-authoring-rules) for operations that need additional care.

## Astra command catalogue

The commands are grouped by the kind of authoring work they perform:

| Purpose | Commands |
| --- | --- |
| Discover and check content | `astra_catalog`, `astra_find`, `astra_describe`, `astra_validate` |
| Create and edit basic assets | `astra_create`, `astra_set`, `astra_set_members`, `astra_delete` |
| Configure classes and progression | `astra_create_class`, `astra_class_formulas`, `astra_growth_formula`, `astra_ensure_config` |
| Configure scaling, fixed values, action steps, and entities | `astra_scaling_weights`, `astra_fixed_values`, `astra_composite_steps`, `astra_entity_setup`, `astra_entity_base_source` |
| Build condition trees | `astra_condition`, `astra_condition_edit` |
| Run package recipes | `astra_recipe` |

Use `astra_catalog` to check which types and fields are available in the current project, including those added by compatible extension packages. It lists asset and component types, conditions, actions, enums, and the field names accepted by commands.

Unity Pipeline tools handle generic project operations, such as creating a GameObject or a Game Action asset. Astra tools configure Astra-specific data and relationships. For example, Pipeline's `create_asset` creates a concrete Game Action asset, while `astra_condition_edit` builds the condition tree stored in it.

The most common workflow is:

1. Use `astra_find` to locate existing assets and components and obtain their identifiers (handles)
2. Use `astra_describe` to inspect the current configuration and derived data
3. Preview creation, deletion, or bulk changes with `dry_run` enabled (`--dry_run true` in the CLI)
4. Repeat the command without `dry_run` after inspecting its planned changes and issues
5. Read the returned `changes` and `issues` to check the result after validation
6. Use `astra_validate` on the affected object, folder, or project to find remaining setup issues

For complete command parameters and end-to-end examples, open the installed skill's `references/commands.md` and `references/workflows.md` files.

## Safe authoring rules

Astra validates content whenever a command writes it. For example, validation may remove an invalid member from a set even though the command itself succeeds. Always inspect the returned `changes` and `issues` to check which changes were kept.

> [!WARNING]
> Asset creation and deletion cannot be undone through Unity's regular Undo stack. Preview these operations with `dry_run` and pass `confirm` when the command requires it.
>
> Pipeline's transactional `batch` can report a rollback even though assets written by an Astra command remain on disk. Do not rely on it to undo Astra asset writes.

> [!NOTE]
> Commands that change content refuse to run in Play Mode. Read-only commands, including `astra_describe` and `astra_validate`, remain available for inspecting a live scene. The exception is `astra_fixed_values` with `allow_runtime` explicitly enabled, which supports temporary runtime tuning.

Some data needs a dedicated command to keep related settings consistent:

- Use `astra_set_members` for stat-set and attribute-set membership
- Use `astra_class_formulas` or `astra_create_class` to create and assign class growth formulas
- Use `astra_scaling_weights` and `astra_fixed_values` for weights and values associated with the stats or attributes in a set
- Use `astra_condition_edit` for `[SerializeReference]` condition trees
- Use `astra_entity_setup` and `astra_entity_base_source` to keep an entity's class-based and fixed-base configurations consistent

## Maintaining the installed skill

### Updating the installed skill

The installer leaves matching files in place. Replacing files that differ from the packaged version requires `--confirm true`. This includes both local edits and files from an older Astra version.

After updating Astra:

1. Repeat the installation command for the client in use with `--dry_run true`, keeping the same `--input` destination if one was specified
2. Inspect the returned `conflicts` and preserve any local edits that are still needed
3. Run the installation command without `--dry_run true`, adding `--confirm true` to replace differing files

### Uninstalling the skill

To uninstall, remove the installed `astra-framework` folder from the Unity project after preserving any local edits:

- `.claude/skills/astra-framework/` for the default Claude Code installation
- `.agents/skills/astra-framework/` for the Codex installation (also usable by OpenCode)
- `.opencode/skills/astra-framework/` for the OpenCode-specific installation

If a custom destination was used, remove that copy instead. OpenCode can discover the skill in several locations, so check for other installed copies as well. Restart the client after removal.

There is no built-in uninstall recipe or Unity menu item in this version. Removing the installed copy leaves the package's source instructions and the Unity MCP configuration in place. Disconnect MCP separately if it is no longer needed.

## Connecting an MCP client

MCP is an optional alternative for agents that need tools exposed through that protocol, such as agents that cannot run shell commands. An MCP *client* is the AI application or agent connecting to Unity's MCP server. The Astra tools use the same names and arguments as the CLI commands.

To use this alternative, first list the clients and configuration paths supported by the installed Unity CLI:

```bash
unity mcp configure --list
```

Then configure the client using its identifier. For example, for Claude Code:

```bash
unity mcp configure claude-code
```

This writes the Unity MCP entry to the client's configuration file. For project-local configuration, clients that support it accept `--local` (for example, `unity mcp configure cursor --local`). The Astra skill can still provide the agent with instructions when using MCP.

The supported clients and configuration paths can change between CLI versions. See the [Unity CLI reference](https://docs.unity.com/en-us/unity-cli/unity-cli-reference) for all options and the [Unity CLI release notes](https://docs.unity.com/en-us/unity-cli/release-notes) for changes.

## Troubleshooting

If the `astra_*` commands are missing after installing or updating a package:

1. Let Unity finish recompiling
2. Check that the installed Pipeline version is within the range listed in [Requirements](#requirements)
3. Inspect the Editor warnings with `unity command console --level warn`

Astra compiles its Pipeline adapter only for the tested version range. If Pipeline is outside that range, Astra reports the supported versions in the Editor warnings.

> [!WARNING]
> `ASTRA_PIPELINE_FORCE` can force the adapter to compile outside the tested Pipeline version range. Use it only after validating the integration in the project, because experimental Pipeline APIs can change between versions.

If using the optional MCP connection, configure it separately from the skill. Installing the skill does not add client identifiers to Unity CLI. For example, before trying to configure OpenCode, check whether it appears in `unity mcp configure --list`.
