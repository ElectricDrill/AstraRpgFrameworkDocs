# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [2.2.0] - Unreleased

### Added
#### Runtime Features
- Added bonus attribute points: `EntityAttributes.GrantBonusAttributePoints`, `RevokeBonusAttributePoints`, and `SetBonusAttributePoints` manage a spendable point pool separate from level-derived points (e.g. granted by items or completed quests). Unlike level points, bonus points survive level-down and re-spec. `BonusAttributePoints` and `LevelAttributePoints` expose the two pools individually, alongside the existing `AvailableAttributePoints` and `TotalAttributePoints`.
- Added permanent per-attribute bonuses: `EntityAttributes.AddPermanentBonus`, `RemovePermanentBonus`, `SetPermanentBonus`, and `SetPermanentBonuses` apply a fixed bonus to an attribute that isn't drawn from any point pool (e.g. a consumed item or book permanently raising an attribute). Permanent bonuses are applied alongside spent points, before percentage modifiers, and are read back with `GetPermanentBonus`/`GetAllPermanentBonuses`. `ClearPermanentBonus` and `ClearAllPermanentBonuses` remove them explicitly.
- Added `AttributePointsChangeInfo` and `EntityAttributes.OnAttributePointsChangedCallback`, raised whenever the attribute-points budget (available and/or total) changes from spending, refunding, granting/revoking bonus points, or level changes.
- Added `EntitySaveId`, an opt-in stable per-entity identity for save systems: a string-backed serializable struct exposed through `EntityCore.SaveId`, `EnsureSaveId` (get-or-create), `RegenerateSaveId`, and `SetSaveId`. The framework never reads the value (no registry, no lookups, no behavior keyed off it), so an entity without an id behaves identically to one with an id; resolving a saved id back to a live entity, and keeping ids unique at runtime, stay the save system's responsibility. `Instantiate` copies a baked id, so runtime-spawned entities that each need their own save record must be given a fresh id by the spawner. See [Saving and Loading](./workflows/saving-and-loading.md).
- Added `EntityAttributes.SetAllSpentPoints`, the bulk setter that pairs with `GetAllSpentPoints` for restoring the invested-points portfolio from a snapshot in a single state assignment, instead of replaying `SpendOn`/`RefundFrom` (whose outcome would depend on the order attributes are replayed in). It replaces rather than merges (attributes absent from the snapshot end at zero), rejects in full any snapshot the current point budget cannot cover, and skips attributes no longer in the `AttributeSet` with a warning rather than failing.
- Added `EntityStats.GetFixedBaseStatValues` and `EntityAttributes.GetFixedBaseAttributeValues`, the bulk getters that pair with the existing `SetFixedStatSource`/`SetFixedAttributeSource` setters, so restoring fixed base values no longer requires walking `StatSet`/`AttributeSet` and calling `GetBase` per entry. Both read through the currently active source, so an entity backed by a `FixedStatValuesAsset` reports the asset's values.

#### Editor Features
- The `EntityAttributesEditor` inspector's `Attribute Points Tracker` now breaks the points summary down into `Level` and `Bonus`, and exposes an editable `Bonus Points` field (prompts for confirmation before a lowering that would force a refund of invested points).
- Added a `Permanent Bonuses` box to the `EntityAttributesEditor` inspector, listing every attribute in the current selection with an editable bonus value and a `Clear All` button.
- Added an optional **Save Identity** section to the `EntityCore` inspector: a read-only **Save Id** field with **Generate** / **Regenerate** / **Copy** actions (regenerating an existing id prompts for confirmation, since it orphans any save records that reference it), multi-object aware. Hidden by default behind the **Show Save Identity section** preference.
- Added `EntitySaveIdRegistry`, an authoring-time guard that keeps save ids unique: duplicating a placed entity, or dropping a second instance of a prefab that already carries a baked id, re-mints the id on the newcomer while the original keeps its id. It never runs in Play Mode and never touches prefab assets or Prefab Mode.
- Added a **Save Identity** page under `Edit > Preferences > Astra Framework`, holding the per-user **Show Save Identity section** and **Auto-bake save id on create** toggles (both off by default, so a project not using save ids gains no new authoring behavior). **Auto-bake save id on create** assigns an id to entities placed in a scene automatically, without a click; prefab assets, Prefab Mode, and Play Mode are left untouched.

