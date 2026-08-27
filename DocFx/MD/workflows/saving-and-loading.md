# Saving and Loading

*🏷️ Version 2.2.0+*

Astra is not a save system, and is not trying to become one. It defines no save format, ships no serializer, keeps no registry of live entities, and never touches the disk. What it does provide is everything a save system needs on the other side of that line: a stable per-entity identity, matching getters and setters for every piece of runtime state, and bulk restore paths that hold up when a whole snapshot is reapplied at once.

This page walks through that surface: what Unity already persists for you, what you capture and restore through the API, the order to restore it in, and what is deliberately left out.

## What stays your responsibility

Astra draws the line in a specific place, and a few jobs sit firmly on your side of it:

- **The save format and the I/O.** JSON, binary, `PlayerPrefs`, an asset-store package (all fine). Astra never expresses an opinion here, it follows the framework philosophy: **maximum flexibility and no limitations**.
- **Mapping a saved id back to a live entity.** There is no `FindEntityById`. Astra never keeps a runtime map of live entities, because charging every project a registration on every `Instantiate()` (projectiles, VFX, pooled objects) to serve a feature most entities never use is the wrong trade. Your save system owns that lookup.
- **Resolving asset references.** `StatSO`, `AttributeSO`, `ClassSO`, `GrowthFormulaSO` and friends are `ScriptableObject` references. Turning them into something a save file can hold (an addressable address, a GUID, a name) and back again is your job.
- **Deciding when to capture and restore.** Nothing on this page happens on its own. Astra exposes the state; you choose what to read, when, and where to put it.

## What Unity already round-trips

If your strategy is to serialize the whole GameObject or prefab instance (a scene save, or a prefab-instance diff), the framework's `[SerializeField]` state comes back on its own. You do not need to touch any of the following through the API:

| Component | Serialized state |
|---|---|
| `EntityCore` | the entity level, owner and owner resolution, tags, save id |
| `EntityStats` | class-vs-fixed base toggle, fixed stat set, inline fixed values, fixed-values-asset toggle and reference, cache toggle |
| `EntityAttributes` | attribute points per level, the points tracker (level points, bonus points, per-attribute investments), point-removal strategy, class-vs-fixed toggle, fixed attribute set, inline fixed values, permanent bonuses, fixed-values-asset toggle and reference, cache toggle |
| `EntityClass` | the assigned class |

For a custom save format, none of that is automatic, and the rest of this page is for you.

## Give every entity a stable id

`EntityCore` has no identity that survives a save/load cycle on its own: the only handle on an entity is the Unity object reference, and that reference does not come back. `EntitySaveId` fills exactly that gap, and nothing more. Its whole API is four members:

```csharp
public EntitySaveId SaveId { get; }        // IsEmpty is true until an id is baked or set
public EntitySaveId EnsureSaveId();        // get-or-create, idempotent
public EntitySaveId RegenerateSaveId();    // always mint a fresh one
public void SetSaveId(EntitySaveId id);    // re-apply a saved id
```

`EntitySaveId` is a string-backed serializable struct. Rebuild one from save data with `new EntitySaveId(savedString)`, and write it back out through its `Value` property.

**Astra never reads this value.** No registry, no lookups, no behavior keyed off it. An entity with an empty id behaves identically to one with an id, and `SetSaveId` accepts anything you give it (including an id another live entity already carries). Uniqueness is your concern.

### The `Instantiate()` caveat

This is the one that bites in practice. `Instantiate()` copies every serialized field, including the save id, so **every runtime copy of a prefab that carries a baked id shares that id**. For entities spawned at runtime that each need their own save record (enemies, dropped loot, pooled objects), the spawner has to break the tie:

```csharp
var enemy = Instantiate(enemyPrefab);
enemy.GetComponent<EntityCore>().RegenerateSaveId();
```

On restore, `SetSaveId(new EntitySaveId(idFromSave))` does the same job with the id your save system already holds.

### Showing the Save Identity section in the inspector

The `EntityCore` inspector can render a **Save Identity** section, but it is hidden by default. To reveal it, open `Edit > Preferences > Astra Framework > Save Identity` and enable **Show Save Identity section**. That page holds two per-user preferences, both off by default:

- **Show Save Identity section** renders the section in the `EntityCore` inspector: a read-only **Save Id** field (showing `(none)` until an id is assigned), a **Generate** button that becomes **Regenerate** once the entity has an id, and **Copy** to put the id on the clipboard. It is multi-object aware.
- **Auto-bake save id on create** gives every entity placed in a scene an id automatically, instead of requiring a click. Prefab assets and Prefab Mode are never touched, and nothing is baked during Play Mode.

> [!NOTE]
> These preferences are per-user and affect authoring only. `SaveId`, `EnsureSaveId`, `RegenerateSaveId`, and `SetSaveId` work exactly the same from code whether the section is shown or not.

