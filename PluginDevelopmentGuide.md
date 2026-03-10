# Plugin Development Guide

## 1. Getting Started

### Prerequisites
- **MSVC v143** (Visual Studio 2022)
- **C++20** (`/std:c++20`)
- **x64 Release** build
- **Runtime Library**: `/MD` (Multi-threaded DLL) — must match the host

### Project Setup
1. Create a new **C++ DLL** project in Visual Studio
2. Set the include path to point to the POEFixer source directory (for SDK and ImGui headers)
3. Add ImGui source files to your project: `imgui.cpp`, `imgui_draw.cpp`, `imgui_tables.cpp`, `imgui_widgets.cpp`
4. Include the Plugin SDK headers in your plugin source:
   ```cpp
   #include "plugin_sdk/PluginAPI.h"
   #include "plugin_sdk/PluginContext.h"
   #include "imgui/imgui.h"
   ```
   Or use the convenience header from ExamplePlugin:
   ```cpp
   #include "sdk/PluginHelpers.h"  // Includes all SDK headers + MemoryReader + utilities
   ```
5. Define `PLUGIN_EXPORTS` and `_CRT_SECURE_NO_WARNINGS` in your project's Preprocessor Definitions

### Folder Structure
```
Plugins/
  YourPlugin/
    YourPlugin.dll      <-- DLL name MUST match folder name
    config/
      settings.txt      <-- Optional settings file
    data/
      ...               <-- Optional data directory (databases, caches, etc.)
```

### ExamplePlugin Project Structure
The ExamplePlugin demonstrates the recommended project layout:
```
Plugins/ExamplePlugin/
  ExamplePlugin.cpp        <-- Main plugin entry point + factory exports
  sdk/
    PluginHelpers.h        <-- MemoryReader, WideToNarrow, entity/rarity helpers
  examples/
    ExampleBuffs.h         <-- Buff list with filtering and progress bars
    ExampleEntities.h      <-- Entity debug list with watch mechanism, component trees, JSON dump
    ExampleInventory.h     <-- ServerData, inventory selector, slot grid, item mods with rarity
    ExampleMemory.h        <-- Hex viewer, Read<T> demo, pattern scanner
    ExampleUiExplorer.h    <-- Full UI element explorer with search, navigation, highlighting
```

### KillCount Plugin Structure
A more complete plugin example with SQLite3, icon atlas, and overlay rendering:
```
Plugins/KillCount/
  KillCount.cpp            <-- Main plugin entry point, IPlugin lifecycle, settings UI
  KillCount.h              <-- Plugin class declaration
  KillTracker.cpp/h        <-- Kill/chest/death counting engine
  OverlayRenderer.cpp/h    <-- ImGui overlay with drag-to-reposition pattern
  IconAtlas.cpp/h           <-- Sprite sheet texture loading (D3D11 + stb_image)
  Database.cpp/h           <-- SQLite3 wrapper for persistent stats
  DisplaySettings.h        <-- Settings struct
  sdk/
    PluginHelpers.h        <-- Copied from ExamplePlugin
  lib/
    sqlite3.c/h            <-- SQLite3 amalgamation (compiled as C)
    sqlite3-vcpkg-config.h <-- Local override for static linking
```

### Naming Convention
The DLL file name **must** match the folder name exactly:
- Folder: `Plugins/MyPlugin/` → DLL: `MyPlugin.dll`
- The host scans each subfolder in `Plugins/` and looks for `<FolderName>.dll`

---

## 2. Plugin Lifecycle

```
Load DLL (LoadLibrary)
  → CreatePlugin()           -- Factory: instantiate your IPlugin
  → SetContext(ctx)           -- Receive host services
  → SetPluginDirectory(dir)   -- Receive your folder path
  → GetSDKVersion()           -- Compatibility check
  → GetName()                 -- Display name for UI
  → [if enabled] OnEnable()  -- Initialize resources
  ↓
  Main Loop (every frame):
    → DrawUI()                -- Render your overlay (only if enabled)
    → DrawSettings()          -- Render settings in Plugins tab
    → WantsOverlay()          -- Host checks if plugin wants overlay mode
  ↓
  Periodically / on shutdown:
    → SaveSettings()          -- Persist your settings
  ↓
  → OnDisable()               -- Cleanup resources
  → DestroyPlugin(plugin)     -- Factory: delete your IPlugin
  → FreeLibrary               -- Unload DLL
```

### Threading
- All `Draw*` methods are called on the **main/render thread**
- `GetSnapshot()` and other PluginContext functions are **thread-safe**
- Do NOT spawn threads that call ImGui — ImGui is not thread-safe

---

## 3. IPlugin Interface Reference

Every plugin must implement the `IPlugin` interface (defined in `plugin_sdk/PluginAPI.h`):

### `void SetPluginDirectory(const char* dir)`
- **When called**: Once, immediately after creation
- **Parameter**: Relative path like `"Plugins/YourPlugin"`
- **Purpose**: Store this path for loading settings/resources

### `void SetContext(PluginContext* context)`
- **When called**: Once, after `SetPluginDirectory`
- **Parameter**: Pointer to the host's `PluginContext` (valid for plugin lifetime)
- **Purpose**: Store this pointer — it's your gateway to all game data
- **Important**: Call `ImGui::SetCurrentContext(ctx->ImGuiContext)` here

### `void OnEnable(bool isGameOpened)`
- **When called**: When user enables the plugin, or on startup if previously enabled
- **Parameter**: `true` if the game process is currently attached
- **Purpose**: Load settings, allocate resources, initialize state

### `void OnDisable()`
- **When called**: When user disables the plugin
- **Purpose**: Release resources, stop background work

### `void DrawUI()`
- **When called**: Every frame, only when plugin is enabled
- **Purpose**: Render your overlay using ImGui
- **Note**: Use unique window IDs like `"MyWindow##MyPlugin"` to avoid conflicts

### `void DrawSettings()`
- **When called**: Every frame, in the Plugins settings tab (only when enabled)
- **Purpose**: Render plugin configuration using ImGui

### `void SaveSettings()`
- **When called**: Periodically and on application shutdown
- **Purpose**: Save your settings to disk (e.g., `Plugins/YourPlugin/config/settings.txt`)

### `const char* GetName()`
- **Returns**: Display name shown in the Plugins tab (e.g., `"My Plugin"`)

