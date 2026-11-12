# gmail-draft

Create and update Gmail drafts through the official Gmail API, so links stay exactly as you wrote them.

## Why

Some AI Gmail connectors (including Claude's built-in Gmail connector) rewrite every link in a draft into a `https://www.google.com/url?q=...&source=gmail&ust=...` redirect. Those redirects expire, and in plain-text emails they aren't even clickable. This is a known bug with no official fix:

- [claude.ai Gmail MCP create_draft rewrites URLs](https://claudeissues.com/issue/66847-bug-claude-ai-gmail-mcp-create-draft-rewrites-plain-text-body-urls-into-corrupte)
- [Gmail connector rewrites URLs and bare domains](https://claudeissues.com/issue/94247-bug-gmail-connector-rewrites-urls-and-bare-domains-in-message-bodies-into-google)

`gmail-draft` skips the connector and talks to the Gmail API directly with your own OAuth client, so the draft holds the real links. It's one Python file with no dependencies (standard library only).

## Setup (once per Google account)

1. **Google Cloud project.** In the [Google Cloud console](https://console.cloud.google.com/), pick or create a project.
2. **Enable the Gmail API.** APIs & Services → Library → "Gmail API" → Enable.
3. **OAuth consent screen.** Google Auth Platform → Branding / Audience. On Google Workspace, choose **Internal** so only your organization can use it. On a personal Gmail account, choose External and add yourself as a test user.
4. **Create the client.** Google Auth Platform → Clients → Create client → **Desktop app**. Download the JSON.
5. **Install:**
   ```sh
   mkdir -p ~/.config/gmail-draft && chmod 700 ~/.config/gmail-draft
   mv ~/Downloads/client_secret_*.json ~/.config/gmail-draft/client.json
   chmod 600 ~/.config/gmail-draft/client.json
   curl -fsSL https://raw.githubusercontent.com/TheSamyar/gmail-draft/main/gmail-draft -o ~/bin/gmail-draft
   chmod +x ~/bin/gmail-draft
   ```
   (Any folder on your `PATH` works instead of `~/bin`.)
6. **Sign in once:**
   ```sh
   gmail-draft auth
   ```
   Your browser opens, you approve, and a token is saved to `~/.config/gmail-draft/token.json` (mode 600). It refreshes itself after that.

Scopes requested: `gmail.compose` (create, update and send drafts) and `gmail.metadata` (read headers only, used to reply in a thread). It can't read email bodies.

## Use

```sh
# New draft
gmail-draft create --to someone@example.com --cc other@example.com \
  --subject "Hello" --body-file msg.txt --html-file msg.html

# Reply in an existing thread (pass the Gmail message id you're replying to)
gmail-draft create --to someone@example.com --body-file msg.txt --reply-to-msg 18c2f0a1b2c3d4e5

# Replace a draft's content (pass recipients and subject again)
gmail-draft update r-1234567890 --to someone@example.com --subject "Hello" --body-file msg.txt

# Print the stored raw email, to check the links
gmail-draft get r-1234567890 | grep -c google.com/url   # should print 0

# Built-in check (no network)
gmail-draft --selfcheck
```

`--html-file` is optional. If you skip it, the email is plain text and mail apps make the URLs clickable on their own.

It only creates and updates **drafts**. You review and send them yourself in Gmail.

## Using it with an AI agent

Tell your agent to write drafts with `gmail-draft` and to keep using its Gmail connector only for reading and searching. For Claude Code, a line like this in your `CLAUDE.md` works:

> Write Gmail drafts with `~/bin/gmail-draft` (never the Gmail connector's create_draft/update_draft, which breaks links). After saving, run `gmail-draft get <id> | grep -c google.com/url` and confirm it prints 0. Never send without asking.

## Security

- Your OAuth client and token stay on your machine in `~/.config/gmail-draft/` with owner-only permissions. Never commit them.
- To revoke access, remove the app at [myaccount.google.com/permissions](https://myaccount.google.com/permissions) and delete `token.json`.

## License

MIT
