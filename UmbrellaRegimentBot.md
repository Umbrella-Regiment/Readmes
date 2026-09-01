# Umbrella Discord Bot - Combined War Tracker & Event Management

A comprehensive Discord bot combining war tracking and event management for Foxhole. Track player statistics across wars and manage community events with sign-ups, all in one unified bot.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Setup](#setup)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Running the Bot](#running-the-bot)
  - [Running Tests](#running-tests)
- [Commands](#commands)
- [Usage](#usage)
- [Database Schema](#database-schema)
- [REST API](#rest-api)
- [OAuth2 Integration](#oauth2-integration)
- [Notes](#notes)
- [Troubleshooting](#troubleshooting)
- [Project Structure](#project-structure)
- [License](#license)

## Overview

Umbrella is an all-in-one Discord bot built for a single Foxhole-focused community server, consolidating tooling a regiment/clan would otherwise run as several separate bots:

- **War performance tracking** — members submit per-war stats (manually or via screenshot OCR), view leaderboards and personal service records, and a voice channel automatically reflects the active war and faction.
- **Community event coordination** — scheduled events with reaction-based RSVP sign-ups (synced with Discord's native scheduled events), polls, and "roll call" tracking of member activity around war transitions.
- **Member lifecycle & moderation** — a ticket-based verification flow for onboarding and role assignment, moderation commands (ban/kick/timeout/warn/purge) with audit logging, and background membership analytics (join dates, voice time, message counts).
- **Quality-of-life utilities** — join-to-create temporary voice channels, a "Top VC" vanity role for the members most active in voice, Foxhole tech-progress screenshot analysis, a community directory of regiments/clans/orgs, referral-code sharing for other games (World of Tanks, World of Warships, Star Citizen, War Thunder), and a PSG gif auto-responder.
- **External integration** — an optional FastAPI REST API (plus reference OAuth2 helpers) so a companion website can read stats/events and manage settings/roles for the same guild.

The bot is designed to run **in one Discord server at a time**, backed by a single SQLite database (`umbrella.db`).

## Features

### Command Index

Every slash command, grouped by bot section and linked to its full documentation in [Commands](#commands):

- **[War Management](#cmd-war-management)**: [`/war_create`](#cmd-war-management), [`/war_initiate`](#cmd-war-management), [`/war_terminate`](#cmd-war-management), [`/war_view`](#cmd-war-management), [`/war_warname`](#cmd-war-management), [`/war_delete`](#cmd-war-management), [`/war_edit`](#cmd-war-management)
- **[Stat Entry & Viewing](#cmd-stat-entry-viewing)**: [`/stats_entry`](#cmd-stat-entry-viewing), [`/stats_edit`](#cmd-stat-entry-viewing), [`/stats_view`](#cmd-stat-entry-viewing), [`/leaderboard`](#cmd-stat-entry-viewing)
- **[Service Record System](#cmd-service-record)**: [`/service_record`](#cmd-service-record), [`/service_rankadd`](#cmd-service-record), [`/service_leveladd`](#cmd-service-record), [`/service_enteredservice`](#cmd-service-record)
- **[Event Management](#cmd-event-management)**: [`/event_create`](#cmd-event-management), [`/event_list`](#cmd-event-management), [`/event_list_past`](#cmd-event-management), [`/event_view`](#cmd-event-management), [`/event_start`](#cmd-event-management), [`/event_end`](#cmd-event-management), [`/event_edit`](#cmd-event-management), [`/event_delete`](#cmd-event-management), [`/event_stats`](#cmd-event-management)
- **[Poll Management](#cmd-poll-management)**: [`/poll_create`](#cmd-poll-management), [`/poll_end`](#cmd-poll-management)
- **[Moderation Commands](#cmd-moderation-commands)**: [`/mod_purge`](#cmd-moderation-commands), [`/mod_ban`](#cmd-moderation-commands), [`/mod_kick`](#cmd-moderation-commands), [`/mod_timeout`](#cmd-moderation-commands), [`/mod_warn`](#cmd-moderation-commands), [`/mod_warnings`](#cmd-moderation-commands), [`/mod_warning_delete`](#cmd-moderation-commands), [`/mod_userinfo`](#cmd-moderation-commands)
- **[Roll Call System](#cmd-roll-call-system)**: [`/rollcall_start`](#cmd-roll-call-system), [`/rollcall_end`](#cmd-roll-call-system), [`/rollcall_regiment_start`](#cmd-roll-call-system), [`/rollcall_regiment_end`](#cmd-roll-call-system)
- **[Temporary VC](#cmd-temporary-vc)**: [`/vcrename`](#cmd-temporary-vc), [`/vclimit`](#cmd-temporary-vc), [`/vclock`](#cmd-temporary-vc), [`/vcunlock`](#cmd-temporary-vc), [`/vcwhitelistrole`](#cmd-temporary-vc), [`/vcblacklistrole`](#cmd-temporary-vc), [`/vcwhitelistuser`](#cmd-temporary-vc), [`/vcblacklistuser`](#cmd-temporary-vc), [`/vc_history`](#cmd-temporary-vc)
- **[Regiments, Clans & Orgs](#cmd-regiments-clans-orgs)**: [`/regiments`](#cmd-regiments-clans-orgs)
- **[Referral Codes](#cmd-referral-codes)**: [`/referral`](#cmd-referral-codes)
- **[Vanity Roles](#cmd-vanity-roles)**: [`/vanity_vc`](#cmd-vanity-roles)
- **[Activity Stats](#cmd-vc-stats)**: [`/vc_stats`](#cmd-vc-stats), [`/vc_leaderboard`](#cmd-vc-stats), [`/stream_leaderboard`](#cmd-vc-stats), [`/message_leaderboard`](#cmd-vc-stats)
- **[Tech Progress Analysis](#cmd-tech-progress-analysis)**: [`/techprogress_bases`](#cmd-tech-progress-analysis), [`/techprogress_engineering`](#cmd-tech-progress-analysis)
- **[Verification & Ticket System](#cmd-verification-ticket-system)**: [`/send_verification_embed`](#cmd-verification-ticket-system), [`/restore_roles`](#cmd-verification-ticket-system)
- **[Rules System](#cmd-rules-system)**: [`/post_rules`](#cmd-rules-system)
- **[API Key Management](#cmd-api-key-management)**: [`/api_key_create`](#cmd-api-key-management), [`/api_key_list`](#cmd-api-key-management), [`/api_key_revoke`](#cmd-api-key-management)
- **[Configuration](#cmd-configuration)**: [`/settings`](#cmd-configuration), [`/database_export`](#cmd-configuration), [`/sync_commands`](#cmd-configuration)
- **[Bot Status](#cmd-bot-status)**: [`/uptime`](#cmd-bot-status), [`/shutdown`](#cmd-bot-status)

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
  - Three messages are maintained in a fixed order: **Lifetime**, **Active War**, and an **info card**
  - Refreshes immediately after a stat submission or edit, instead of waiting for the next 30-minute tick
  - All posting, updating, and reordering is serialized behind a single lock, and transient Discord errors skip rather than resend — so restarts and overlapping updates no longer create duplicate embeds
  - Ending a war relabels its leaderboard as **🏁 Final Leaderboard** and leaves it up until the next war starts; stale war leaderboards are removed automatically
- **Guild Display Names**: Leaderboards show users' guild nicknames when available
- **Data Governance**: Input validation ensuring data quality (numbers only, 0-999,999,999 range checks)
- **War Channel Updates**: Automatic voice channel name updates reflecting active war and faction (updates every 5 minutes)
  - Creates/updates "Active War: {shard} {war_number}" voice channel
  - Creates/updates "Faction: {faction}" voice channel
  - Channels are set to private (no connect permission for default role)
- **Service Records**: Comprehensive service record system tracking member history
  - Rank and level tracking
  - Service history based on assigned war roles
  - Specializations and certifications from Discord roles
  - Rank insignia display
  - War participation tracking across shards (Able/Baker/Charlie)

### Event Management
- ✅ Create events with custom names, descriptions, and times
- ✅ 12-hour time format, with the timezone picked from a slash-command dropdown (no typing timezone names)
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
- ✅ Guided setup — poll type, voting type, and (for time polls) timezone are chosen from dropdowns before the modal opens
- ✅ Single or multiple choice voting
- ✅ Automatic expiration based on duration
- ✅ Reaction-based voting system
- ✅ Thread creation for discussion
- ✅ Vote count display after poll ends

### Roll Call System
- ✅ Track member activity status during war transitions
- ✅ Main roll call: automatic timer-based expiration (configurable); regiment roll call: no timer, manual end
- ✅ Reaction-based status tracking (Active/Inactive)
- ✅ Regiment roll call: ✅ reaction assigns Current War Verification role (separate channel)
- ✅ Persistent embeds across bot restarts
- ✅ Automatic result processing

### Temporary VC (Join-to-Create)
- ✅ When a user joins a voice channel named **"Join to Create VC"** (in the configured category), the bot creates a temporary voice channel for them (or moves them to their existing one)
- ✅ Channels are named after the creator (e.g. *Username's VC*) and are deleted when empty
- ✅ **Creator-only commands**: Rename VC, set user limit (1–99), lock/unlock, whitelist/blacklist by role or user
- ✅ Configure the Join-to-Create category in `/settings` (Event Management → Join-to-Create VC Category)
- ✅ A "Customize your VC" embed with the command list is sent in the new channel (Regiment role mentioned for permission context)
- ✅ **Every closed VC leaves a history record** — its name, owner, how long it was up, peak occupancy, and how many distinct members passed through — written *before* the channel is deleted, because Discord retains nothing about a deleted channel
- ✅ **`/vc_history`** — anyone can list recently closed VCs with their uptime, read server totals, and see a top-five ranking of owners by total VC uptime; optionally filtered to one member
- ✅ The awkward endings are recorded too: a VC deleted by a moderator, or one that emptied (or vanished entirely) while the bot was offline, is archived on the next startup rather than lost

### Tech Progress Analysis
- ✅ Image-based tech progress detection from screenshots
- ✅ Automatic percentage calculation
- ✅ Visual progress bar display

### Moderation System
- ✅ Comprehensive moderation commands (ban, kick, timeout, warn, purge)
- ✅ Warning system with persistent tracking
- ✅ Automatic moderation action logging
- ✅ Role hierarchy protection
- ✅ Moderator role-based access control
- ✅ **Turing Test** honeypot channel — anyone who posts there is auto-banned for spam, with a live-updating catch counter (see [Moderation Commands](#cmd-moderation-commands))

### Verification & Ticket System
- ✅ Ticket-based verification system with private channels
- ✅ Automatic ticket creation via button
- ✅ Role assignment via ticket buttons (Diplomat, Regiment, Non-Foxhole)
- ✅ Automatic ticket closure 24 hours after role assignment
- ✅ Full transcript generation on ticket closure (JSON download + Discord-style HTML)
- ✅ **Transcripts on the website** — transcripts are served through the REST API for the website to read and render, gated on moderator access (see [Ticket Transcripts](#ticket-transcripts))
- ✅ Manual ticket closure option
- ✅ Persistent verification embeds and ticket buttons
- ✅ **Role retention** — a member's roles are always saved when they leave, are kicked, or are banned (and kept current while they're in the server). When restoration is enabled, staff can re-apply them with `/restore_roles` or the **Restore Saved Roles** button in a returning member's verification ticket. Restorable by Admins, the Moderator role, or the Recruiter role.
  - The ticket panel only appears for **genuinely returning** members (a snapshot taken on departure), never for ordinary role changes on someone who never left
  - The panel previews **roles when they left vs. what they have now**, and what would/wouldn't be re-applied — nothing changes until staff click and confirm
  - **Selective restore** — staff can pick a subset of roles (**Restore Selected**) or take everything (**Restore All**)
  - Roles the verification flow assigns itself are excluded, so restoration can't fight the verification flow
  - **Expiry window** (default 180 days, `0` = never) — snapshots older than the window are no longer offered
  - Restorations are posted to the moderation log, and the snapshot is dropped once the returning member's ticket closes

### Vanity Roles (Top VC)
- ✅ Awards a configurable **Top VC** role to the **3 most active members in voice** over the last **30 days**
- ✅ Ranking is computed from a per-session voice log (`vc_sessions`), so it is a true rolling window, not an all-time total
- ✅ Recomputed at exactly two moments and no others — **00:00 UTC on Sunday** and the **Force Resync** button — when the role is added to the current top 3 and removed from anyone who dropped out
- ✅ The underlying task still ticks daily (discord.py can schedule a time of day but not a day of the week); the non-Sunday ticks only prune the session logs and never touch the role
- ✅ **A missed Sunday is caught up.** If the bot was down at 00:00 Sunday, the next daily tick notices the scheduled sync went unserved and runs it then, instead of leaving the role stale for another week
- ✅ Bots and members who have left the server are skipped, backfilling from further down the ranking so the podium stays full
- ✅ Optional **announcement channel** — changes are posted with the current standings and who is new
- ✅ **`/vanity_vc`** — anyone can view the current standings; administrators get a **Force Resync** button on the same message
- ✅ Configured under `/settings → General` (Top VC Role + Top VC Announcement Channel); the role takes effect at the next scheduled sync, or immediately via **Force Resync**
- ✅ Session rows older than the window are pruned automatically, so the log stays bounded

### Activity Stats (all-time)
- ✅ **Command-only** leaderboards — nothing is auto-posted or refreshed on a timer
- ✅ **`/vc_stats`** — server-wide totals in three blocks: **voice** (members tracked, members with VC activity, total VC time as raw seconds and a d/h/m/s breakdown, total joins, average time per join, top 25's share), **Go Live** (members who have streamed, total stream time, streams started, average per stream, share of VC time spent streaming), and **messages** (members who have chatted, total messages, average per chatting member, messages per hour of VC time, top 25's share)
- ✅ **`/vc_leaderboard`** — top 25 members by all-time voice time, with **stream time**, **messages**, and each member's rank by join count (`J#`) alongside
- ✅ **`/stream_leaderboard`** — top 25 members by all-time Go Live time, with how many streams they started and what share of their *own* voice time they spent streaming
- ✅ **`/message_leaderboard`** — top 25 members by all-time messages sent, with each member's share of all server messages and their voice time for context
- ✅ Reads the **all-time** totals in `membership_tracking`, so it is a lifetime view — distinct from the rolling 30-day window behind the Top VC vanity role
- ✅ Members who chat but have **never joined a voice channel** still rank on the message leaderboard (they show `-` in the voice columns) rather than being invisible
- ✅ Bots are excluded, and current server nicknames are used where the member is still in the guild

### Membership Tracking (optional)
- ✅ Records server join date, regiment role assignment date, voice channel time, **Go Live streaming time**, and message count per user
- ✅ Message counts come from `on_message` for any guild message; bots return early there, so they never inflate the totals
- ✅ Data stored in `membership_tracking` table; used for analytics and backfill via `helpers/membership_helpers.py`

### Streaming vs. VC Tracking
- ✅ Tracks **Go Live** (screenshare) separately from plain voice time, using the `self_stream` flag on Discord's voice state
- ✅ Stream time is a **subset** of VC time, not an alternative to it — a member who streams for an hour accrues an hour of both, so you can read "how much of our voice time is people streaming?"
- ✅ Per-user all-time totals land in `membership_tracking` (`stream_starts_count`, `stream_time_seconds`); per-session rows land in `vc_stream_sessions` for rolling-window queries
- ✅ A stream **survives a channel move** (Discord keeps `self_stream` set) but always ends on disconnect
- ✅ The two timers are fully independent, so streaming accrues **both** VC time and stream time for the same period — stream time is a subset of VC time, never a substitute for it
- ✅ When voice logging is enabled, **Started Streaming** / **Stopped Streaming** are posted to the voice log channel alongside joins, leaves, and moves
- ℹ️ Discord exposes only *that* a member is streaming — never what they are streaming, and never who is watching
- ℹ️ This is Go Live screenshare, which is distinct from the purple "Streaming" presence status (a linked Twitch/YouTube broadcast), which would require the privileged presence intent

### API Key Management
- ✅ Guild-scoped API keys for REST API authentication (`/api_key_create`, `/api_key_list`, `/api_key_revoke`)

### Regiments, Clans & Orgs
- ✅ **`/regiments`** — Public listing of community regiments, clans, and orgs grouped by game
- ✅ Optional join link and symbol (emoji, image URL, or uploaded image) per entry
- ✅ **Add Regiment** button on the listing (Moderator role from bot settings or Administrator only)
- ✅ Modal-driven entry: game name and regiment name required; join link and symbol optional

### Referral Codes
- ✅ **`/referral`** — Random community referral codes (one per supported game that has submissions)
- ✅ **Add/Modify** button on the same message — anyone can submit or update their own codes
- ✅ Modal with optional fields for: World of Tanks, World of Warships, Star Citizen, War Thunder
- ✅ Submitter name/mention shown alongside each displayed code
- ✅ One code per user per game (resubmitting updates your existing entry)

### Bot Status
- ✅ **`/uptime`** — status, uptime, gateway ping, and API round-trip time in one embed
- ✅ Distinguishes **process uptime** from **gateway connection age**: discord.py reconnects without restarting, so a bot that has been up for days may have re-established its connection many times
- ✅ Surfaces a **reconnect count** once the connection has flapped, which is the signal that something is wrong even though the bot looks online
- ✅ Public — replaces the old admin-only `/ping`
- ✅ **`/shutdown`** — administrator-only, behind a confirmation, stops the bot cleanly for an update
- ✅ A clean stop is what **saves the open voice and stream timers**; killing the process discards whatever was in flight

### PSG Auto-Responder
- ✅ Any message mentioning **PSG** as a standalone word (any casing) gets the PSG gif back as a reply
- ✅ Fires **anywhere in the message**, not just on its own: `go PSG go`, `PSG!`, `i love psg.`, and `PSG vs Madrid` all trigger it
- ✅ Matched on **word boundaries**, so the letters buried inside a longer word do not count — `PSGG`, `xPSG`, `psg_thing`, and `nopsg` deliberately stay silent
- ✅ Replies without pinging the author, and only in servers (never in DMs). The message is not consumed, so normal handling continues afterwards
- ⚙️ The gif is read from **`images/psg.gif`** — drop the file there to enable it. If it is missing, the trigger silently does nothing and logs a one-line note to the console
- ⚙️ A **15-second per-channel cooldown** stops an active conversation about PSG from re-uploading a multi-megabyte file on every line and burning the bot's global rate limit. Set `PSG_COOLDOWN_SECONDS = 0` in `helpers/autoresponder_helpers.py` to respond every single time

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
   API_ENABLED=true                  # Set to false to disable API server
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
   - Enable "Message Content Intent" and "Server Members Intent" (Privileged Gateway Intents) in the Bot settings
   - Reactions don't require a privileged intent toggle, but the bot needs the "Read Message History" and "Add Reactions" permissions (granted below) to use them

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
     - Manage Messages (for moderation commands)
     - Ban Members (for moderation commands)
     - Kick Members (for moderation commands)
     - Moderate Members (for timeout command)
     - Manage Roles (for verification and role syncing)
     - Manage Channels (for war stat voice channel updates and ticket creation)
   - Copy the generated URL and open it in your browser to invite the bot

### Running the Bot

```bash
python bot.py
```

Startup runs as a sequence of named steps, each of which reports how long it took:

```
  ✓ Databases (0.1s)
  ✓ Command registration (1.2s)
  ✓ Command sync (2.4s)
  ✓ Startup state restore (0.3s)
  ✓ Persistent views/embeds (0.4s)
  📋 Membership backfill: 348 members, 12 regiment-role dates filled
  ✓ Membership backfill (2.2s)
  🎙️ Voice tracking: 4 connected (1 streaming) — resumed 4 voice, 1 stream timer(s)
  ✓ Voice tracking resume (0.0s)
  🎤 Custom VCs: 2 archived, 1 still active
  ✓ Custom VC reconcile (0.1s)

✅ Bot is ready! (startup took 6.6s)
```

- **Every step is bounded by a timeout.** A step that overruns is abandoned with a `⏱️` line and startup continues, so the bot always reaches ready instead of stalling forever with no explanation. The timeouts are safety nets set well above any healthy run, not budgets
- **Every step reports failures** with the exception and a traceback rather than silently skipping
- This matters because `guild.chunk()` — used to populate the member cache during the membership backfill — is a websocket round trip whose internal wait has **no timeout of its own**: it awaits a future that only resolves when the gateway sends member chunks. It is now called through `ensure_guild_chunked()`, which skips it when the guild is already chunked, bounds it, and reports failures instead of swallowing them
- If `✅ Bot is ready!` never appears, the last `✓` line names the step before the problem and the next line says whether it timed out or failed

### Running Tests

```bash
python -m unittest discover -s tests -v
```

## Commands

<a id="cmd-war-management"></a>
### War Management (War Tracker System)

- `/war_create <war_number> <shard> <faction> <war_role>` - Create a new war with associated Discord role **[Admin only]**
  - Requires: war number, shard (Able/Charlie/Baker), faction (Warden/Colonial), and a Discord role
  - The war role is used to track which users participated in each war for service records
- `/war_initiate <war_number>` - Start/activate a war (deactivates all other wars) **[Admin only]**
  - Autocomplete shows all available wars with their status
  - Preserves existing shard information if already set
  - Removes the previous war's final leaderboard and refreshes the leaderboard channel immediately
- `/war_terminate <war_number>` - End/deactivate a war **[Admin only]**
  - Autocomplete shows only active wars
  - Choose the result (Win / Loss / Draw); the war's leaderboard is then relabelled **🏁 Final Leaderboard** and left up until the next war starts
- `/war_view` - View all wars in the database with status indicators (🟢 active, 🔴 inactive)
- `/war_warname <war_number> <war_name>` - Set or update a friendly name for a war **[Admin only]**
  - War names are displayed in service records instead of just war numbers
- `/war_delete <war_number>` - Permanently delete a war and all associated stats **[Admin only]**
  - Requires confirmation before deletion
  - **WARNING:** This action cannot be undone and will delete all user stats for that war
  - Autocomplete shows all wars
- `/war_edit <war_number>` - Edit a war's information **[Admin only]**
  - Opens a setup panel with **Faction** and **Result** dropdowns (pre-selected to the war's current values), then an **Edit War Details** button that opens the modal
  - The modal covers war number, start date, and end date (dropdowns can't live inside a Discord modal, hence the two steps)
  - Date format: YYYY-MM-DD or YYYY-MM-DDTHH:MM:SS
  - Result options: Win, Loss, or Draw
  - Autocomplete shows all wars

**Note:** These commands use the War Tracker system and are separate from event war tracking.

<a id="cmd-stat-entry-viewing"></a>
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

<a id="cmd-service-record"></a>
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

- `/service_enteredservice <entered_service> [user]` - Add or update a member's entered service date
  - Date format: YYYY-MM-DD (e.g., 2021-07-16)
  - Defaults to your own date if no user specified
  - Users with Manage Server permissions can update other members' dates

**Service Record Features:**
- **Specializations**: Automatically populated from assigned Discord roles (Combat Engineer, Armored Cavalry, Naval Specialist, etc.)
- **Certifications**: Automatically populated from assigned Discord roles (Facility, Naval, Op, Combat Engineering, Logistics, Recruiter)
- **Service History**: Shows wars based on assigned war roles, grouped by shard (Able/Baker/Charlie)
- **Rank Insignia**: Displays rank badge thumbnail from `images/rank_insignia/` folder
- **Total Wars**: Counts wars based on assigned war roles across all shards

<a id="cmd-event-management"></a>
### Event Management (Events System)

- `/event_create <timezone>` - Create a new event
  - Pick the timezone from the command's dropdown, then fill in the modal: name, optional description, start time, end time
  - Times use `YYYY-MM-DD h:mm AM/PM` (e.g. `2026-01-15 8:00 PM`) — the expected format is shown in each field's label
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

<a id="cmd-poll-management"></a>
### Poll Management

- `/poll_create` - Create a new poll (guided dropdowns, then a modal)
  - **1) Poll type** — Text options or Time options
  - **2) Voting type** — Single choice or Multiple choice
  - **3) Timezone** — required for time polls only
  - Click **Continue** to open the matching modal for the question, options, and duration
  - Time options use `YYYY-MM-DD h:mm AM/PM`, one per line
  - Set poll duration (minutes, hours, or days)
  - Polls are automatically posted to a designated channel with a thread
  - Votes are collected via emoji reactions
  - Polls automatically end when their duration expires

- `/poll_end [poll]` - Manually end a poll
  - Autocomplete shows only active/open polls
  - Displays final vote counts and highlights winning option(s)

<a id="cmd-moderation-commands"></a>
### Moderation Commands **[Moderator Role Required]**

- `/mod_purge <target_type> <purge_type> [user] [amount] [date]` - Purge messages from a channel or server
  - Target types: User (in channel), All humans, All bots, User (server-wide)
  - Purge types: All messages, Number of messages, Back to date
  - For date purges, use format: YYYY-MM-DD
  - Server-wide purges work across all text channels
  - All purge actions are logged to the moderation log channel

- `/mod_ban <user> <reason>` - Ban a user from the server
  - Requires reason for the ban
  - Respects role hierarchy (cannot ban users with higher or equal roles)
  - Logs ban action to moderation log channel

- `/mod_kick <user> <reason>` - Kick a user from the server
  - Requires reason for the kick
  - Respects role hierarchy (cannot kick users with higher or equal roles)
  - Logs kick action to moderation log channel

- `/mod_timeout <user> <duration> <reason>` - Timeout a user (prevent them from chatting)
  - Duration format: `1h`, `30m`, `2d`, etc. (maximum 28 days)
  - Requires reason for the timeout
  - Respects role hierarchy (cannot timeout users with higher or equal roles)
  - Logs timeout action to moderation log channel

- `/mod_warn <user> <reason>` - Warn a user
  - Adds a warning to the user's record
  - Displays total warning count after warning
  - Warnings are stored in the database
  - Logs warn action to moderation log channel

- `/mod_warnings <user>` - View all warnings for a user
  - Shows all warnings with moderator, reason, and timestamp
  - Displays up to 10 most recent warnings
  - Shows total warning count

- `/mod_warning_delete <user> <warning_id>` - Delete a warning from a user
  - Requires the warning ID (shown in `/mod_warnings`)
  - Autocomplete shows warnings for the selected user with reason and date
  - Logs deletion to moderation log channel

- `/mod_userinfo <user>` - View detailed information about a user
  - Shows user ID, account creation date, server join date
  - Displays user status (Server Owner, Admin, Moderator, etc.)
  - Shows key permissions and roles (top 10 roles)
  - Displays warning count
  - Works for users not in the server (limited information)
  - Includes Discord timestamps for dates

**Moderation Features:**
- All moderation actions are logged to a designated moderation log channel (if configured)
- Role hierarchy protection (moderators cannot moderate users with higher roles)
- Warning system tracks user infractions with timestamps and reasons
- Moderator role required (configured via Discord role, not permissions)

**Turing Test (Bot Honeypot Channel):**
- Configure a hidden "honeypot" channel via `/settings` (Moderation → Set Turing Test Channel, Enable/Disable Turing Test)
- Anyone who posts in that channel — human or bot — is immediately banned, following the same ban flow as `/mod_ban`
- The offending message is deleted and the ban is logged to the moderation log channel (if configured), with the bot itself shown as the acting moderator
- The bot posts a persistent embed in the channel explaining its purpose, showing whether the feature is active and a live count of catches; the embed is updated in place (not re-posted) each time someone is caught
- Server owners and Administrators are never auto-banned by this feature, to prevent accidental lockouts while configuring the channel

<a id="cmd-roll-call-system"></a>
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

- `/rollcall_regiment_start <ending_war> <upcoming_war> [channel]` - Start a regiment roll call **[Admin only]**
  - Posts the same style of roll call embed in the Regiment Channel (or specified channel)
  - **No timer** — does not auto-end. Use `/rollcall_regiment_end` to end it.
  - Reacting ✅ assigns the **Current War Verification** role to the user; ❌ does nothing
  - Used for regiment members to opt in to the current war role
  - Autocomplete available for war selection

- `/rollcall_regiment_end` - End the active regiment roll call **[Admin only]**
  - Stops the regiment roll call so reactions no longer assign the Current War Verification role
  - Updates the embed and clears reactions

- Roll call settings are configured in `/settings` → **Roll Call** **[Admin only]**
  - Set the diplomat channel where (main) roll calls are posted
  - Set the Regiment Channel where regiment roll calls are posted
  - Configure roll call timer duration (in hours) for main roll calls
  - Interactive dropdown interface for easy configuration

**Roll Call Features:**
- Main roll call: automatic timer-based expiration; regiment roll call: no timer, manual end only
- Reaction-based member status tracking
- Persistent embeds that survive bot restarts
- Automatic processing when main roll call timer expires

<a id="cmd-temporary-vc"></a>
### Temporary VC (Join-to-Create)

When a voice channel named **"Join to Create VC"** is placed in the configured category (set in `/settings` → Event Management), users who join it get a temporary VC created (or are moved to their existing one). Only the VC creator can use these commands:

- `/vcrename <name>` - Rename your temporary VC (up to 100 characters)
- `/vclimit <limit>` - Set user limit (1–99) for your VC
- `/vclock` - Lock your VC so no one new can join (only you can connect)
- `/vcunlock` - Unlock your VC so everyone can join again
- `/vcwhitelistrole <role>` - Only members with this role (and you) can join
- `/vcblacklistrole <role>` - Members with this role cannot join
- `/vcwhitelistuser <user>` - Only this user (and you) can join
- `/vcblacklistuser <user>` - This user cannot join

Empty temporary VCs are automatically deleted. The bot sends a "Customize your VC" embed in the new channel with the command list.

**Available to everyone:**

- `/vc_history [user] [limit]` - Custom VCs that have closed: how long each was up and who was in it
  - Each entry shows the VC's name, its uptime, how many distinct members passed through and the peak number at once, and how it ended (emptied, deleted, or settled at restart)
  - Server totals alongside the list: VCs closed, total uptime, and the longest single VC
  - A **Most VC uptime** ranking of the top five owners by total time their VCs were up (omitted when the list is filtered to one member)
  - `user` limits the list to VCs created by that member; `limit` is 1–25 (default 10)
  - Records start at the first VC that closed after this feature was added — VCs that closed before it were never recorded and cannot be recovered

<a id="cmd-regiments-clans-orgs"></a>
### Regiments, Clans & Orgs

- `/regiments` - View all community regiments, clans, and orgs grouped by game
  - Each entry shows name, optional join link, and optional symbol
  - Multiple embeds may be sent if the list is long

- **Add Regiment** (button on `/regiments` message) - Add a new entry **[Moderator role or Administrator]**
  - Opens a modal:
    - **Game name** (required)
    - **Regiment / clan / org name** (required)
    - **Link to join** (optional)
    - **Symbol** — URL or emoji (optional)
  - After saving, optionally upload a symbol image via **Upload symbol image** or **Skip**
  - Requires the **Moderator** role configured in `/settings`, or Administrator permission

<a id="cmd-referral-codes"></a>
### Referral Codes

- `/referral` - Get random community referral codes for supported games
  - Shows one random code per game that has at least one submission
  - Each field includes the code and who submitted it (mention or stored nickname)
  - Run again anytime for a new random selection
  - Includes an **Add/Modify** button on the same message

- **Add/Modify** (button on `/referral` message) - Submit or update your referral codes
  - Opens a modal with optional fields (fill in only the games you have):
    - World of Tanks
    - World of Warships
    - Star Citizen
    - War Thunder
  - Anyone can use the button; codes are saved for **your** Discord account only
  - Submitting again updates your code for any game you enter
  - Confirmation is ephemeral (only you see it)

<a id="cmd-vanity-roles"></a>
### Vanity Roles

- `/vanity_vc` - Show the **Top VC** standings for the last 30 days
  - Available to everyone; posts a public embed with the current top 3 and their total voice time
  - Also shows which role is configured as the Top VC role (or a hint to set one)
  - **Force Resync** (button on the same message) — recomputes the standings and re-applies the role right away **[Administrator only]**
  - Configure the role and optional announcement channel under `/settings → General`
  - The role is otherwise only recomputed at **00:00 UTC on Sunday** — this button is the only other trigger

<a id="cmd-vc-stats"></a>
### Activity Stats

- `/vc_stats` - Server-wide voice, streaming, and message stats (**all-time**), posted as **Umbrella Discord Stats**
  - Members tracked, members with VC activity, total VC time in raw seconds plus a days/hours/minutes/seconds breakdown
  - Total VC joins, average time per join, and the share of all VC time held by the top 25
  - A **Go Live Streaming** block with the same shape: members who have streamed, total stream time, streams started, average time per stream, and the share of VC time spent streaming
  - A **Messages** block: members who have chatted, total messages sent, average per chatting member, messages per hour of VC time, and the share held by the top 25
- `/vc_leaderboard [limit]` - Top 25 members by **all-time** time spent in voice channels
  - Columns: rank, member, total time, **stream time**, VC joins, **messages**, and `J#` (that member's rank by number of joins)
  - Message counts are abbreviated past 10,000 (`15.0k`) so the extra column doesn't widen the table further
  - Below the table: the top 25's combined **VC time**, **stream time**, and **messages**, each with its share of the server total, so all three read side by side
  - Ranking is always by **VC time** — the stream and message columns are context, not the sort key
  - `limit` (optional, 1-25) shortens the table; defaults to 25
- `/stream_leaderboard [limit]` - Top 25 members by **all-time** Go Live streaming time
  - Columns: rank, member, stream time, number of streams started, and `% VC` (share of that member's *own* voice time spent streaming)
  - Members who have never streamed are left out entirely, so a quiet server shows a short table rather than a page of zeroes
  - `limit` (optional, 1-25) shortens the table; defaults to 25
- `/message_leaderboard [limit]` - Top 25 members by **all-time** messages sent
  - Columns: rank, member, messages, share of all server messages, and VC time for context
  - Members who have never posted are left out; text-only members rank normally and show `-` for VC time
  - A field below the table counts how many of the top 25 have never joined a voice channel
  - `limit` (optional, 1-25) shortens the table; defaults to 25
- All four are **command-only** — they post a public embed on demand and are never refreshed automatically
- All four read the all-time totals in `membership_tracking` (a lifetime view), unlike `/vanity_vc`, which uses a rolling 30-day window
- Stream time is a **subset** of VC time, so `% VC` and "share of VC time spent streaming" can never exceed 100%. Messages are independent of both

<a id="cmd-tech-progress-analysis"></a>
### Tech Progress Analysis

- `/techprogress_bases <image>` - Analyze tech progress from a screenshot (bunker bases, townhalls, relics)
- `/techprogress_engineering <image>` - Analyze engineering center tech (how far up the lightest grey extends %)
  - Upload an image containing the tech progress box from Foxhole
  - Automatically detects and calculates the progress percentage
  - Displays a visual progress bar and percentage
  - Useful for tracking research progress across the faction

<a id="cmd-verification-ticket-system"></a>
### Verification & Ticket System

- `/send_verification_embed` - Post or update the verification embed in the configured channel **[Admin only]**
  - Creates or updates the verification embed with ticket creation button
  - Embed includes instructions for verification process
  - Button allows users to create verification tickets
  - Automatically checks for existing embed before posting

- **Verification Ticket System**:
  - Users click "Open Verification Ticket" button to create a private ticket channel
  - Each user can only have one open ticket at a time
  - Ticket channels are created in the configured verification category
  - Staff (Admins or Recruiter Certification role) can assign roles via buttons in ticket
  - Tickets automatically close 24 hours after role assignment
  - Manual closure available via "Close Ticket" button
  - Full transcripts generated on closure (saved to `transcripts/` folder)
  - Transcripts can be sent to configured transcript channel
  - Channel names reflect ticket status (open, completed, closed)
  - If the ticket creator is a **genuinely returning** member with a non-expired snapshot (and role retention is enabled), a **Returning Member — Saved Roles** panel is posted in the ticket
    - Shows the roles they held when they left, what they have now, what is available to restore, and anything that can't be restored (too high, managed, deleted, or verification-assigned)
    - The **Restore Saved Roles** button opens an ephemeral, staff-scoped picker: select a subset and **Restore Selected**, or **Restore All**. Nothing is applied until confirmed
    - Closing the ticket clears the departure snapshot, so the panel doesn't reappear on future tickets

- `/restore_roles user:<member>` - Restore a returning member's saved roles **[Admin, Moderator role, or Recruiter role]**
  - Re-applies **all** assignable roles the member had when they left/were kicked/banned (for a pick-and-choose restore, use the ticket panel below)
  - Skips `@everyone`, integration/managed roles, deleted roles, roles the verification flow assigns itself, and any role above the bot's highest role (all reported back so staff can handle them manually)
  - Refuses snapshots older than the **Role Retention Expiry** window (default 180 days; `0` = never expire)
  - Requires **Role Retention** to be enabled in `/settings → Moderation`
  - Capture is always on; the toggle only controls restoration
  - Successful restores are posted to the moderation log channel

<a id="cmd-rules-system"></a>
### Rules System

- `/post_rules` - Post server rules to the configured rule channel **[Admin only]**
  - Posts formatted rule embeds to the designated rule channel
  - Requires rule channel to be configured in `/settings`
  - Creates multiple embeds for comprehensive rule display
  - Bot must have Send Messages and Embed Links permissions

<a id="cmd-api-key-management"></a>
### API Key Management **[Admin only]**

- `/api_key_create [site_name]` - Create a new API key for this server
  - Key is shown once; save it immediately. Use in `X-API-Key` header for API requests.
  - Optional `site_name` identifies the service using the key

- `/api_key_list` - List all API keys for this server
  - Shows key ID, site name, created by, created date; does not show the secret key

- `/api_key_revoke <key_id>` - Revoke an API key
  - Deactivates the key so it can no longer be used for API access

### Event War Tracking
- Events can be tracked by war number (separate from War Tracker wars)
- Events created while a war is active will be automatically assigned to that war
- Use `/event_stats war_filter:100` to view statistics for a specific war

<a id="cmd-configuration"></a>
### Configuration (Admin Only)

- `/settings` - View and manage **all bot settings** in a stateful category-driven panel
  - Home summary with quick status checks
  - Category navigation:
    - War Tracker
    - Events
    - Verification
    - Roll Call
    - Moderation (includes the Turing Test honeypot channel + toggle, and Role Retention)
    - Logging
    - General (rule channel, bot status message, **Top VC role**, **Top VC announcement channel**)
  - Uses reusable action handlers for toggles, channels, roles, and text fields
  - Uses modal input for text settings:
    - verification welcome message (pre-filled with the built-in default so admins edit from it — the setting only *overrides* the default, it never replaces it)
    - bot status message
    - roll call timer (hours)
    - role retention expiry (days, `0` = never)
  - Setting the Top VC role does **not** sync it right away — it takes effect at the next Sunday 00:00 UTC pass, or immediately via **Force Resync** on `/vanity_vc`
  - Includes session safety:
    - user-scoped interaction checks
    - timeout handling that disables controls
  - Single unified panel: War Tracker, Events, Verification, Roll Call, Moderation, Logging, and General

#### Settings UI Implementation Notes
- UI render helpers and action maps live in `helpers/settings_ui.py`
- `/settings` view classes and callbacks live in `commands/admin_commands.py`
- Data flow:
  - load snapshot (`load_settings_snapshot`)
  - build embed (`build_home_embed` / `build_category_embed`)
  - apply updates via action handlers (`handle_toggle_action`, `handle_channel_action`, `handle_role_action`, `handle_text_action`)

- `/database_export` - Export full database to CSV file (sent via DM) **[Admin only]**
  - Exports all tables from the single `umbrella.db` database
  - The CSV file will be sent to your DMs (or in the channel if DMs are disabled)
  - Files are timestamped (e.g. `umbrella_export_YYYYMMDD_HHMMSS.csv`)

- Logging is configured in `/settings` → **Logging** **[Admin only]**
  - Enable/disable message logging (tracks message edits and deletions)
  - Set message log channel
  - Enable/disable role logging (tracks role assignments and removals)
  - Set role log channel
  - Enable/disable moderation logging (tracks moderation actions)
  - Set moderation log channel
  - Enable/disable voice logging (tracks voice channel joins, leaves, and moves)
  - Set voice log channel
  - Enable/disable join/leave logging (tracks member joins and leaves)
  - Set join/leave log channel
  - Interactive dropdown interface for configuration

- `/sync_commands` - Re-register all bot commands (useful after updates) **[Admin only]**
  - Clears and re-registers all commands
  - Useful when commands aren't appearing or after bot updates
  - Can sync to a test guild if `TEST_GUILD_ID` is set in `.env`

<a id="cmd-bot-status"></a>
### Bot Status

- `/uptime` - Show the bot's status, uptime, and ping
  - **Status** line, colour-coded from gateway latency: healthy (<150ms), elevated (<300ms), or high
  - **Uptime** since the process started, and a relative **Started** timestamp that renders in each viewer's own timezone
  - **Gateway ping** (heartbeat latency) and **Response time** (a real round trip to Discord's API, measured while acknowledging the command)
  - **Reconnects** and a **Connected** timestamp, shown only once the gateway has actually dropped and come back — on a clean run they would just repeat the uptime
  - Server name and member count, plus the running discord.py version
  - A **Voice tracking** panel comparing how many members Discord reports in voice against how many timers are actually running. If the timers are behind, the field turns ⚠️ and says so — the quickest way to check that voice time is being counted, without stopping the bot
  - Available to everyone: it reports the bot's own health and exposes nothing an ordinary member cannot already see
  - Replaces the old admin-only `/ping`, which showed latency and a hardcoded "Online" and nothing more

- `/shutdown` - Stop the bot cleanly for an update or restart **[Admin only]**
  - Shows a confirmation first with the current uptime and how many voice/stream sessions are open, since this takes the bot **offline for everyone**
  - Only the administrator who ran the command can confirm it, and the permission is re-checked when the button is pressed, not just when the command is invoked
  - The prompt expires after 60 seconds and the **Cancel** button aborts it
  - On confirm the bot writes every open voice and stream timer to the database, then closes the connection — so the time accrued so far is kept, and members still sitting in voice are picked back up automatically on the next start
  - **It does not restart the bot.** The process exits cleanly (status 0); something else has to bring it back — a supervisor with `Restart=always` under systemd, a restart policy in Docker, or starting `python bot.py` again by hand. With `Restart=on-failure` it will **not** come back, because a clean shutdown is not a failure
  - Prefer this over `kill`/`kill -9` when updating: `kill -9` cannot be caught, so any in-flight voice sessions are lost

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

### Regiments Workflow

1. **View the list**:
   ```
   /regiments
   ```

2. **Add an entry** (mods/admins only):
   - Run `/regiments` and click **Add Regiment**
   - Complete the modal (game + name required)
   - Optionally upload a symbol image afterward

### Top VC Vanity Role Workflow

1. Under `/settings → General`, set **Top VC Role** (and optionally **Top VC Announcement Channel**)
2. Make sure the bot has **Manage Roles** and that the Top VC role sits below the bot's highest role
3. The role is recomputed at **00:00 UTC each Sunday** — setting it does not sync straight away, so use **Force Resync** if you want it applied now
4. Members accumulate voice time as they use voice channels — a session counts once they leave it
5. Anyone can check standings with `/vanity_vc`; administrators can hit **Force Resync** on that message to update on demand
6. When the top 3 changes, the role moves and (if configured) the announcement channel gets the new standings

### Referral Codes Workflow

1. **Get random codes**:
   ```
   /referral
   ```

2. **Submit or update your codes**:
   - On any `/referral` message, click **Add/Modify**
   - Enter codes for any supported games (all fields optional)
   - Run `/referral` again later for a different random pick

## Database Schema

The bot uses a **single SQLite database** (`umbrella.db`) for all data. Data is guild-scoped; the bot is designed to run in one Discord server at a time.

### Main tables in `umbrella.db`
- **wars**: War information (number, name, start/end dates, active status, shard, faction, associated Discord role, result)
- **user_stats**: User statistics per war
- **service_records**: Service record profiles (rank, level, entered service date)
- **settings**: Bot configuration for war tracking (leaderboard channel, screenshot processing, etc.)
- **roll_calls**: Roll call information (guild, channel, message, wars, timer, status)
- **roll_call_responses**: Roll call member responses (rollcall_id, user_id, status, timestamp)
- **regiment_rollcalls**: Regiment roll call messages (reactions assign Current War Verification role; no timer)
- **regiment_rollcall_responses**: Per-user responses to regiment roll calls (message_id, user_id, is_active, withdrawn)
- **user_warnings**: User warnings (guild, user, moderator, reason, timestamp)
- **membership_tracking**: Optional membership metrics (user_id, discord_join_date, regiment_role_assigned_date, vc_joins_count, vc_time_seconds, stream_starts_count, stream_time_seconds, messages_sent)
- **vc_sessions**: One row per completed voice session (user_id, user_nickname, seconds, ended_at) — powers the rolling 30-day **Top VC** ranking; rows older than the window are pruned automatically
- **vc_stream_sessions**: One row per completed **Go Live** stream session (same shape as `vc_sessions`) — pruned on the same schedule
- **task_state**: Internal key/value bookkeeping for scheduled tasks (currently just when the Top VC sync last completed, so a missed run can be caught up). Deliberately separate from `guild_settings`, which is the user-facing surface behind `/settings` and the web API
- **retained_roles**: Saved role snapshots for role retention (user_id, user_nickname, role_ids JSON, left_at). A `left_at` value marks a genuine departure — only those snapshots trigger the returning-member panel, and they expire after the configured window
- **events**: Event information (name, description, times, creator, status, required_attendees, voice_channel_id, war_id)
- **signups**: Event attendance (user_id, position, emoji)
- **guild_settings**: Server-specific settings (logging, moderation role, roll call, verification, rule channel, role retention expiry, Top VC role/announcement channel, etc.)
- **events_wars**: War tracking for events (separate from war tracker wars)
- **polls**, **poll_options**, **poll_votes**: Poll data
- **verification_tickets**: Verification ticket information (ticket_id, ticket_number, guild_id, user_id, channel_id, status, created_at, completed_at, closed_at, assigned_role_id, assigned_by_user_id, auto_close_scheduled)
- **api_keys**: API keys for this server (guild-scoped; used for API authentication)
- **vc_created_channels**: Temporary VC (Join-to-Create) ownership for the VCs that exist **right now** (guild_id, channel_id, owner_user_id, channel_name, created_at, peak_members) — a row leaves this table the moment its VC closes, so a live lookup never has to filter out dead channels
- **vc_created_participants**: Who has been in a live temporary VC (channel_id, user_id, user_nickname, first_joined_at) — collapses into the `unique_members` count when the VC is archived, and the rows are then deleted
- **vc_created_history**: One row per **closed** temporary VC (owner, name at close, created_at, closed_at, duration_seconds, peak_members, unique_members, closed_reason) — written before the channel is deleted, since Discord retains nothing about a deleted channel. `duration_seconds` is `NULL` for VCs that were already open when history tracking was added, which is honest where a `0` would read as a VC nobody ever used
- **regiments**: Community regiment/clan/org entries (game_name, regiment_name, symbol_url, join_link, created_by_id)
- **referral_codes**: Per-user referral codes (game_name, user_id, user_nickname, referral_code; unique per user per game)

### Migrating from legacy databases

If you have existing `foxhole_stats.db`, `events.db`, or `polls.db` from an older setup, use the script in `scripts/merge_legacy_databases.py` to merge them into a single `umbrella.db`. See `scripts/README.md` for usage.

## REST API

The bot includes a FastAPI-based REST API for external integrations.

### API Setup

1. **Add to `.env`**:
   ```env
   API_ENABLED=true                  # Set to false to disable API server
   API_KEY=your-secure-api-key-here
   API_HOST=0.0.0.0
   API_PORT=8080
   API_CORS_ORIGINS=http://localhost:3000
   ```

2. **API starts automatically** when the bot starts (if `API_ENABLED=true`):
   - Base URL: `http://localhost:8080`
   - Documentation: `http://localhost:8080/docs`
   - Health Check: `http://localhost:8080/health`
   - Set `API_ENABLED=false` to disable the API server entirely

### API Endpoints

All endpoints (except `/` and `/health`) require the `X-API-Key` header. API keys are **guild-scoped**: create them with `/api_key_create`; the key must belong to the guild you are requesting data for.

- **GET `/api/v1/events`** - Get all events for a guild
- **GET `/api/v1/events/{event_id}`** - Get a specific event
- **GET `/api/v1/leaderboard/{guild_id}`** - Get leaderboard data (lifetime and active war); optional query `?limit=10` (1–100) for top N per category. See `api/LEADERBOARD_API_DOCUMENTATION.md`.
- **GET `/api/v1/war-statistics/{guild_id}`** - Summary war stats (total/won/lost, active war, past wars) for website Stats/Home pages
- **GET `/api/v1/verify`** - Validate an API key and return pairing info for its guild
- **GET `/api/v1/guild/{guild_id}/metadata`** - Channels + roles for a guild (used for web panel dropdowns)
- **GET `/api/v1/status`** - Detailed runtime status (uptime, gateway reconnects, guild info)

Endpoints below also require an `Authorization: Bearer <access_token>` obtained from
`POST /api/v1/auth/token`. The tier shown is the minimum; it is re-checked against
Discord on every request.

**Auth** — `POST /api/v1/auth/token` (exchange a Discord OAuth code for a tiered token),
`POST /api/v1/auth/admin-token` (admin-only, legacy), `GET /api/v1/auth/me` (identity + live tier)

**Members** *(member)* — `GET /api/v1/members/me`, `GET /api/v1/members/{user_id}`,
`GET /api/v1/members?guild_id=` (paginated directory)

**Activity** *(member)* — `GET /api/v1/activity/{guild_id}/leaderboard` (voice / stream /
message metrics), `/summary`, `/rolling-vc`, `/custom-vc`

**Community** *(member)* — `GET /api/v1/polls/{guild_id}` and `/{poll_id}`,
`GET /api/v1/rollcalls/{guild_id}` and `/{rollcall_id}`, `/regiment/active`,
`GET /api/v1/regiments/{guild_id}`

**Events** — `POST /api/v1/events` *(member, rate-limited)*,
`PUT`/`DELETE /api/v1/events/{event_id}` *(creator or admin)*

**Moderation** *(mod)* — `GET /api/v1/moderation/{guild_id}/warnings`,
`GET /api/v1/moderation/{guild_id}/bans`,
`DELETE /api/v1/moderation/{guild_id}/warnings/{warning_id}`

**Admin** *(admin)* — `GET`/`POST /api/v1/settings/{guild_id}`,
`POST /api/v1/roles/assign`, `POST /api/v1/roles/remove`, `PUT /api/v1/roles/modify`,
and the full `/api/v1/wars/{guild_id}` CRUD plus `/start` and `/end`

> **Removed:** admin endpoints previously accepted a self-asserted `X-Discord-User-Id`
> header in place of a signed token, selected by `ADMIN_AUTH_MODE`. Any API-key holder
> could set that header and claim to be an administrator, so the path has been deleted
> and `ADMIN_AUTH_MODE` no longer exists.

See `api/README.md` for complete API documentation, request/response examples, tiers, and error handling.

## OAuth2 Integration

`helpers/sync_role.py` (token exchange, token refresh, role-sync functions) is used in two ways:

1. **Actively wired in**: the website's Discord login, which posts its authorization code to `POST /api/v1/auth/token` for the bot to redeem (`api/routes.py`), using only the `identify` scope.
2. **Reference only**: `helpers/oauth2_example.py` is an example of adapting the same helpers (with the broader `guilds`/`guilds.members.read` scopes) into a separate website/web framework for full role syncing. It is a starting point, not a runnable endpoint, and is not wired into the bot.

## Ticket Transcripts

When a verification ticket closes, the bot writes the whole conversation to its local
`transcripts/` folder — one JSON file per ticket, plus a sibling directory holding every
image that was posted. That file is the record; reading it is the website's job.

**The bot does not serve transcripts to browsers.** It used to: there was a second
Discord OAuth flow, a cookie session, and a route that handed out files from
`transcripts/`, optionally fronted by nginx. All of that is gone. Transcripts now leave
the bot only through three REST endpoints, using the same API key plus bearer token as
every other endpoint:

| Endpoint | Returns |
| --- | --- |
| `GET /api/v1/transcripts/{guild_id}` | Paginated ticket list, each marked `has_transcript` |
| `GET /api/v1/transcripts/{guild_id}/{ticket_id}` | One transcript: every message in order, with its attachments |
| `GET /api/v1/transcripts/{guild_id}/{ticket_id}/attachments/{path}` | One saved image, as bytes |

All three require **moderator access or better** — the same bar the old viewer applied,
since transcripts hold verification screenshots and whatever was said in the ticket.

Two things to know when building against these:

- **Message content is raw.** It is exactly what the member typed, unescaped. A
  transcript that rewrote its own contents would not be a transcript, so escaping is the
  renderer's job. The bot no longer produces HTML for a transcript at all.
- **Attachments need credentials**, so a page cannot load them from an `<img src>`
  directly. Proxy them from the site's own server. Each is served with `nosniff`, and
  only ever declares an image content type — anything else comes back as an opaque
  download.

### Setup

Nothing is required for the API side beyond a working API key and `ADMIN_TOKEN_SECRET`.
Two optional pieces:

1. **Set a Moderator role** via `/settings` if you haven't already. That is the role
   checked for transcript access (Administrators always have it).

2. **Add a "🌐 View Online" button** to the transcript embed the bot posts in Discord by
   pointing `TRANSCRIPT_BASE_URL` at the website's transcript page:

   ```env
   TRANSCRIPT_BASE_URL=https://yourdomain.com/admin/tickets
   ```

   The bot appends the ticket id, so ticket 1234 links to
   `https://yourdomain.com/admin/tickets/1234`. Leave it blank and the embed carries only
   the "📥 Download Transcript" button.

## Notes

### War Tracking
- Leaderboard updates every 30 minutes automatically (can be enabled/disabled in `/settings`), and immediately after a stat submission or edit
- The leaderboard channel holds exactly three messages in a fixed order: **Lifetime**, **Active War**, then the **info card**
- Leaderboards edit their existing messages rather than creating duplicates. Every post/update/reorder path (startup reconcile, the 30-minute loop, manual refreshes) shares one lock, and a transient Discord error skips that cycle instead of resending — a new message is only sent when the stored one is genuinely gone
- Ending a war rewrites its leaderboard as **🏁 Final Leaderboard - War N (Ended — Result)** and leaves it up; starting the next war deletes it. Any stale war leaderboards left in the channel are cleaned up on the next pass
- Leaderboards display users' guild display names (nicknames) when available
- Stats **replace** existing entries when resubmitted for the same war
- War stat voice channels automatically update every 5 minutes:
  - "Active War: {shard} {war_number}" - Shows current active war with shard
  - "Faction: {faction}" - Shows current active war faction
  - Channels are auto-created if they don't exist
  - Channels are set to private (no connect permission for default role)
  - Requires bot to have Manage Channels permission
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
- Polls are created via `/poll_create` command using modal interface
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

### Temporary VC (Join-to-Create)
- Place a voice channel named exactly **"Join to Create VC"** in the category set in `/settings` (Event Management → Join-to-Create VC Category)
- When a user joins that channel, the bot creates a new voice channel in the same category (or moves them to their existing one) and sends a "Customize your VC" embed
- Only the creator of the VC can use `/vcrename`, `/vclimit`, `/vclock`, `/vcunlock`, and the whitelist/blacklist commands
- When the temporary VC is empty, it is automatically deleted
- **The history record is written before the delete call** — the last moment the channel's name, age, and occupancy are still knowable. Discord keeps nothing about a deleted channel, so there is no second chance to write it
- All four ways a VC can end are recorded: it emptied, a moderator deleted it, or the bot was down when it went away — either gone entirely or left sitting empty, both settled by the `Custom VC reconcile` startup step. A VC that is still in use at startup is left alone and recorded when it finally empties
- That startup step also clears the stale ownership row those offline endings used to leave behind. The table allows one VC per owner, so a leftover row blocked that member from ever getting a new one
- Peak occupancy and the unique-member count are collected as people join, so both are already known by the time the VC closes
- Renaming with `/vcrename` is preserved: the record stores the name the channel had when it closed, not the one it was created with

### Vanity Roles (Top VC)
- Voice time comes from the `vc_sessions` log, which is written when a member **leaves** a voice channel — an in-progress session doesn't count until it ends
- The set is recomputed at **00:00 UTC on Sunday** and at no other scheduled time — there is deliberately no sync on startup and none when the role is configured; administrators can force a pass from `/vanity_vc`
- A **missed Sunday is caught up automatically**: the completion time of each sync is persisted, and the next daily tick after the bot returns spots that the scheduled Sunday went by unserved and runs it. Worst case the role is up to a day late instead of a week
- The catch-up fires at the next 00:00 UTC tick, not the moment the bot starts — startup deliberately does not sync. Use **Force Resync** if you need it applied sooner
- A sync that raises does **not** record a completion, so the next tick retries it. A sync that runs but finds no role configured *does* record — that is a configuration problem, not a missed run, and it already prints its own warning
- A bot that has never synced counts as overdue, so a fresh install picks the role up at the next daily tick rather than waiting up to a week for the first Sunday
- The bot needs **Manage Roles**, and the Top VC role must sit **below** the bot's highest role — otherwise the sync logs a warning and does nothing
- Role changes are attributed to "Top VC" in the role log rather than showing up as an unexplained audit-log entry
- If no announcement channel is set (or the bot can't post there), the sync still runs silently
- Members who leave the server drop out of the ranking, and the next member down takes their place

### Streaming vs. VC Tracking
- VC time and stream time run as **two independent timers at the same time**, not as alternatives: a member streaming for an hour accrues an hour of VC time *and* an hour of stream time. Starting or stopping a stream never touches the VC timer
- In-progress sessions are **checkpointed to the database every 30 minutes**, so an unclean stop — a crash, a `kill -9`, a power cut — costs at most one interval per member instead of the whole session. Nobody is disconnected or re-counted; only the accounting mark moves forward
- The timers store *the moment time was last accounted for*, not *the moment the member joined*, which is what makes a checkpoint exact: each write covers only the interval since the previous one, and the sub-second remainder is carried rather than shaved off
- Checkpointing splits one long session into several `vc_sessions` rows. The rolling 30-day ranking sums seconds per member, so the totals are identical — only the row count differs, and even 60 members permanently in voice produce ~100k rows across the whole 35-day retention window
- `CHECKPOINT_INTERVAL_MINUTES` in `helpers/membership_helpers.py` sets the interval, and is the ceiling on how much voice time a crash can cost
- Both timers are **in-memory and written on completion**. To stop a restart discarding them, the bot **writes out every open session on graceful shutdown** and **restarts the timers for everyone already in voice on startup** (`resume_voice_tracking` in `bot.py`), so a planned restart loses only the downtime itself
- No special shutdown procedure is needed. `Ctrl+C`, the **`/shutdown`** command, and a supervisor stop all flush cleanly. Only `Ctrl+C` unwinds by default, so `bot.py` installs handlers that make the stop signals behave the same way
- **Which signal arrives depends on the platform.** On Linux/macOS it is `SIGTERM` (`systemctl stop`, `docker stop`). On **Windows, SIGTERM is never delivered between processes** — a supervisor stopping a console app (AMP, NSSM, PM2) sends `CTRL_C_EVENT` (SIGINT) or `CTRL_BREAK_EVENT` (SIGBREAK), and SIGBREAK's default disposition kills without cleanup. Handlers are registered for both wherever they exist
- `/shutdown` calls `bot.close()` directly, and the flush still runs exactly once: discord.py's context manager awaits the already-running close instead of starting a second one
- Shutdown order is **save state → stop serving → close the connection**. A supervisor usually allows only a short grace period before killing the process, so the one irreplaceable step goes first; the API server never writes voice data, so stopping it earlier would buy nothing and risk the flush being cut off. Each step is isolated — a failure in one does not block the others
- The shutdown sequence logs unconditionally (`🛑 Shutting down`, `💾 Saved N…`, `🌐 API server stopped`, `👋 Shutdown complete`), including when there was nothing to save, so a stop that silently skipped the flush is visible in the console rather than indistinguishable from a clean one
- Every step of the sequence is guarded against `BaseException`, not just `Exception`. A stop signal makes `asyncio.run` cancel every task, so any `await` during shutdown can raise `CancelledError` — a `BaseException` that would otherwise escape and abandon the remaining steps partway through, ending the log mid-sequence
- The steps are independent: a failing flush, a wedged API server, or an error closing the gateway each log their own message and the sequence continues. The state flush runs first precisely so it is never the step that gets skipped
- The API server runs inside the bot's event loop as `_EmbeddedServer`, a `uvicorn.Server` subclass whose `capture_signals()` is a no-op. Stock uvicorn **replaces the process SIGINT/SIGTERM handlers** for the lifetime of `serve()`, which would hand shutdown to uvicorn instead of the bot and skip the session flush on `systemctl stop`
- A **SIGKILL** (`kill -9`, power loss, OOM killer) still bypasses everything, since nothing can catch it. The startup resume is the safety net there: the in-flight portion is lost, but the member keeps accruing from boot instead of losing the whole session
- The startup resume seeds timers only — it never bumps `vc_joins_count` or `stream_starts_count`, which would otherwise inflate on every restart
- It is also safe on a gateway reconnect: `on_ready` fires again in the same process, and the resume never restarts a timer that is already running
- The resume reads the guild's **voice-state cache** directly rather than `VoiceChannel.members`. That property resolves every voice state against the *member* cache and silently drops anyone it cannot resolve, so a guild that has not finished chunking reports empty voice channels even with people sitting in them
- Startup logs the result unconditionally (`🎙️ Voice tracking: N connected (N streaming) — resumed N voice, N stream timer(s)`), so a zero is visible rather than silent
- A background task banks in-progress time every 30 minutes, so voice time no longer depends on the bot stopping cleanly or the member disconnecting
- Leaving a voice channel always closes an open stream; moving between channels does not, since Discord keeps `self_stream` set across a move
- Redundant voice-state updates (a mute toggle while already streaming) do not start a second stream session
- Stream sessions older than the retention window are pruned by the same 00:00 UTC pass that prunes `vc_sessions`, which stays **daily** even though the role sync is weekly
- Stream totals surface in `/vc_stats`, `/vc_leaderboard`, and `/stream_leaderboard`, and are included in the `/database_export` CSV
- Because tracking started when the feature was added, stream totals begin at zero for everyone while VC totals carry full history — expect `% VC` to look artificially low until the log fills in

### Regiments, Clans & Orgs
- `/regiments` is available to everyone; only mods/admins see **Add Regiment**
- Moderator access uses the role set in `/settings` (Moderation), not Discord permission flags alone
- Symbol can be an emoji, image URL, or file uploaded after the modal
- Entries persist until removed manually (no self-serve delete command yet)

### Referral Codes
- Supported games are fixed in code: World of Tanks, World of Warships, Star Citizen, War Thunder
- `/referral` picks random codes with SQLite `ORDER BY RANDOM()` — each run may show different submitters
- **Add/Modify** works on any `/referral` message; each user can only change their own stored codes
- Empty modal submit (no fields filled) shows an ephemeral reminder to enter at least one code

### Moderation System
- Comprehensive moderation command suite (ban, kick, timeout, warn, purge)
- Warning system with persistent storage and tracking
- Automatic moderation action logging to designated channel
- Role hierarchy protection prevents moderating users with higher roles
- Moderator role-based access control (not permission-based)

### Service Records
- Service records track member service history based on assigned Discord roles
- Specializations and certifications are automatically detected from role assignments
- Service history is populated from war roles assigned during `/war_create`
- Rank insignia images are stored in `images/rank_insignia/` folder
- Total wars count is based on assigned war roles across all shards
- If no specializations or certifications are found, displays "No Current Specializations" or "No Current Certifications"

### Moderation
- Moderator role must be configured in `/settings` (Moderation Settings) by selecting a Discord role
- Users with the configured moderator role can use all moderation commands
- All moderation actions are logged if moderation logging is enabled
- Warning system tracks infractions with timestamps and reasons
- Role hierarchy is enforced (cannot moderate users with higher roles)
- Maximum timeout duration is 28 days (Discord limit)
- `/mod_userinfo` provides comprehensive user information including permissions, roles, and warning count

### Logging
- **Message Logging**: Automatically tracks message edits and deletions
  - Logs original content, edited content, author, channel, and timestamps
  - Shows before/after comparison for edits
- **Role Logging**: Automatically tracks role assignments and removals
  - Logs role changes with member, role, and moderator information (or source label)
  - **Accurate moderator attribution**: When the bot changes roles (verification ticket, API, or roll call), the log shows the correct source—the staff member who assigned the role in the ticket, **API** for API-driven changes, or **Roll call** for roll call reactions—instead of relying only on Discord’s audit log
  - For other role changes (e.g. staff using Discord directly), the bot uses the guild audit log (single query per member, up to 25 recent entries) to determine the moderator
  - When multiple roles are added or removed in one update, a single moderator is shown for the batch and a note explains: *"Multiple roles changed in one update; moderator may reflect the most recent change."*
  - Includes timestamps and action type (added/removed)
- **Moderation Logging**: Tracks all moderation actions (ban, kick, timeout, warn, purge)
  - Logs moderator, target user, reason, and duration (for timeouts)
  - Includes timestamps and action details
- **Voice Logging**: Automatically tracks voice channel joins, leaves, and moves
  - Logs member voice state changes with channel information
  - Tracks when members join, leave, or move between voice channels
- **Join/Leave Logging**: Automatically tracks member joins and leaves
  - Logs new member joins with account creation date
  - Logs member leaves with join date and time in server
- All logging features can be enabled/disabled and configured via `/settings` → **Logging**
- Each log type requires a designated channel to be set
- Logging handlers run automatically in the background

### Verification & Ticket System
- Verification embeds are posted in the configured verification embed channel (use `/send_verification_embed` to post/update)
- Users click "Open Verification Ticket" button to create a private ticket channel
- Each user can only have one open ticket at a time
- Ticket channels are created in the configured verification category
- Only users with Administrator permissions or the Recruiter Certification role can assign verification roles via ticket buttons
- The current war verification role is automatically assigned with verification roles (except for Non-Foxhole)
- Tickets automatically close 24 hours after a role is assigned (background task checks every hour)
- Manual ticket closure available via "Close Ticket" button in ticket channel
- Full transcripts are generated when tickets close (includes all messages, embeds, and attachments)
- Transcripts are saved to the `transcripts/` folder as JSON files, with posted images in a sibling directory per ticket, and are read by the website through the API (see [Ticket Transcripts](#ticket-transcripts))
- Transcripts can be sent to configured transcript channel if set, with "📥 Download Transcript" and (if `TRANSCRIPT_BASE_URL` is configured) "🌐 View Online" buttons — the latter links to the website, not to the bot
- Ticket channel names reflect status: `{ticket_number}-{username}-{status}` (open/completed/closed)
- Verification embed and ticket buttons persist across bot restarts and are automatically restored on startup
- The verification welcome message has a **built-in default** that is used whenever no custom message is set; `/settings` shows it (labelled *built-in default*) and pre-fills the edit modal with it
- Role retention: only snapshots with a departure stamp count as a "return". The returning-member panel previews the change and applies nothing until staff confirm, staff can restore a subset or everything, and the snapshot is cleared when the ticket closes
- Saved roles expire after the **Role Retention Expiry** window (`/settings → Moderation`, default 180 days, `0` = never); expired snapshots are reported instead of restored
- All verification roles (Diplomat, Regiment, Non-Foxhole, Charlie) are configurable via `/settings`
- Charlie role is configurable but not included in the verification ticket role buttons

### General
- Both systems run independently but share the same bot instance
- War Tracker wars and Event wars are separate systems
- Make sure users have DMs enabled for stat entry and event creation
- The bot ignores messages starting with `!` to avoid conflicts
- Bot status message can be customized via `/settings` and affects the bot's activity display

## Troubleshooting

- **OCR not working**: Ensure Tesseract is installed and screenshot processing is enabled in `/settings` (War Tracker settings)
- **DM not received**: Check that DMs are enabled from server members
- **Commands not appearing**: Use `/sync_commands` (Admin only)
- **Reactions don't work**: Check bot permissions (Add Reactions, Read Message History)
- **API not working**: Verify `API_ENABLED=true` in `.env`, `API_KEY` is set, and port is available
- **API trying to start when disabled**: Ensure `API_ENABLED=false` in `.env` file
- **Database errors**: Delete `umbrella.db` to reset (this deletes all data)
- **Bot not responding**: Verify bot token and permissions are correct
- **Moderation commands not working**: Ensure users have the moderator role assigned (not just permissions), and bot has required permissions (Ban Members, Kick Members, Moderate Members, Manage Messages)
- **Moderation logging not working**: Check that moderation logging is enabled in `/settings` and a moderation log channel is set
- **Temporary VC not creating**: Ensure the voice channel is named exactly **"Join to Create VC"**, it is in the category set in `/settings` (Event Management → Join-to-Create VC Category), and the bot has Manage Channels permission
- **Top VC role not being assigned**: Set the role under `/settings → General`, give the bot **Manage Roles**, and move the Top VC role **below** the bot's highest role. Voice time is only recorded when a session ends, so a brand-new install has no data until members leave voice
- **Top VC standings look empty or stale**: Use the **Force Resync** button on `/vanity_vc` (admins only) — the automatic pass only runs at 00:00 UTC on Sundays, or on the next daily tick if that Sunday was missed
- **Returning-member panel not showing in a ticket**: It only appears for members who actually left (a snapshot with `left_at`), when Role Retention is enabled, and when the snapshot is inside the expiry window

## Project Structure

```
├── bot.py                          # Main bot file
├── commands/
│   ├── admin_commands.py          # Admin commands (settings, export, sync, logging)
│   ├── stats_commands.py           # Stat entry/viewing commands
│   ├── war_commands.py             # War management commands
│   ├── leaderboard_commands.py    # Leaderboard commands
│   ├── event_commands.py           # All event management commands (consolidated)
│   ├── poll_commands.py            # Poll creation and management commands
│   ├── rollcall_commands.py        # Roll call management commands
│   ├── vc_commands.py               # Temporary VC commands (vcrename, vclimit, vclock, etc.)
│   ├── techprogress_commands.py    # Tech progress analysis commands
│   ├── mod_commands.py             # Moderation commands (ban, kick, timeout, warn, purge)
│   ├── verification_commands.py    # Verification channel role assignment
│   ├── ticket_commands.py          # Verification ticket system commands
│   ├── servicerecord_commands.py   # Service record commands
│   ├── api_commands.py             # API key management (create, list, revoke)
│   ├── regiment_commands.py        # Regiment/clan/org listing and add modal
│   ├── referral_commands.py        # Referral code random display and Add/Modify modal
│   ├── vanity_commands.py          # /vanity_vc standings + admin Force Resync button
│   ├── vc_stats_commands.py        # /vc_stats + the three Top 25 leaderboards
│   ├── status_commands.py          # /uptime + /shutdown (bot status and clean stop)
│   ├── all_commands_extension.py   # Extension entry point that loads all command modules
│   └── utils.py                    # Command utilities
├── helpers/
│   ├── database.py                 # Database functions (all systems)
│   ├── membership_helpers.py       # VC/stream/message tracking, 30-min checkpoints, resume + flush
│   ├── dm_handler.py               # War tracker DM handler
│   ├── events_dm_handler.py        # Events/Polls DM handler
│   ├── dm_conversations.py        # DM conversation state management
│   ├── dm_prompts.py               # DM prompt embeds and views
│   ├── event_helpers.py            # Event helper functions (embeds, reactions, sync)
│   ├── poll_helpers.py             # Poll helper functions (embeds, voting, expiration)
│   ├── rollcall_helpers.py         # Roll call helper functions (embeds, timers)
│   ├── vc_helpers.py               # Join-to-Create VC (create on join, empty-channel cleanup)
│   ├── stats_helpers.py            # Stat helper functions (embeds, leaderboards)
│   ├── war_helpers.py              # War helper functions (embeds)
│   ├── verification_helpers.py     # Verification helper functions (incl. the default welcome message)
│   ├── ticket_helpers.py           # Ticket system helper functions (creation, closure, transcripts)
│   ├── role_retention_helpers.py   # Saved-role snapshots, returning-member panel, selective restore
│   ├── turing_test_helpers.py      # Turing Test honeypot embed and catch counter
│   ├── vanity_helpers.py           # Top VC vanity role (rolling 30-day ranking, Sunday 00:00 UTC sync)
│   ├── vc_stats_helpers.py         # All-time voice/stream/message summary + leaderboard embeds
│   ├── autoresponder_helpers.py    # PSG gif auto-responder (word-boundary message trigger)
│   ├── uptime_helpers.py           # Process uptime + gateway connection tracking for /uptime
│   ├── servicerecord_helpers.py    # Service record helper functions
│   ├── moderation_helpers.py       # Moderation helper functions (role checks, regiment permissions)
│   ├── regiment_helpers.py         # Regiment embed building and URL helpers
│   ├── referral_helpers.py         # Referral game list and random embed helpers
│   ├── logging_helpers.py          # Logging helpers (role-change actor cache, moderation, role/message/voice logs)
│   ├── reactions.py                # Raw reaction event handlers
│   ├── discord_event_sync.py       # Discord scheduled event sync
│   ├── screenshot_processor.py    # OCR processing for stats
│   ├── leaderboard_updater.py      # Leaderboard posting/reordering, final-war handling, instant refresh
│   ├── war_channel_updater.py      # Auto war stat voice channel updates
│   ├── embeds.py                    # Shared embed helper functions
│   ├── startup.py                  # Startup tasks (restore reactions, views, timers)
│   ├── models.py                    # Data models
│   ├── api_helpers.py              # API helpers
│   ├── settings_schema.py          # Shared settings keys/constants used by API and persistence layers
│   ├── settings_ui.py              # /settings render helpers and action maps
│   ├── sync_role.py                # OAuth2 role-sync helpers (reference implementation, not wired into the bot)
│   ├── oauth2_example.py           # Example website route handlers using sync_role.py
│   └── config.py                   # Configuration
├── handlers/
│   └── logging.py                  # Message and role change logging handlers
├── images/
│   ├── psg.gif                     # Sent by the PSG auto-responder (add this file yourself)
│   └── rank_insignia/              # Rank insignia images for service records
├── transcripts/                    # Ticket transcripts (auto-generated JSON + saved images)
├── api/                            # REST API
│   ├── server.py
│   ├── routes.py
│   ├── auth.py                     # API key authentication
│   ├── admin_token.py              # Admin token issuing/verification
│   ├── routes_transcripts.py       # Ticket transcript listing, bodies and attachments
│   ├── bot_instance.py             # Shared bot handle for API routes
│   ├── README.md
│   ├── ADMIN_AUTH_DESIGN.md
│   └── LEADERBOARD_API_DOCUMENTATION.md
├── scripts/                        # Maintenance and migration scripts
│   ├── merge_legacy_databases.py   # Merge legacy war/events/polls DBs into umbrella.db
│   └── README.md
├── tests/
│   ├── test_settings_contract.py   # Settings schema/contract tests
│   ├── test_admin_auth.py          # Admin token/auth tests
│   └── test_transcripts_api.py     # Transcript endpoint tier gate and path confinement
├── requirements.txt
└── umbrella.db                     # Single SQLite database (created on first run)
```

## License

This project is provided as-is for personal use.