### `int GetSDKVersion()`
- **Returns**: `PLUGIN_SDK_VERSION` (currently 4)
- **Purpose**: Host checks this for compatibility — must match

### `bool WantsOverlay()` *(SDK v2)*
- **Returns**: `true` if the plugin wants to render in overlay mode (transparent overlay on top of game)
- **Default**: `false` — plugin only renders in normal settings window
- **Purpose**: When any plugin returns `true`, the host enters overlay mode even if no built-in features require it

### Factory Exports
Your DLL must export these two C functions:
```cpp
extern "C" PLUGIN_API IPlugin* CreatePlugin() {
    return new MyPlugin();
}

extern "C" PLUGIN_API void DestroyPlugin(IPlugin* plugin) {
    delete plugin;
}
```

---

## 4. PluginContext API Reference

The `PluginContext` struct (defined in `plugin_sdk/PluginContext.h`) provides function pointers for accessing game data. All types are in the `PluginSDK` namespace.

### Game Data Access

#### `GetSnapshot()` → `shared_ptr<const PluginGameSnapshot>`
Returns a complete snapshot of the game state. Updated once per frame. Contains:

| Field | Type | Description |
|-------|------|-------------|
| `CurrentState` | `GameStateTypes` | Current game state |
| `CurrentAreaName` | `string` | Area name (e.g., "The Riverways") |
| `CurrentAreaHash` | `string` | Unique area instance hash |
| `CurrentAreaLevel` | `uint8_t` | Monster level of current area |
| `IsTown` | `bool` | True if in town |
| `IsHideout` | `bool` | True if in hideout |
| `IsPaused` | `bool` | True if game is paused |
| `IsSkillTreeVisible` | `bool` | True if skill tree panel is open |
| `WorldToGridConvertor` | `float` | Conversion factor for world→grid |
| `Player` | `RadarEntity` | Local player entity data |
| `Entities` | `vector<RadarEntity>` | All nearby entities |
| `LargeMap` / `MiniMap` | `MapData` | Map overlay data |
| `Vitals` | `PlayerVitals` | Player HP/ES/MP + buffs |
| `ScreenWidth` / `ScreenHeight` | `int` | Game window dimensions |
| `ProcessId` | `DWORD` | Game process ID |
| `GameWindow` | `HWND` | Game window handle |
| `GameWindowForeground` | `bool` | True if game is foreground |
| `IsAttached` | `bool` | True if attached to game process |
| `AreaChangeCounter` | `uint64_t` | Increments on area change |
| `Inventories` | `vector<InventoryInfo>` | Player inventory contents |
| `CurrencyTotals` | `map<string,int>` | Currency counts by path |
| `InventoryGrid` | `InventoryGridInfo` | Inventory UI grid info |
| `WorldToScreenMatrix` | `XMFLOAT4X4` | 3D→2D projection matrix |

