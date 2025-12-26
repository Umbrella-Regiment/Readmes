# Umbrella Discord Bot - Combined War Tracker & Event Management

A comprehensive Discord bot combining war tracking and event management for Foxhole. Track player statistics across wars and manage community events with sign-ups, all in one unified bot.

## Features

### War Tracking & Statistics
- **War Management**: Setup, start, and end wars with autocomplete support
- **Stat Entry**: Manual entry or screenshot processing through DM interactions
- **Screenshot Processing**: Automatic stat extraction from screenshots using OCR (optional)
- **Stat Editing**: Edit previously submitted stats with interactive selection
- **Stat Tracking**: Store stats in SQLite database with war association
- **Stat Categories**: Tracks 14+ stat categories including:
  - Damage stats (enemy/friendly player damage, structure/vehicle damage)
  - Support stats (construction, repairing, healing, revivals)
  - Vehicle stats (captured vehicles, self-damage by faction)
  - Material stats (gathering, submission, supply delivery)
- **Stat Display**: View individual user stats for specific wars or lifetime with rank indicators
- **Real-time Leaderboards**: Automatic leaderboard updates in a specified channel (top 3 per category)
- **Guild Display Names**: Leaderboards show users' guild nicknames when available
- **Data Governance**: Input validation ensuring data quality (numbers only, 0-999,999,999 range checks)
- **Service Records**: Comprehensive service record system tracking member history
  - Rank and level tracking
  - Service history based on assigned war roles
  - Specializations and certifications from Discord roles
  - Rank insignia display
  - War participation tracking across shards (Able/Baker/Charlie)

