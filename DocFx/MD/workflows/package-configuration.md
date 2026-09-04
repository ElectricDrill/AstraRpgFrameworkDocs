# Package Configuration

The base Astra Framework is designed for **zero setup out of the box** for runtime configuration: importing the **Utils** sample is all you need to get a fully working setup. The sample provides a pre-built `AstraFrameworkConfigSO` asset placed in a `Resources` subfolder (named `Astra Framework Config`), with the three required global events already wired up to the event instances also found in the sample. The editor bootstrapper auto-detects this asset and assigns it to the Global Settings the moment Unity loads, so no manual steps are required.

Authoring defaults are a separate **editor-only** convenience layer. They are fully optional, they do not affect runtime behavior, and leaving them empty preserves the same manual authoring workflow.

The configuration asset holds **Global Events** — the `GameEvent` ScriptableObjects that broadcast entity, level, stat, and attribute information to the entire game — plus an optional shared stat binding for experience gain modifiers. Three of these events are required for the framework to function correctly; the remaining event fields and the stat binding are optional.

## Zero Setup via Samples

The quickest way to configure the framework is to import the **Utils** sample from the Package Manager:

1. Open **Window → Package Manager**
2. Select **Astra Framework** in your project packages
3. Expand **Samples** and click **Import** next to **Utils**

After import, your project will contain:

- **`Assets/Samples/Astra Framework/<version>/Utils/Resources/Astra Framework Config.asset`** — the ready-to-use configuration with all required events already assigned
- **`Assets/Samples/Astra Framework/<version>/Utils/EventInstances/`** — the individual event `ScriptableObject` assets referenced by the configuration

The editor bootstrapper detects the config automatically and assigns it to the Global Settings. No Project Settings step is needed.

