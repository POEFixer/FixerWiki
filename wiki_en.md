# POEFixer Wiki

Welcome to the POEFixer documentation. Below is a breakdown of the program's functionality, organized by interface tabs.

---

## 1. Login (Trade Browser)

The **Login** tab is the starting point for your trading experience. It integrates a full-featured web browser directly into the application, allowing you to interact with the official Path of Exile Trade website without leaving the tool.

### Key Features

#### Built-in Web Browser

Powered by **WebView2**, the embedded browser provides a seamless experience identical to using Chrome or Edge.

- **Session Management:** The application automatically captures your session cookies. This is crucial for the **Trade** tab to connect to the official WebSocket API and receive live updates.
- **Game Mode Support:** Automatically detects and supports both Path of Exile 1 and Path of Exile 2 trade sites based on your settings.
- **Navigation:** Includes an address bar for manual URL entry and a **Home** button to quickly return to the main trade search page.

#### Adding Trade Filters

Once you have configured a search on the official site (e.g., "Headhunter" with specific stats), you can add it to the application's monitoring system.

1. **Configure Search:** Use the browser to set up your filters on the trade site.
2. **Click "Add":** Press the large **Add** button at the top of the tab.
3. **Validation:** The tool checks if the URL is valid and matches your current game mode (PoE 1 vs PoE 2).
4. **Confirmation:** A dialog will appear, allowing you to name the search before it starts monitoring in the **Trade** tab.

> **Note:** If you see a "WebView2 Missing" error, the application will provide a link to download the necessary runtime from Microsoft.

*(Screenshot: Login Tab Overview)*

*(Screenshot: Adding a new trade link)*

---

## 2. Trade

The **Trade** tab is the command center for real-time market monitoring. It utilizes the official Path of Exile WebSocket API to receive instant notifications about new item listings that match your search criteria.

### Interface Overview

The main view displays a list of all your active trade search connections.

*(Screenshot: Trade Tab Main View)*

#### Connection List Columns

1.  **Status:**
    -   **Green:** Connected and listening for new items.
    -   **Orange:** Connecting...
    -   **Red:** Disconnected.
    -   **Rate Limit:** If you are rate-limited by the API, a countdown timer will appear here showing when you can reconnect.
2.  **Name:** The name of your search (e.g., "Headhunter").
    -   **Click:** Opens the search URL in your default web browser.
    -   **Tooltip:** Hover to see the full URL.
3.  **Statistics:**
    -   **Handshake Icon:** Number of items found in this session.
    -   **Check Icon:** Number of successful whisper messages sent.
    -   **X Icon:** Number of failed whisper attempts.
4.  **Actions:**
    -   **Play/Stop Button:** Manually start or stop the live search for this specific connection.
    -   **Shopping Cart:** Opens the **Manual Buy** menu to bulk purchase items from this search (WIP).
    -   **Pen (Edit):** Update the Name or URL of the connection.
    -   **Gear (Settings):** Opens the detailed **Settings & Logs** window for this connection.
    -   **Trash (Delete):** Permanently remove this search connection.

### Global Controls

At the bottom of the tab, you will find global controls:

-   **Connect All:** Sequentially connects to all enabled searches. It adds a small delay between connections to prevent triggering API rate limits.
-   **Disconnect All:** Immediately stops all active searches.
-   **Add Link:** Manually add a trade search URL if you prefer not to use the **Login** tab browser.

### Connection Settings & Logs

Clicking the **Gear** icon on any connection opens a detailed modal window.

*(Screenshot: Connection Settings Window)*

#### 1. General Settings

-   **Auto Reconnect:** If enabled, the tool will automatically try to reconnect if the connection drops.
-   **Enable Notification:** Shows a popup notification overlay in-game when a new item is found.
-   **Multi-Item Whisper Delay:** Adds a configurable delay (in milliseconds) between whisper messages. This is crucial for high-volume searches to avoid being flagged for spamming.

#### 2. Currency Filters

Filter incoming notifications based on the listing price. This allows you to ignore listings that are too cheap or too expensive without changing the website filters.

-   **PoE 1 / PoE 2 Support:** The list of currencies automatically adjusts based on the selected game mode.
-   **Min/Max:** Set minimum and maximum quantities for specific currencies (e.g., only notify for items listed between 100 and 150 Chaos Orbs).

#### 3. Trade Logs

On the right side of the settings window, you can see a history of items found by this specific search.

*(Screenshot: Trade Logs View)*

-   **Item Card:** Displays the item icon, name, and listed price.
-   **Seller Info:** Shows the player name and how long ago the item was listed.
-   **Mods:** Lists the explicit modifiers of the item.
-   **Status Indicators:**
    -   **Travel:** Indicates if the seller's hideout was successfully visited.
    -   **Buy:** Indicates if the trade message was successfully copied/sent.

---

## 3. Settings

The **Settings** tab is the control center for configuring the application's global behavior, game integration, and user interface preferences.

### Game Selection

- **Path of Exile 2 / Path of Exile 1:** Select the game mode you are currently playing. This adjusts the trade search URLs, window detection logic, and scanner compatibility.
- **League:** Choose your current league (e.g., Standard, Hardcore, or temporary leagues) from the dropdown menu to ensure trade searches target the correct economy.

### Scanner Type

Choose how the application reads data from the game client.

