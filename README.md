# n8n workflows: Blueprint Systems AI

Free, import-ready n8n workflows from the [Blueprint Systems AI](https://www.youtube.com/@BlueprintSystemsAI) YouTube channel. Each folder has the workflow JSON, a short README, and the video that builds it step by step.

| Workflow | What it does | Video |
|----------|--------------|-------|
| [self-host-kit](./self-host-kit) | Self-host n8n on any VPS: Docker Compose + Postgres 17 + Caddy (automatic HTTPS), pinned versions, backup one-liner | [Watch](https://www.youtube.com/watch?v=jn7h7wNESLw) |
| [ai-email-triage](./ai-email-triage) | GPT-4o-mini classifies every Gmail email, drafts the reply in the thread, alerts Slack only for urgent sales leads | [Watch](https://www.youtube.com/watch?v=JZn3UsmQW4g) |

More coming: Make vs n8n vs Zapier comparison.

## How to import
Workflows: in n8n, **Workflows → Import from file** → choose the `workflow.json`, then add your own credentials in each node. The self-host kit: follow its README. Nothing here contains credentials, secrets or personal IDs.

## License
MIT
