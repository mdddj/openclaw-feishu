---
name: openclaw-feishu
description: >
  Configure, audit, and troubleshoot the OpenClaw Feishu or Lark channel from
  the official OpenClaw docs. Use when the user asks to connect OpenClaw to
  Feishu, create or configure a Feishu bot app, edit `channels.feishu` in
  `~/.openclaw/openclaw.json`, switch between WebSocket and webhook delivery,
  manage DM pairing or group allowlists, enable ACP or Drive comment
  workflows, or debug a Feishu bot that is not receiving or sending messages.
  Triggers: "配置 OpenClaw 飞书", "接入 Feishu/Lark", "飞书机器人不回复",
  "pairing 配对", "群聊 allowlist", "Drive 评论", "ACP 会话".
---

# OpenClaw Feishu

## Overview

Use the official OpenClaw Feishu channel documentation to guide setup, configuration, access control, advanced routing, and troubleshooting. Keep `SKILL.md` focused on execution and load references only for the part of the workflow that the user actually needs.

## Workflow

1. Classify the request before giving instructions.

- New integration or first-time setup: read [references/setup.md](references/setup.md).
- Existing bot behavior, access control, ACP, Drive comments, or troubleshooting: read [references/operations.md](references/operations.md).
- If the user asks for the latest exact behavior or a field that may have changed, re-open the official docs page before answering.

2. Default to the documented happy path unless the user clearly wants a variant.

- Prefer `openclaw onboard` for brand-new installs.
- Prefer `openclaw channels add` for an existing OpenClaw install.
- Prefer WebSocket event delivery over webhook unless the user explicitly needs webhook.
- Default `dmPolicy` to `pairing` and `groupPolicy` to `allowlist` when showing a minimal config.

3. Walk the user through the Feishu-side setup in the documented order.

- Confirm whether the tenant is Feishu or Lark.
- Have the user create an enterprise app, copy `App ID` and `App Secret`, enable bot capability, batch-import the documented permissions, configure event subscription, and publish the app.
- Call out two non-obvious constraints from the docs:
  - Long-connection event subscription is the default path and does not require a public webhook URL.
  - Gateway should already be running before Feishu long-connection settings are saved, or the setting may fail to persist.

4. Apply OpenClaw configuration with exact snippets instead of vague prose.

- Use `~/.openclaw/openclaw.json` for manual configuration.
- Only mention `verificationToken` and `encryptKey` when `connectionMode: "webhook"`.
- Only mention `webhookHost` changes when the user explicitly needs a non-default bind address.
- Use `domain: "lark"` for international tenants.
- Preserve unrelated existing config keys if the user shows a larger config file.

5. Verify and test in the documented order.

- Check or start Gateway with `openclaw gateway status`, `openclaw gateway`, and `openclaw logs --follow`.
- Ask the user to send a test message after the app is published.
- If direct messages use pairing, approve with `openclaw pairing approve feishu <CODE>`.

6. Use the documented control model for permissions and routing.

- For private chats, choose between `pairing`, `allowlist`, `open`, and `disabled`.
- For groups, reason through `groupPolicy`, `groupAllowFrom`, `requireMention`, and optional per-group sender `allowFrom`.
- Use log-based discovery for `chat_id` and `open_id` unless the user already has them.
- For ACP and multi-agent routing, only promise support for direct chats and topic threads that the docs explicitly mention.

7. Troubleshoot from the outside in.

- If the bot does not receive messages, check publish status, event subscription, long connection, permissions, and Gateway status before suggesting deeper debugging.
- If the bot ignores group messages, check whether it was added to the group, whether mention is required, and whether `groupPolicy` or sender allowlists block the message.
- If sending fails, check `im:message:send_as_bot`, publish status, and logs.
- If secrets leaked, rotate the secret in Feishu, update config, and restart Gateway.

## Response Rules

- Give concrete commands and config fragments, not abstract summaries.
- Distinguish Feishu and Lark when domain selection matters.
- Do not invent undocumented permission names, config keys, or unsupported ACP scopes.
- Do not tell the user to expose a public webhook URL unless they explicitly choose webhook mode.
- Mention the official source URL when handing over a final solution or checklist.

## Resources

- [references/setup.md](references/setup.md): first-time setup, required permissions, event subscription, config examples, test flow
- [references/operations.md](references/operations.md): access control, IDs, advanced routing, ACP, Drive comments, limits, troubleshooting