- **WGC (Windows Graphics Capture):**
  -   **Description:** Captures the game window visually even when it's in the background. Recommended for PoE 1 or if Memory Scanner is unavailable.
  -   **Corner Density:** Adjusts the threshold for frame analysis (advanced setting).
  -   **Use Full Window:** When enabled, scans the entire game window. If disabled, you can manually define a specific screen area to scan (useful for ultrawide monitors or specific setups).
  -   **Select Area:** Interactive tool to draw the scanning rectangle on your screen.
- **Memory Scanner (PoE 2 Only):**
  -   **Description:** Directly reads game memory to identify items and entities. This is generally faster and more accurate for PoE 2 but requires the game to be in a compatible state.

### Click Settings

Configure how the application interacts with the game mouse input, primarily for automated trading or inventory management.

- **Enable Auto Click:** Master switch to allow the application to perform mouse clicks.
- **Click Count:** Number of clicks to perform per action (if there is no currency in the inventory, the item may not be bought the first time).
- **Click Interval:** Delay (in milliseconds) between clicks.
- **Scan Timeout:** Maximum time (in seconds) the scanner will attempt to find an item before giving up.
- **Hold Ctrl:** Automatically holds the `Ctrl` key during clicks (useful for fast-moving items between inventory and stash).
- **Return Cursor:** Returns the mouse cursor to its original position after performing a click action.
- **Block Input:** Temporarily blocks user mouse input while the application is performing an automated action to prevent interference.
- **Enable Rate Limit:** Limits the frequency of actions to prevent flagging by game servers.
- **Fast Trade:** Optimizes settings for rapid trading interactions.
- **Fast Mouse Click:** Reduces the input delay for mouse down/up events.

### Advanced

- **Return to Hideout:** Automatically attempts to type `/hideout` in chat after certain actions (if configured).
- **Streamer Mode:** Hides the application's overlay and windows from streaming software (like OBS) so viewers only see the game.

### Desktop Notifications

Customize the pop-up notifications that appear when a trade search finds an item.

- **Enable Desktop Notifications:** Toggle system-level notifications on/off.
- **Position:** Choose where notifications appear on your screen (Top-Right, Top-Left, Bottom-Right, Bottom-Left).
- **Display Time:** How long the notification stays visible (in seconds).
- **Enable Sound:** Plays a sound effect when a notification appears.
- **Display Options:**
  -   **Show Icon:** Displays the item's icon.
  -   **Show Price:** Shows the listed price.
  -   **Show Seller:** Shows the seller's character name.
  -   **Show Mods:** Lists the item's explicit modifiers.
- **Test Notification:** Sends a sample notification to test your current settings.

### Theme & Appearance

- **Theme:** Choose from 4 visual themes for the entire application:
  - **Dark** — Default dark theme with subtle contrast.
  - **Light** — Light theme for well-lit environments.
  - **Blue** — Dark theme with blue accent colors.
  - **Midnight** — Deep dark theme with minimal brightness.
- **Font Scale:** Adjust the global text size (0.8x to 1.5x) for better readability on high-resolution displays.

### Language & Interface

- **Language:** Switch the application's interface language (e.g., English, Russian, Korean, etc.). 11 languages are supported.
- All theme-related strings are localized across all supported languages.

*(Screenshot: Settings Tab Overview)*

---

## 4. AutoPot [PoE 2]

The **AutoPot** tab provides automated flask usage and utility features specifically for Path of Exile 2. It includes a comprehensive system for managing your character's Health (HP), Energy Shield (ES), and Mana (MP) through automated actions and visual overlays.

### Key Features

#### 1. Input Device & Driver Status

The system relies on a virtual controller to simulate input safely.

- **ViGEmBus Driver:** Checks if the required ViGEmBus driver is installed and provides a download link if missing.
- **Controller Detection:** Automatically detects physical controllers (Gamepad) and virtual controllers.
- **Safety Warnings:** Warns if you have bound gamepad keys but no controller is detected.

#### 2. Vitals Overlay (HUD)

A customizable visual overlay that displays your current HP, ES, and MP on top of the game.

- **Display Modes:** Choose from 7 different styles:
  - *Horizontal Bars* (Classic look)
  - *Vertical Bars* (Compact side view)
  - *Percent Text* (Simple text readout)
  - *Circles* (Ring indicators)
  - *Compact Blocks* (Minimalist squares)
  - *Minimal Dots* (Unobtrusive dots)
  - *Glass Panel* (Transparent background)
- **Customization:**
  - **Scale:** Adjust the size of the overlay (0.5x to 3.0x).
  - **Toggle Visibility:** Independently show or hide HP, ES, and MP bars.
  - **Numeric Values:** Toggle between percentage and absolute numbers.
  - **Draggable:** When the menu is open, a "Drag Hint" appears, allowing you to move the overlay anywhere on the screen.

#### 3. Automation Modes

Select the logic that best fits your character build:

- **Life:** Monitors HP only.
- **Energy Shield (ES):** Monitors ES only.
- **Life + ES:** Monitors both.
- **Mind Over Matter (MOM):** Monitors MP (often used for damage mitigation).
- **MOM + Life:** Monitors MP and HP.
- **MOM + ES:** Monitors MP and ES.
- **All:** Monitors HP, ES, and MP simultaneously.

#### 4. Threshold Configuration

Precise control over when potions are used.

- **Sliders:** Set the percentage threshold (1-100%) for HP, ES, and MP. When values drop below this level, the corresponding key is triggered.
- **Smart Visibility:** Only sliders relevant to your selected **Mode** are shown to keep the UI clean.

