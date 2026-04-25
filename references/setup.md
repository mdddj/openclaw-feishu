# OpenClaw Feishu Setup

Source: https://docs.openclaw.ai/zh-CN/channels/feishu
Verified against the live docs on 2026-04-25.

## Contents

- Built-in plugin and recommended entry points
- Feishu app creation checklist
- Required permissions
- Event subscription
- OpenClaw configuration
- Test flow

## Built-in Plugin and Recommended Entry Points

- The Feishu channel is built into current OpenClaw releases.
- If the user is on an older or custom build without the built-in plugin, install it with:

```bash
openclaw plugins install @openclaw/feishu
```

- Prefer:

```bash
openclaw onboard
```

for first-time OpenClaw setup.

- Prefer:

```bash
openclaw channels add
```

for adding Feishu to an existing install.

## Feishu App Creation Checklist

1. Open the Feishu Open Platform.
   Feishu CN tenants: `https://open.feishu.cn`
   Lark global tenants: `https://open.larksuite.com/app`

2. Create an enterprise app.

3. Copy credentials from `Credentials & Basic Info`.
   - `App ID` format: `cli_xxx`
   - `App Secret`

4. Enable bot capability in `App Capability > Bot`.

5. Configure permissions with batch import.

6. Configure event subscription.

7. Publish the app in `Version Management & Release`.

## Required Permissions

Use the permission batch-import block from the docs:

```json
{
  "scopes": {
    "tenant": [
      "aily:file:read",
      "aily:file:write",
      "application:application.app_message_stats.overview:readonly",
      "application:application:self_manage",
      "application:bot.menu:write",
      "cardkit:card:read",
      "cardkit:card:write",
      "contact:user.employee_id:readonly",
      "corehr:file:download",
      "event:ip_list",
      "im:chat.access_event.bot_p2p_chat:read",
      "im:chat.members:bot_access",
      "im:message",
      "im:message.group_at_msg:readonly",
      "im:message.p2p_msg:readonly",
      "im:message:readonly",
      "im:message:send_as_bot",
      "im:resource"
    ],
    "user": [
      "aily:file:read",
      "aily:file:write",
      "im:chat.access_event.bot_p2p_chat:read"
    ]
  }
}
```

If the user reports send failures, missing `im:message:send_as_bot` is one of the first checks.

## Event Subscription

Before configuring events, the docs require both of these to already be true:

- `openclaw channels add` has been run for Feishu
- Gateway is running and healthy:

```bash
openclaw gateway status
```

In Feishu `Event Subscription`:

- Choose `Use long connection to receive events` for the default WebSocket path.
- Add `im.message.receive_v1`.
- Add `drive.notice.comment_add_v1` only if the user wants Drive comment workflows.

Important documented caveat:

- If Gateway is not running, the long-connection setting may fail to save.

## OpenClaw Configuration

### Minimal file-based config

Edit `~/.openclaw/openclaw.json`:

```json
{
  "channels": {
    "feishu": {
      "enabled": true,
      "dmPolicy": "pairing",
      "accounts": {
        "main": {
          "appId": "cli_xxx",
          "appSecret": "xxx",
          "name": "My AI assistant"
        }
      }
    }
  }
}
```

### Environment variables

```bash
export FEISHU_APP_ID="cli_xxx"
export FEISHU_APP_SECRET="xxx"
```

### Webhook mode

WebSocket is the documented default. Only use webhook mode if the user explicitly needs it.

When `connectionMode: "webhook"` is used, also set:

- `channels.feishu.verificationToken`
- `channels.feishu.encryptKey`

The docs say Feishu webhook server binding defaults to `127.0.0.1`; only change `webhookHost` when a different bind address is required.

### Lark domain

For international tenants, set `domain: "lark"` at the channel or account level:

```json
{
  "channels": {
    "feishu": {
      "domain": "lark",
      "accounts": {
        "main": {
          "appId": "cli_xxx",
          "appSecret": "xxx"
        }
      }
    }
  }
}
```

### Quota optimization flags

Use these only when the user needs to reduce Feishu API usage:

- `typingIndicator: false`
- `resolveSenderNames: false`

Both can be set at the channel level or per account.

## Test Flow

1. Start Gateway:

```bash
openclaw gateway
```

2. Send a test message to the bot in Feishu.

3. If `dmPolicy` is `pairing`, approve the returned code:

```bash
openclaw pairing approve feishu <CODE>
```

4. Use these checks during rollout:

```bash
openclaw gateway status
openclaw logs --follow
```
