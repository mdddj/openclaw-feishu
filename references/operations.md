# OpenClaw Feishu Operations

Source: https://docs.openclaw.ai/zh-CN/channels/feishu
Verified against the live docs on 2026-04-25.

## Contents

- Access control
- Group policy patterns
- Finding IDs
- Troubleshooting
- Advanced configuration
- ACP and routing
- Message and Drive capabilities

## Access Control

### Direct messages

- Default: `dmPolicy: "pairing"`
- Approve pending pairings:

```bash
openclaw pairing list feishu
openclaw pairing approve feishu <CODE>
```

- `dmPolicy` behaviors:
  - `"pairing"`: unknown users receive a pairing code and must be approved
  - `"allowlist"`: only users in `allowFrom` may chat
  - `"open"`: all users may chat, but the docs require `allowFrom` to include `"*"`
  - `"disabled"`: disable direct messages

### Group messages

- `groupPolicy: "open"`: allow all groups
- `groupPolicy: "allowlist"`: only allow groups in `groupAllowFrom`
- `groupPolicy: "disabled"`: disable group messages

`requireMention` defaults are conditional:

- explicit `true`: must mention the bot
- explicit `false`: no mention required
- unset with `groupPolicy: "open"`: defaults to `false`
- unset with any non-open group policy: defaults to `true`

Per-group `channels.feishu.groups.<chat_id>.requireMention` overrides the channel-level default.

## Group Policy Patterns

### Allow all groups without mention

```json
{
  "channels": {
    "feishu": {
      "groupPolicy": "open"
    }
  }
}
```

### Allow all groups but still require mention

```json
{
  "channels": {
    "feishu": {
      "groupPolicy": "open",
      "requireMention": true
    }
  }
}
```

### Allow only specific groups

```json
{
  "channels": {
    "feishu": {
      "groupPolicy": "allowlist",
      "groupAllowFrom": ["oc_xxx", "oc_yyy"]
    }
  }
}
```

### Restrict which senders may talk inside an allowed group

```json
{
  "channels": {
    "feishu": {
      "groupPolicy": "allowlist",
      "groupAllowFrom": ["oc_xxx"],
      "groups": {
        "oc_xxx": {
          "allowFrom": ["ou_user1", "ou_user2"]
        }
      }
    }
  }
}
```

The docs are explicit that `groups.<chat_id>.allowFrom` gates all messages from senders in that group, not only control commands.

## Finding IDs

### Group `chat_id`

- Format: `oc_xxx`
- Recommended path:
  1. Start Gateway
  2. Mention the bot in the target group
  3. Run:

```bash
openclaw logs --follow
```

and capture `chat_id`

### User `open_id`

- Format: `ou_xxx`
- Recommended path:
  1. Start Gateway
  2. Send the bot a direct message
  3. Run:

```bash
openclaw logs --follow
```

and capture `open_id`

- You can also inspect pairing requests:

```bash
openclaw pairing list feishu
```

## Troubleshooting

### Bot does not reply in a group

Check, in order:

1. The bot was added to the group
2. The user mentioned the bot when mention is required
3. `groupPolicy` is not `"disabled"`
4. Sender-level allowlists are not blocking the user
5. Logs:

```bash
openclaw logs --follow
```

### Bot does not receive any messages

Check, in order:

1. The app was published and approved
2. Event subscription includes `im.message.receive_v1`
3. Long connection is enabled
4. Required permissions were granted
5. Gateway is healthy:

```bash
openclaw gateway status
```

6. Logs:

```bash
openclaw logs --follow
```

### Message send failure

Check:

1. `im:message:send_as_bot` permission
2. App publish state
3. Detailed errors in logs

### Secret leak

Rotate in the documented order:

1. Reset `App Secret` in the Feishu Open Platform
2. Update the local OpenClaw config
3. Restart Gateway

## Advanced Configuration

### Multi-account

Use `defaultAccount` when more than one bot account exists:

```json
{
  "channels": {
    "feishu": {
      "defaultAccount": "main",
      "accounts": {
        "main": {
          "appId": "cli_xxx",
          "appSecret": "xxx",
          "name": "Primary bot"
        },
        "backup": {
          "appId": "cli_yyy",
          "appSecret": "yyy",
          "name": "Backup bot",
          "enabled": false
        }
      }
    }
  }
}
```

### Message limits

- `textChunkLimit`: default `2000`
- `mediaMaxMb`: default `30`

### Streaming

Feishu streaming uses interactive cards.

```json
{
  "channels": {
    "feishu": {
      "streaming": true,
      "blockStreaming": true
    }
  }
}
```

Set `streaming: false` to wait for the full reply before sending.

## ACP and Routing

### Documented ACP scope

The docs only state ACP support for:

- direct messages
- topic conversations inside groups

The docs explicitly say v1 does not support ordinary non-topic group chats for `/acp spawn ... --thread here`.

### In-chat ACP spawn

```text
/acp spawn codex --thread here
```

Use this only in direct chats or Feishu topic threads.

### Persistent ACP binding

Use typed ACP bindings when the user wants a stable direct-chat or topic-thread mapping:

```json
{
  "bindings": [
    {
      "type": "acp",
      "agentId": "codex",
      "match": {
        "channel": "feishu",
        "accountId": "default",
        "peer": { "kind": "direct", "id": "ou_1234567890" }
      }
    },
    {
      "type": "acp",
      "agentId": "codex",
      "match": {
        "channel": "feishu",
        "accountId": "default",
        "peer": { "kind": "group", "id": "oc_group_chat:topic:om_topic_root" }
      },
      "acp": { "label": "codex-feishu-topic" }
    }
  ]
}
```

### Multi-agent routing

Bindings may route different peers to different agents. The relevant match fields are:

- `match.channel`: `"feishu"`
- `match.peer.kind`: `"direct"` or `"group"`
- `match.peer.id`: user `open_id` or group `chat_id`

## Message and Drive Capabilities

### Supported message types

Receive:

- text
- rich text (`post`)
- image
- file
- audio
- video or media
- sticker

Send:

- text
- image
- file
- audio
- video or media
- interactive card
- limited rich text support for post-style formatting and cards

Threading:

- inline replies are supported
- topic-thread replies are supported when Feishu exposes `reply_in_thread`
- media replies remain thread-aware when replying in threads or topics

### Drive comments

Drive comment workflows require:

- `drive.notice.comment_add_v1` in event subscription
- `channels.feishu.tools.drive` left enabled, unless the user intentionally disables it

Documented `feishu_drive` comment actions:

- `list_comments`
- `list_comment_replies`
- `add_comment`
- `reply_comment`

When a Drive comment event is handled, the docs say the agent receives:

- comment text and sender
- document metadata
- thread context for replies

After editing the document and replying through `feishu_drive.reply_comment`, the documented pattern is to end with the exact silent marker `NO_REPLY` or `no_reply` to avoid duplicate chat replies.
