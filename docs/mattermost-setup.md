# Mattermost Setup

Connect OpenClaw to a self-hosted Mattermost instance. Useful when you want full control over your chat infrastructure.

## 1. Deploy Mattermost

If you don't have a Mattermost instance yet:

```bash
# Clone the official Docker setup
git clone https://github.com/mattermost/docker ~/mattermost-docker
cd ~/mattermost-docker

# Copy and edit the env file
cp env.example .env
# Edit .env — set DOMAIN, database credentials, etc.

# Start
docker compose up -d
```

Default URL: `http://localhost:8065`

## 2. Create a Bot Account

1. Log in to Mattermost as a System Admin
2. Go to **System Console > Integrations > Bot Accounts**
3. Enable **bot account creation**
4. Go to **Integrations > Bot Accounts > Add Bot Account**
5. Fill in:
   - Username: `openclaw` (or whatever you prefer)
   - Display Name: OpenClaw Bot
   - Role: System Admin (needed to post to channels)
6. Click **Create** and copy the **Access Token** — this is your bot token

## 3. Get Your Channel ID

1. Open the target channel in Mattermost
2. Click the channel name in the header > **View Info**
3. The channel ID is shown in the info panel, or in the URL: `.../<team>/channels/<channel-id>`

## 4. Configure openclaw.json

```json
{
  "channels": {
    "mattermost": {
      "enabled": true,
      "botToken": "YOUR_BOT_TOKEN",
      "baseUrl": "http://localhost:8065",
      "dmPolicy": "pairing",
      "groupPolicy": "open",
      "replyToMode": "off",
      "network": {
        "dangerouslyAllowPrivateNetwork": true
      },
      "groups": {
        "YOUR_CHANNEL_ID": {
          "requireMention": false
        }
      }
    }
  }
}
```

### Network Note

If Mattermost runs on the same machine as the OpenClaw gateway (localhost), you need `dangerouslyAllowPrivateNetwork: true`. Without it, OpenClaw blocks connections to private/local IPs.

If Mattermost runs on a different server with a public or Tailscale IP, you can remove this flag.

## 5. Route Messages to an Agent

```json
{
  "bindings": [
    {
      "type": "route",
      "agentId": "main",
      "match": {
        "channel": "mattermost"
      }
    }
  ]
}
```

## 6. Enable the Plugin

Make sure the Mattermost plugin is enabled:

```json
{
  "plugins": {
    "entries": {
      "mattermost": {
        "enabled": true
      }
    }
  }
}
```

## Tips

- **Self-hosted advantage:** Full control over data, no rate limits, works on air-gapped networks
- Use **separate channels** for separate workflows (one channel per pod)
- Mattermost and Discord can run simultaneously — both route to the same agents via bindings
- The `replyToMode: "off"` setting posts flat messages instead of threaded replies
