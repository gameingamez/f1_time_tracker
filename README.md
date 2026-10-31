# F1 25 Discord Lap-Time Listener

This Python program listens for F1 25 UDP telemetry on port `6767`. When the
player's lap number advances and the packet contains a completed lap time, it
posts the track name and time to a Discord channel using a channel webhook or
an official Discord bot. Posts include the player name from telemetry and
whether the lap is the fastest valid lap observed by this listener for the
current session; it does not compare against the game's historical personal
best.

It uses Python's standard library only.

## Configure F1 25

In the game's telemetry settings:

- Enable UDP telemetry.
- Set the UDP address to the listener PC's network address. For example, if
  that PC is `192.168.1.160`, use `192.168.1.160`. Use `127.0.0.1` only if
  the game and listener are on the same PC and you prefer loopback.
- Set the UDP port to `6767`.
- Select the F1 25 packet format.

## Option A: Discord bot (recommended if webhooks are blocked)

1. In the [Discord Developer Portal](https://discord.com/developers/applications),
   create an application, open its **Bot** page, and create/reset the bot token.
   Keep the token secret; never paste it into chat or publish it.
2. Invite the bot to your server using OAuth2 URL Generator with the `bot`
   scope and only the **View Channel** and **Send Messages** permissions.
   A server admin must authorize the invite. The bot must be allowed to see the
   target channel.
3. Enable Developer Mode in Discord, right-click the destination channel, and
   choose **Copy Channel ID**.
4. In `config.json`, set `discord_mode` to `"bot"`, replace
   `PASTE_TARGET_CHANNEL_ID_HERE` with that channel ID, and replace
   `PASTE_BOT_TOKEN_HERE` in `discord_bot_token` with the bot token.
5. Start the listener:

   ```powershell
   python f1_discord_laptime.py
   ```

The bot token is a password. Do not share `config.json`, upload it, or commit it
to version control. If you prefer not to store the token in the file, leave the
placeholder and set `DISCORD_BOT_TOKEN` in PowerShell before starting.

## Option B: Channel webhook

In Discord, open the target channel's **Edit Channel → Integrations → Webhooks**
settings, create a webhook, and copy its URL. You need permission to manage
webhooks in that server/channel. In `config.json`, set `discord_mode` to
`"webhook"` and paste the URL into `discord_webhook_url`:

```json
{
  "discord_mode": "webhook",
  "discord_webhook_url": "https://discord.com/api/webhooks/...",
  "telemetry_host": "0.0.0.0",
  "telemetry_port": 6767
}
```

For bot mode, use `"discord_mode": "bot"`, add
`"discord_channel_id": "your-channel-id"`, and set `"discord_bot_token"` to
your bot token (or leave the placeholder and use the environment variable).

`telemetry_host` and `telemetry_port` are the local address and port the
listener binds to. Set `telemetry_host` to the same network address configured
as F1 25's UDP destination. Use `0.0.0.0` to listen on all network interfaces.
The game's UDP destination port and `telemetry_port` must both be `6767`.

Treat the webhook URL like a password: anyone with it can post to its channel.
Do not share it or publish `config.json`. If it is exposed, delete or regenerate
the webhook in Discord.

## Run

From this folder:

```powershell
python f1_discord_laptime.py
```

Stop it with `Ctrl+C`. The listener logs invalid packets and reports webhook
delivery failures rather than silently treating them as successful.
On startup, it logs the first received packet of each type. If it only prints
the listening line and never reports received packets while the game is
sending telemetry, check the destination IP and Windows Firewall UDP access.

## Test

```powershell
python -m unittest -v
```