> **Important: Entity Filtering**
> Dead entities (those with `EntityState == Useless`) are filtered OUT of the snapshot before plugins receive it. This means you will **never** observe an HP transition from alive to dead. If you need to detect kills, use **disappearance-based detection** instead — track entity IDs by zone and count them as killed when they disappear from the entity list while in `InnerCircle` or `OuterCircle` proximity. See [Section 8: Common Recipes](#detect-monster-kills-disappearance-based) for details.

#### `GetPlayerVitals()` → `PlayerVitals`
Convenience shortcut for player vitals.

#### `GetCurrentState()` → `GameStateTypes`
Returns the current game state enum.

#### `IsAttached()` → `bool`
True if the game process is attached and readable.

#### `IsInGame()` → `bool`
True if currently in game (not loading, not on login screen).

#### `IsGameForeground()` → `bool`
True if the game window is the foreground window.

#### `GetProcessId()` → `DWORD`
Returns the game process ID.

### Item Data Access

#### `ReadExtendedItemMods(entityAddress)` → `ExtendedItemModInfo`
Read all mods from an item entity.

#### `ReadItemRarity(entityAddress)` → `int`
Returns: 0=Normal, 1=Magic, 2=Rare, 3=Unique

#### `ReadItemStackCount(entityAddress)` → `int`
Returns stack count for currency/stackable items.

#### `ReadItemName(entityAddress)` → `string`
Returns the item's base type name.

#### `ReadItemPath(entityAddress)` → `string`
Returns the item's metadata path.

### Overlay Mode *(SDK v2)*

#### `IsOverlayMode()` → `bool`
Returns true if the host is currently in overlay mode (transparent overlay on top of the game window). Use this to adjust your rendering — e.g., draw on game overlay vs. draw in a settings window.

### UI State *(SDK v4)*

#### `IsMenuVisible()` → `bool`
Returns true when the host's settings menu is visible (overlay is interactive). When the menu is hidden, the overlay window is click-through (`WS_EX_TRANSPARENT`), so ImGui windows cannot receive mouse input.

Use this to implement the **draggable overlay pattern**:
- **Menu visible**: Show a drag handle, allow interaction (tabs, buttons)
- **Menu hidden**: Remove drag handle, add `ImGuiWindowFlags_NoInputs` to make the window non-interactive

See [Section 6: Draggable Overlay Pattern](#draggable-overlay-pattern-sdk-v4) for the full implementation.

### Memory Reading *(SDK v2)*

Direct access to game process memory. All reads are safe (return 0/empty on failure).

#### `GetBaseAddress()` → `uintptr_t`
Returns the game executable module base address. Returns 0 if not attached.

#### `GetModuleSize()` → `uintptr_t`
Returns the game module size in bytes. Returns 0 if not attached.

#### `ReadProcessMemory(address, buffer, size)` → `bool`
Read a block of raw bytes from the game process. The `buffer` must have at least `size` bytes allocated. Returns true on success.

```cpp
// Example: Read a 4-byte integer from game memory
uint32_t value = 0;
m_Context->ReadProcessMemory(address, &value, sizeof(value));

// Example: Read a struct
MyStruct data{};
m_Context->ReadProcessMemory(structAddress, &data, sizeof(data));
```

#### `ReadString(address)` → `string`
Read a null-terminated ASCII string from game memory (max 128 chars).

#### `ReadUnicodeString(address)` → `wstring`
Read a null-terminated Unicode (wide) string from game memory (max 128 wchars).

#### `GetPatternAddress(patternName)` → `uintptr_t`
Get a resolved pattern scan address by name. Returns 0 if not found.

Standard patterns:
| Name | Description |
|------|-------------|
| `"Game States"` | GameStates vector root |
| `"File Root"` | File registry |
| `"AreaChangeCounter"` | Area transition counter |
| `"Terrain Rotator Helper"` | Rotation data |
| `"Terrain Rotation Selector"` | Rotation selector |
| `"GameCullSize"` | Screen cull value |

### World-to-Screen Projection *(SDK v2)*

#### `WorldToScreen(worldX, worldY, worldZ, outX, outY)` → `bool`
Convert a world-space position to screen coordinates. Returns true if the position is visible on screen.

```cpp
float screenX, screenY;
if (m_Context->WorldToScreen(entity.WorldX, entity.WorldY, entity.WorldZ, &screenX, &screenY)) {
    ImGui::GetBackgroundDrawList()->AddText(ImVec2(screenX, screenY), IM_COL32_WHITE, "Label");
}
```

### Inventory *(SDK v2)*

#### `RequestInventoryScan(inventoryId)`
Request the host to scan inventories. Pass `-1` to scan all inventories, or a specific inventory ID. Inventory data in the snapshot is populated after the scan completes (next frame).

**Note**: Inventory data is not automatically refreshed — you must call this function to trigger a scan. Call it periodically (e.g., every 2 seconds) if you need continuous inventory data.

### Terrain Data *(SDK v2)*

#### `GetWalkableGrid(outWidth, outHeight)` → `const uint8_t*`
Returns a pointer to the walkable grid data. The grid is a 2D array where 0 = not walkable, non-zero = walkable. Returns nullptr if data is not available.

#### `GetTerrainHeight(gridX, gridY)` → `float`
Returns the terrain height at a grid position. Returns 0 if out of bounds or data unavailable.

### Native Container Reading *(SDK v3)*

These functions read C++ standard library containers directly from game memory, mirroring the host's `Core::Process` methods.

#### `ReadStdVector(containerAddress, elementSize, outCount)` → `void*`
Read a `StdVector` (24-byte struct: `{First, Last, End}`) from game memory. Returns a `malloc`'d buffer of elements. Caller must `free()` the returned pointer. Returns `nullptr` on failure.

```cpp
// Example: Read a vector of uint32_t
int count = 0;
void* data = m_Context->ReadStdVector(vectorAddr, sizeof(uint32_t), &count);
if (data && count > 0) {
    uint32_t* values = static_cast<uint32_t*>(data);
    for (int i = 0; i < count; i++) { /* values[i] */ }
    free(data);
}
```

#### `ReadStdList(containerAddress, elementSize, outCount)` → `void*`
Read a `StdList` (16-byte struct: `{Head, Size}`) from game memory. Traverses the linked list and returns a contiguous buffer. Caller must `free()`.

#### `ReadStdBucket(containerAddress, elementSize, outCount)` → `void*`
Read a `StdBucket` from game memory (reads the embedded `StdVector`). Caller must `free()`.

#### `ReadStdMap(containerAddress, keySize, valueSize, callback, userData)` → `int`
Traverse a `StdMap` (16-byte struct: `{Head, Size}`) and call `callback` for each key-value pair. Returns the number of nodes visited.

```cpp
// Example: Read a map<uint32_t, float>
struct MapResult { std::vector<std::pair<uint32_t, float>> entries; };
MapResult result;
m_Context->ReadStdMap(mapAddr, sizeof(uint32_t), sizeof(float),
    [](const void* key, const void* value, void* userData) {
        auto* r = static_cast<MapResult*>(userData);
        uint32_t k; float v;
        memcpy(&k, key, sizeof(k));
        memcpy(&v, value, sizeof(v));
        r->entries.push_back({k, v});
    }, &result);
```

#### `ReadStdWString(containerAddress)` → `wstring`
Read a `StdWString` (32-byte struct with inline/heap buffer) from game memory.

#### `GetInventoryName(inventoryId)` → `const char*`
Returns the human-readable name for an inventory ID (e.g., 1 → `"MainInventory1"`, 3 → `"Weapon1"`, 64 → `"Currency1"`).

### Debug Data Access *(SDK v4)*

SDK v4 provides direct access to the host's debug data — entity components, inventory details, and UI element tree — matching the built-in Debug tabs.

#### Entity Debug List

#### `GetEntityDebugList()` → `vector<DebugEntityInfo>`
Returns a list of all entities with debug metadata (Id, Address, Path, Type, SubType, State, Rarity, Zone). This mirrors the Debug→Entity List tab.

#### `WatchEntity(entityId)`
Start watching an entity's components. The host's worker thread will read full component data for this entity each frame.

#### `UnwatchEntity(entityId)`
Stop watching an entity's components. Call this when the user collapses the entity tree node to free resources.

#### `GetWatchedEntityData(entityId)` → `DebugEntityComponents`
Returns the full component data for a watched entity. Contains sub-structs for all 8 recognized components (Life, Render, Positioned, Targetable, Animated, Stats, Actor, Buffs) plus the list of all component addresses.

```cpp
// Example: Watch entity on expand, read components
auto entities = m_Context->GetEntityDebugList();
for (auto& e : entities) {
    if (ImGui::TreeNode(e.Path.c_str())) {
        m_Context->WatchEntity(e.Id);
        auto data = m_Context->GetWatchedEntityData(e.Id);
        if (data.HasLife) {
            ImGui::Text("HP: %d / %d", data.Life.Current.Value, data.Life.Max.Value);
        }
        ImGui::TreePop();
    } else {
        m_Context->UnwatchEntity(e.Id);
    }
}
```

#### Inventory Debug

#### `GetServerDataAddress()` → `uintptr_t`
Returns the ServerData component base address.

#### `GetPlayerInventoryList()` → `vector<pair<int, uintptr_t>>`
Returns all player inventory IDs and their addresses (from ServerData).

#### `WatchInventory(inventoryId)`
Start watching an inventory for detailed debug inspection. The host reads slot occupancy, item details, and mods.

#### `GetWatchedInventoryData()` → `DebugInventoryData`
Returns full data for the currently watched inventory: grid dimensions, slot occupancy, items with rarity and mods.

```cpp
// Example: Inventory inspector
auto invList = m_Context->GetPlayerInventoryList();
m_Context->WatchInventory(invList[0].first);
auto inv = m_Context->GetWatchedInventoryData();
for (auto& item : inv.Items) {
    ImGui::Text("[%s] Rarity=%d Mods=%d", item.Path.c_str(), item.Rarity,
        (int)(item.ImplicitMods.size() + item.ExplicitMods.size()));
}
```

#### UI Element Tree

#### `GetGameUiRootAddress()` → `uintptr_t`
Returns the root game UI element address (for in-game UI tree navigation).

#### `GetUiRootAddress()` → `uintptr_t`
Returns the top-level UI root address.

#### `GetGameCullValue()` → `int`
Returns the current GameCullSize value, used for UI scale calculations. Combined with screen dimensions, this enables accurate UI element position/size computation.

```cpp
// Example: UI scale calculation (matching host logic)
int cullValue = m_Context->GetGameCullValue();
auto snapshot = m_Context->GetSnapshot();
// Scale for index 1 (width): screenWidth / (cullValue / baseWidth)
// Scale for index 2 (height): screenHeight / (cullValue / baseHeight)
```

### PluginHelpers.h — Convenience Wrapper *(SDK v3)*

The `sdk/PluginHelpers.h` header (included with ExamplePlugin) provides a type-safe `MemoryReader` class that wraps the raw PluginContext functions:

```cpp
PluginSDK::MemoryReader mem(m_Context);

// Read a single struct
auto data = mem.Read<MyStruct>(address);

// Read an array
auto arr = mem.ReadArray<uint32_t>(address, count);

// Read native containers — returns std::vector<T>
auto vec = mem.ReadStdVector<uint32_t>(vectorAddr);
auto list = mem.ReadStdList<MyNode>(listAddr);
auto bucket = mem.ReadStdBucket<MyEntry>(bucketAddr);
auto map = mem.ReadStdMap<uint32_t, float>(mapAddr);
auto wstr = mem.ReadStdWString(wstringAddr);
```

Additional utilities in `PluginHelpers.h`:
- `WideToNarrow(wstring)` — safe wstring→string conversion
- `GetEntityTypeName(type)` — enum to display name
- `GetNearbyZoneName(zone)` — zone to display name
- `GetRarityName(rarity)` / `GetRarityColor(rarity)` — rarity display helpers

### Host Services

#### `Log(level, message)`
Write to the host's log system. Levels: `"Debug"`, `"Info"`, `"Warning"`, `"Error"`

#### `ImGuiContext` (void*)
The host's ImGui context. Call `ImGui::SetCurrentContext()` with this in `SetContext()`.

#### `D3DDevice` (void*)
The host's `ID3D11Device*`. Cast and use for loading textures.

---

## 5. Data Structures Reference

All types are in the `PluginSDK` namespace. Plugins typically add `using namespace PluginSDK;`.

### `RadarEntity`
Per-entity data available in `snapshot->Entities`:

| Field | Type | Description |
|-------|------|-------------|
| `Id` | `uint32_t` | Unique entity ID |
| `Address` | `uintptr_t` | Memory address (for item API calls) |
| `IsValid` | `bool` | Entity validity flag |
| `entityType` | `EntityTypes` | Entity category |
| `entitySubtype` | `EntitySubtypes` | Entity subcategory |
| `entityState` | `EntityStates` | Entity state |
| `Rarity` | `int` | 0=Normal, 1=Magic, 2=Rare, 3=Unique |
| `Reaction` | `uint8_t` | 0=Hostile, 1=Neutral, 2=Friendly |
| `GridPositionX/Y` | `float` | Position on terrain grid |
| `WorldX/Y/Z` | `float` | World-space position |
| `ModelBoundsZ` | `float` | Model height |
| `Path` | `wstring` | Entity metadata path |
| `PlayerName` | `wstring` | Player name (if player entity) |
| `CurrentHP/MaxHP` | `int` | Entity health |
| `CurrentES/MaxES` | `int` | Entity energy shield |
| `IsSleeping` | `bool` | Far-away entity flag |
| `IsChestOpened` | `bool` | Chest opened state |
| `Zone` | `NearbyZone` | Proximity to player |
| `ComponentCache` | `EntityComponentCache` | Component addresses |

> **Important**: Dead entities (`EntityState::Useless`) are removed from the snapshot before it reaches plugins. You will never see a monster's HP drop to zero — it simply disappears from the list. Use the `Zone` field to distinguish kills (entity disappeared from InnerCircle/OuterCircle) from entities going out of range (entity was in Far zone).

### `Buff`
Active buff/debuff:

| Field | Type | Description |
|-------|------|-------------|
| `Name` | `string` | Internal buff name (e.g., `"flask_effect_life"`) |
| `TimeLeft` | `float` | Seconds remaining |
| `Charges` | `short` | Stack count |
| `TotalTime` | `float` | Total duration |

### `MapData`
Minimap/largemap state:

| Field | Type | Description |
|-------|------|-------------|
| `CenterX/Y` | `float` | Map center |
| `SizeX/Y` | `float` | Map dimensions |
| `ShiftX/Y` | `float` | Current pan offset |
| `Zoom` | `float` | Zoom level |
| `IsVisible` | `bool` | Map is currently shown |

### `InventoryInfo` / `InventoryItemInfo`

| Field | Type | Description |
|-------|------|-------------|
| `Id` | `int` | Inventory ID |
| `TotalBoxesX/Y` | `int` | Grid dimensions |
| `Items` | `vector<InventoryItemInfo>` | Items in inventory |

Item fields: `Address`, `Name`, `Path`, `SlotX/Y`, `Width/Height`, `StackCount`, `IsCurrency`

### `DebugEntityInfo` *(SDK v4)*
Entity metadata from `GetEntityDebugList()`:

| Field | Type | Description |
|-------|------|-------------|
| `Id` | `uint32_t` | Entity ID |
| `Address` | `uintptr_t` | Memory address |
| `Path` | `string` | Metadata path |
| `entityType` | `EntityTypes` | Entity category |
| `entitySubtype` | `EntitySubtypes` | Entity subcategory |
| `entityState` | `EntityStates` | Entity state |
| `Rarity` | `int` | 0=Normal, 1=Magic, 2=Rare, 3=Unique |
| `Zone` | `NearbyZone` | Proximity to player |

### `DebugEntityComponents` *(SDK v4)*
Full component data from `GetWatchedEntityData()`:

| Field | Type | Description |
|-------|------|-------------|
| `EntityId` | `uint32_t` | Which entity this data is for |
| `ComponentAddresses` | `vector<pair<string,uintptr_t>>` | All component name→address pairs |
| `HasLife` / `Life` | `bool` / `DebugLifeComp` | Life component (HP/ES/MP vitals) |
| `HasRender` / `Render` | `bool` / `DebugRenderComp` | Render component (position, bounds, model name) |
| `HasPositioned` / `Positioned` | `bool` / `DebugPositionedComp` | Grid position, rotation, scale |
| `HasTargetable` / `Targetable` | `bool` / `DebugTargetableComp` | Targetable/hostile/friendly flags |
| `HasAnimated` / `Animated` | `bool` / `DebugAnimatedComp` | Animation path, current animation ID |
| `HasStats` / `Stats` | `bool` / `DebugStatsComp` | Game stats (stat ID → value map) |
| `HasActor` / `Actor` | `bool` / `DebugActorComp` | Active skills, action ID |
| `HasBuffs` / `Buffs` | `bool` / `vector<DebugBuff>` | Active buffs with full details |

### `DebugInventoryData` *(SDK v4)*
Inventory details from `GetWatchedInventoryData()`:

| Field | Type | Description |
|-------|------|-------------|
| `InventoryId` | `int` | Inventory ID (-1 if none) |
| `Address` | `uintptr_t` | Inventory address |
| `TotalBoxesX/Y` | `int` | Grid dimensions |
| `ServerRequestCounter` | `int` | Server sync counter |
| `SlotOccupied` | `vector<bool>` | Per-slot occupancy |
| `Items` | `vector<DebugInventoryItem>` | Items with path, rarity, and mods |

### `DebugInventoryItem` *(SDK v4)*

| Field | Type | Description |
|-------|------|-------------|
| `Address` | `uintptr_t` | Item entity address |
| `Path` | `string` | Item metadata path |
| `Rarity` | `int` | 0=Normal, 1=Magic, 2=Rare, 3=Unique |
| `ImplicitMods` | `vector<DebugModInfo>` | Implicit modifiers |
| `ExplicitMods` | `vector<DebugModInfo>` | Explicit modifiers |
| `EnchantMods` | `vector<DebugModInfo>` | Enchant modifiers |
| `HellscapeMods` | `vector<DebugModInfo>` | Hellscape modifiers |

### `DebugModInfo` *(SDK v4)*

| Field | Type | Description |
|-------|------|-------------|
| `Name` | `string` | Mod display name |
| `Value0` | `float` | First value (NaN if none) |
| `Value1` | `float` | Second value (NaN if none) |

### Enums

**`EntityTypes`**: Unidentified(0), Chest(1), NPC(2), Player(3), Shrine(4), Monster(5), DeliriumBomb(6), DeliriumSpawner(7), OtherImportantObjects(8), Item(9), Renderable(10), AreaTransition(11)

**`EntitySubtypes`**: _Unidentified(0), _None(1), PlayerSelf(2), PlayerOther(3), ChestWithMagicRarity(4), ChestWithRareRarity(5), ExpeditionChest(6), BreachChest(7), Strongbox(8), SpecialNPC(9), POIMonster(10), PinnacleBoss(11), WorldItem(12), InventoryItem(13)

**`EntityStates`**: None(0), Useless(1), PlayerLeader(2), MonsterFriendly(3), PinnacleBossHidden(4)

**`NearbyZone`**: None(0), InnerCircle(1, ~60 grid units), OuterCircle(2, ~120 grid units), Far(3)

**`GameStateTypes`**: AreaLoadingState(0), ChangePasswordState(1), CreditsState(2), EscapeState(3), InGameState(4), PreGameState(5), LoginState(6), WaitingState(7), CreateCharacterState(8), SelectCharacterState(9), DeleteCharacterState(10), LoadingState(11), GameNotLoaded(12)

---

## 6. ImGui Usage in Plugins

### Shared Context
The host and plugin share the same ImGui context. You **must** call:
```cpp
ImGui::SetCurrentContext(static_cast<ImGuiContext*>(m_Context->ImGuiContext));
```
in your `SetContext()` method.

### Window IDs
Always use unique window IDs to avoid conflicts with host or other plugins:
```cpp
ImGui::Begin("My Window##MyPluginName", &showWindow);
```

### Available Features
- Windows, tabs, trees, tables, draw lists
- Texture loading via D3D11 device
- Overlay rendering via `ImGui::GetBackgroundDrawList()`
- FontAwesome 6 icons via `#include "imgui/IconsFontAwesome6.h"` (e.g., `ICON_FA_ARROWS_UP_DOWN_LEFT_RIGHT`)

### Overlay Rendering *(SDK v2)*
Use `WorldToScreen()` to draw labels/shapes at entity positions:
```cpp
float sx, sy;
if (m_Context->WorldToScreen(entity.WorldX, entity.WorldY, entity.WorldZ, &sx, &sy)) {
    auto* drawList = ImGui::GetBackgroundDrawList();
    drawList->AddText(ImVec2(sx, sy - 20), IM_COL32(255, 255, 0, 255), "Monster");
    drawList->AddCircleFilled(ImVec2(sx, sy), 4.0f, IM_COL32(255, 0, 0, 255));
}
```

### Overlay Mode
Override `WantsOverlay()` to return `true` to request the host to enter overlay mode:
```cpp
bool WantsOverlay() override { return m_OverlayEnabled; }
```
When in overlay mode, the host window is transparent and positioned over the game. Your `DrawUI()` calls render directly onto the game screen.

### Draggable Overlay Pattern *(SDK v4)*

The host overlay uses `WS_EX_TRANSPARENT` to make the window click-through when the menu is hidden. This means ImGui windows **cannot receive mouse input** unless the menu is visible. To create a draggable overlay window (like the built-in Vitals Overlay), use this dual-mode pattern:

```cpp
#include "imgui/IconsFontAwesome6.h"

void MyPlugin::RenderOverlay() {
    bool menuVisible = m_Context->IsMenuVisible ? m_Context->IsMenuVisible() : false;

    if (menuVisible) {
        // === DRAGGABLE MODE ===
        // Window with background, drag hint, interactive controls
        ImGui::SetNextWindowPos(ImVec2(m_PosX, m_PosY), ImGuiCond_Appearing);
        ImGui::SetNextWindowBgAlpha(m_Alpha);

        ImGuiWindowFlags flags = ImGuiWindowFlags_NoTitleBar |
            ImGuiWindowFlags_NoScrollbar | ImGuiWindowFlags_AlwaysAutoResize |
            ImGuiWindowFlags_NoSavedSettings | ImGuiWindowFlags_NoCollapse |
            ImGuiWindowFlags_NoFocusOnAppearing;

        ImGui::Begin("##MyOverlay", nullptr, flags);

        // Drag hint (the entire window is draggable since there's no title bar)
        ImGui::TextColored(ImVec4(0.5f, 0.5f, 0.5f, 1.0f),
            ICON_FA_ARROWS_UP_DOWN_LEFT_RIGHT " Drag to reposition");
        ImGui::Spacing();

        // ... your content here (tabs, text, icons, buttons — all interactive) ...

        // Persist position when user drags the window
        ImVec2 pos = ImGui::GetWindowPos();
        if (pos.x != m_PosX || pos.y != m_PosY) {
            m_PosX = pos.x;
            m_PosY = pos.y;
            // Save to settings on next SaveSettings() call
        }

        ImGui::End();
    }
    else {
        // === NON-INTERACTIVE MODE ===
        // Static overlay — no drag, no mouse interaction
        ImGui::SetNextWindowPos(ImVec2(m_PosX, m_PosY));
        ImGui::SetNextWindowBgAlpha(m_Alpha);

        ImGuiWindowFlags flags = ImGuiWindowFlags_NoTitleBar |
            ImGuiWindowFlags_NoScrollbar | ImGuiWindowFlags_AlwaysAutoResize |
            ImGuiWindowFlags_NoSavedSettings | ImGuiWindowFlags_NoCollapse |
            ImGuiWindowFlags_NoFocusOnAppearing |
            ImGuiWindowFlags_NoMove | ImGuiWindowFlags_NoResize |
            ImGuiWindowFlags_NoInputs;

        ImGui::Begin("##MyOverlay", nullptr, flags);
        // ... your content here (display-only, no interactive controls) ...
        ImGui::End();
    }
}
```

**Key points:**
- `ImGuiCond_Appearing` sets position only on first show; ImGui then tracks drag position
- `NoTitleBar` + no `NoMove` = window is draggable from any empty area (ImGui default behavior)
- `NoInputs` in non-interactive mode prevents the overlay from stealing focus through `WS_EX_TRANSPARENT`
- Always null-check `IsMenuVisible` pointer for backward compatibility: `m_Context->IsMenuVisible ? m_Context->IsMenuVisible() : false`
- Save position to your settings file so it persists across sessions

---

## 7. Settings Persistence

### Recommended Pattern
```cpp
void OnEnable(bool isGameOpened) override {
    LoadSettings();  // Load from Plugins/YourPlugin/config/settings.txt
}

void SaveSettings() override {
    // Write to Plugins/YourPlugin/config/settings.txt
    // The host calls this periodically and on shutdown
}
```

### File Location
Store settings in `<PluginDirectory>/config/`:
```cpp
std::filesystem::path settingsPath =
    std::filesystem::path(m_Directory) / "config" / "settings.txt";
```

### Simple Key-Value Format
For plugins with many settings, a simple key=value text format works well:
```cpp
// Save
std::ofstream file(configDir / "settings.txt");
file << "ShowOverlay=" << (m_Settings.ShowOverlay ? 1 : 0) << "\n";
file << "WindowAlpha=" << m_Settings.WindowAlpha << "\n";
file << "PosX=" << m_Settings.PosX << "\n";
file << "PosY=" << m_Settings.PosY << "\n";

// Load
std::ifstream file(settingsPath);
std::string line;
while (std::getline(file, line)) {
    auto eq = line.find('=');
    if (eq == std::string::npos) continue;
    std::string key = line.substr(0, eq);
    std::string val = line.substr(eq + 1);
    if (key == "ShowOverlay") m_Settings.ShowOverlay = (val == "1");
    else if (key == "WindowAlpha") m_Settings.WindowAlpha = std::stof(val);
    else if (key == "PosX") m_Settings.PosX = std::stof(val);
    else if (key == "PosY") m_Settings.PosY = std::stof(val);
}
```

---

## 8. Common Recipes

### Get player HP percentage
```cpp
auto vitals = m_Context->GetPlayerVitals();
int hpPercent = vitals.HPPercent; // 0-100
```

### List all monsters within inner circle
```cpp
auto snapshot = m_Context->GetSnapshot();
for (auto& e : snapshot->Entities) {
    if (e.entityType == EntityTypes::Monster &&
        e.Zone == NearbyZone::InnerCircle) {
        // e.CurrentHP, e.Path, e.WorldX/Y/Z...
    }
}
```

### Check if player has a specific buff
```cpp
auto vitals = m_Context->GetPlayerVitals();
for (auto& buff : vitals.Buffs) {
    if (buff.Name == "flask_effect_life") {
        // buff.TimeLeft, buff.Charges...
    }
}
```

### Get current area info
```cpp
auto snapshot = m_Context->GetSnapshot();
std::string area = snapshot->CurrentAreaName;
bool isTown = snapshot->IsTown;
int level = snapshot->CurrentAreaLevel;
```

### Draw text at entity world position *(SDK v2)*
```cpp
for (auto& e : snapshot->Entities) {
    if (e.entityType != EntityTypes::Monster) continue;
    float sx, sy;
    if (m_Context->WorldToScreen(e.WorldX, e.WorldY, e.WorldZ, &sx, &sy)) {
        auto* dl = ImGui::GetBackgroundDrawList();
        dl->AddText(ImVec2(sx, sy - 15), IM_COL32(255, 255, 0, 255), "Monster");
    }
}
```

### Read item mods from inventory
```cpp
// First, request an inventory scan (call periodically, e.g. every 2s)
m_Context->RequestInventoryScan(-1);

// Then read from snapshot (next frame)
auto snapshot = m_Context->GetSnapshot();
for (auto& inv : snapshot->Inventories) {
    for (auto& item : inv.Items) {
        auto mods = m_Context->ReadExtendedItemMods(item.Address);
        // mods.ExplicitMods, mods.ImplicitMods...
    }
}
```

### Count entities by type
```cpp
auto snapshot = m_Context->GetSnapshot();
int monsters = 0, items = 0;
for (auto& e : snapshot->Entities) {
    if (e.entityType == EntityTypes::Monster) monsters++;
    if (e.entityType == EntityTypes::Item) items++;
}
```

### Detect area change
```cpp
static uint64_t lastAreaChange = 0;
auto snapshot = m_Context->GetSnapshot();
if (snapshot->AreaChangeCounter != lastAreaChange) {
    lastAreaChange = snapshot->AreaChangeCounter;
    // Area changed! Reset state...
}
```

### Check if in loading screen
```cpp
if (m_Context->GetCurrentState() == GameStateTypes::AreaLoadingState) {
    // Currently loading...
}
```

### Detect monster kills (disappearance-based)
Dead entities are filtered out of the snapshot (`EntityState::Useless`), so you cannot detect HP dropping to 0. Instead, track entities by ID and detect when they disappear from nearby zones:

```cpp
// In your tracker class:
struct TrackedEntity {
    uint32_t Id;
    int Rarity;
    PluginSDK::NearbyZone Zone;
};

std::unordered_map<uint32_t, TrackedEntity> m_PrevEntities;

void DetectKills(const std::vector<PluginSDK::RadarEntity>& entities) {
    std::unordered_set<uint32_t> currentIds;

    // Update tracking map with current monsters
    for (const auto& e : entities) {
        if (e.entityType != EntityTypes::Monster) continue;
        if (e.entityState == EntityStates::MonsterFriendly) continue;
        currentIds.insert(e.Id);
        m_PrevEntities[e.Id] = { e.Id, e.Rarity, e.Zone };
    }

    // Disappeared from InnerCircle/OuterCircle = killed
    for (auto it = m_PrevEntities.begin(); it != m_PrevEntities.end(); ) {
        if (currentIds.count(it->first) == 0) {
            if (it->second.Zone == NearbyZone::InnerCircle ||
                it->second.Zone == NearbyZone::OuterCircle) {
                OnMonsterKilled(it->second.Rarity);
            }
            it = m_PrevEntities.erase(it);
        } else {
            ++it;
        }
    }
}
```

**Why this works**: Entities within ~120 grid units that suddenly vanish were almost certainly killed (not just walked out of range). Entities in the `Far` zone naturally pop in and out of the entity list — don't count those.

**Important**: Clear `m_PrevEntities` on area change (`AreaChangeCounter` changed) to avoid false positives.

### Read raw game memory *(SDK v2)*
```cpp
// Read a struct from a known address
struct MyGameStruct { int field1; float field2; };
MyGameStruct data{};
if (m_Context->ReadProcessMemory(someAddress, &data, sizeof(data))) {
    // data.field1, data.field2 are now populated
}

// Read a string from memory
std::string str = m_Context->ReadString(stringAddress);
```

### Use pattern scan results *(SDK v2)*
```cpp
uintptr_t gameStatesAddr = m_Context->GetPatternAddress("Game States");
if (gameStatesAddr != 0) {
    // Read data at the resolved pattern address
    uint64_t value = 0;
    m_Context->ReadProcessMemory(gameStatesAddr, &value, sizeof(value));
}
```

### Check walkable terrain *(SDK v2)*
```cpp
int gridW = 0, gridH = 0;
const uint8_t* grid = m_Context->GetWalkableGrid(&gridW, &gridH);
if (grid && gridW > 0 && gridH > 0) {
    int x = (int)snapshot->Player.GridPositionX;
    int y = (int)snapshot->Player.GridPositionY;
    if (x >= 0 && x < gridW && y >= 0 && y < gridH) {
        bool walkable = grid[y * gridW + x] != 0;
    }
}
```

### Read typed memory with MemoryReader *(SDK v3)*
```cpp
PluginSDK::MemoryReader mem(m_Context);

// Read a struct from a known address
struct GameData { int level; float health; };
auto data = mem.Read<GameData>(address);

// Read a StdVector of pointers
auto ptrs = mem.ReadStdVector<uintptr_t>(vectorAddr);
for (auto ptr : ptrs) { /* process each pointer */ }

// Read a StdMap<int, float>
auto entries = mem.ReadStdMap<int, float>(mapAddr);
for (auto& [key, value] : entries) { /* key, value */ }
```

### Get inventory name *(SDK v3)*
```cpp
for (auto& inv : snapshot->Inventories) {
    const char* name = m_Context->GetInventoryName(inv.Id);
    // name is e.g. "MainInventory1", "Weapon1", "Currency1"
}
```

### Read entity component addresses
```cpp
for (auto& e : snapshot->Entities) {
    auto& cc = e.ComponentCache;
    if (cc.HasLife()) {
        // cc.LifeAddr contains the Life component address
        // Use MemoryReader to read component structs
    }
    if (cc.HasRender()) {
        // cc.RenderAddr has the Render component address
    }
}
```

### Inspect entity components via debug watch *(SDK v4)*
```cpp
// Get all entities with debug info
auto entities = m_Context->GetEntityDebugList();
for (auto& e : entities) {
    bool open = ImGui::TreeNode(e.Path.c_str());
    if (open) {
        m_Context->WatchEntity(e.Id);
        auto comp = m_Context->GetWatchedEntityData(e.Id);
        if (comp.HasLife) {
            ImGui::Text("HP: %d/%d  ES: %d/%d",
                comp.Life.Current.Value, comp.Life.Max.Value,
                comp.Life.CurrentES.Value, comp.Life.MaxES.Value);
        }
        if (comp.HasActor) {
            ImGui::Text("Action: %d  Skills: %d",
                comp.Actor.ActionId, (int)comp.Actor.Skills.size());
        }
        ImGui::TreePop();
    } else {
        m_Context->UnwatchEntity(e.Id);
    }
}
```

### Inspect inventory with slot grid *(SDK v4)*
```cpp
auto invList = m_Context->GetPlayerInventoryList();
if (!invList.empty()) {
    m_Context->WatchInventory(invList[0].first);
    auto inv = m_Context->GetWatchedInventoryData();
    if (inv.InventoryId >= 0) {
        ImGui::Text("Grid: %dx%d  Items: %d",
            inv.TotalBoxesX, inv.TotalBoxesY, (int)inv.Items.size());
        for (auto& item : inv.Items) {
            ImGui::Text("[R%d] %s  Mods: %d/%d/%d/%d",
                item.Rarity, item.Path.c_str(),
                (int)item.ImplicitMods.size(), (int)item.ExplicitMods.size(),
                (int)item.EnchantMods.size(), (int)item.HellscapeMods.size());
        }
    }
}
```

### Navigate UI element tree *(SDK v4)*
```cpp
uintptr_t uiRoot = m_Context->GetGameUiRootAddress();
if (uiRoot) {
    PluginSDK::MemoryReader mem(m_Context);
    // Read children vector at offset 0x010
    auto children = mem.ReadStdVector<uintptr_t>(uiRoot + 0x010);
    for (auto childAddr : children) {
        // Read StringId at offset 0x448
        uintptr_t strPtr = mem.Read<uintptr_t>(childAddr + 0x448);
        if (strPtr) {
            std::string name = m_Context->ReadString(strPtr);
            ImGui::Text("Child: %s (0x%llX)", name.c_str(), childAddr);
        }
    }
}
```

### Read item mods with display helpers *(SDK v3)*
```cpp
auto mods = m_Context->ReadExtendedItemMods(item.Address);
ImGui::TextColored(
    PluginSDK::GetRarityColor(mods.Rarity),
    "Rarity: %s", PluginSDK::GetRarityName(mods.Rarity));
for (auto& mod : mods.ExplicitMods) {
    ImGui::BulletText("%s", mod.Key.c_str());
}
```

---

## 9. Building and Deploying

### Build Settings
| Setting | Value |
|---------|-------|
| Configuration | Release |
| Platform | x64 |
| C++ Standard | `/std:c++20` |
| Runtime Library | `/MD` (Multi-threaded DLL) |
| Configuration Type | DLL |

### Required Files in Plugin Project
- Your plugin `.cpp` file(s)
- ImGui source files: `imgui.cpp`, `imgui_draw.cpp`, `imgui_tables.cpp`, `imgui_widgets.cpp`
- Include path to POEFixer root (for SDK headers and ImGui headers)
- Optional: Copy `Plugins/ExamplePlugin/sdk/PluginHelpers.h` for the `MemoryReader` wrapper and utility functions

### Include Paths
Your `.vcxproj` should have these additional include directories:
```xml
<AdditionalIncludeDirectories>$(SolutionDir)POEFixer;%(AdditionalIncludeDirectories)</AdditionalIncludeDirectories>
```
If using local third-party libraries (e.g., SQLite3 in a `lib/` subfolder), add `$(ProjectDir)lib` before the solution path so local headers take priority:
```xml
<AdditionalIncludeDirectories>$(ProjectDir)lib;$(SolutionDir)POEFixer;%(AdditionalIncludeDirectories)</AdditionalIncludeDirectories>
```

### Third-Party Libraries

#### SQLite3 (Static Linking)
To use SQLite3 in a plugin, you must compile the amalgamation source directly into your DLL — Windows `LoadLibrary` does not search the DLL's own directory for dependencies, so dynamically linking `sqlite3.dll` will fail with error 126.

Steps:
1. Copy `sqlite3.c` and `sqlite3.h` into your plugin's `lib/` directory
2. Create `lib/sqlite3-vcpkg-config.h` to override `SQLITE_API` (prevents `__declspec(dllimport)` errors):
   ```c
   #ifndef SQLITE_API
   #define SQLITE_API
   #endif
   #define SQLITE_ENABLE_UNLOCK_NOTIFY 1
   #define SQLITE_OS_WIN 1
   #define SQLITE_ENABLE_COLUMN_METADATA 1
   ```
3. Add `sqlite3.c` to your `.vcxproj` as a C file with warnings disabled:
   ```xml
   <ClCompile Include="lib\sqlite3.c">
     <CompileAs>CompileAsC</CompileAs>
     <WarningLevel>TurnOffAllWarnings</WarningLevel>
     <SDLCheck>false</SDLCheck>
     <PreprocessorDefinitions>SQLITE_THREADSAFE=1;_CRT_SECURE_NO_WARNINGS;%(PreprocessorDefinitions)</PreprocessorDefinitions>
   </ClCompile>
   ```

#### stb_image (Texture Loading)
For loading textures from image files (PNG, JPG), include stb_image in one `.cpp` file:
```cpp
#define STB_IMAGE_IMPLEMENTATION
#include "stb_image.h"
```
Then use D3D11 device from `m_Context->D3DDevice` to create GPU textures.

### Output
Set your output directory to:
```
$(SolutionDir)x64\Release\Plugins\YourPlugin\
```

### Deployment
Copy your built DLL to `Plugins/YourPlugin/YourPlugin.dll` next to the main executable.

### Debugging
1. Build your plugin DLL in Debug mode
2. Start the host application
3. In Visual Studio: Debug → Attach to Process → select the host .exe
4. Set breakpoints in your plugin source
5. The debugger will break when your code is called

---

## 10. Troubleshooting

| Problem | Solution |
|---------|----------|
| Plugin not loading | Check DLL name matches folder name exactly |
| "SDK version mismatch" | Rebuild plugin with latest SDK headers (current version: 4) |
| LoadLibrary error 126 | DLL has unresolved dependencies. For third-party libs like SQLite3, compile them statically into the DLL (see Section 9). Use `dumpbin /dependents YourPlugin.dll` to check. |
| Crash on load | Check CRT mismatch — both must use `/MD` |
| ImGui not rendering | Ensure `ImGui::SetCurrentContext()` is called in `SetContext()` |
| Data is empty/zero | Check `IsAttached()` and `IsInGame()` before reading data |
| Inventory is empty | Call `RequestInventoryScan(-1)` — inventory data is on-demand |
| "Missing exports" error | Ensure `CreatePlugin` and `DestroyPlugin` are exported with `extern "C"` |
| Plugin crashes host | This shouldn't happen — all plugin calls are SEH-protected. Check the logs. |
| Stale data | `GetSnapshot()` returns the latest frame's data. Don't cache the pointer. |
| Memory read returns 0 | Verify `IsAttached()` is true and the address is valid |
| WorldToScreen returns false | The position may be behind the camera or off-screen |
| Overlay window not clickable | The host uses `WS_EX_TRANSPARENT` when menu is hidden. Use `IsMenuVisible()` to show interactive controls only when menu is active. See the Draggable Overlay Pattern in Section 6. |
| Kill/death detection doesn't work | Dead entities are removed from the snapshot. Use disappearance-based detection instead of HP transition. See Section 8. |
| C2491 "dllimport function" errors | Your third-party library headers define `__declspec(dllimport)`. Create a local override header that sets the API macro to empty (see SQLite3 example in Section 9). |