#### 5. Auto Disconnect (Chicken)

A safety feature to automatically log out or disconnect if your vitals drop to a critical level.

- **Separate Triggers:** Independent enable toggles and thresholds for HP, ES, and MP.
- **Visual Indicators:** Thresholds are visualized on the Vitals Overlay as dashed lines or markers, showing exactly when the disconnect will trigger.

#### 6. Key Bindings & Delays

Configure which keys to press for each action.

- **Key 1:** Typically used for Health (HP) or Energy Shield (ES) depending on the selected mode.
- **Key 2:** Typically used for Mana (MP) or as a secondary flask trigger.
- **Key 3:** Dedicated key for Energy Shield (ES), available exclusively in "All" mode.
- **Input Types:** Supports both Keyboard keys and Gamepad buttons.
- **Delays:** Set a cooldown (in milliseconds) for each key to prevent spamming actions too quickly.

*(Screenshot: AutoPot Tab Overview)*

*(Screenshot: Vitals Overlay Examples)*

---

## 5. Radar [PoE 2]

An advanced visual assistant and map overlay system designed specifically for Path of Exile 2. It helps with navigation, entity tracking, and filtering.

### Key Features

#### 1. General Settings

Customize the core behavior of the radar.

- **Draw Walkable Map:** Toggles the visualization of the game's navigation mesh (walkable terrain). Useful for seeing valid paths in dark areas.
  - **Color:** Customize the color of the map overlay.
  - **Border Thickness:** Adjust the outline thickness of the map borders.
- **Map Fix:** Toggles a correction mode for map alignment issues.
- **Visibility Logic:**
  - **Hide in Town/Hideout:** Automatically disables the radar in safe zones to reduce clutter.
  - **Hide in Background:** Hides the overlay when the game window is not in focus (alt-tabbed).
  - **Hide on Pause:** Hides the overlay when the game is paused.
  - **Hide Outside Network Bubble:** Hides entities that are outside the server's update range to prevent "ghost" entities (stale data).
- **Show Player Names:** Toggles the display of names above players.

*(Screenshot: Radar General Settings)*

#### 2. Icons

Customize the visual representation of entities on the radar.

- **Groups:** Entities are categorized into groups (Base, League, Breach, Delirium, Expedition, etc.).
- **Customization:** For each group, you can assign specific icons and adjust their size. This allows you to differentiate between a normal monster, a boss, or a specific league mechanic at a glance.

*(Screenshot: Radar Icon Settings)*

#### 3. Path Finder

A powerful navigation tool that draws lines/paths to specific targets.

- **Enable Path Finder:** Toggles the path drawing system.
- **Multicolor:** Randomizes path colors for better distinction between multiple targets.
- **Important POI:** Toggles the display of "Points of Interest" (like Quest Objectives or Waypoints) with optional text backgrounds.
- **Target Management:**
  - **Add Target:** Manually add a target by name.
  - **Add from Map:** Enter an interactive mode to click on an object on the radar to add it as a target.
  - **Add to Global/Area:** Choose whether the target should be tracked in all areas (Global) or only the current zone (Area).
- **Ignored Areas:** You can disable the Path Finder in specific zones to avoid clutter.

*(Screenshot: Path Finder Tab)*

#### 4. Filters (Entity Management)

Fine-tune which entities are shown on the radar and how they are displayed.

- **Add Nearby Entities:** A dropdown list that scans the current memory for valid entities near the player (Monsters, Chests, NPCs, etc.). Select an entity and click "Add" to instantly create a filter for it.
- **Manual Input:** For advanced users, allows adding filters by manually typing the metadata path.
- **Edit Mode:** Click the "E" button next to any filter to open the detailed editor:
  - **Name:** Set a custom display name.
  - **Color:** Choose a custom text color.
  - **Icon:** Toggle icon display, choose a specific icon, and adjust its size.
  - **Draw Path:** Specific toggle for drawing a path to this entity (if Path Finder is enabled).
  - **Ignore:** Completely hide this entity from the radar.

*(Screenshot: Filters Tab)*

*(Screenshot: Edit Entity Modal)*

---

## 6. Smoother

The **Smoother** tab is a comprehensive tool designed to optimize game performance and customize visual elements. It works by modifying the game's data files (`.ggpk` or similar) to disable or alter specific graphical features.

### Key Features

#### Game Selection & Setup

- **Game Version:** Support for both Path of Exile 1 and Path of Exile 2.
- **Path Configuration:** You must specify the path to the game's `Content.ggpk` file (or `Bundles2` directory for PoE 2). The tool includes a file browser and validation to ensure the correct path is selected.
- **Backup System:**
  - **Create Backup:** Automatically creates backups of original files before applying any patches.
  - **Restore All:** A single button to revert all changes and restore the game to its original state.
  - **Clear Backup:** Deletes stored backups and hashes (useful if you want to start fresh).
  - **Game Updates:** Automatically detects if the game has been updated and prompts you to create a new backup to prevent corruption.

#### Performance Patches

Toggle individual patches to disable specific graphical effects. This can significantly boost FPS on lower-end systems.

- **Fog / Atlas Fog:** Removes fog effects from maps and the Atlas.
- **Shadows & Lights:** Disables dynamic shadows and lighting calculations.
- **Particles:** Removes environmental particles (rain, dust) and general particle effects.
- **Corpses:** Prevents corpses from rendering (PoE 1 only).
- **Delirium:** Removes the visual fog and overlay associated with the Delirium league.
- **EpkRemoval:** Removes effect packages for cleaner visuals.
- **Camera:** Enables extended camera zoom limits (configurable via a slider).