### Changed
#### Runtime Features
- `EntityAttributes.RefundAllSpentPoints` now also refunds points invested from the bonus pool, not just level-derived points, since invested points are fungible and not tagged by the pool they were drawn from. Permanent per-attribute bonuses are never touched by `RefundAllSpentPoints`, level-down, or re-spec; only the explicit `ClearPermanentBonus`/`ClearAllPermanentBonuses` APIs remove them.
- `EntityStats.UseClassBaseStats` now has a public setter (previously `internal`), matching its twin `EntityAttributes.UseBaseAttributesFromClass`, so a save system can restore which base-stat source an entity was using.
- `EntityCore.OwnerResolution` is now a public settable property; it was previously reachable only through the inspector. It is still read once in `Awake`, so restore it before the entity awakes.
- `EntityLevel` now raises a single `OnEntityLevelUp`/`OnEntityLevelDown` event per level transition, regardless of how many levels are crossed (previously one event per level). A jump from level 1 to 50 arrives as one event with `PreviousValue = 1`, `NewValue = 50`, and `AbsAmount = 49`, and runs one bulk stat and attribute pass. This applies to every level change path: `AddExp`, `RemoveExp`, `SetTotalCurrentExp`, direct `Level` assignment, and `ResetToLevelOne`. Listeners that need per-level granularity (one reward or fanfare per level) should iterate over `AbsAmount`.


## [2.1.1] - 2026-07-29
- Improved support for Unity 6.4+ editors

## [2.1.0] - 2026-07-13

### Added
#### Runtime Features
- Added `IFixedAttributeSource` and `IFixedStatSource`, minimal point-queried interfaces that supply fixed (non-class-derived) base attribute/stat values to `EntityAttributes`/`EntityStats`, plus the `FixedAttributeValues`/`FixedStatValues` serializable wrapper types that back the framework's own inline and asset-backed sources.
- Added `FixedAttributeValuesSO`/`FixedStatValuesSO`, standalone assets holding fixed base values that can be shared across multiple entities, together with the `EntityAttributes.FixedAttributeValuesAsset`/`EntityStats.FixedStatValuesAsset` fields and the `Use Fixed Attribute Values Asset`/`Use Fixed Stat Values Asset` toggle.
- Added `EntityAttributes.SetFixedAttributeSource`/`EntityStats.SetFixedStatSource`, with an `IReadOnlyDictionary` overload, to inject fixed base values from any custom source (save data, procedural generation, live-ops config) at runtime.
- Added `EntityAttributes.SetFixedAttributeSet`/`EntityStats.SetFixedStatSet` to swap the `AttributeSetSO`/`StatSetSO` used for fixed base values at runtime.
- Added `Assets > Create > Astra Framework > Fixed Attribute Values` and `Fixed Stat Values` asset creation menu items, with dedicated custom editors for authoring their values.
- Added the `Use Fixed Attribute Values Asset`/`Use Fixed Stat Values Asset` toggle to the `EntityAttributesEditor`/`EntityStatsEditor` inspectors.
- Added `EntityCore.Owner`, `EntityCore.Root`, `EntityCore.IsOwnedBy`, and the `OnOwnerChanged` event, an opt-in ownership edge between entities that lets one entity be recognized as owned by another (for example, a weapon entity owned by the ship entity carrying it).
- Added `OwnerResolution` (`Explicit`/`NearestAncestor`) so `EntityCore.Owner` can be assigned explicitly or resolved automatically from the nearest `EntityCore` up the transform hierarchy.
- Added `EntityAttribution` and `EntityOwnership.Resolve`, letting custom systems resolve "the responsible entity" for an entity as itself (`Direct`), its `Owner`, or its `Root`.
- Added the `HolderOwner`, `PerformerOwner`, and `PayloadPerformerOwner` `ConditionTarget` values, and the `IsOwnedByCondition` condition, so condition trees can reason about entity ownership.

#### Editor Features
- Added an **Ownership** section to the `EntityCoreEditor` inspector, exposing the **Owner** and **Owner Resolution** fields, plus a read-only **Root** display and a cycle warning in Play Mode.

#### Editor Features
- Added Class Dashboard Window: Allows to inspect all defined classes in a single place. You can visualize all the GrowthFormula used in a certain class in the same graph.
- Added Growth Curves Comparison Window: Lets you to compare multiple classes (up to 4), easing balancing and tuning. Also allows to compare multiple GrowthFormulas of any kind.
- Added Game Tag Browser Window: Enables browsing of assets basing of assigned Game Tags. If you used Game Tags along your project, will greatly ease the navigation of your assets.

