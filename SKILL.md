# Codehooks Backend Skill

Give your OpenClaw agent a serverless backend for webhooks, data storage, and background jobs.

## What this skill enables

- **Webhook endpoints** that external services can call (Stripe, GitHub, Shopify, etc.)
- **Persistent storage** beyond local memory (NoSQL + key-value)
- **Background jobs** and scheduled tasks that run 24/7
- **Queue workers** for async processing
- **Autonomous workflows** with retries, branching, and state management

## Setup

```bash
# Install the CLI
npm install -g codehooks
```

**For agents (non-interactive):** Use an admin token instead of `coho login`:

```bash
# All commands accept --admintoken flag
coho create openclaw-backend --admintoken YOUR_ADMIN_TOKEN
coho deploy --admintoken YOUR_ADMIN_TOKEN
```

Store the token in an environment variable:
```bash
export CODEHOOKS_ADMIN_TOKEN="your-token-here"
coho deploy --admintoken $CODEHOOKS_ADMIN_TOKEN
```

**To get an admin token:** Run `coho add-admintoken` from an authenticated session (human does this once, gives token to agent).

Your backend is now live. Took about 30 seconds.

---

## Essential: Load the development context

Before building anything, run:

```bash
coho prompt
```

This outputs the complete Codehooks development prompt — routing, database, queues, jobs, workflows, and the full codehooks-js API. Copy it into your context to build any backend feature correctly.

**macOS shortcut:**
```bash
coho prompt | pbcopy
```

---

## Commands your agent can use

All commands accept `--admintoken $CODEHOOKS_ADMIN_TOKEN` for non-interactive use.

| Command | What it does |
|---------|--------------|
| `coho prompt` | Get the full development context |
| `coho deploy` | Deploy code (5 seconds to live) |
| `coho info` | Get your endpoint URL |
| `coho log -f` | Stream logs in real-time |
| `coho data <collection> --query '{}'` | Query the database |
| `coho data <collection> --add '{...}'` | Add data |

---

## Code examples

### Webhook that stores incoming data

```javascript
import { app, Datastore } from 'codehooks-js';

app.post('/webhook', async (req, res) => {
  const db = await Datastore.open();
  await db.insertOne('events', {
    ...req.body,
    receivedAt: new Date().toISOString()
  });
  res.json({ ok: true });
});

export default app.init();
```

### Scheduled job (runs daily at 9am)

```javascript
import { app, Datastore } from 'codehooks-js';

app.job('0 9 * * *', async (_, { jobId }) => {
  console.log(`Running job: ${jobId}`);
  const conn = await Datastore.open();
  const events = await conn.getMany('events', {}).toArray();
  console.log('Daily summary:', events.length, 'events');
});

export default app.init();
```

### Queue worker for async processing

```javascript
import { app, Datastore } from 'codehooks-js';

app.worker('processTask', async (req, res) => {
  const { task } = req.body.payload;
  const conn = await Datastore.open();
  await conn.updateOne('tasks', { _id: task.id }, { $set: { status: 'completed' } });
  res.end();
});

export default app.init();
```

### Autonomous workflow (multi-step with retries)

```javascript
import { app } from 'codehooks-js';

const workflow = app.createWorkflow('myTask', 'Process tasks autonomously', {
  begin: async function (state, goto) {
    console.log('Starting task:', state.taskId);
    goto('process', state);
  },
  process: async function (state, goto) {
    // Do work here - workflow handles retries and state
    state.result = await doWork(state.data);
    goto('complete', state);
  },
  complete: function (state, goto) {
    console.log('Done:', state.result);
    goto(null, state); // End workflow
  }
});

// Agent starts workflow via API
app.post('/start', async (req, res) => {
  const result = await workflow.start(req.body);
  res.json(result);
});

export default app.init();
```

---

## Development workflow: Let your agent build new endpoints

1. Agent runs `coho prompt` and loads the development context
2. Agent writes code using codehooks-js patterns
3. Agent runs `coho deploy` (5 seconds to live)
4. Agent verifies with `coho log -f` or `coho data`
5. Agent iterates — the fast deploy loop won't break your flow

---

## When to use this skill

- You need a reliable webhook URL for Stripe, GitHub, Shopify, etc.
- You want persistent storage outside your local machine
- You need scheduled jobs that run even when your device is off
- You want to offload sensitive API integrations to a sandboxed environment
- You need queues for async processing
- You want autonomous multi-step workflows that run independently with retries

---

## Resources

- **Documentation:** https://codehooks.io/docs
- **Full prompt:** Run `coho prompt` or visit https://codehooks.io/llms.txt
- **Templates:** https://github.com/RestDB/codehooks-io-templates
- **MCP Server:** https://github.com/RestDB/codehooks-mcp-server