*(Screenshot: List of available patches)*

#### Advanced Customization Tools

##### 1. Color Mods

Customize the text colors of item modifiers and stats.

- **Search & Add:** Search for specific stats (e.g., "Physical Damage") and assign them a custom color.
- **Presets:** Includes default presets for dangerous mods (Reflect, No Regen) or valuable drops.
- **Export/Import:** Share your color configurations with others via JSON files.
- **Edit Descriptions:** You can even rewrite the text description of mods for better clarity.

*(Screenshot: Color Mods configuration window)*

##### 2. Spell Effects

A powerful tree-based editor to disable specific spell effects.

- **Tree View:** Browse through all game effects organized by category.
- **Selective Disabling:** Disable effects for specific skills (e.g., "Discharge", "Tornado Shot") to reduce visual clutter without removing everything.
- **Search:** Quickly find effects by name.

*(Screenshot: Spell Effects tree editor)*

##### 3. Black Screen (Performance Mode)

An extreme optimization mode that removes textures and shaders, rendering the game world mostly black while keeping essential gameplay elements visible.

- **File Extensions:** Choose which file types to ignore (e.g., `.dds` textures, `.wav` sounds).
- **Animation Blocks:** Disable specific animation categories like ParticleEffects, Lights, or WindEvents.
- **SkinMesh Settings:**
  - **Ignore List:** Select specific 3D models (metadata) to keep visible even in Black Screen mode. This is crucial for keeping your character, specific bosses, or important mechanics visible.
  - **Scanning:** The tool scans game files to build a complete list of available models for you to toggle.

*(Screenshot: Black Screen advanced settings)*

---

## 7. AutoCraft (still in development!)

The **AutoCraft** tab provides a powerful automation tool for crafting items using currency spam (e.g., Chaos Orb) until specific modifiers are hit.

### Overview

AutoCraft works by automatically applying a selected currency to an item repeatedly. After each application, it scans the item's modifiers to check if they match your desired criteria. If a match is found, the process stops immediately.

### Getting Started

Go to the Smoother tab -> Specify the path to the .ggpk file -> Click Load Data (this is very important! as we need to get mod information from the game client)

1. **Enable the Module:** Go to the **AutoCraft** tab in the main POEFixer window and check the **"Enable AutoCraft"** box.
2. **In-Game Activation:**
   - Switch to the game window.
   - Hover your mouse cursor over the item you wish to craft in your inventory.
   - Press **`Ctrl + Alt`**.
   - This will open the **AutoCraft Overlay** window.

### Interface Description (The Overlay)

The crafting process is managed entirely through the in-game overlay window.

*(Screenshot: AutoCraft Overlay Window)*

- **Target Item:** Displays the name of the item currently selected for crafting.
- **Currency:** A dropdown menu to select the currency type you want to use (e.g., Chaos Orb, Orb of Alteration). The tool automatically detects available currency in your inventory.
- **Limit:** A safety feature that sets the maximum number of currency items to use. The default is 10. The bot will stop if this limit is reached, preventing accidental loss of all currency.
- **Search Mods:**
  - Type in this field to search for specific modifiers (e.g., "maximum life", "resistance").
  - A list of matching mods will appear below. Click a mod to select it for configuration.
- **Mod Configuration:**
  - Once a mod is selected, you can set minimum values for its stats (e.g., if you want at least "+80 to Maximum Life", set the value to 80).
  - Click the **"+ Add"** button to add it to your Target List.
- **Target Mods List:**
  - Displays all the criteria you are looking for.
  - **Logic:** The system uses **"OR"** logic. This means if *any single mod* in this list is found on the item with the required values, the crafting process will stop as a "Success".
  - Use the "Edit" or "X" (Delete) buttons to manage your targets.

### Crafting Process

1. **Setup:** Ensure you have enough of the selected currency in your inventory.
2. **Start:** Click the green **"START CRAFT"** button in the overlay.
3. **Execution:**
   - The bot will automatically right-click the currency.
   - It will hold **Shift** to enable bulk application.
   - It will left-click the target item to apply the currency.
   - After each click, it instantly reads the item's new modifiers.
4. **Stopping:**
   - **Success:** A matching mod is found.
   - **Limit Reached:** The configured currency limit is hit.
   - **Manual Stop:** You click the "STOP" button.
   - **Error:** If the item or currency is moved/lost.

> **Warning:** The bot simulates mouse clicks. Do not move your mouse or use the keyboard while the crafting process is running (except to press the emergency Stop hotkey if configured, or clicking Stop in the UI).

### Settings

The AutoCraft module exposes configurable settings directly in the UI:

- **Timing:** Adjustable delays for click interval, scan timeout, and action cooldowns.
- **Safety:** Currency limit per session, emergency stop hotkey support.
- **Behavior:** Configurable shift-hold for bulk crafting, cursor return options.

> **Emergency Stop:** You can configure a global hotkey to immediately stop the crafting process at any time.

---

## 8. HealthBars

The **HealthBars** tab offers a highly customizable overlay system that displays Health (HP) and Energy Shield (ES) bars directly over entities in the game world. This visual aid helps you quickly identify high-priority targets, monitor your own status, and execute mechanics like Culling Strike with precision.

### Global Settings

These settings apply to the entire HealthBars system or define defaults.