> [!IMPORTANT]
> The sample events are generic, shared instances intended for getting started quickly. If your game already has its own event architecture — or if you need to subscribe to entity spawned, level-up, or level-down events from your own systems — you should create a **custom configuration** and assign your own events to it (see [Creating Configuration Assets](#creating-configuration-assets) and [Project Settings](#project-settings)).

---

## Runtime Configuration Overview

The framework uses a **two-tier runtime configuration architecture**:

1. **Global Settings** (`AstraFrameworkGlobalSettingsSO`) — A lightweight pointer stored in `Resources` that references your active configuration
2. **Framework Configuration** (`AstraFrameworkConfigSO`) — The actual configuration containing all global event references and shared stat bindings

This separation allows you to:
- Switch between different configuration profiles easily (e.g., for testing)
- Keep configuration data separate from the loading mechanism
- Support convention-based fallbacks for quick prototyping

For editor-time asset and component pre-filling, see [Authoring Defaults](#authoring-defaults).

---

## Global Settings

### Automatic Setup

The package automatically creates the Global Settings asset on first import or when the editor loads. You don't need to do anything manually.

**What happens automatically:**
1. The `AstraFrameworkGlobalSettings.asset` is created in `Assets/Resources/`
2. If a default configuration exists (e.g., from imported samples), it is automatically assigned
3. The system is immediately ready to use

> [!NOTE]
> Default location: `Assets/Resources/AstraFrameworkGlobalSettings.asset`

### Project Settings

You can manage the package configuration through Unity's Project Settings window:

1. Open **Edit → Project Settings**
2. Navigate to **Astra Framework**
3. Assign your desired **Active Config Profile**

**Status Indicators:**
- ✅ **"Using Explicit Configuration"** — A configuration is explicitly assigned
- ⚠️ **"No explicit profile assigned. System is using the Fallback..."** — No explicit configuration; using convention-based fallback
- 🛑 **"No Configuration found!"** — No configuration available; global events will not be dispatched

**Quick Actions:**
- **Create New Config Asset** — Opens a save dialog to create a new configuration

### Manual Configuration (alternative to Project Settings)

You can also manage the configuration by directly editing the `AstraFrameworkGlobalSettingsSO` asset in the `Assets/Resources` folder. This lets you assign a specific `AstraFrameworkConfigSO` without using the Project Settings window.

### Convention Over Configuration

The package follows a **convention-over-configuration** philosophy to reduce setup friction.

#### Fallback Resolution

If no explicit configuration is assigned in Project Settings, the system automatically searches for a configuration named:

> **`Astra Framework Config`**

located in any **`Resources`** folder in your project.

#### Search Order

The configuration provider uses a **three-step loading strategy**:

1. **Explicit Configuration** (Project Settings)
   - Loads `AstraFrameworkGlobalSettingsSO` from `Resources/AstraFrameworkGlobalSettings`
   - If it has an `ActiveConfig` assigned, use it

2. **Convention-Based Fallback**
   - Searches for `Astra Framework Config` in any `Resources` folder
   - Logs a message indicating fallback usage

3. **Error State**
   - If neither is found, logs an error with instructions
   - Global events will not be dispatched until a configuration is provided

> [!TIP]
> For production projects, always use **explicit configuration** via Project Settings for clarity and control.

---

## Authoring Defaults

Authoring defaults are **project-scoped, editor-only defaults** configured in `Edit > Project Settings > Astra Framework > Authoring Defaults`. They pre-fill selected references when you add framework components or create framework assets. They are never read by gameplay code, and every assignment in this page is optional.

The manual **Authoring Defaults** section shown in supported inspectors is controlled separately through the user preference at `Edit > Preferences... > Astra Framework > Authoring Defaults`. This preference affects only whether the extra **Apply** and **Replace** actions are rendered in inspectors. It does **not** change the project-wide defaults configured in Project Settings, and it does **not** disable automatic default assignment when new supported objects are created.

> [!NOTE]
> Authoring defaults do **not** change the framework's zero-setup philosophy. The framework works without them, and they exist only to make repeated authoring safer, clearer, and more customizable.

### Why authoring defaults replaced script default references

Earlier authoring flows could rely on script-level default references that pointed to assets imported from the package samples. That was convenient, but it could also create silent project drift after a package update: newly created objects could start referencing assets from the newly imported sample version, while older objects still referenced assets from an older sample version.

Authoring defaults move that responsibility into explicit project settings. This makes the choice visible, project-owned, and easy to standardize across a team. If no default is configured, the corresponding field simply stays empty and can still be assigned manually.

### What can be configured

Each authoring-default group has its own enable toggle, so you can automate only the parts of the workflow that are useful for your project:

- **Level Defaults** — applied when `EntityCore` is added; can assign **Default Max Level** and **Default Experience Formula**
- **Growth Formula Defaults** — applied when `GrowthFormulaSO` assets are created; can assign **Default Max Level**
- **Class Defaults** — applied when `ClassSO` assets are created; can assign **Default Stat Set**, **Default Attribute Set**, and an optional **Default Max HP Formula**
- **Entity Stats Defaults** — applied when `EntityStats` is added; can assign **Default Fixed Base Stats Set** when the component uses fixed base stats
- **Entity Attributes Defaults** — applied when `EntityAttributes` is added; can assign **Default Fixed Base Attribute Set** when the component uses fixed base attributes

For class authoring, the same settings page also exposes the optional automatic growth-formula creation helpers used when new `ClassSO` assets are created.

> [!NOTE]
> A field left empty stays manual. You can configure only a shared `Max Level`, only a fixed-base set, only class defaults, or nothing at all.

### Inspector actions

Supported inspectors can expose a manual **Authoring Defaults** section with two actions:

- **Apply Authoring Defaults** — fills only empty or unassigned fields using the current project defaults
- **Replace With Authoring Defaults** — asks for confirmation and then also overwrites fields that are already configured

This section is available for `EntityCore`, `GrowthFormulaSO`, `ClassSO`, `EntityStats`, and `EntityAttributes`.

If you want to show it, open `Edit > Preferences > Astra Framework > Authoring Defaults` and enable **Show Authoring Defaults inspector actions**.

If you leave this preference disabled, the inspectors simply hide these manual buttons. The automatic authoring-default workflow configured in Project Settings still applies normally when supported components or assets are created.

This is useful when a project decides to standardize on a shared set of authoring assets after some objects have already been created.

### Recommended usage

If your project always expects the same `Max Level`, experience formula, class sets, or fixed-base sets, assign those project-owned assets in **Authoring Defaults** so newly created objects follow one explicit standard. If you prefer per-object setup, leave the defaults empty and continue authoring manually.

When possible, prefer project-owned assets over assets living inside imported sample folders. The goal of authoring defaults is to keep these references explicit and under project control.

---

## Configuration Loading Strategy

The configuration is loaded lazily on first access and cached for performance:

```csharp
// Automatically loads configuration on first access
var config = AstraFrameworkConfigProvider.Instance;

// Pre-load during initialization to avoid runtime overhead
AstraFrameworkConfigProvider.WarmUp();

// Force reload (useful for testing or after domain reload)
AstraFrameworkConfigProvider.Reset();
```

**When is the configuration loaded?**
- Automatically before the first scene loads (via `RuntimeInitializeOnLoadMethod`)
- Lazily when first accessed via `AstraFrameworkConfigProvider.Instance`
- Explicitly when calling `WarmUp()`

---

## Creating Configuration Assets

### Via Project Settings

1. Open **Edit → Project Settings → Astra Framework**
2. Unassign any existing configuration, if any
3. Click **Create New Config Asset**
4. Choose a save location
5. The new configuration is automatically assigned

### Via Asset Menu

1. Right-click in the **Project Window**
2. Select **Create → Astra Framework → Config**
3. Name your configuration
4. Assign it in Project Settings or in the Global Settings asset

---

## Configuration Reference

> [!NOTE]
> In the `AstraFrameworkConfigSO` asset, you can hover over each field to see a tooltip with a brief description.

The `AstraFrameworkConfigSO` asset contains the global event references and shared stat bindings used by the core entity components.

The red asterisks (<span style="color:red;">*</span>) indicate fields that are required for the system to function properly. Make sure to assign them before entering Play Mode to avoid missing event dispatches.

### Global Events

Global Events are `GameEvent` ScriptableObjects that broadcast framework-level information to the **entire game**. They are assigned once in the configuration asset and shared by all entities, ensuring a single authoritative source for framework-level communication.

Each global event is broadcast through the **`FrameworkEventChannel`** system. This means that alongside the global event configured here, individual entity components can also hold per-entity extra events that are raised together with the global one when the channel fires. All entity components that expose event channels implement `IEventRegistrar`, which provides `Subscribe<TEvent>` and `Unsubscribe<TEvent>` for per-entity extra event registration.

> [!WARNING]
> Three events are **required** — if they are missing, the corresponding entity lifecycle events are never dispatched, which breaks systems that depend on entity spawn, level-up, or level-down notifications. Assign them before entering Play Mode.

> [!IMPORTANT]
> If your game already has systems built around your own entity spawned, level-up, or level-down events, **do not reuse the sample events**. Create your own `AstraFrameworkConfigSO`, assign your events to it, and register it in **Project Settings → Astra Framework**. That way your existing subscriptions continue to work seamlessly.

#### Global Entity Spawned Event <span style="color:red;">*</span>
**Type:** `EntityCoreGameEvent`  
**Required:** Yes  
**Raised by:** `EntityCore`  
**Description:** Raised once, at the end of the entity's `Start()` lifecycle, after the level, stats, and attribute components have all initialized. Carries a direct reference to the `EntityCore` that just spawned.

Subscribe to this event to track entity registration in global systems (e.g., registering an entity with an AI manager or a health-bar tracker).

**Per-entity extra events API:**
```csharp
entityCore.Subscribe<EntityCoreGameEvent>(myEntitySpecificEvent);
entityCore.Unsubscribe<EntityCoreGameEvent>(myEntitySpecificEvent);
```

---

#### Global Entity Level Up Event <span style="color:red;">*</span>
**Type:** `EntityLevelUpGameEvent`  
**Required:** Yes  
**Raised by:** `EntityLevel`  
**Description:** Raised when any entity's level increases. Carries an `EntityLevelChangedContext` payload. Raised once per transition, even when several levels are crossed at once (`AbsAmount` holds the number of levels gained).

**Payload — `EntityLevelChangedContext`:**

| Property | Type | Description |
|---|---|---|
| `Target` | `EntityCore` | The entity whose level changed |
| `PreviousValue` | `int` | The level before the increase |
| `NewValue` | `int` | The level after the increase |
| `AbsAmount` | `int` | The absolute number of levels gained |

**Per-entity extra events API:**
```csharp
entityCore.Level.Subscribe<EntityLevelUpGameEvent>(myLevelUpEvent);
entityCore.Level.Unsubscribe<EntityLevelUpGameEvent>(myLevelUpEvent);
```

---

#### Global Entity Level Down Event <span style="color:red;">*</span>
**Type:** `EntityLevelDownGameEvent`  
**Required:** Yes  
**Raised by:** `EntityLevel`  
**Description:** Raised when any entity's level decreases. Carries an `EntityLevelChangedContext` payload with the same fields as the level-up event. Raised once per transition, even when several levels are crossed at once (`AbsAmount` holds the number of levels lost).

**Per-entity extra events API:**
```csharp
entityCore.Level.Subscribe<EntityLevelDownGameEvent>(myLevelDownEvent);
entityCore.Level.Unsubscribe<EntityLevelDownGameEvent>(myLevelDownEvent);
```

---

#### Experience Gain Modifier Stat
**Type:** `StatSO`  
**Required:** No  
**Read by:** `EntityCore` / `EntityLevel`  
**Description:** Optional shared `StatSO` binding used when `EntityLevel` applies experience gain modifiers. Assign the stat asset once in the active configuration, and each entity will read its own value for that stat from `EntityStats` when calling `AddExp(...)`.

If this field is empty, or if a given entity does not expose that stat in `EntityStats`, the entity gains experience without an additional modifier.

> [!TIP]
> This replaces the older per-entity setup. You now choose the stat asset once in `AstraFrameworkConfigSO` instead of assigning the same stat on every entity.

---

#### Global Stat Changed Event
**Type:** `StatChangedGameEvent`  
**Required:** No  
**Raised by:** `EntityStats`  
**Description:** Raised whenever any stat's computed value changes on any entity — whether due to a flat modifier, a percentage modifier, a stat-to-stat dependency, or an attribute-driven scaling update. Carries a `StatChangeInfo` payload.

**Payload — `StatChangeInfo`:**

| Property | Type | Description |
|---|---|---|
| `EntityStats` | `EntityStats` | The component that owns the changed stat |
| `Stat` | `Stat` | The stat asset that changed |
| `Target` / `Entity` | `EntityCore` | The owning entity |
| `PreviousValue` | `long` | The stat's computed value before the change |
| `NewValue` | `long` | The stat's computed value after the change |
| `AbsAmount` | `long` | The absolute magnitude of the change |

**Example use cases:**
- Updating a HUD bar when a character's Attack stat changes
- Triggering ability unlock logic when a threshold stat is reached
- Logging stat changes for analytics

**Per-entity extra events API:**
```csharp
entityStats.Subscribe<StatChangedGameEvent>(myStatChangedEvent);
entityStats.Unsubscribe<StatChangedGameEvent>(myStatChangedEvent);
```

---

#### Global Attribute Changed Event
**Type:** `AttributeChangedGameEvent`  
**Required:** No  
**Raised by:** `EntityAttributes`  
**Description:** Raised whenever any attribute value changes on any entity. Carries an `AttributeChangeInfo` payload.

**Payload — `AttributeChangeInfo`:**

| Property | Type | Description |
|---|---|---|
| `EntityAttributes` | `EntityAttributes` | The component that owns the changed attribute |
| `Attribute` | `Attribute` | The attribute asset that changed |
| `Target` / `Entity` | `EntityCore` | The owning entity |
| `PreviousValue` | `long` | The attribute's value before the change |
| `NewValue` | `long` | The attribute's value after the change |
| `AbsAmount` | `long` | The absolute magnitude of the change |

**Per-entity extra events API:**
```csharp
entityAttributes.Subscribe<AttributeChangedGameEvent>(myAttributeChangedEvent);
entityAttributes.Unsubscribe<AttributeChangedGameEvent>(myAttributeChangedEvent);
```

> [!NOTE]
> Changing an attribute may also trigger the **Global Stat Changed Event** for any stats whose scaling formula depends on that attribute. Both events are dispatched independently.

---

## Troubleshooting

### "No configuration found" error

**Cause:** No configuration is assigned and no fallback exists.

**Solution:**
1. Check **Project Settings → Astra Framework**
2. Assign a configuration or create a new one
3. Alternatively, create a config named `Astra Framework Config` in a `Resources` folder

### "Using Fallback" log message

**Cause:** No explicit configuration assigned in Project Settings.

**Solution:**
1. Open **Project Settings → Astra Framework**
2. Assign the fallback configuration explicitly
3. This message is informational only and won't break functionality

### Configuration not updating in Play Mode

**Cause:** Configuration is cached on first access.

**Solution:**
```csharp
// Force reload
AstraFrameworkConfigProvider.Reset();
```

### Missing Resources folder

**Cause:** The `Assets/Resources/` folder doesn't exist.

**Solution:** The package creates it automatically. If it's missing, create it manually:
1. Right-click `Assets`
2. Create → Folder → Name it `Resources`
3. Restart Unity to trigger the editor bootstrapper