> [!CAUTION]
> **Regenerate** discards the current id, and any save records already written against it stop resolving to this entity. That is why it asks for confirmation, while the first **Generate** on an entity with no id does not.

### Keeping ids unique while authoring

An editor-only guard keeps ids unique while you work: duplicating a placed entity, or dropping a second instance of a prefab that already carries a baked id, re-mints the id on the newcomer and leaves the original alone. It never touches prefab assets or Prefab Mode, and never runs in Play Mode. It cannot help with runtime `Instantiate()` (nothing editor-only can), which is why the spawner rule above exists.

## Capturing state

For a custom format, this is the read side. Everything here is public, and every getter has a matching setter covered in [Restoring](#restoring).

### `EntityCore`

- `SaveId` for identity
- `Owner` for the ownership edge (see [Entity Ownership](workflows.md#entity-ownership))

### `EntityCore.Level`

Save `CurrentTotalExperience`, not the level. The growth formula is authoritative: if it changed since the save was written, the same experience should resolve to the new level, and it will only do that if you saved the experience.

### `EntityStats`

- `UseClassBaseStats`, `StatSet`
- `GetFixedBaseStatValues()` for the fixed base values, already in the exact shape the setter expects
- `UseFixedStatValuesAsset`, `FixedStatValuesAsset`

### `EntityAttributes`

- `UseBaseAttributesFromClass`, `AttributeSet`
- `GetFixedBaseAttributeValues()` for fixed base values
- `GetAllSpentPoints()` for the invested-points portfolio
- `BonusAttributePoints` for the bonus pool (see [Bonus attribute points](workflows.md#bonus-attribute-points))
- `GetAllPermanentBonuses()` for permanent per-attribute bonuses (see [Permanent attribute bonuses](workflows.md#permanent-attribute-bonuses))
- `UseFixedAttributeValuesAsset`, `FixedAttributeValuesAsset`

Do not save `AvailableAttributePoints`, `TotalAttributePoints`, or `LevelAttributePoints`. They are derived from the level and the bonus pool, and are rebuilt during restore.

### `EntityClass`

- `Class`

A capture pass might look like this, with asset references already turned into your own string keys:

```csharp
EntitySaveData Capture(EntityCore core)
{
    var stats = core.GetComponent<EntityStats>();
    var attributes = core.GetComponent<EntityAttributes>();

    return new EntitySaveData {
        saveId = core.SaveId.Value,
        classKey = AssetKey.For(core.GetComponent<EntityClass>().Class),
        totalExperience = core.Level.CurrentTotalExperience,
        bonusAttributePoints = attributes.BonusAttributePoints,
        baseStats = ToKeyed(stats.GetFixedBaseStatValues()),
        baseAttributes = ToKeyed(attributes.GetFixedBaseAttributeValues()),
        spentPoints = ToKeyed(attributes.GetAllSpentPoints()),
        permanentBonuses = ToKeyed(attributes.GetAllPermanentBonuses()),
    };
}
```

## What not to try to save

Some state is derived, transient, or design-time, and snapshotting it is a mistake rather than a gap:

- **Transient modifiers.** Flat, percentage, and stat-to-stat modifiers are add-only by design: there is no bulk read and no removal API, because their lifetime (duration, stacking, refresh, source) belongs to the companion package `com.electricdrill.astra-rpg-modifiers`. Save the *sources* (which buff is active, and for how long) and let modifiers reapply them; do not try to snapshot the computed modifier state.
- **Caches.** The cache toggle is configuration; the cached values are derived and rebuilt on demand. There is nothing to save.
- **Design-time configuration.** Max level, the experience growth formula, attribute points per level, the point-removal strategy, stat and attribute set composition, and everything inside a `ClassSO` are authored data, not session state. They belong in your project, not in your save file.
- **Tag contents.** `EntityCore.Tags` is read-only today: authored tags round-trip through Unity serialization, but there is no API to mutate (and therefore to restore) them. If your game needs runtime tag changes, that is a feature to design with its own change events, not something to bolt onto a save path.

## Restoring

This is where a save system actually breaks, so it is worth being precise about both *when* and *in what order*.

### When

**Restore at or after `EntityCore.OnBeforeSpawned`.** That event fires from `EntityCore`'s `Start`, after the level has been wired to its owning entity and before the spawned event goes out. Subscribing from your own component's `Awake` is the intended pattern:

```csharp
private void Awake() => _core.OnBeforeSpawned += Restore;
```

Restoring earlier is not safe. Before `Start`, `EntityLevel` has no reference to its entity, so the level-changed events it raises carry a null target, and `EntityStats` / `EntityAttributes` filter on that target and skip. The attribute point budget then never grows to match the level you just restored.

`OnBeforeSpawned` is safe regardless of script execution order. `EntityAttributes` initializes its point budget from the level both in `OnEnable` and in `Start`, and that step is idempotent: whether it runs before or after your restore, you land on the same budget.

> [!CAUTION]
> Do not restore onto a deactivated instance. Instantiating inactive, restoring, then activating does not work for anything level- or points-related: with the GameObject inactive, `Start` has not run and `EntityAttributes` has not subscribed to level events, so the restore writes into state that activation then re-initializes over. Identity (`SetSaveId`) and plain configuration (`Class`, `OwnerResolution`) are fine to set before activation; everything else waits until the entity has spawned.

### In what order

Each step builds on the ones above it.

1. **Identity.** `SetSaveId`, or use the saved id to locate the entity you are restoring into.
2. **Structure**, before anything that depends on which stats and attributes exist: `EntityClass.Class`, `EntityStats.UseClassBaseStats` / `SetFixedStatSet`, `EntityAttributes.UseBaseAttributesFromClass` / `SetFixedAttributeSet`, and `EntityCore.Owner` / `OwnerResolution`. Changing the attribute set refunds every spent point (with a warning), so do it here, before you restore investments.
3. **Level and experience.** `EntityCore.Level.SetTotalCurrentExp(savedExperience)`. This is the restore entry point for the level: it clamps the experience, derives the target level from the growth formula, and drives the level there through the normal setter, so every dependent component reacts exactly as it does in play. It also grows the level-derived attribute point budget.
4. **Bonus points.** `EntityAttributes.SetBonusAttributePoints(saved)`. Together with step 3 this fixes the total budget.
5. **Base values.** `EntityStats.SetFixedStatSource(...)` and `EntityAttributes.SetFixedAttributeSource(...)`, each with the full set of values (see [Pluggable fixed base value sources](advanced-topics.md#pluggable-fixed-base-value-sources)).
6. **Spent points.** `EntityAttributes.SetAllSpentPoints(...)`. This needs the budget from steps 3 and 4: a snapshot the budget cannot cover is rejected in full, with an error, and nothing is written.
7. **Permanent bonuses.** `EntityAttributes.SetPermanentBonuses(...)`.

Steps 5 to 7 are independent of each other and can be reordered freely.

> [!NOTE]
> `SetFixedStatSource` / `SetFixedAttributeSource` always write into the entity's inline values, never into an assigned fixed-values asset (which may be shared by other entities). If the entity was using an asset for its base values, restore `UseFixedStatValuesAsset` / `UseFixedAttributeValuesAsset` and the asset reference in step 2 and skip step 5 for that component; injected inline values are ignored while the asset toggle is on.

Restoring the owner builds on [Entity Ownership](workflows.md#entity-ownership), which is currently experimental; treat that part of the restore as provisional until the ownership edge stabilizes.

### Why the bulk setters exist

`SetAllSpentPoints`, `SetFixedStatSource`, `SetFixedAttributeSource`, and `SetPermanentBonuses` apply a whole snapshot as a single state assignment. This matters because the incremental in-game APIs (`SpendOn` / `RefundFrom` and similar) each validate against the state at that instant, so replaying a saved snapshot through them makes the result depend on the order attributes happen to be replayed in, and logs an error for every intermediate step that momentarily overshoots the budget.

Two behaviors are worth knowing:

- `SetAllSpentPoints` **replaces**: attributes absent from the dictionary end at zero. Attributes no longer in the set (removed from the game since the save was written) are skipped with a warning, and everything else still applies (that is a migration, not a corruption).
- `SetPermanentBonuses` **merges**: attributes it does not mention keep the bonus they already have. Permanent bonuses accumulate from many independent sources, whereas the invested-points portfolio is one coherent allocation of one budget.

### Events during a restore

A restore raises the same events as ordinary gameplay: stat-changed, attribute-changed, attribute-points-changed, level-up / down, class-changed. That is deliberate: UI and derived systems converge on the restored state through the same code path they already use. If a listener must not react to a restore, gate that listener rather than asking the framework to restore silently.

The bulk restore paths already coalesce (one event per value that genuinely changed, not one per write). To coalesce further, across steps, wrap the whole restore in a bulk scope:

```csharp
void Restore()
{
    using (_stats.BeginBulk())
    using (_attributes.BeginBulk())
    {
        // steps 2 to 7
    }
}
```

## Where the line is

Astra's side of the contract is: stable ids, symmetric getters and setters, and bulk restore paths that survive being called with a whole snapshot at once.

Resolving asset references into keys your save file can hold, deciding what a save file contains, and finding a live entity from a saved `EntitySaveId` stay with your save system. That boundary is intentional.

For the related runtime APIs, see [Pluggable fixed base value sources](advanced-topics.md#pluggable-fixed-base-value-sources) and [Reader APIs and safe value access](advanced-topics.md#reader-apis-and-safe-value-access).