- **Enable HealthBars:** Toggles the entire feature on or off.
- **Draw in Town / Draw in Hideout:** Choose whether to display health bars when you are in safe zones like towns or your hideout.
- **Interpolate Position:** Smooths the movement of the health bars to match the entity's movement.
  - **Interpolation Rate:** Adjusts the smoothness speed (in milliseconds). Lower values are more responsive, while higher values are smoother.
- **Height Offset:** Globally adjusts the vertical position of all health bars relative to the entity model (useful if bars are floating too high or clipping into the model).
- **Culling Strike Thresholds:**
  - Define the percentage of HP at which the bar changes color to indicate the monster is within Culling Strike range.
  - Configurable separately for **White**, **Magic**, **Rare**, and **Unique** monsters.

*(Screenshot: Global Settings panel)*

### Entity Groups

You can independently configure health bars for different types of entities. This allows you, for example, to have large, distinct bars for Unique bosses while keeping White monster bars subtle or hidden.

**Supported Groups:**

- **White Monsters:** Normal enemies.
- **Magic Monsters:** Blue enemies.
- **Rare Monsters:** Yellow enemies.
- **Unique Monsters:** Bosses and unique enemies.
- **Friendly Monsters:** Minions and allied NPCs.
- **Player:** Your character and other players.

### Per-Group Configuration

Each group has its own collapsible section with the following options:

#### Visibility & Elements

- **Enable Group:** Toggles health bars for this specific group.
- **Show Health Bar:** Displays the main HP bar.
- **Show ES Bar:** Displays the Energy Shield bar (stacked above the HP bar).
- **Show Culling Strike:** Changes the HP bar color when the target is below the defined Culling Strike threshold.
- **Show Text:** Displays the numerical HP value (e.g., "1.5M", "100K").
  - **Show Text Background:** Adds a background box behind the text for better readability.

#### Colors

Customize the visual appearance with a full color picker (supports transparency):

- **Health Color:** Color of the HP bar.
- **ES Color:** Color of the Energy Shield bar.
- **Background Color:** Background color of the empty portion of the bars.
- **Text Color:** Color of the numerical text.
- **Text Background:** Color of the text background box.

#### Dimensions & Layout

- **Scale (Width/Height):** Adjusts the size of the health bar.
- **Shift (X/Y):** Fine-tune the position of the bar relative to the entity (pixel offset).
- **Graduations:** Adds vertical tick marks to the bar to help estimate HP percentages.
- **Border Thickness:** Adjusts the thickness of the black outline around the bars.
- **Bar Gap:** Sets the vertical spacing between the HP bar and the ES bar.

*(Screenshot: Configuration for a specific group, e.g., Unique Monsters)*

---

## 9. Atlas

The **Atlas** tab provides a comprehensive overlay for the in-game Atlas, offering enhanced visualization, pathfinding, and management tools to streamline your mapping strategy. It helps you track completion, find specific maps, and plan your route through the Atlas.

### Key Features

#### 1. Interactive Overlay

The core feature is a highly customizable overlay that draws directly over the in-game Atlas screen.

- **Enable Overlay:** Toggles the entire Atlas overlay system on or off.
- **Hide When Not Focused:** Automatically hides the overlay when the game window is not in focus to save resources and screen space.
- **Controller Mode:** Optimized navigation and display adjustments for players using a controller.

*(Screenshot: Atlas Overlay in action)*

#### 2. Search & Navigation

Easily locate specific maps or plan your progression path.

- **Search Maps:** A dedicated search bar allows you to type a map name (partial matches supported). The overlay will highlight matching maps.
- **Visual Pathfinding (Draw Lines):**
  - **Route Lines:** Draws connection lines between nodes to visualize paths.
  - **Path Thickness:** Adjustable thickness for better visibility.
  - **Search Lines:** Draws lines from your current position/start node to the maps matching your search query.
  - **Citadel/Tower Lines:** Draws lines to important objectives like Citadels or Towers.
  - **Range Sliders:** Configure the maximum distance for drawing these lines to avoid cluttering the screen.

#### 3. Display Customization

Filter what is shown to focus on your current goals.

- **Hide Completed:** Hides maps you have already completed (bonus objective fulfilled).
- **Hide Inaccessible:** Hides maps that you cannot currently reach or run.
- **Grid System:**
  - **Show Grid:** Overlays a grid on the Atlas for easier coordinate estimation.
  - **Skip Completed:** Removes grid cells for completed areas.
  - **Custom Color:** Fully customizable grid line colors.

#### 4. Visual Enhancements

Customize the look and feel of the overlay to match your preference.

- **Map Badges:** Displays icons or text badges on maps indicating their tier, biome, or specific properties.
- **Biome Borders:** Draws a colored border around maps belonging to specific biomes.
  - **Thickness:** Adjustable border thickness.
- **Colors & Scaling:**
  - **Background/Font Colors:** Set global default colors for the overlay elements.
  - **Scale Multiplier:** Adjust the size of overlay elements (text, icons) relative to the game UI.
  - **Anchor Nudge:** Fine-tune the alignment of the overlay if it doesn't perfectly match the game window.

#### 5. Biome & Content Management

Detailed control over how specific game mechanics and regions are highlighted.

- **Biome Settings:** Toggle visibility and customize border colors for each specific biome.
- **Content Settings:**
  - **Badges:** Configure highlights for specific content types (e.g., Breach, Delirium, Expedition, Ritual) associated with maps.
  - **Overrides:** For each content type, you can set custom background and font colors to make them stand out.

