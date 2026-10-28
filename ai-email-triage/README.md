# AI Email Triage & Auto-Draft (n8n + GPT-4o-mini)

Every new Gmail email is classified by GPT-4o-mini, gets a ready-to-send **draft reply in the same thread**, and only **urgent sales leads** trigger a Slack alert. Nothing is sent automatically.

▶️ Video walkthrough: https://www.youtube.com/watch?v=JZn3UsmQW4g

## How it works
1. **Gmail Trigger**: polls every minute, filter `-from:me -in:drafts` (so your own drafts don't loop back in)
2. **AI Classifier**: HTTP Request to OpenAI `gpt-4o-mini` in JSON mode → `category` (sales_lead / billing_invoice / support), `urgency` (high / normal), `summary`, `draft_reply`
3. **Switch**: `hot_lead` = sales_lead + high urgency
4. **Gmail**: create a draft reply in the thread (every email)
5. **Slack**: alert in `#sales-alerts` (hot leads only)

## Import
1. Download `workflow.json` → in n8n: **Workflows → Import from file**.
2. Create credentials: Gmail (OAuth2), OpenAI (API key), Slack.
3. Open each node, select your credential, set your Slack channel and the signature in the prompt.
4. Test with a few emails, then **Publish**.

## Requirements
- n8n (Cloud or self-hosted, tested on 2.43.x)
- Gmail account, OpenAI API key (gpt-4o-mini costs a fraction of a cent per email), Slack workspace

The workflow contains no credentials or personal IDs. MIT licensed.
