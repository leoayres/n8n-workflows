# n8n workflows: Blueprint Systems AI

Free, import-ready n8n workflows from the [Blueprint Systems AI](https://www.youtube.com/@BlueprintSystemsAI) YouTube channel. Each folder has the workflow JSON, a short README, and the video that builds it step by step.

| Workflow | What it does | Video |
|----------|--------------|-------|
| [ai-email-triage](./ai-email-triage) | GPT-4o-mini classifies every Gmail email, drafts the reply in the thread, alerts Slack only for urgent sales leads | [Watch](https://www.youtube.com/watch?v=JZn3UsmQW4g) |

More coming: self-hosting kit (Docker Compose + Caddy + Postgres), Make vs n8n vs Zapier comparison.

## How to import
In n8n: **Workflows → Import from file** → choose the `workflow.json`. Then add your own credentials in each node. No workflow here contains credentials or personal IDs.

## License
MIT