#### 6. Map Groups

Create custom collections of maps for tracking specific strategies (e.g., "Farming", "Boss Rushing").

- **Custom Groups:** Create named groups of maps.
- **Organization:** Reorder groups, delete them, or rename them.
- **Styling:** Assign unique background and font colors to each group for quick visual identification on the Atlas.
- **Management:** Easily add or remove maps from a group by name.

*(Screenshot: Map Groups configuration)*

---

## 10. Hotkeys (Custom Hotkeys)

The **Hotkeys** tab is a powerful automation and macro system that allows you to create complex behaviors, auto-aim scripts, and reaction-based triggers. It is divided into two main sections: **Custom Hotkeys** and **System Hotkeys**.

### Custom Hotkeys

This section allows you to define your own macros. You can organize them into groups, set specific conditions, and chain multiple actions together.

#### Groups

- **Add Group:** Create a new folder to organize your hotkeys (e.g., "Combat", "Buffs").
- **Toggle:** Enable or disable entire groups at once.
- **Rename:** Double-click a group name to rename it.

#### Hotkey Configuration

Each hotkey consists of an identity, a trigger/output key, global conditions, and a list of actions.

1.  **Identity:**
    -   **Enabled:** Toggle individual hotkeys.
    -   **Name:** Double-click to rename (e.g., "Auto Pot", "Aim Bot").
    -   **ID:** A unique number (e.g., #1) used for chaining macros.

2.  **Global Conditions:**
    -   **Focus:** If checked, the hotkey only works when the game window is in the foreground (prevents accidental triggers while alt-tabbed).
    -   **Safe Zone:** If checked, the hotkey is disabled in Towns and Hideouts (prevents wasting resources).

3.  **Output Key (Trigger):**
    -   Click the button to assign a key (Keyboard, Mouse, or Gamepad).
    -   **Function:** This is the key the program will simulate pressing when actions are executed. It can also serve as a manual trigger for "Chain" actions.
    -   **Modifiers:** Supports Ctrl, Shift, Alt, and Gamepad LT.

#### Actions

You can add multiple actions to a single hotkey. They execute in sequence or loop based on their settings.

-   **Repeat:**
    -   Simply presses the Output Key repeatedly.
    -   **Infinite:** Runs forever while the hotkey is active.
    -   **Count:** Runs a specific number of times.
    -   **Interval:** Time (ms) between presses.

-   **Chain (Link Hotkeys):**
    -   Triggers *another* hotkey by its ID.
    -   **Trigger Mode:**
        -   *Program Only:* Fires only if this hotkey was triggered automatically.
        -   *User Only:* Fires only if YOU physically pressed the key.
        -   *Always:* Fires in both cases.
    -   **Delay:** Time to wait before triggering the next hotkey.

-   **Vitals (Health / Energy Shield / Mana):**
    -   Triggers the key press when your HP/ES/Mana meets a condition (e.g., `< 50%`).
    -   Useful for Auto-Pot, Auto-Logout, or panic skills.

-   **Buffs / Charges:**
    -   **Check Charges:** Triggers based on Power/Frenzy/Endurance/Spirit charge counts.
    -   **Check Buff:** Triggers if a specific buff name is present/absent or has a certain stack count/time left.
    -   **Charged Staff:** Specific check for "charged_staff_stack".

-   **Monster Count:**
    -   Triggers when a certain number of monsters (with optional Rarity filters) are within a specific radius.
    -   Useful for using AoE skills automatically when surrounded.

-   **Hold:**
    -   Holds the Output Key down for a specified duration (ms).

-   **MouseMove (Auto-Aim):**
    -   The most advanced action. It finds a target, moves the mouse cursor to it, and (optionally) presses the key.
    -   **Target Types:**
        -   **Monster:** Filter by Rarity (Normal/Magic/Rare/Unique).
        -   **Chest:** Filter by type (Strongbox, Expedition, Breach, etc.).
        -   **Player/NPC:** Select specific names from the current area.
        -   **Custom:** Target any entity by its Metadata path.
    -   **Settings:**
        -   **Radius:** Max distance to search for targets.
        -   **Hold Time:** How long to track the target.
        -   **Debug Mode:** Draws a red circle (radius) and white text (entity metadata) in the overlay.
        -   **Scan All:** Ignores standard filters and targets everything (for testing).

### System Hotkeys

Built-in global shortcuts for controlling the application.

- **Menu Toggle:** Show/Hide the overlay menu.
- **Stop Trade:** Emergency stop for trade automation.
- **Radar - Add from Map:** Adds the entity under your cursor to the Radar filter list.
- **Radar - Add POI:** Adds a Point of Interest at the cursor location.

*(Screenshot: Custom Hotkeys Interface)*

*(Screenshot: MouseMove Action Settings)*

---

## 11. Launcher (Limited User)

The **Launcher** tab provides a secure and isolated environment for running the game. It allows you to create and manage "Limited Users"—Windows accounts with restricted permissions—and launch the game under these specific user contexts. This is particularly useful for security (Sandboxing) and for running multiple instances of the game with separate configurations/permissions.

### Key Features

#### User Management

The core of this tab is the management of restricted Windows user accounts.

- **User Selection:** A dropdown menu lets you choose which Limited User account to use for launching the game.
- **Create New User:** Click the **"Create New"** button to open a dialog where you can define a new username and password. This creates a real Windows user account on your system with limited privileges.
- **Delete User:** Remove the currently selected Limited User from your system.
- **Manage All Users:** View a list of all users on the system. **Warning:** This allows you to delete *any* user, so use this with caution.

*(Screenshot: Launcher Tab Overview)*
*(Screenshot: Creating a new limited user)*

#### Game Configuration

- **Game Path:** Specify the full path to your game executable (e.g., `PathofExile.exe` or `PathofExile_x64.exe`).
- **Browse:** Use the **"..."** button to open a file explorer and select the executable file.

#### Launching

- **Launch Game:** The large button at the bottom starts the game using the credentials of the selected Limited User. The application handles the complex process of logging in the user in the background and starting the process securely.

### Detailed Functionality

1.  **Isolation:** By running the game as a different user, the game process is isolated from your main user's files and other running processes.
2.  **Configuration Saving:** The launcher automatically saves your selected game path and user preferences to `limitedUser/config.json`, so you don't need to re-enter them every time.
3.  **Security:** Passwords and credentials are managed securely by the application's backend (`LimitedUserManager`).

---

## 12. Logs

The **Logs** tab is a powerful system event viewer that allows you to monitor the application's internal state, debug issues, and track automated actions in real-time. It offers extensive filtering and management tools to keep the information relevant and readable.

### Interface Overview

The interface is divided into four main areas: the Control Bar, Category Filters, the Log List, and the Status Bar.

*(Screenshot: Logs Tab Overview)*

### 1. Control Bar (Top)

Located at the top of the tab, this bar provides quick access to global filters and window management.

- **Search:** A real-time text filter. Type here to instantly filter logs by message content or timestamp.
- **Level Filters:** Toggle buttons to show/hide logs based on severity:
  - **Info:** General informational messages (Blue).
  - **Warning:** Potential issues or important alerts (Yellow).
  - **Error:** Critical failures or errors (Red).
  - **Debug:** Deep internal state information (Grey, available in Debug builds only).
- **Category Filter Toggle:** Click the filter icon to expand/collapse the detailed Category selection panel.
- **Detach/Attach Window:** Moves the Logs tab into a separate, floating window. This is useful for keeping logs visible on a second monitor while using other tabs in the main application.
- **Save to File:** Exports the current log history to a text file in the `logs/` directory. Useful for sharing with developers for support.

### 2. Category Filters (Collapsible)

When expanded, this section allows fine-grained control over which modules generate logs.

- **Categories:** You can enable or disable logs for specific features like **Trade**, **Radar**, **AutoPot**, **AutoCraft**, **Hotkeys**, etc.
- **Trade Subgroups:** The **Trade** category is further divided into subgroups for precise debugging:
  - *Connection:* Status of the trade site connection.
  - *Buy:* Buy actions and clipboard operations.
  - *WebSocket:* Live data stream events.
  - *RateLimit:* Warnings about API rate limits.
  - *Icon:* Item icon loading status.
- **All / None:** Quick buttons to enable or disable all categories at once.

*(Screenshot: Expanded Category Filters)*

### 3. Log List (Center)

The main area displays the chronological history of events.

- **Format:** Each entry shows the Timestamp, Level Icon, Category Icon, and the Message.
- **Context Menu:** Right-click on any log entry to access copy options:
  - **Copy Line:** Copy the selected log line to clipboard.
  - **Copy Visible:** Copy only the currently filtered logs.
  - **Copy All:** Copy the entire log history.
- **Auto-Scroll:** The list automatically scrolls to the newest entry unless you manually scroll up to inspect older logs.

### 4. Status Bar (Bottom)

Displays statistics and maintenance tools.

- **Counters:** Shows the "Total" number of logs in memory and the "Shown" number based on current filters.
- **Clear All:** Permanently deletes all log entries from memory.
- **Clear Filtered:** Deletes only the logs that are currently visible (passing the active filters).

---

## 13. Follow Bot

The **Follow Bot** tab allows your character to automatically follow another player or entity. This is particularly useful for playing with a support character (Aurabot) or dual-boxing.

### Key Features

#### 1. General Control

- **Enable Follow Bot:** Master switch to start/stop the bot.
- **Status Indicator:** Displays the current state of the bot (e.g., *Following*, *WaitingForTarget*, *Stuck*) with color coding for quick status checks.

#### 2. Target Settings

Configure who to follow and how close to stay.

- **Target Name:** Manually enter the name of the player/character to follow.
- **Pick Player:** A dropdown list showing nearby players. Select one to automatically fill the Target Name.
- **Stop Distance:** The distance at which the bot will stop moving towards the target (to avoid standing exactly on top of them).

#### 3. Movement Settings

Fine-tune how the bot moves and interacts with the game.

- **Move Key:** Assign the key used for movement (usually your "Force Move" key in-game). Click the button and press the desired key to bind it.
- **Click Ahead Distance:** How far ahead of the current position the bot should try to click.
- **Click Interval:** Time (in ms) between movement clicks.
- **Update Interval:** How often the bot logic runs.
- **Path Recalc Interval:** How often the pathfinding algorithm recalculates the route to the target.
- **Return Cursor:** If enabled, returns the mouse cursor to its original position after a movement click.
- **Require Focus:** If enabled, the bot only moves when the game window is in the foreground.
- **Disable in Town:** Automatically pauses the bot when in town or hideout.

#### 4. Anti-Stuck Settings

Mechanisms to handle situations where the bot gets stuck.

- **Stuck Timeout:** How long (in ms) to wait without successful movement before declaring "Stuck".
- **Max Recovery:** Number of attempts to un-stuck before giving up or resetting.

#### 5. Debug Settings

Visual tools to diagnose pathfinding and logic.

- **Debug Overlay:** Enables drawing debug information on the screen.
- **Show Path:** Draws the calculated path to the target.
- **Show Waypoints:** Highlights the specific points the bot is clicking.
- **Show Stop Radius:** Visualizes the circle around the target where the bot will stop.
- **Metadata Debug:** Tools to view entity metadata and test pathfinding to specific objects (similar to Custom Hotkeys).

---

## 14. Debug [PoE 2]

The **Debug** tab provides low-level inspection tools for examining game memory structures in real-time. It is primarily designed for plugin developers and advanced users who need to understand entity data, component values, and UI element hierarchy.

### 1. Entity List

A filterable table showing all entities currently loaded in the game.

- **Columns:** Entity ID, Address, Metadata Path, Type, SubType, State, Rarity, Zone (InnerCircle/OuterCircle/Far).
- **Type Filters:** Quickly filter by entity type — Monsters, Chests, NPCs, Players, Items, etc.
- **Search:** Text-based search by metadata path.
- **Count Display:** Shows total number of entities and filtered count.

### 2. Entity Watch (Component Inspector)

Click on an entity in the list to "watch" it. The application's worker thread will then read detailed component data for that entity every frame.

**Available Components:**
- **Life:** Current/Max HP, ES, MP, Reserved amounts, recovery rates.
- **Render:** World position (X/Y/Z), model bounds, model name/path.
- **Positioned:** Grid position, rotation, scale factor.
- **Targetable:** IsTargetable, IsHostile, IsFriendly flags.
- **Animated:** Current animation path, animation ID.
- **Stats:** Full stat dictionary (stat ID → value).
- **Actor:** Active skills list, current action ID.
- **Buffs:** All active buffs with name, charges, time remaining, total duration.

- **All Components:** Displays a full list of all component name→address pairs, even for components without dedicated parsing (useful for advanced memory reading).

### 3. Inventory Debug

Inspect the game's inventory system in detail.

- **ServerData:** Displays the ServerData component address.
- **Inventory List:** Shows all player inventories (ID, Address, Name).
- **Inventory Watch:** Select an inventory to inspect:
  - **Slot Grid:** Visual representation of occupied/empty slots.
  - **Item List:** All items with their metadata path, rarity, slot position.
  - **Item Mods:** Full mod breakdown (Implicit, Explicit, Enchant, Hellscape) with values.

### 4. UI Explorer

Navigate the game's UI element tree hierarchy.

- **Game UI Root:** Inspect elements under the in-game UI root (panels, menus, buttons).
- **Top-Level UI Root:** Access the top-level UI root for login/loading screens.
- **Tree Navigation:** Expand/collapse UI elements to see children, addresses, and properties.
- **Search:** Filter elements by name/path.
- **GameCullValue:** Displayed for UI scale calculations.

---

## 15. Plugins

The **Plugins** tab manages native C++ DLL plugins that extend POEFixer's functionality. Plugins can render overlays, access game data, and integrate seamlessly with the existing UI.

### Plugin Management

- **Plugin List:** Shows all discovered plugins from the `Plugins/` directory.
- **Enable/Disable:** Toggle individual plugins on or off. State is saved between sessions.
- **Per-Plugin Settings:** Each enabled plugin can render its own settings section directly in the Plugins tab.
- **SDK Version Check:** The host validates plugin SDK version compatibility on load. Current SDK version: **4**.

### Available Plugins

#### Kill Counter

A comprehensive kill/chest/death tracking overlay.

- **Monster Kill Tracking:** Counts killed monsters by rarity (Normal, Magic, Rare, Unique, Unknown) using disappearance-based detection.
- **Chest Tracking:** Counts opened chests by type (Magic, Rare, Expedition, Breach, Strongbox).
- **Death Counter:** Tracks player deaths.
- **Per-Map Counters:** Automatic reset when entering a new area and killing the first monster (old counters remain visible until first kill in new area).
- **Lifetime Totals:** Persistent statistics stored in a SQLite3 database.
- **Overlay:**
  - Draggable when the program menu is active ("Drag to reposition" hint appears).
  - Non-interactive and title-bar-free when menu is hidden.
  - Configurable opacity (0-100%), adjustable size.
  - Tab bar for switching between Map and Total views.
- **Icon Atlas:** Displays entity type icons from the shared `Resources/radar/icons.png` sprite sheet.
- **Settings:** Configure which counters to display, opacity, overlay mode, reset buttons.

#### Example Plugin

A reference implementation demonstrating all Plugin SDK features:

- **Buff Inspector:** Lists player buffs with filtering and progress bars.
- **Entity Explorer:** Entity debug list with watch mechanism, component trees.
- **Inventory Inspector:** ServerData viewer, inventory selector, slot grid, item mods.
- **Memory Viewer:** Hex dump viewer, `Read<T>` demo, pattern scanner.
- **UI Explorer:** Full UI element tree navigator with search and highlighting.

### Creating Plugins

See the [Plugin Development Guide](../PluginDevelopmentGuide.md) for detailed instructions on creating your own plugins, including:
- Project setup and build configuration
- IPlugin interface implementation
- PluginContext API reference (v1-v4)
- Data structures and enums
- Common recipes and patterns
- Draggable overlay pattern
- Troubleshooting guide
