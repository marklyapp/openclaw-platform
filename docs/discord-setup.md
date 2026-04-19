# Discord Bot Setup

Connect OpenClaw to a Discord server so agents can receive messages and post updates.

## 1. Create a Discord Application

1. Go to the [Discord Developer Portal](https://discord.com/developers/applications)
2. Click **New Application** — name it whatever you want (e.g., "OpenClaw Bot")
3. Go to the **Bot** tab
4. Click **Reset Token** and copy it — this is your bot token
5. Under **Privileged Gateway Intents**, enable:
   - **Message Content Intent** (required — OpenClaw reads message text)

## 2. Invite the Bot to Your Server

1. Go to **OAuth2 > URL Generator**
2. Scopes: select `bot`
3. Bot permissions: select these:
   - Send Messages
   - Read Messages / View Channels
   - Add Reactions
   - Read Message History
4. Copy the generated URL and open it in your browser
5. Select your server and authorize

## 3. Get Your IDs

Enable Developer Mode: **Discord Settings > Advanced > Developer Mode**

| What | How | Used as |
|------|-----|---------|
| Server (guild) ID | Right-click server name > Copy Server ID | `channels.discord.guilds.<ID>` |
| Channel ID | Right-click channel > Copy Channel ID | Delivery targets, env vars |
| Your user ID | Right-click your username > Copy User ID | `OWNER_MENTION` for @mentions |

## 4. Configure openclaw.json

Add the following to your `openclaw.json`:

```json
{
  "channels": {
    "discord": {
      "enabled": true,
      "token": "YOUR_BOT_TOKEN",
      "streaming": {
        "mode": "block"
      },
      "groupPolicy": "open",
      "ackReaction": "eyes",
      "ackReactionScope": "all",
      "guilds": {
        "YOUR_GUILD_ID": {
          "requireMention": false
        }
      }
    }
  }
}
```

Set `requireMention: true` if you want the bot to only respond when @mentioned. Set to `false` for it to respond to every message in the channel.

## 5. Route Messages to an Agent

Add a binding so Discord messages go to a specific agent:

```json
{
  "bindings": [
    {
      "type": "route",
      "agentId": "main",
      "match": {
        "channel": "discord"
      }
    }
  ]
}
```

## 6. Pass the Token to Sandboxed Agents (Optional)

If agents inside Docker need to post to Discord (e.g., via the Discord API), pass the token as an env var:

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "docker": {
          "env": {
            "DISCORD_BOT_TOKEN": "YOUR_BOT_TOKEN"
          }
        }
      }
    }
  }
}
```

## Tips

- **One channel per workflow** — keeps context clean and avoids cross-talk
- Use `"actions": { "messages": false }` to suppress the bot reacting to its own messages
- Use `"streaming": { "mode": "block" }` for clean single-message responses instead of streaming edits
- The `ackReaction` emoji (e.g., "eyes" = `eyes`) appears when the bot starts processing a message
