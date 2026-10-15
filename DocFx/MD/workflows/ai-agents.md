# Using Astra with AI agents

*🏷️ Version 2.3.0+*

> [!WARNING]
> AI features are currently experimental and may change at any time.
>
> This is due to the fact that the Unity pipeline package dependency is experimental itself. Therefore, Astra'a AI feature are experimental so that they will change along with the underlying Unity package.

Astra Framework can expose its editor authoring workflows to an AI agent through the [Unity Pipeline package](https://docs.unity.com/en-us/unity-production-pipeline/local-tools-cli/unity-pipeline-package). The integration adds twenty `astra_*` commands to the [Unity CLI](https://docs.unity.com/en-us/unity-cli), its local HTTP server, and its MCP server. They create and configure Astra assets, wire entity components, inspect authored content, and run the same validation rules that protect the Inspector workflows.

This is useful when an agent needs to work with the actual project state rather than only generate C# or suggest Inspector changes. The commands are editor-only and optional: projects that do not install Unity Pipeline keep the normal Astra Inspector workflows unchanged.

## Requirements

The integration requires:

- Unity 6.2 or later with the Astra Framework package installed
- The [Unity CLI](https://docs.unity.com/en-us/unity-cli) available for the project
- `com.unity.pipeline` version `0.7.0-exp.1` through (but excluding) `0.8.0`
- A running Unity Editor instance for commands that inspect or change project content

Install Pipeline once for the project, then check that the Editor is ready:

```bash
unity pipeline install
unity status
unity command --query astra_ --detail compact
```

The last command should list twenty `astra_*` commands. Astra compiles its Pipeline adapter only when Pipeline is in its tested version range. If the commands are missing after installing or updating a package, let Unity finish recompiling and inspect `unity command console --level warn`. Astra reports an out-of-range Pipeline version there, including the supported range.

> [!WARNING]
> `ASTRA_PIPELINE_FORCE` can force the adapter to compile with a Pipeline version outside the tested range. Use it only after validating the integration in the project, because Unity Pipeline is an experimental dependency and its command APIs can change between versions.

## Connecting an MCP client

The CLI commands are also published as MCP tools with the same names and arguments. Here, a *client* is an AI-enabled desktop application, IDE, editor extension, or command-line agent that can launch and use an MCP server. `unity mcp configure` writes the Unity MCP entry to the configuration file that the selected client reads.

Use a supported client identifier in place of `<client>`:

```bash
unity mcp configure <client>
```

The Unity CLI currently supports these client identifiers:

| Client type | Identifiers |
| --- | --- |
| Anthropic clients | `claude` (Claude Desktop), `claude-code` |
| Code editors and IDEs | `cursor`, `vscode`, `vscode-insiders`, `windsurf`, `cline`, `kiro`, `zed`, `continue` |
| Command-line agents | `copilot-cli`, `codex`, `kimi` |
| Other clients and utilities | `trae`, `openclaw`, `antigravity`, `inspect` |

For example, configure Claude Code with `unity mcp configure claude-code`. Use `unity mcp configure cursor --local` or `unity mcp configure vscode --local` to create project-local configuration where the selected client supports it.

The supported client set and configuration paths can change as the experimental Unity CLI evolves. Run the following command to list the exact identifiers and destinations supported by the installed CLI:

```bash
unity mcp configure --list
```

See the [Unity CLI reference](https://docs.unity.com/en-us/unity-cli/unity-cli-reference) for all options, and the [Unity CLI release notes](https://docs.unity.com/en-us/unity-cli/release-notes) for changes to supported clients. The configured client can then call `astra_find`, `astra_create`, and the other Astra tools alongside Unity's standard Pipeline tools.

Use Unity Pipeline tools for generic project operations such as creating a GameObject or a Game Action asset. Use the Astra tools to author Astra-specific data and relationships. For example, Pipeline's `create_asset` creates a concrete Game Action asset, while `astra_condition_edit` safely builds the condition tree stored in it.

## Installing the Astra agent skill

The package ships an agent skill in `Documentation~/agent/`. It describes Astra concepts, logical field names, command arguments, worked workflows, and recovery steps.

For Claude Code, install a project-scoped copy at `.claude/skills/astra-framework/` from the Unity menu:

`Tools > Astra Framework > AI > Install Agent Skill…`

You can also ask an agent to install it through the command surface:

```bash
unity command astra_recipe --name install_skill --dry_run true
unity command astra_recipe --name install_skill
```

The installer is idempotent. It leaves matching files in place and requires `--confirm true` before replacing a locally modified file. Run it again after updating Astra to refresh the installed skill.

Claude Code is the automated installation target for the Astra skill in this version. This does not limit MCP connectivity: any client in the previous table can call the `astra_*` tools after it is configured. Other clients can consume the same Markdown files directly from `Packages/com.electricdrill.astra-rpg-framework/Documentation~/agent/` when their own configuration supports project instructions.

## Astra command catalogue

The commands are grouped by the kind of authoring work they perform:

| Purpose | Commands |
| --- | --- |
| Discover and check content | `astra_catalog`, `astra_find`, `astra_describe`, `astra_validate` |
| Create and edit basic assets | `astra_create`, `astra_set`, `astra_set_members`, `astra_delete` |
| Configure classes and progression | `astra_create_class`, `astra_class_formulas`, `astra_growth_formula`, `astra_ensure_config` |
| Write structured Astra data | `astra_scaling_weights`, `astra_fixed_values`, `astra_composite_steps`, `astra_entity_setup`, `astra_entity_base_source` |
| Build condition trees | `astra_condition`, `astra_condition_edit` |
| Run package recipes | `astra_recipe` |

`astra_catalog` is the source of truth for the currently available asset types, component types, conditions, actions, enums, and logical fields. It also includes contributions from compatible extension packages, so use it before assuming a type name or serialized field path.

The most common workflow is:

1. Use `astra_find` to resolve existing assets and components to unambiguous handles
2. Use `astra_describe` to inspect the current configuration and derived data
3. Run an asset-creating, destructive, or bulk command with `dry_run: true`
4. Repeat the command without `dry_run` after inspecting its planned changes and issues
5. Read the returned `changes` and `issues`, which reflect the state after Astra validation ran
6. Use `astra_validate` on the affected object, folder, or project to find remaining setup issues

For complete command parameters and end-to-end examples, open the installed skill's `references/commands.md` and `references/workflows.md` files.

## Safe authoring rules

Astra validates authored content whenever a command writes it. A request can therefore be accepted while a set member or an invalid relationship is removed by validation. Always inspect the returned `changes` and `issues`; do not assume that the requested value survived the write.

Asset creation and deletion cannot be undone through Unity's regular Undo stack. Use `dry_run` before operations that create or delete assets, and pass `confirm` when a command requires an explicit destructive-action acknowledgement. Do not use Pipeline's transactional `batch` as a safety mechanism for Astra asset writes: a batch can report a rollback even though assets written by a custom command remain on disk.

> [!NOTE]
> Mutating `astra_*` commands refuse to run in Play Mode. `astra_describe`, `astra_validate`, and the other read-only commands remain available, which makes them useful for inspecting a live scene. The `astra_fixed_values` runtime path is the documented exception when explicitly enabled; use it for temporary runtime tuning rather than to author persistent asset data.

Some structures need their dedicated command instead of a generic field write:

- Use `astra_set_members` for stat-set and attribute-set membership
- Use `astra_class_formulas` or `astra_create_class` to create and assign class growth formulas
- Use `astra_scaling_weights` and `astra_fixed_values` for set-keyed value tables
- Use `astra_condition_edit` for `[SerializeReference]` condition trees
- Use `astra_entity_setup` and `astra_entity_base_source` to keep an entity's class-based and fixed-base configurations consistent

These commands encode the same consistency rules used by Astra's editor tooling and return actionable validation results when a setup is incomplete.
