# Arcade Duels — Bot specification

**Archetype:** custom

**Voice:** playful and encouraging — write every user-facing message, button label, error, and empty state in this voice.

Arcade Duels is a Telegram bot offering casual arcade mini-games with tap/reaction challenges, supporting both solo and real-time multiplayer matches. Users can choose between persistent profiles with stats and leaderboards or session-only play. Multiplayer matches are coordinated through a public lobby and quick-match system, with notifications sent to a configured Telegram group/channel.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Casual mobile Telegram users who enjoy short reaction/tap games
- Players wanting quick head-to-head duels or solo practice sessions
- Small communities or channels that want to host multiplayer activity and visibility in a group

## Success criteria

- Users can complete a solo game session with score tracking
- Users can join a multiplayer match through the lobby or quick-match
- Match notifications are delivered to both private chats and the configured group/channel
- Leaderboard updates are visible for users with persistent profiles

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu and onboarding flow
- **Play** (button, actor: user, callback: game:select) — Navigate to game selection screen
- **Join Lobby** (button, actor: user, callback: lobby:join) — Enter the public multiplayer lobby
- **Quick Match** (button, actor: user, callback: match:quick) — Automatically find a compatible opponent for a match
- **Rematch** (button, actor: user, callback: match:rematch) — Request a rematch with the same opponent
- **Share Score** (button, actor: user, callback: score:share) — Share match score with others

## Flows

### Onboarding
_Trigger:_ /start

1. Display welcome message
2. Explain game options
3. Prompt for persistent profile choice (Yes/No)

_Data touched:_ User

### Game Selection
_Trigger:_ game:select

1. Display solo vs multiplayer options
2. Show available mini-games (initially Reaction Tap)

### Solo Game
_Trigger:_ game:solo

1. Start countdown
2. Run reaction challenge
3. Display score and results
4. Save score if persistent profile enabled

_Data touched:_ Session, Match, Leaderboard entry

### Multiplayer Lobby
_Trigger:_ lobby:join

1. Display list of waiting players
2. Show invite/challenge/quick-match options

_Data touched:_ Lobby entry

### Quick Match
_Trigger:_ match:quick

1. Find compatible opponent
2. Notify both players privately
3. Announce match in group/channel

_Data touched:_ Match, Lobby entry

### Direct Match
_Trigger:_ match:challenge

1. Confirm match request
2. Start synchronized countdown
3. Run real-time match
4. Display results to both players
5. Post results to group/channel

_Data touched:_ Match, Leaderboard entry

### Match Notification
_Trigger:_ match:notification

1. Send private chat notification with accept/decline buttons
2. Forward match info to group/channel

_Data touched:_ Match

### Post-Match Actions
_Trigger:_ match:end

1. Display results
2. Offer rematch and share options

_Data touched:_ Match, Leaderboard entry

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram account linked to a profile when user opts into persistence
  - fields: Telegram ID, Display name, Avatar, Persistent stats (wins, best score, streak)
- **Session** _(retention: session)_ — Single play instance (solo or multiplayer) with persistence choice
  - fields: Session ID, User ID (if persistent), Game type, Start time, Persistence mode
- **Lobby entry** _(retention: session)_ — Record of players waiting for opponents
  - fields: Player ID, Display name, Match type, Visibility setting, Timestamp
- **Match** _(retention: session)_ — Active real-time game between two players
  - fields: Match ID, Player 1 ID, Player 2 ID, Game type, Start time, Scores, Result, Notification status
- **Leaderboard entry** _(retention: persistent)_ — Aggregated stats for persistent users
  - fields: User ID, Total games, Wins, Best score, Current streak, Weekly stats

## Integrations

- **Telegram** (required) — Bot API messaging for all interactions
- **Telegram Group/Channel** (required) — Notification forwarding for lobby and match events
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure the Telegram group/channel for notifications
- View and manage active lobby entries
- Access global and weekly leaderboards
- Monitor active matches and session data

## Notifications

- Lobby join/leave announcements
- Quick-match found notifications
- Match start/end announcements with player mentions
- Leaderboard updates for persistent users

## Permissions & privacy

- Users must opt-in to persistent profile storage
- Private chat interactions remain confidential
- Group/channel notifications only show basic match info (who vs who, game type)
- No external data collection beyond Telegram account ID and opt-in profile data

## Edge cases

- Player disconnects during a match
- Quick-match timeout when no compatible opponents found
- User declines an active match invitation
- Multiple simultaneous match requests to the same player
- Group/channel notification fails to post

## Required tests

- Verify solo game completes with score tracking
- Confirm multiplayer match starts through lobby and quick-match
- Validate notifications appear in both private chats and group/channel
- Test persistent profile data retention across sessions
- Simulate player disconnection during match

## Assumptions

- Persistent-profile opt-in is decided at session start
- Initial mini-game is Reaction Tap
- Lobby shows minimal player info
- Matchmaking supports both manual and automatic pairing
- Notifications go to both private and group/channel
- No payments at launch
- Group/channel is set by owner at setup
