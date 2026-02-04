# codehooks-openclaw-skill

Give your OpenClaw agent a serverless backend for webhooks, data storage, and background jobs.

## Why this exists

OpenClaw agents run locally. That's great for privacy, but tricky when you need:

- Webhook endpoints that work when your Mac is asleep
- Persistent storage beyond local files
- Scheduled jobs that run 24/7
- Reliable integrations with Stripe, Shopify, GitHub
- Multi-step workflows that run autonomously with retries and state management

This skill gives your agent everything it needs to deploy and manage serverless backends.

## Quick start

```bash
# Install the Codehooks CLI
npm install -g codehooks

# Authenticate
coho login

# Create and deploy a backend
coho create my-backend
coho deploy
```

Your agent now has a live webhook URL, database, and job runner.

## Give your agent the full context

Run this to get the complete development prompt:

```bash
coho prompt
```

This outputs everything your agent needs to build with codehooks-js — routes, database operations, queues, jobs, workflows. Feed it into your agent's context.

**macOS shortcut:**
```bash
coho prompt | pbcopy
```

## What's included

- **SKILL.md** — The skill file for OpenClaw
- **examples/** — Ready-to-use code templates
  - `webhook-handler.js` — Webhook with signature verification
  - `daily-job.js` — Scheduled task example
  - `queue-worker.js` — Async queue processing
  - `workflow-automation.js` — Multi-step autonomous workflows

## Commands reference

| Command | Description |
|---------|-------------|
| `coho prompt` | Get full development context |
| `coho deploy` | Deploy code (5 seconds) |
| `coho info` | Get your endpoint URL |
| `coho log -f` | Stream logs |
| `coho data <collection> --query '{}'` | Query database |
| `coho data <collection> --add '{...}'` | Add data |

## Example: Stripe webhook handler

```javascript
import { app, Datastore } from 'codehooks-js';

app.post('/stripe-webhook', async (req, res) => {
  const db = await Datastore.open();
  await db.insertOne('payments', {
    event: req.body,
    receivedAt: new Date().toISOString()
  });
  res.json({ received: true });
});

export default app.init();
```

## Workflows for autonomous agents

Workflows let your agent kick off multi-step processes that run independently:

```javascript
// Agent starts a workflow
const { workflowId } = await fetch('/workflow/start', {
  method: 'POST',
  body: JSON.stringify({
    data: taskData,
    callbackUrl: 'https://my-agent/webhook' // Get notified on completion
  })
}).then(r => r.json());

// Workflow runs autonomously with retries, state persistence, and error handling
// Agent gets notified via callback when done
```

See `examples/workflow-automation.js` for a complete example.

## Resources

- [Codehooks Documentation](https://codehooks.io/docs)
- [Full development prompt](https://codehooks.io/llms.txt)
- [More templates](https://github.com/RestDB/codehooks-io-templates)
- [MCP Server](https://github.com/RestDB/codehooks-mcp-server)

## License

MIT

---

**Your agent writes the code. Codehooks runs it.**
