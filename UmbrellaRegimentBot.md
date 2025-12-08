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

## Setup

### Prerequisites

1. **Python 3.12.10** (recommended) or Python 3.12+
2. **Tesseract OCR** (optional, for screenshot processing):
   - Windows: Download from [GitHub](https://github.com/UB-Mannheim/tesseract/wiki)
   - Linux: `sudo apt-get install tesseract-ocr`
   - macOS: `brew install tesseract`

### Installation

1. **Ensure Python 3.12.10 is installed**:
   ```bash
   python --version
   # Should show: Python 3.12.10
   ```
   If you need to install Python 3.12.10, download it from [python.org](https://www.python.org/downloads/)

2. **Clone or download this repository**

3. **Install Python dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

4. **Create a `.env` file** in the project root:
   ```
   DISCORD_BOT_TOKEN=your_bot_token_here
   TEST_GUILD_ID=your_guild_id_here  # Optional: For guild-level command syncing
   
   # Optional: API Configuration
   API_KEY=your-secure-api-key-here
   API_HOST=0.0.0.0
   API_PORT=8080
   API_CORS_ORIGINS=http://localhost:3000
   
   # Optional: OAuth2 Configuration
   DISCORD_CLIENT_ID=your_client_id_here
   DISCORD_CLIENT_SECRET=your_client_secret_here
   DISCORD_REDIRECT_URI=https://yourwebsite.com/auth/discord/callback
   ```

5. **Get your Discord bot token**:
   - Go to [Discord Developer Portal](https://discord.com/developers/applications)
   - Create a new application or select an existing one
   - Go to the "Bot" section
   - Create a bot and copy the token
   - Enable "Message Content Intent" in the Bot settings
   - Enable "Server Members Intent" and "Reactions Intent"

6. **Invite your bot to your server**:
   - In the Developer Portal, go to "OAuth2" > "URL Generator"
   - Select scopes: `bot`, `applications.commands`
   - Select bot permissions:
     - Send Messages
     - Read Message History
     - Attach Files
     - Embed Links
     - Add Reactions
     - Manage Events
   - Copy the generated URL and open it in your browser to invite the bot

### Running the Bot

```bash
python bot.py
```

## Commands

### War Management (War Tracker System)

- `/war_setup <war_number>` - Setup a new war number **[Admin only]**
- `/war_start <war_number>` - Start a war (deactivates all other wars) **[Admin only]**
  - Autocomplete shows all available wars with their status
- `/war_end <war_number>` - End a war **[Admin only]**
  - Autocomplete shows only active wars
- `/war_view` - View all wars in the database with status indicators (🟢 active, 🔴 inactive)

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
  - Interactive interface with buttons and dropdowns for easy configuration

- `/event_settings` - Alternative event settings command (legacy)
  - Provides similar functionality to `/settings` but only for event management
  - Uses action-based parameter selection
  - **Note:** The unified `/settings` command is recommended for managing all settings

- `/export_database` - Export database to CSV file (sent via DM)
  - Select which database to export using dropdown menu:
    - **Foxhole/War Tracker Database** - War tracking and statistics
    - **Events Database** - Events management data
  - The CSV file will be sent to your DMs (or in the channel if DMs are disabled)
  - Files are timestamped for easy identification

- `/sync_commands` - Re-register all bot commands (useful after updates) **[Admin only]**
  - Clears and re-registers all commands
  - Useful when commands aren't appearing or after bot updates
  - Can sync to a test guild if `TEST_GUILD_ID` is set in `.env`

- `/ping` - Check bot latency and status **[Admin only]**
  - Shows API latency in milliseconds
  - Color-coded status (green/orange/red based on latency)

## Usage

### War Tracking Workflow

1. **Setup a War**:
   ```
   /war_setup war_number:100
   /war_start war_number:100
   ```
   Use `/war_view` to see all wars and their status

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
- **wars**: War information (number, start/end dates, active status)
- **user_stats**: User statistics per war
- **settings**: Bot configuration for war tracking

### Events Database (`events.db`)
- **events**: Event information (name, description, times, creator, status)
- **signups**: Event attendance (user_id, position, emoji)
- **guild_settings**: Server-specific settings for events
- **events_wars**: War tracking for events (separate from war tracker wars)

## REST API

The bot includes a FastAPI-based REST API for external integrations.

### API Setup

1. **Add to `.env`**:
   ```env
   API_KEY=your-secure-api-key-here
   API_HOST=0.0.0.0
   API_PORT=8080
   API_CORS_ORIGINS=http://localhost:3000
   ```

2. **API starts automatically** when the bot starts:
   - Base URL: `http://localhost:8080`
   - Documentation: `http://localhost:8080/docs`
   - Health Check: `http://localhost:8080/health`

### API Endpoints

All endpoints (except `/` and `/health`) require `X-API-Key` header.

- **GET `/api/v1/events`** - Get all events for a guild
- **GET `/api/v1/events/{event_id}`** - Get a specific event
- **POST `/api/v1/roles/assign`** - Assign a role to a user
- **POST `/api/v1/roles/remove`** - Remove a role from a user
- **PUT `/api/v1/roles/modify`** - Modify user roles

See `api/README.md` for complete API documentation.

## OAuth2 Integration

The bot includes Discord OAuth2 helpers for website authentication and role syncing.

1. **Configure in Discord Developer Portal**:
   - Add redirect URIs
   - Select scopes: `identify`, `guilds`, `guilds.members.read`

2. **Add to `.env`**:
   ```env
   DISCORD_CLIENT_ID=your_client_id
   DISCORD_CLIENT_SECRET=your_client_secret
   DISCORD_REDIRECT_URI=https://yourwebsite.com/auth/discord/callback
   ```

See `OAUTH2_SETUP.md` for complete setup instructions.

## Notes

### War Tracking
- Leaderboard updates every 5 minutes automatically (can be enabled/disabled in `/settings`)
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
- **API not working**: Verify `API_KEY` is set and port is available
- **Database errors**: Delete database files to reset (this deletes all data)
- **Bot not responding**: Verify bot token and permissions are correct

## Project Structure

```
├── bot.py                          # Main bot file
├── commands/
│   ├── admin_commands.py          # War tracker admin commands
│   ├── stats_commands.py          # Stat entry/viewing commands
│   ├── war_commands.py            # War management commands
│   ├── leaderboard_commands.py    # Leaderboard commands
│   ├── create_event.py            # Create event command
│   ├── list_events.py             # List events commands
│   ├── view_event.py              # View event command
│   ├── start_event.py             # Start event command
│   ├── end_event.py               # End event command
│   ├── edit_event.py              # Edit event command
│   ├── delete_event.py            # Delete event command
│   ├── event_stats.py             # Event statistics command
│   ├── settings.py                # Event settings command
│   └── utils.py                   # Command utilities
├── helpers/
│   ├── database.py                # Database functions (both systems)
│   ├── dm_handler.py              # War tracker DM handler
│   ├── events_dm_handler.py       # Events DM handler
│   ├── embed_helper.py            # War tracker embeds
│   ├── embeds.py                  # Event embeds
│   ├── screenshot_processor.py    # OCR processing
│   ├── leaderboard_updater.py     # Auto leaderboard updates
│   ├── reactions.py               # Event reaction handling
│   ├── startup.py                 # Startup tasks
│   ├── discord_event_sync.py      # Discord event sync
│   ├── dm_conversations.py        # Event DM conversations
│   ├── dm_prompts.py              # Event DM prompts
│   ├── models.py                  # Data models
│   ├── api_helpers.py             # API helpers
│   ├── sync_role.py               # Role syncing
│   └── config.py                  # Configuration
├── api/                            # REST API
│   ├── server.py
│   ├── routes.py
│   ├── auth.py
│   └── README.md
├── java/                           # Java integration helpers
│   ├── ApiClient.java
│   ├── DiscordOAuth2Helper.java
│   └── README.md
├── requirements.txt
├── foxhole_stats.db                # War tracker database
└── events.db                       # Events database
```

## License

This project is provided as-is for personal use.