### Event Management
- ✅ Create events with custom names, descriptions, and times
- ✅ 12-hour time format with timezone selection
- ✅ Automatic Discord scheduled event creation
- ✅ Simple Yes/No attendance system with reaction-based sign-ups
- ✅ Required attendees tracking with signup count display
- ✅ Real-time embed updates showing sign-ups with Discord mentions
- ✅ Local time display (automatically shows in each user's timezone)
- ✅ Event management (start, end, delete, edit, view) with autocomplete
- ✅ Status tracking (Upcoming, In Progress, Ended)
- ✅ Event start notifications with user mentions
- ✅ **Reaction persistence across bot restarts**
- ✅ **Discord scheduled event sync** - Users marking "Interested" automatically sync to bot signups
- ✅ **War tracking** - Track events by war number, automatically assign to active wars
- ✅ **REST API** - FastAPI-based REST API for external integrations
- ✅ **OAuth2 Integration** - Discord OAuth2 helpers for website authentication

### Poll Management
- ✅ Create polls with text-based or time-based options
- ✅ Single or multiple choice voting
- ✅ Automatic expiration based on duration
- ✅ Reaction-based voting system
- ✅ Thread creation for discussion
- ✅ Vote count display after poll ends

### Roll Call System
- ✅ Track member activity status during war transitions
- ✅ Automatic timer-based expiration (configurable)
- ✅ Reaction-based status tracking (Active/Inactive)
- ✅ Persistent embeds across bot restarts
- ✅ Automatic result processing

### Tech Progress Analysis
- ✅ Image-based tech progress detection from screenshots
- ✅ Automatic percentage calculation
- ✅ Visual progress bar display

## Commands

### War Management (War Tracker System)

- `/war_create <war_number> <shard> <faction> <war_role>` - Create a new war with associated Discord role **[Admin only]**
  - Requires: war number, shard (Able/Charlie/Baker), faction (Warden/Colonial), and a Discord role
  - The war role is used to track which users participated in each war for service records
- `/war_initiate <war_number>` - Start/activate a war (deactivates all other wars) **[Admin only]**
  - Autocomplete shows all available wars with their status
  - Preserves existing shard information if already set
- `/war_terminate <war_number>` - End/deactivate a war **[Admin only]**
  - Autocomplete shows only active wars
- `/war_view` - View all wars in the database with status indicators (🟢 active, 🔴 inactive)
- `/war_warname <war_number> <war_name>` - Set or update a friendly name for a war **[Admin only]**
  - War names are displayed in service records instead of just war numbers
- `/war_delete <war_number>` - Permanently delete a war and all associated stats **[Admin only]**
  - Requires confirmation before deletion
  - **WARNING:** This action cannot be undone and will delete all user stats for that war
  - Autocomplete shows all wars

**Note:** These commands use the War Tracker system and are separate from event war tracking.

### Stat Entry & Viewing (War Tracker System)

- `/stats_entry` - Opens a DM to enter your stats
  - Choose between **Manual Entry** (step-by-step prompts) or **Screenshot Submission** (OCR processing)
  - Manual entry uses buttons for Skip and Cancel actions
  - Screenshot processing extracts stats automatically from images
  - If you already have stats for the current war, new submissions will **replace** existing stats
  - Tracks multiple stat categories including damage, construction, healing, materials, and more

- `/stats_edit` - Opens a DM to edit your most recent stats for the current war
  - Select which stat to edit from a list
  - View current values and update individual stats

- `/stats_view [user] [war_number]` - View stats for a user (defaults to you, current war, or lifetime)
  - Shows all stat categories with ranks if available
  - Displays war-specific or lifetime statistics

- `/leaderboard [leaderboard_type] [war_number]` - View the leaderboard (top 3 per category)
  - Options: "Active War" or "Lifetime"
  - Defaults to current active war, or lifetime if no active war
  - Shows top 3 players per stat category (14+ different categories)
  - Displays users' guild display names (nicknames) when available

### Service Record System

- `/service_record [user]` - Display a member's service record **[DECLASSIFIED]**
  - Shows comprehensive service history including:
    - Rank and level
    - Total wars participated (based on assigned war roles)
    - Entered service date
    - Highest stat and record amount
    - Specializations (based on assigned Discord roles)
    - Certifications (based on assigned Discord roles)
    - Service history grouped by shard (Able/Baker/Charlie)
  - Displays rank insignia thumbnail if available
  - Defaults to your own record if no user specified

- `/service_rankadd <rank> [user]` - Add or update a member's service rank
  - Rank options include: Private, Lance Corporal, Corporal, Sergeant, Staff Sergeant, Warrant Officer 2, Warrant Officer 1, Officer Cadet, Second Lieutenant, Lieutenant, Captain, Major, Lieutenant Colonel, Colonel, Brigadier General, Major General, Lieutenant General, General, Field Marshal
  - Autocomplete available for rank selection
  - Defaults to your own rank if no user specified
  - Users with Manage Server permissions can update other members' ranks

- `/service_leveladd <level> [user]` - Add or update a member's service level
  - Level must be 0 or higher
  - Defaults to your own level if no user specified
  - Users with Manage Server permissions can update other members' levels

- `/server_enteredservice <entered_service> [user]` - Add or update a member's entered service date
  - Date format: YYYY-MM-DD (e.g., 2021-07-16)
  - Defaults to your own date if no user specified
  - Users with Manage Server permissions can update other members' dates

**Service Record Features:**
- **Specializations**: Automatically populated from assigned Discord roles (Combat Engineer, Armored Cavalry, Naval Specialist, etc.)
- **Certifications**: Automatically populated from assigned Discord roles (Facility, Naval, Op, Combat Engineering, Logistics, Recruiter)
- **Service History**: Shows wars based on assigned war roles, grouped by shard (Able/Baker/Charlie)
- **Rank Insignia**: Displays rank badge thumbnail from `images/rank_insignia/` folder
- **Total Wars**: Counts wars based on assigned war roles across all shards

### Event Management (Events System)

- `/event_create` - Create a new event (opens DM conversation flow)
  - Autocomplete available for selecting existing events in some contexts
- `/event_list` - List all active events (upcoming and in-progress)
- `/event_list_past` - List all past/ended events
- `/event_view` - View detailed information about a specific event
  - Autocomplete shows all available events
- `/event_start` - Mark an event as started (changes status to "In Progress")
  - Autocomplete shows upcoming events
- `/event_end` - Mark an event as ended
  - Autocomplete shows in-progress events
- `/event_edit` - Edit an existing event's details (Creator or Admin only, opens DM flow)
  - Autocomplete shows events you can edit
- `/event_delete` - Permanently delete an event (Creator or Admin only)
  - Autocomplete shows events you can delete
- `/event_stats [war_filter]` - View event statistics (filter by war number or "lifetime")

### Poll Management

- `/poll` - Create a new poll (opens DM conversation flow)
  - Choose between "Text Only Options" or "Time Only Options"
  - For time polls, specify your timezone and enter times in 12-hour format
  - Add multiple response options
  - Choose single or multiple choice voting
  - Set poll duration (minutes, hours, or days)
  - Polls are automatically posted to a designated channel with a thread
  - Votes are collected via emoji reactions
  - Polls automatically end when their duration expires

- `/poll_end [poll]` - Manually end a poll
  - Autocomplete shows only active/open polls
  - Displays final vote counts and highlights winning option(s)

### Roll Call System

- `/rollcall_start <ending_war> <upcoming_war> [channel]` - Start a new roll call **[Admin only]**
  - Creates a roll call embed in the specified channel (or configured diplomat channel)
  - Tracks which members are active/inactive for the transition between wars
  - Automatically ends after a configured timer (default: 48 hours)
  - Members react with ✅ (Active) or ❌ (Inactive) to indicate their status
  - Autocomplete available for war selection

- `/rollcall_end` - End the active roll call early **[Admin only]**
  - Manually terminates the current roll call before the timer expires
  - Processes final results and updates member status

- `/settings_rollcall` - Configure roll call settings **[Admin only]**
  - Set the diplomat channel where roll calls are posted
  - Configure roll call timer duration (in hours)
  - Interactive dropdown interface for easy configuration

**Roll Call Features:**
- Automatic timer-based expiration
- Reaction-based member status tracking
- Persistent embeds that survive bot restarts
- Automatic processing when timer expires

### Tech Progress Analysis

- `/techprogress <image>` - Analyze tech progress from a screenshot
  - Upload an image containing the tech progress box from Foxhole
  - Automatically detects and calculates the progress percentage
  - Displays a visual progress bar and percentage
  - Useful for tracking research progress across the faction

### Verification System

- **Automatic Channel Setup**: When a new text channel is created in the verification category, the bot automatically posts a verification embed with user selection dropdowns
- **Role Assignment**: Admins and users with Recruiter Certification role can select users from dropdown menus to assign verification roles:
  - Diplomat
  - [☂] (Umbrella)
  - Non-Foxhole
  - Charlie
- **Automatic Role Assignment**: When a role is assigned, the current war verification role is also automatically assigned (except for Non-Foxhole)
- **Welcome Messages**: Automated welcome messages are sent after role assignment
- **Persistent Views**: Verification embeds and dropdowns persist across bot restarts

**Event War Tracking:**
- Events can be tracked by war number (separate from War Tracker wars)
- Events created while a war is active will be automatically assigned to that war
- Use `/event_stats war_filter:100` to view statistics for a specific war

### Configuration (Admin Only)

- `/settings` - View and manage **all bot settings** in one unified interface
  - **War Tracker Settings:**
    - Toggle screenshot processing on/off (dropdown menu)
    - Set leaderboard channel for automatic updates (dropdown menu)
    - Enable/disable automatic leaderboard updates (dropdown menu)
  - **Event Management Settings:**
    - Enable/disable event output channel (dropdown menu)
    - Set event output channel (dropdown menu)
    - Enable/disable event start channel (dropdown menu)
    - Set event start channel (dropdown menu)
    - Enable/disable event attendance channel (dropdown menu)
    - Set event attendance channel (dropdown menu)
    - Set default voice channel for Discord scheduled events (dropdown menu)
  - **Verification Settings:**
    - Set current war verification role (dropdown menu)
    - Used for automatic role assignment in verification channels
  - **Roll Call Settings:**
    - Set diplomat channel for roll calls (dropdown menu)
    - Configure roll call timer duration (dropdown menu)
  - **Logging Settings:**
    - Enable/disable message logging (dropdown menu)
    - Set message log channel (dropdown menu)
    - Enable/disable role logging (dropdown menu)
    - Set role log channel (dropdown menu)
  - Interactive interface with buttons and dropdowns for easy configuration

- `/event_settings` - Alternative event settings command (legacy)
  - Provides similar functionality to `/settings` but only for event management
  - Uses action-based parameter selection
  - **Note:** The unified `/settings` command is recommended for managing all settings

- `/database_export` - Export database to CSV file (sent via DM) **[Admin only]**
  - Select which database to export using dropdown menu:
    - **Foxhole/War Tracker Database** - War tracking and statistics
    - **Events Database** - Events management data
  - The CSV file will be sent to your DMs (or in the channel if DMs are disabled)
  - Files are timestamped for easy identification

- `/settings_logging` - Configure logging settings **[Admin only]**
  - Enable/disable message logging (tracks message edits and deletions)
  - Set message log channel
  - Enable/disable role logging (tracks role assignments and removals)
  - Set role log channel
  - Interactive dropdown interface for configuration

- `/sync_commands` - Re-register all bot commands (useful after updates) **[Admin only]**
  - Clears and re-registers all commands
  - Useful when commands aren't appearing or after bot updates
  - Can sync to a test guild if `TEST_GUILD_ID` is set in `.env`

- `/ping` - Check bot latency and status **[Admin only]**
  - Shows API latency in milliseconds
  - Color-coded status (green/orange/red based on latency)

## Usage

### War Tracking Workflow

1. **Create a War**:
   ```
   /war_create war_number:100 shard:Able faction:Warden war_role:@War100Role
   /war_initiate war_number:100
   ```
   - Each war requires a Discord role to track participants for service records
   - Use `/war_warname` to set a friendly name for the war (e.g., "Wind Hills Campaign")
   - Use `/war_view` to see all wars and their status
   - Use `/war_terminate` to end an active war
   - Use `/war_delete` to permanently remove a war (with confirmation)

2. **Submit Stats**:
   - Use `/stats_entry` to open a DM with the bot
   - Choose your entry method:
     - **Manual Entry**: Step-by-step prompts with buttons
     - **Screenshot Submission**: Attach a screenshot for OCR processing
   - **Important**: New submissions **replace** existing stats for the same war

3. **View Stats**:
   ```
   /stats_view                                    # Your stats for current war
   /stats_view user:@username war_number:100      # Specific user and war
   /leaderboard leaderboard_type:Active War      # Leaderboard for active war (top 3 per category)
   /leaderboard leaderboard_type:Lifetime         # Lifetime leaderboard (top 3 per category)
   /stats_view                                    # Your stats with rank positions for each stat
   ```

### Event Management Workflow

1. **Create an Event**:
   - Use `/event_create` to start a DM conversation
   - The bot will guide you through:
     - Event name
     - Description
     - Timezone
     - Start time (12-hour format)
     - End time (12-hour format)
     - Required attendees
     - Voice channel (optional)
     - Discord scheduled event creation (optional)

2. **Manage Events**:
   ```
   /event_list              # View active events
   /event_view event:"..."   # View event details
   /event_start event:"..."  # Start an event
   /event_end event:"..."    # End an event
   /event_edit event:"..."   # Edit event details
   /event_delete event:"..." # Delete an event
   ```

3. **View Statistics**:
   ```
   /event_stats                      # Lifetime statistics
   /event_stats war_filter:100       # Statistics for War 100
   /event_stats war_filter:lifetime  # All events
   ```

### Attendance System

Users can indicate attendance by reacting to event embed messages:
- ✅ **Yes** - User will be attending
- ❌ **No** - User will not be attending

The embed automatically updates to show sign-ups. Users can change their response by reacting with the other emoji.

## Database Schema

The bot uses **two separate SQLite databases**:

### War Tracker Database (`foxhole_stats.db`)
- **wars**: War information (number, name, start/end dates, active status, shard, faction, associated Discord role)
- **user_stats**: User statistics per war
- **service_records**: Service record profiles (rank, level, entered service date)
- **settings**: Bot configuration for war tracking
- **rollcalls**: Roll call information (guild, channel, message, wars, timer, status)

### Events Database (`events.db`)
- **events**: Event information (name, description, times, creator, status)
- **signups**: Event attendance (user_id, position, emoji)
- **guild_settings**: Server-specific settings for events
- **events_wars**: War tracking for events (separate from war tracker wars)
- **polls**: Poll information (title, question, options, voting type, duration, status)

## Notes

### War Tracking
- Leaderboard updates every 30 minutes automatically (can be enabled/disabled in `/settings`)
- Leaderboards automatically update existing embed messages rather than creating duplicates
- Leaderboards display users' guild display names (nicknames) when available
- Stats **replace** existing entries when resubmitted for the same war
   - Screenshot processing requires Tesseract OCR (optional)
   - Input validation: numbers only (0-999,999,999), no spaces or text
   - Use `/settings` to configure war tracker options (screenshot processing, leaderboard channel, and automatic updates)

### Event Management
- Event IDs use format: `YYYY-MM-DD_XXX`
- Times stored in UTC, displayed in user's local timezone
- Reactions persist across bot restarts
- Discord scheduled events sync automatically
- Events can be tracked by war number (separate from war tracker)

### Poll Management
- Polls support both text-based and time-based options
- Time options are displayed using Discord time codes (shows in reader's local timezone)
- Votes are collected via emoji reactions and persist across bot restarts
- Polls automatically end when their duration expires
- Vote counts are hidden while polls are open, only shown after ending
- Emoji reactions are cleaned up after polls end

### Roll Call System
- Roll calls track member activity status during war transitions
- Members react with Active/Inactive emojis to indicate their status
- Automatic timer-based expiration (configurable, default 48 hours)
- Roll call embeds persist across bot restarts
- Results are processed automatically when timer expires or manually ended

### Tech Progress Analysis
- Image-based tech progress detection from Foxhole screenshots
- Automatic percentage calculation and visual progress bar display
- Supports various image formats

### Service Records
- Service records track member service history based on assigned Discord roles
- Specializations and certifications are automatically detected from role assignments
- Service history is populated from war roles assigned during `/war_create`
- Rank insignia images are stored in `images/rank_insignia/` folder
- Total wars count is based on assigned war roles across all shards
- If no specializations or certifications are found, displays "No Current Specializations" or "No Current Certifications"

### General
- Both systems run independently but share the same bot instance
- War Tracker wars and Event wars are separate systems
- Make sure users have DMs enabled for stat entry and event creation
- The bot ignores messages starting with `!` to avoid conflicts

## Troubleshooting

- **OCR not working**: Ensure Tesseract is installed and screenshot processing is enabled in `/settings` (War Tracker settings)
- **DM not received**: Check that DMs are enabled from server members
- **Commands not appearing**: Use `/sync_commands` (Admin only)
- **Reactions don't work**: Check bot permissions (Add Reactions, Read Message History)
- **Database errors**: Delete database files to reset (this deletes all data)
- **Bot not responding**: Verify bot token and permissions are correct

## License

This project is provided as-is for personal use.