### Changed
#### Runtime Features
- `EntityAttributes` and `EntityStats` now store fixed base values internally through the new `FixedAttributeValues`/`FixedStatValues` wrapper types instead of a raw dictionary field. Existing serialized data migrates automatically the first time each object loads; see [Migrating to v2.1.0](./migration-guide.md#migrating-to-v210) in the Migration Guide.

## [2.0.1] - 2026-06-21

### Added
- Added support for Unity v6.5

### Changed
- Rebranded the framework to "Astra Framework" ("RPG" was dropped)
## [2.1.0] - Unreleased

### Added
#### Editor Features
- Added Class Dashboard Window: Allows to inspect all defined classes in a single place. You can visualize all the GrowthFormula used in a certain class in the same graph.
- Added Growth Curves Comparison Window: Lets you to compare multiple classes (up to 4), easing balancing and tuning. Also allows to compare multiple GrowthFormulas of any kind.
- Added Game Tag Browser Window: Enables browsing of assets basing of assigned Game Tags. If you used Game Tags along your project, will greatly ease the navigation of your assets.

## [2.0.1] - 2026-06-21

### Added
- Added support for Unity v6.5

### Changed
- Rebranded the framework to "Astra Framework" ("RPG" was dropped)

## [2.0.0] - 2026-05-05

> [!WARNING]
> This update includes breaking changes and behavior changes that can affect existing projects. Refer to the [Migrating to v2.0.0](./migration-guide.md#migrating-to-v200) section of the migration guide before updating.

### Added
#### Runtime Features
- Added the `Conditions` system, including composite conditions, entity/tag/value-change/stat/attribute/random leaf conditions, and `ConditionalGameAction`.
- Added `ConditionEvaluationAvailability` and `ConditionTargetAvailability` so condition compatibility checks can describe both the payload contract and the entity target slots available in a given evaluation context.
- Added the `Game Tags` system with `GameTag`, `GameTagSet`, `ITaggable`, and tag-based conditions.
- Added framework-level configuration assets for built-in shared events through `AstraFrameworkConfigSO`, `AstraFrameworkGlobalSettingsSO`, and `AstraFrameworkConfigProvider`.
- Added reactive trigger infrastructure with `IReactiveTrigger`, typed `GameEventTrigger<TContext>` implementations, and `EventSource` support for shared subscription coalescing.
- Added `IHasEntity` as the common bridge contract for entity-aware payloads and game actions, plus projection actions for entity-context workflows.
- Added owner-aware execution paths for `GameAction`s and introduced `GameActionBase` to support inspector-facing polymorphic action references.
- Added `IStatReader` and `IAttributeReader`, then extended the API with `TryGetBase`.
- Added `RoundingMode` and its helper extensions for explicit integer conversion from double-based calculations.

#### Editor Features
- Added dedicated authoring helpers for conditions, including quick setup utilities, managed-reference body drawers for condition fields, and condition tooltips.
- Added deeper condition authoring support through `ConditionEditorContext` and `ConditionTargetDrawer`, so payload-aware editors can also filter invalid `ConditionTarget` values instead of only filtering condition types.
- Added the Game Tag header-pill workflow, including popup search, multi-select add/remove, double-click quick add, intersection handling, overflow display, drag reordering, click-to-ping, and visual customization support.
- Added custom managed-reference drawers for condition-related attribute and stat fields.
- Added optional project-scoped **Authoring Defaults** in `Edit > Project Settings > Astra Framework > Authoring Defaults` for `EntityCore`, `GrowthFormulaSO`, `ClassSO`, `EntityStats`, and `EntityAttributes`, together with inspector actions to apply or replace those defaults on existing objects.
- Added multi-object editing support to the `EntityCore`, `EntityStats`, `EntityAttributes`, `EntityClass`, `Class`, `Scaling Formula`, `Attribute`, `Stat`, `Attribute Scaling Component`, and `Stat Scaling Component` (both attributes and stats) inspectors, including bulk editing for shared values across multiple selected objects.
- Added the `Migrate Managed References (SO Rename)` tool (`Tools → Astra Framework → v2.0.0 Migration -> Migrate Managed References (SO Rename)`) to rewrite `[SerializeReference]` managed-reference type strings that `[MovedFrom]` alone cannot patch after the ScriptableObject type renames. The tool supports a dry-run mode, per-session reporting to `Library/AstraMigrationReport_<timestamp>.txt`, and optional scanning of closed scenes.
- Added `TypeSelectableClosedGenericValidator`, an editor startup check that emits a warning when a `[SerializeReference, TypeSelectable]` field targets a closed generic type — a pattern that breaks silently when any type inside the generic argument is renamed (Unity bug [UUM-44729](https://issuetracker.unity3d.com/issues/movedfrom-attribute-is-not-working-for-generic-types)).

### Changed
#### Runtime Features
- Built-in `Spawned`, `Level Up`, `Level Down`, `Stat Changed`, and `Attribute Changed` events are dispatched through the active Astra Framework configuration asset.
- The optional experience gain modifier stat binding is now assigned once in `AstraFrameworkConfigSO` through **Experience Gain Modifier Stat** instead of being configured per entity.
- `GameAction` workflows now lean on `IHasEntity` as the primary context for event-driven authoring, with owner propagation preserved across wrapper and projection actions.
- `EntityCore` now implements both `IStatReader` and `IAttributeReader`, making the entity itself a convenient read facade for gameplay code.
- `ConditionCompatibility` now performs deeper validation of condition trees, including recursive checks on authored `ConditionTarget` selections inside composite conditions.
- Corrected the `EntityStats.AddStatToStatModifer(...)` typo to `AddStatToStatModifier(...)`.
- Stat and attribute changed notifications now cover broader effective changes, including dependent recalculations and bulk level transitions. See [Addressed Limitations](./limitations.md#addressed-limitations) for details.
- `EntityCore`, `EntityLevel`, `EntityStats`, and `EntityAttributes` implement `IEventRegistrar`, providing generic `Subscribe<TEvent>` and `Unsubscribe<TEvent>` methods for per-entity extra event registration.
- All core ScriptableObject types have been renamed with an `SO` suffix to better reflect their `ScriptableObject` nature. The full rename table and migration instructions are in the [Migration Guide](./migration-guide.md#scriptableobject-type-renames-so-suffix).

  | Old name | New name |
  |---|---|
  | `Stat` | `StatSO` |
  | `Attribute` | `AttributeSO` |
  | `Class` | `ClassSO` |
  | `StatSet` | `StatSetSO` |
  | `AttributeSet` | `AttributeSetSO` |
  | `GrowthFormula` | `GrowthFormulaSO` |
  | `ScalingFormula` | `ScalingFormulaSO` |
  | `ScalingComponent` | `ScalingComponentSO` |
  | `AttributesScalingComponent` | `AttributesScalingComponentSO` |
  | `StatsScalingComponent` | `StatsScalingComponentSO` |
  | `IntVar` | `IntVarSO` |
  | `LongVar` | `LongVarSO` |
  | `GameTag` | `GameTagSO` |
  | `BoundedValue` | `BoundedValueSO` |

  All renamed types carry `[MovedFrom(autoUpdateAPI = true)]`; Unity's API Updater handles C# script references automatically on reimport.
- `IDisplaySONameProvider<T>` was replaced by the non-generic `IDisplaySONameProvider` (parameter type changed from `T` to `ScriptableObject`). The change was necessary to work around [Unity bug UUM-44729](https://issuetracker.unity3d.com/issues/movedfrom-attribute-is-not-working-for-generic-types), where `[MovedFrom]` does not propagate into generic arguments of serialized type strings. Custom implementations must now cast the `ScriptableObject` parameter internally. See [Migrating `IDisplaySONameProvider<T>`](./managed-reference-migration.md#1-migrating-idisplaysonameprovidert-custom-implementations).

#### Editor Features
- Improved the fixed-base attribute handling in `EntityAttributesEditor` and `EntityStatsEditor` for Play Mode usage.
- Replaced sample-backed script default references with optional authoring defaults, so new objects no longer inherit hidden references from package samples and repeated authoring can be standardized explicitly at the project level.
- `ConditionFieldWithQuickSetup` and `PayloadFilteredConditionPicker` now work from a full `ConditionEvaluationAvailability`, whether it is inferred from `IReactiveTrigger.PayloadType` or passed explicitly by a custom editor.
- Payload-aware condition authoring now keeps nested composite conditions under the same availability filter and shows inline errors when a condition type or `ConditionTarget` is incompatible with the current context.

### Removed
#### Runtime Features
- Removed the deprecated legacy `EntityLeveledUpGameEvent` and `EntityLeveledDownGameEvent` types together with the remaining deprecated legacy hooks in `EntityLevel`. These had already been deprecated in `v1.4.0`; if your project still uses them, complete the migration described in [Migrating to v1.4.0](./migration-guide.md#migrating-to-v140) before upgrading.
- Removed the deprecated `AttributePointsTracker` compatibility layer from `EntityAttributes`. The transition to `EntityPointsTracker` and `AttributePortfolio` started in `v1.3.0`.

#### Samples
- Removed the separate `Utils` sample folder. Its helper assets now live under `Examples`, and the samples are documented as reference content rather than project-owned runtime data.

#### Editor Features
- Removed the deprecated `DerivedTypePicker` editor utility. The replacement path to `TypeSelectionMenu` was introduced in `v1.2.0`.
### Fixed
#### Runtime Features
- Fixed level-down handling so fixed-base attribute snapshots are preserved correctly while raising change events.
- Fixed a null-reference path in `GameEventListener` validation when the UnityEvent response is missing.

## [1.5.1] - 2026-04-04
### Changed
#### Editor Features
- Reordered the EntityAttributes fields in the custom inspector to prevent the bug mentioned below
### Fixed
#### Editor Features
- Fixed a bug in the EntityAttributes custom inspector that was not allowing to set either the fixed base Attribute Set nor the "Use Class Attributes From Class"


## [1.5.0] - 2026-03-23
### Added
#### Runtime Features
- Added `IScalingFormula` interface exposing the 4 main methods for scaling formulas. `ScalingFormula` and `ScalingFormulaInstance` now implement this interface.
- Added `ScalingFormulaInstance`. This class shall be used in place of `ScalingFormula` when you need to have a scaling formula that may use temporary scaling components. Temporary scaling components on the SO `ScalingFormula` have been deprecated an will be removed (see below). This ensures that the temporary scaling components do not modify the original `ScriptableObject` instance of the scaling formula, which could cause unintended side effects across different entities using the same scaling formula.
- Added several public read-only properties to `ScalingFormula`:
  - `UseScalingBaseValue`
  - `HasSelfComponents`
  - `HasTargetComponents`
  - `SelfScalingComponents`
  - `TargetScalingComponents`
- Added a public `MaxLevel` get property to `EntityLevel`
- All game event listeners log now an error if play mode is entered while no game event SO instance is assigned to them

#### Editor Features
- Added `Percentage` custom property drawer

### Changed
#### Editor Features
- Improved the `GrowthFormula`'s custom inspector in several ways:
  - Repositioned remove buttons (Now just a "-" sign) to the top right of each growth expression (previously at the bottom)
  - When the `Use Constant Value At Level 1` is checked/unchecked, the first growth expression starting level is automatically set to 2 or 1 respectively, without the need for user input.
  - When adding the first growth expression, if the `Use Constant Value At Level 1` is checked, the starting level of the growth expression is automatically set to 2. If it's unchecked, it's set to 1.
  - When adding a growth expression, the starting level of the new expression is automatically set to one level higher than the previous last expression. The user can of course change it after, but this provides a more intuitive default behavior.
  - When a certain growth expression level is changed, the validation is not performed right away any more, but only after the user finishes editing the field (presses Enter or changes focus). This prevents weird behavior of the inspector while editing the growth expressions, such as automatic changes of the level of other expressions while editing a certain expression's level, which could be confusing.
  - Expressions can now be reordered with up and down buttons.
  - The growth expressions now have dedicated colored headers.
  - The table of the level to value mappings has been redesigned and now clearly shows the various growth expression ranges by using the same colors as the growth expressions.
  - The graph has been redesigned and now light vertical lines are shown to better identify the ranges.
  - The on-hover popup shown over the graph has been redesigned. It now also shows the growth expression and its range to which the hovered level belongs.

#### Samples
- Reviewed the style of the sample scene

### Deprecated
#### Runtime Features
- Deprecated `TmpSelfScalingComponents` and `TmpTargetScalingComponents` in `ScalingFormula`

### Removed
#### Runtime Features
- `ToggleHarvestedExpSourceGameAction` no longer prints a warning when no `ExpSource` component is attached to the entity on which is being executed

## [1.4.2] - 2026-02-13
### Changed
#### Samples
- The sample scene is now using custom materials and a custom shader graph to support multiple render pipelines.

### Fixed
- Fixed a "Stat cannot be read" error being thrown upon project opening/scene loading.

## [1.4.1] - 2026-02-09
### Fixed
- Fixed an issue with the display of Included Stat Sets.

## [1.4.0] - 2026-02-04

> [!WARNING]
> This update involves breaking changes. Refer to the [Migrating to v1.4.0](./migration-guide.md#migrating-to-v140) section of the migration guide for detailed steps to update your project.

### Added
#### Runtime Features
- Added `GameAction` to encapsulate game logic as `ScriptableObject`s. Has a generic context type parameter to specify the type of object the action operates on.
- Added `IExecutable` interface. Implemented by `GameAction`.
- Added the following implementations of `GameAction`:
  - `ToggleRenderersGameAction`: Toggles the enabled state of all `Renderer` components on a GameObject and its children.
  - `ToggleHarvestedExpSourceGameAction`: Toggles the harvested state of an `ExpResource` component.
  - `ToggleCollidersGameAction`: Toggles the enabled state of all `Collider` components on a GameObject and its children.
  - `ToggleActiveGameObjectGameAction`: Toggles the active state of a GameObject.
  - `IncreaseCounterGameAction`: Increases a LongVar by a specified amount. Amount can also be negative to decrease the counter.
  - `DoNothingGameAction`: A no-op action that does nothing when executed. Reserved mostly for Astra Health for neutral responses to certain events.
  - `DestroyGameAction`: Destroys the target GameObject.
  - `DelayedGameAction`: Executes another GameAction after a specified delay.
  - `CompositeGameAction`: Executes a list of GameActions in sequence.
- Added the Create Menu entries for all the above GameActions, with `EntityEngine.Component` as context type parameter.
- Added `GameActionRunner` MonoBehaviour. Used to execute GameActions in a fire-and-forget manner on a specific owner `GameObject`. Useful, for example, if the action is expected to complete and the GameObject owning the GameAction may be destroyed at any moment, or if you want to centralize the execution of GameActions on a specific GameObject (such as a manager).
- Added `IHasSource` interface to establish a contract for objects that have a source EntityCore. Used by Game Event contexts that have a source entity doing something. Currently reserved for future use in Astra Health.
- Added `IHasTarget` interface to establish a contract for objects that have a target EntityCore. Used by Game Event contexts that have a target entity receiving something. Currently used by the `EntityLevelChangedContext` for the new Entity Level Up and Entity Level Down Game Events (see changes for details).
- `IHasValueChange<out T>` interface to establish a contract for objects that represent a change in value of type T. Used by Game Event contexts that represent a change in value:
  - `EntityLevelChangedContext` (int value change)
  - `AttributeChangeInfo` (long value change)
  - `StatChangeInfo` (long value change)
  - `IHasVictim` interface to establish a contract for objects that have a victim EntityCore. Reseved for future use in Astra Health.
- Added custom icon for `ExpSource` MonoBehaviour.

#### Editor Features
- Added a `BrokenEventFinder` utility to help identify broken responses in Unity Events (and therefore in Game Events). Can be accessed from the menu: `Tools/Astra Framework/Find Broken UnityEvents`.

#### Samples
- Added custom font for the Sample Scene.

### Changed
#### Runtime Features
- Simplified `ExpSource` implementation. Now `Harvested` property is no longer set to true upon getting `Exp` property. Now the external code is responsible for setting the `Harvested` property to true when appropriate. This change allows for more flexible usage of the `ExpSource` component, as it can now be used in scenarios where the experience points are not immediately harvested upon retrieval. This change was made to better accommodate future features in Astra Health.

#### Samples
- Updated the input system in the Sample Scene so that no manual action from the user is required when importing the project in Unity v6.2+.
- Renamed `SampleScene` to `Astra Framework Sample Scene`.
- Updated `CommonApiCheatSheet` to reflect new EntityLevelUp and EntityLevelDown game events (see Deprecated section).

### Fixed
#### Samples
- Fixed the input handling in the Sample Scene for Unity 6.2+

### Deprecated
- Mark `EntityLeveledDown` and `EntityLeveledUp` events as obsolete, suggesting new alternatives. Use the new `EntityLevelUpGameEvent` and `EntityLevelDownGameEvent`, and the respective listeners, instead. **Notice that the new events don't have the `ed` suffix after `EntityLevel`**. See the [Migration Guide](./migration-guide.md#migrating-to-v140) for detailed migration steps.

### Removed
- Removed previously deprecated `_includedStatSets` field in `StatSet`


## [1.3.1] - 2026-01-10
### Fixed
- Fixed an editor-scripts related issue that was causing build problems.

## [1.3.0] - 2025-12-16
### Added
#### Runtime Features
- Added `PRV[N]` and `AT[N]` keywords for growth formulas' expressions. 
  - `PRV[N]`: Represents the n-previous Growth Formula value. For example, `PRV[3]` at level 8 would return the Growth Formula value at level 5.
  - `AT[N]`: Represents the Growth Formula value at a specific level N. For example, `AT[10]` would return the Growth Formula value at level 10.
- `BoundedValue.Clamp` now supports also clamping of double values. For now is used internally by the framework and is reserved for future use is Astra Health.

#### Editor Features
- Added support for `enum` formatting in `InspectorTypography`.
- Added utility to provide compact representation of large numbers in the editor (e.g., 1.5K, 4M). Reserved for future use in Astra Health.

### Changed
#### Runtime Features
- Changed the internal `AttributePointsTracker` used by `EntityAttributes` to track spendable attribute points. `EntityPointsTracker` replaces the legacy tracker, and relies on a new `AttributePortfolio` to manage the allocation of attribute points to attributes. This new system is more readable, more robust, and more maintainable. The old system is left for backward compatibility. By updating the framework to this version, existing projects will automatically migrate to the new system.

### Fixed
#### Runtime Features
- Fixed Round Robin Strategy for attribute points removal on level down or reset to level 1.

## [1.2.0] - 2025-11-25
### Added
#### Runtime Features
- Added `OnLevelDownEvent` Game Event. This event is triggered whenever an entity levels down, allowing you to respond to level down events in your game.
- Added XP deduction feature. Now you can deduct XP from an entity using the `EntityCore.Level.RemoveExp(long amount)` method. This will also handle level downs if the deducted XP causes the entity to drop below the current level's XP threshold. Triggers the `OnLevelDownEvent` once for each level the entity drops.
- Added reset to level 1 feature. You can now reset an entity's level and XP to level 1 using the `EntityCore.Level.ResetToLevelOne()` method. This will set the entity's level to 1 and XP to 0. Triggers the `OnLevelDownEvent` once for each level the entity drops.
- Added basic support for statistics, attributes, and classes names presentation. This includes:
  - `IDisplaySONameProvider<in T> where T : ScriptableObject` interface for providing custom display names for ScriptableObjects.
  - `DefaultDisplaySONameProvider<T>` class that implements the above interface and returns the ScriptableObject's name as the display name.
  - `DefaultAttributeCompactDisplaySONameProvider` class that provides compact display names for attributes (e.g., "Str" for "Strength"). This one simply takes the first three letters of the attribute's name, capitalize them, and return that as the display name.  
  You can implement your own display name providers by implementing the `IDisplaySONameProvider<T>` interface to provide custom display names for your statistics, attributes, and classes or to use **localization systems**.

#### Editor Features
- Added *Read Only* (`RO`) property in InspectorTypography. Marks a field in the inspector as read only, in the sense that the component automatically manages it, and the user should not manually edit any associate LongVar/IntVar associated values. Currently reserved for future use.
- Added `IValidatable` interface. Used by the framework to propagate validation events to depending objects. Used to ensure that when a MonoBehaviour/ScriptableObject is validated in the inspector, all depending objects are also validated to ensure data consistency.
- Added `TypeSelectionMenu`. Provides a reusable editor context menu to pick a concrete type derived from a given base type.
Configurable display (optional "None" or "Default" entries, optional name trimming, exclusion of test types, extra filters).
Returns the chosen type via a callback so callers can create or clear instances accordingly.
- Added `[TypeSelectable]` attribute for use on `[SerializeReference]` fields; supports an optional DefaultType to present a "Default" choice. Shows also "None" to clear the reference.
- Added `SerializeReferenceDrawer` custom property drawer. New editor PropertyDrawer showing current concrete type and a Select button for managed references.
Opens centralized `TypeSelectionMenu`, supports choosing None, Default or concrete types and creates/assigns instances.
- Added `SerializeReferenceInitializer`. Runs in the editor to ensure `[SerializeReference]` fields marked with `[TypeSelectable]` that declare a default type are populated when scenes/prefabs are opened or on demand (during script compilation).
Automatically instantiates and assigns default instances where fields are null, provides a menu command to initialize all applicable fields, and marks modified assets/scenes as dirty.

#### Samples
- Added `ExpUtils` MonoBehaviour. Adds utility methods that can be invoked from the Unity Editor (in play mode) to add or remove XP from an entity for testing purposes.

### Changed
#### Runtime
- `SoSetScalingComponentBase` now logs a warning if no entity set is assigned during the lookup of values to use for scaling calculations.
- `AttributeScalingComponent` and `StatScalingComponent` now return `0` as the scaled value if no entity set is assigned, instead of rasing an error. This behavior allows for more flexible setups with entities with different sets of attributes/stats. The warning logged by the base class will help identify potential misconfigurations.
- Marked `OnEntityLevelUp` and `OnEntityLevelDown` events in `EntityCore` as required (red *).
Such methods are becoming crucial for the correct functioning of the various components of the framework. Therefore, it's important to ensure that these events are properly assigned.  
Entities that are missing these events will raise an error during their `Start()` method.

#### Editor
- Improved hot-reloading support for StatSet and AttributeSet.
- Attributes of `AttributeSet` are not "Re-play" (yellow R) any more. This means that it supports changes (addition/removal) of attributes in the AttributeSet without exiting play-mode.
- Improved `InspectorTypography` class. Now it supports also property drawn within GUI rects.

#### Samples
- `ValuesReader` and `ValueEntry` now use `[SerializeReference]` and `[TypeSelectable]` attributes for allowing easier selection of the presentation strategy for each value entry (`IDisplaySONameProvider` implementation).

### Deprecated
- Deprecated `DerivedTypePicker` in favor of the new `TypeSelectionMenu` system. Use the new system for better type selection in the editor. `DerivedTypePicker` will be removed in future releases.

### Fixed
- Added missing invalidation of the cached values for the statistics in `EntityStats.OnEnable()` method.
- Fixed `SetTotalCurrentExp` that was not raising level up and level down events when appropriate.
- Fixed the assignment of total available attribute points. Previously, this was handled only in OnValidate, which caused issues in builds (where editor-only code is stripped). Initialization is now also performed in `EntityCore.Start()`. Additionally, total available attribute points are now updated within `AddPoints()`, ensuring they are correctly recalculated whenever points are added—not only during `Init()`.

## [1.1.2] - 2025-11-12
### Changed
- Updated the "Example scene and instances" samples. Now by default the Hero prefab uses a IntVar for the attribute spendable points per level. This makes more consistent the amount of spendable attribute points acquired by each character on level up
- Updated offline docs with Installation Guide section

## [1.1.1] - 2025-11-06
### Changed
- Updated the links in `package.json` to match the new docs URL

## [1.1.0] - 2025-11-06
### Changed
#### Updated branding from "SOAP RPG Framework" to "Astra RPG Framework"
IMPORTANT: Refer to the [migration guide](./migration-guide.md) for updating existing projects that used the previous version.
- UPM package name changed to "com.electricdrill.astra-rpg-framework".
- UPM package display name changed to "Astra RPG Framework".
- Namespaces updated to reflect new branding: "ElectricDrill.AstraRPGFramework".
- Menu paths in Unity Editor updated to "Astra RPG Framework".


### Deprecated
- `_includedStatSets` field in `StatSet` will become an internal property in future releases.

## [1.0.0] - 2025-10-30
### Added
- Initial release of SOAP RPG Framework.
- ScriptableObject-based toolkit for core RPG mechanics.
- MonoBehaviour-based components for easy integration.

#### ScriptableObject Features
- Statistics & Attributes: Define core stats like Strength and Intelligence, as well as derived attributes like Health or Mana.
- StatSets & AttributeSets: Group related stats and attributes together for organization. Sets can be nested for complex structures.
- Experience & Leveling System: Custom experience curves and character progression.
- Classes: Define character classes with unique progressions and abilities.
- Progression System: Control how stats and attributes grow as characters level up.
- Scaling Formulas: Custom formulas to calculate final values of stats and attributes.
- Scaling Components: Modular scaling formulas reusable across stats and attributes.
- Game Events: Design and trigger custom game events.
- Growth Formulas: Determine how values progress based on a character's level.
- Game Event Generators: Define custom game event types.
- Variables: Use Long and Int variables as ScriptableObjects for persistent data storage.

#### MonoBehaviour Features
- EntityCore: Manage a character's core data, including experience and level.
- EntityStats & EntityAttributes: Assign statistics and attributes to any entity. Components can retrieve values from the assigned EntityClass based on level.
- EntityClass: Assign a class to an entity, linking it to specific progression paths.

#### API & Modifiers
- Comprehensive API for advanced customization (see API Docs).
- Dynamic application of modifiers to stats and attributes:
  - Flat & Percentage Modifiers for attributes.
  - Flat, Percentage, & Stat-to-Stat Modifiers for statistics.
