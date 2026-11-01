# AI Email Triage & Auto-Draft (n8n + GPT-4o-mini)

Every new Gmail email is classified by GPT-4o-mini, gets a ready-to-send **draft reply in the same thread**, and only **urgent sales leads** trigger a Slack alert. Nothing is sent automatically.

▶️ Video walkthrough: https://www.youtube.com/watch?v=JZn3UsmQW4g

## How it works (v1.3)
1. **Gmail Trigger**: polls every minute, filter `-from:me -in:drafts` (so your own drafts don't loop back in)
2. **Clean & Dedupe**: strips hidden text (display:none, zero-size text, invisible Unicode) before the prompt, and skips messages already handled
3. **AI Classifier**: HTTP Request to OpenAI `gpt-4o-mini` in JSON mode → `category` (sales_lead / billing_invoice / support), `urgency` (high / normal), `summary`, `draft_reply`. The prompt treats the email as untrusted data
4. **Parse & Validate**: broken JSON, unknown category, empty draft or API error → `needs_review` (never silently dropped)
5. **Switch**: `hot_lead` (sales + high) · `urgent` (other high) · `normal` · `needs_review`
6. **Gmail**: draft reply in the thread (all except needs_review) + labels `Urgent` / `Needs review`
7. **Slack**: alert in `#sales-alerts` for hot leads, with summary and a direct link to the Gmail thread
8. **Mark as processed**: remembers message IDs so a re-run never stacks a second draft

## Changelog
- **v1.3**: hidden-text stripping + untrusted-input prompt, JSON validation with a needs-review fallback, Gmail labels for urgency/review, Slack alert links to the thread, duplicate-draft guard. Suggestions from r/n8n, thanks!
- v1.2: initial public version.

## Import
1. Download `workflow.json` → in n8n: **Workflows → Import from file**.
2. Create credentials: Gmail (OAuth2), OpenAI (API key), Slack. In Gmail, create two labels (e.g. `AI/Urgent`, `AI/Needs review`) and pick them in nodes 4C and 4D.
3. Open each node, select your credential, set your Slack channel and the signature in the prompt.
4. Test with a few emails, then **Publish**.

## Requirements
- n8n (Cloud or self-hosted, tested on 2.43.x)
- Gmail account, OpenAI API key (gpt-4o-mini costs a fraction of a cent per email), Slack workspace

The workflow contains no credentials or personal IDs. MIT licensed.
