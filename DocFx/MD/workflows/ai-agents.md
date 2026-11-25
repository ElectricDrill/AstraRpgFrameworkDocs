# Using Astra with AI agents

*🏷️ Version 2.3.0+*

Astra Framework allows AI agents to create and configure Astra assets directly in the Unity Editor. For example, an agent can create a class, configure its growth formulas, or inspect an entity's setup. Changes go through the same validation rules used by Astra's Inspector tools.

The integration uses the [Unity Pipeline package](https://docs.unity.com/en-us/unity-cli/unity-pipeline/unity-pipeline-package), which makes twenty Astra commands, grouped under `astra`, available through the [Unity CLI](https://docs.unity.com/en-us/unity-cli). The recommended workflow is to install the Astra agent skill and let the agent run these commands directly from the project directory. The skill provides Astra-specific instructions, while the CLI lets the agent inspect and change the project.

Unity also [recommends direct CLI commands for agents that can run shell commands](https://docs.unity.com/en-us/unity-cli/replace-mcp-server-unity-cli). An MCP server remains available as an [alternative](#connecting-an-mcp-client), for example when an agent cannot run shell commands.

The integration is optional and runs in the Editor. The usual Inspector workflows remain available with or without it.

## Requirements

The integration requires:

- Unity 6.2 or later with the Astra Framework package installed
- The [Unity CLI](https://docs.unity.com/en-us/unity-cli) available for the project
- `com.unity.pipeline` version `1.0.0` or later, up to (but excluding) `2.0.0`
- A running Unity Editor instance for commands that inspect or change project content

> [!NOTE]
> The experimental `0.x` Pipeline releases are not supported. If the project still uses one, upgrade it with `unity pipeline install --force`.

From the Unity project root, install Pipeline once for the project, then check that the Editor is ready:

```bash
unity pipeline install
unity status
unity command astra
```

`unity status` should report the Editor as ready. The last command lists the `astra` command group, which should contain twenty commands. If it is missing, see [Troubleshooting](#troubleshooting).

## Installing the Astra agent skill

The agent skill teaches an agent how to work with Astra: which concepts and commands to use, how to configure assets, and how to resolve setup issues. The package includes these instructions in `Documentation~/agent/`.

The same skill supports Claude Code, Codex, and OpenCode. Its `SKILL.md` includes the `name` and `description` metadata those clients require. Two project folders cover all three clients:

| Folder | Read by |
| --- | --- |
| `.claude/skills/astra-framework/` | Claude Code, OpenCode |
| `.agents/skills/astra-framework/` | Codex, OpenCode |

All relative paths start at the Unity project root (the folder containing `Assets`).

> [!NOTE]
> For the recommended workflow, the agent needs to be able to run Unity CLI commands from the project directory. Install the skill below, then ask the agent to use it with the CLI. MCP configuration is optional.

### From the Unity menu

Open `Tools > Astra Framework > AI > Install Agent Skill…` and choose the clients to install the skill for:

- **Claude Code** installs it at `.claude/skills/astra-framework/`
- **Codex / OpenCode** installs it at `.agents/skills/astra-framework/`

Both can be selected at once. The window remembers the last choice. If a file in a destination differs from the packaged version, Unity asks before replacing it, and cancelling leaves every folder untouched. The result is reported in the Console.

### From the CLI

The same installation is available as a recipe, which is useful from a script or when the agent itself installs the skill. Preview it first, then apply it:

```bash
unity command astra recipe --name install_skill --dry-run true
unity command astra recipe --name install_skill
```

Without arguments the recipe installs the Claude Code copy. Pick the clients with `input.clients`:

```bash
unity command astra recipe --name install_skill --input '{"clients":["claude-code"]}'
unity command astra recipe --name install_skill --input '{"clients":["codex"]}'
unity command astra recipe --name install_skill --input '{"clients":["claude-code","codex"]}'
```

The accepted client names are `claude-code` and `agents`, plus the aliases `claude`, `codex`, and `opencode`. Aliases that share a folder install it once: `codex` and `opencode` both resolve to `.agents/skills/astra-framework/`.

To install into one custom folder instead (inside the project, but not under `Assets` or `Packages`), set a destination. A destination cannot be combined with `clients`:

```bash
unity command astra recipe --name install_skill --input '{"destination":".opencode/skills/astra-framework"}'
```

### Claude Code

Install the skill at `.claude/skills/astra-framework/`, either with **Claude Code** in the menu window or with the default recipe. Open Claude Code in the Unity project repository, and it will find the skill. Ask it to use the `astra-framework` skill for Astra authoring tasks.

### Codex

[Codex discovers project skills](https://learn.chatgpt.com/docs/build-skills#where-codex-loads-local-skills) in `.agents/skills/`. Install the skill with **Codex / OpenCode** in the menu window, or:

```bash
unity command astra recipe --name install_skill --input '{"clients":["codex"]}'
```

Open Codex in the Unity project repository. Codex detects skill changes automatically; restart it if the skill does not appear.

### OpenCode

[OpenCode discovers project skills](https://opencode.ai/docs/skills/) in `.opencode/skills/`, `.claude/skills/`, and `.agents/skills/`. Either of the two standard installations above therefore works for OpenCode, so there is no need to install a dedicated copy. Keep a single copy in a supported location to avoid duplicate skill names.

If a copy under `.opencode/skills/astra-framework/` is preferred, install it with the `destination` input shown above. Open OpenCode in the Unity project repository and ask it to load the `astra-framework` skill. If the skill is missing, check that the `skill` tool is enabled and that the client's skill permissions allow it.

> [!TIP]
> `unity skill install <client>` installs Unity's own skills for the Unity CLI into an agent client, including `claude-code`, `codex`, and `opencode`. It is independent of the Astra skill, and the two work well together. Use `unity skill install --list` to see the supported clients and install status.

## Creating an attribute with an agent

Let's create a `Strength` attribute, like the one in [Create attributes](workflows.md#create-attributes). Keep the Unity Editor open in Edit Mode and open the agent in the Unity project repository, with the Astra skill installed and permission to run Unity CLI commands.

Ask the agent:

> Use the `astra-framework` skill to create a `Strength` attribute at `Assets/Astra/Attributes/`. First check whether it already exists. If it does, inspect it and leave it unchanged. Otherwise, show me a dry run and wait for my approval before creating it. I want `Strength` to have an upper cap set at 99 as max value. After creation, inspect the asset and report any validation issues.

The agent runs `unity command astra find` to locate existing attributes and `unity command astra describe` to inspect them. If the asset does not exist, it can preview creation with `unity command astra create` and `--dry-run true`. The preview reports the planned changes without writing the asset.

After reviewing the preview, ask the agent to proceed. In Unity's Project window, select `Strength` under `Assets/Astra/Attributes`. Its Inspector shows the same attribute settings as an asset created manually. Review **Has Min Value** and **Has Max Value** to check the limits before using it in an attribute set.

### Running the same commands directly

These are the CLI commands the agent uses. They can also be run manually from the project directory. Parameter names are written in kebab-case (`--dry-run`, `--if-exists`). First, search for the attribute:

```bash
unity command astra find --type attribute --name Strength
```

If `Assets/Astra/Attributes/Strength.asset` already exists, inspect it with `astra describe` as shown below. Otherwise, preview its creation:

```bash
unity command astra create --type attribute --path "Assets/Astra/Attributes/Strength.asset" --init '{"has_max_value":true,"max_value":99}' --dry-run true
```

Review the returned `changes` and `issues`. If the preview matches the intended setup, create the asset by repeating the command without `--dry-run`:

```bash
unity command astra create --type attribute --path "Assets/Astra/Attributes/Strength.asset" --init '{"has_max_value":true,"max_value":99}'
```

Missing folders are created automatically. Finally, inspect the saved asset:

```bash
unity command astra describe --handle "Assets/Astra/Attributes/Strength.asset"
```

Compare the returned `data.fields` with the Inspector settings. Writes run Astra's validation rules, so the returned values show the configuration that was actually kept. See [Safe authoring rules](#safe-authoring-rules) for operations that need additional care.

## Astra command catalogue

The commands are grouped by the kind of authoring work they perform:

| Purpose | Commands |
| --- | --- |
| Discover and check content | `astra catalog`, `astra find`, `astra describe`, `astra validate` |
| Create and edit basic assets | `astra create`, `astra set`, `astra set_members`, `astra delete` |
| Configure classes and progression | `astra create_class`, `astra class_formulas`, `astra growth_formula`, `astra ensure_config` |
| Configure scaling, fixed values, action steps, and entities | `astra scaling_weights`, `astra fixed_values`, `astra composite_steps`, `astra entity_setup`, `astra entity_base_source` |
| Build condition trees | `astra condition`, `astra condition_edit` |
| Run package recipes | `astra recipe` |

Use `astra catalog` to check which types and fields are available in the current project, including those added by compatible extension packages. It lists asset and component types, conditions, actions, enums, and the field names accepted by commands. Run `unity command astra <name> --help` to see the parameters of one command.

Unity Pipeline tools handle generic project operations, such as creating a GameObject or a Game Action asset. Astra tools configure Astra-specific data and relationships. For example, Pipeline's `asset create` creates a concrete Game Action asset, while `astra condition_edit` builds the condition tree stored in it.

Each command declares its effect, which MCP clients and the command catalog expose as a hint to decide whether to ask before running it:

- **Read-only**: `astra describe`, `astra catalog`, `astra find`, `astra validate`, `astra condition`
- **Idempotent**: `astra set`, `astra scaling_weights`, `astra fixed_values`, `astra entity_setup`, `astra entity_base_source`, `astra class_formulas`, `astra growth_formula`, `astra ensure_config`
- **Destructive**, because they take a `confirm` parameter: `astra create`, `astra set_members`, `astra create_class`, `astra composite_steps`, `astra condition_edit`, `astra recipe`, `astra delete`

The most common workflow is:

1. Use `astra find` to locate existing assets and components and obtain their identifiers (handles)
2. Use `astra describe` to inspect the current configuration and derived data
3. Preview creation, deletion, or bulk changes with `--dry-run true`
4. Repeat the command without `--dry-run` after inspecting its planned changes and issues
5. Read the returned `changes` and `issues` to check the result after validation
6. Use `astra validate` on the affected object, folder, or project to find remaining setup issues

For complete command parameters and end-to-end examples, open the installed skill's `references/commands.md` and `references/workflows.md` files.

## Safe authoring rules

Astra validates content whenever a command writes it. For example, validation may remove an invalid member from a set even though the command itself succeeds. Always inspect the returned `changes` and `issues` to check which changes were kept.

> [!WARNING]
> Asset creation and deletion cannot be undone through Unity's regular Undo stack. Preview these operations with `--dry-run true` and pass `--confirm true` when the command requires it.
>
> Pipeline's transactional `job batch` can report a rollback (`reverted: true`) even though assets written by an Astra command remain on disk. Do not rely on it to undo Astra asset writes.

> [!NOTE]
> Commands that change content refuse to run in Play Mode. Read-only commands, including `astra describe` and `astra validate`, remain available for inspecting a live scene. The exceptions are `astra fixed_values` with `--allow-runtime true`, which supports temporary runtime tuning, and the `install_skill` recipe, which only writes files outside `Assets`.

Pipeline checks the arguments before an Astra command runs: a parameter or a nested row key that the command does not have (for example `--dry_run` instead of `--dry-run`) is refused instead of being ignored, so a misspelling can never turn a preview into a real write. A wrong argument exits with code 2, while a refusal that depends on the project state, such as a missing `--confirm true`, exits with code 6 and reports an `ASTRA_E_*` code.

Some data needs a dedicated command to keep related settings consistent:

- Use `astra set_members` for stat-set and attribute-set membership
- Use `astra class_formulas` or `astra create_class` to create and assign class growth formulas
- Use `astra scaling_weights` and `astra fixed_values` for weights and values associated with the stats or attributes in a set
- Use `astra condition_edit` for `[SerializeReference]` condition trees
- Use `astra entity_setup` and `astra entity_base_source` to keep an entity's class-based and fixed-base configurations consistent

## Maintaining the installed skill

### Updating the installed skill

The installer leaves matching files in place. Replacing files that differ from the packaged version requires `--confirm true`. This includes both local edits and files from an older Astra version. When several clients are installed at once, a conflict in one folder blocks all of them, and `--confirm true` replaces the differing files in every folder.

After updating Astra, either repeat the menu installation (it asks before replacing anything), or use the CLI:

1. Repeat the installation command for the client in use with `--dry-run true`, keeping the same `--input` if one was specified
2. Inspect the returned `conflicts` and preserve any local edits that are still needed
3. Run the installation command without `--dry-run true`, adding `--confirm true` to replace differing files

### Uninstalling the skill

To uninstall, remove the installed `astra-framework` folder from the Unity project after preserving any local edits:

- `.claude/skills/astra-framework/` for the Claude Code installation (also read by OpenCode)
- `.agents/skills/astra-framework/` for the Codex installation (also read by OpenCode)
- `.opencode/skills/astra-framework/` for a custom OpenCode-specific installation

If a custom destination was used, remove that copy instead. OpenCode can discover the skill in several locations, so check for other installed copies as well. Restart the client after removal.

There is no built-in uninstall recipe or Unity menu item in this version. Removing the installed copy leaves the package's source instructions and any MCP configuration in place. See [Removing the MCP server](#removing-the-mcp-server) to disconnect MCP.

## Connecting an MCP client

MCP is an alternative for agents that need tools exposed through that protocol, such as agents that cannot run shell commands. An MCP *client* is the AI application or agent connecting to Unity's MCP server, which the Unity CLI starts with `unity mcp`. The Astra commands are exposed with the same arguments as the CLI commands.

> [!NOTE]
> Over MCP, a command is addressed by its group and name joined by an underscore: `astra describe` becomes `astra_describe`, and `astra set_members` becomes `astra_set_members`. Parameters keep their kebab-case names (`dry-run`, `if-exists`). Each tool carries the command's effect as an annotation, so clients can ask before running a destructive one.

The Unity Editor must be running for the tools to work, and the Pipeline package must be installed as described in [Requirements](#requirements). The Astra skill can still provide the agent with instructions when using MCP.

### Listing the supported clients

List the clients and configuration paths supported by the installed Unity CLI:

```bash
unity mcp configure --list
```

The list changes between CLI versions. Preview what a configuration would write, without modifying any file, with `--dry-run`:

```bash
unity mcp configure <client> --dry-run
```

The server entry points at the `unity` executable and starts it with `mcp --tool-groups`, which exposes one tool per Editor command group instead of one tool per command. To expose one tool per command, remove `--tool-groups` from the entry's arguments in the client configuration. The same entry accepts other `unity mcp` options, such as `--tools`, `--exclude-tools`, `--exclude-annotations destructive`, and `--project-path` (which pins the entry to one project).

### Claude Code

Add the server to Claude Code with:

```bash
unity mcp configure claude-code
```

This runs `claude mcp add` for you, registering a user-scoped server named `unity-editor-mcp`. Check it with `claude mcp list`.

### Claude Desktop

Claude Desktop is a separate client, named `claude` in the Unity CLI:

```bash
unity mcp configure claude
```

This writes a `unity` entry under `mcpServers` in `claude_desktop_config.json` (the exact path is shown by `unity mcp configure --list`). Restart Claude Desktop afterwards.

### Codex

Add the server to Codex with:

```bash
unity mcp configure codex
```

This writes a `[mcp_servers.unity]` entry to `~/.codex/config.toml`. It also relaxes Codex's sandbox network policy, so that `unity command` can reach the Unity Editor's Pipeline server over localhost. Check the server with `codex mcp list`.

### OpenCode

OpenCode does not appear in `unity mcp configure --list`, so the server has to be added to OpenCode's own configuration. Either use the OpenCode CLI, which writes the entry in the format of the installed version:

```bash
opencode mcp add
```

or add the entry by hand to the project's OpenCode configuration (for example `opencode.jsonc`), under `mcp.servers`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "servers": {
      "unity": {
        "type": "local",
        "command": ["unity", "mcp", "--tool-groups"]
      }
    }
  }
}
```

Check the result with `opencode mcp list`. Older OpenCode versions place the server name directly under `mcp` instead of `mcp.servers`; follow the format of the installed version.

### Other clients

Other supported clients, such as Cursor, VS Code, Windsurf, Cline, and Zed, are configured the same way, using the identifier shown by `unity mcp configure --list`. Clients that support project-local configuration accept `--local`, for example `unity mcp configure cursor --local`. Clients listed with no configuration file need a manual setup: point them at the command `unity mcp --tool-groups`.

The supported clients and configuration paths can change between CLI versions. See the [Unity CLI reference](https://docs.unity.com/en-us/unity-cli/unity-cli-reference) for all options and the [Unity CLI release notes](https://docs.unity.com/en-us/unity-cli/release-notes) for changes.

### Removing the MCP server

The Unity CLI only writes the configuration: it has no command to remove it. Remove the entry with the client's own tools. Restart the client afterwards.

| Client | How to remove the Unity server |
| --- | --- |
| Claude Code | `claude mcp remove unity-editor-mcp --scope user` |
| Claude Desktop | Delete the `unity` entry under `mcpServers` in `claude_desktop_config.json` |
| Codex | `codex mcp remove unity`, then check `~/.codex/config.toml` for the sandbox network setting added by the configuration, if it is no longer needed |
| OpenCode | Delete the `unity` entry from `mcp.servers` in the configuration where it was added, or set `"disabled": true` in it to keep it without connecting |
| Cursor, VS Code, and other clients | Delete the `unity` entry from the file listed by `unity mcp configure --list` |

Removing the MCP server does not uninstall the Astra skill, and the other way around. Remove the skill as described in [Uninstalling the skill](#uninstalling-the-skill). The Pipeline package itself can be removed from the project in the Unity Package Manager, which also removes the `astra` commands until it is installed again.

## Troubleshooting

If the `astra` command group is missing after installing or updating a package (`No command named 'astra'`):

1. Let Unity finish recompiling, or trigger it with `unity command script recompile` and wait for `unity command script recompile_status` to report that it is done
2. Check that the installed Pipeline version is within the range listed in [Requirements](#requirements)
3. Inspect the Editor warnings with `unity command console get --level warn`

Astra compiles its Pipeline adapter only for the tested version range. If Pipeline is outside that range, Astra reports the supported versions in the Editor warnings.

> [!WARNING]
> `ASTRA_PIPELINE_FORCE` can force the adapter to compile outside the tested Pipeline version range. Define it in `Project Settings > Player > Other Settings > Scripting Define Symbols` only after validating the integration in the project, because Pipeline APIs can change between major versions.

If using the MCP connection, configure it separately from the skill. Installing the skill does not add client identifiers to the Unity CLI. For example, before trying to configure a client, check whether it appears in `unity mcp configure --list`.
