# SagaLoom Project Scaffold

## Simple, Practical Structure for Prompt-as-System with Mastra

```
/sagaloom/
├── prompts/                     # PROMPT-AS-SYSTEM (Your BIS modules)
│   ├── personas/               # Agent system prompts
│   │   ├── spinner.md         # Narrative Spinner persona
│   │   ├── weaver.md          # Scene Weaver persona
│   │   ├── canonist.md        # Canonist persona
│   │   └── qa.md              # QA Gatekeeper persona
│   ├── bis/                   # Bigass Instruction Sets (policies)
│   │   ├── chapter-planning.md
│   │   ├── scene-realization.md
│   │   ├── canon-merge.md
│   │   └── pacing-policy.md
│   └── tools/                 # Tool descriptions for agents
│       ├── retrieve-context.md
│       ├── search-canon.md
│       └── validate-output.md
│
├── src/                        # TYPESCRIPT/MASTRA CODE
│   ├── agents/                # Mastra agent definitions
│   │   ├── spinner.ts         # Loads spinner persona + BIS
│   │   ├── weaver.ts          # Loads weaver persona + BIS
│   │   ├── canonist.ts        # Loads canonist persona + BIS
│   │   └── qa.ts              # Loads qa persona + BIS
│   ├── workflows/             # Mastra workflow orchestration
│   │   ├── chapter-cycle.ts   # Main chapter generation workflow
│   │   └── planning-flow.ts   # Arc and story planning workflow
│   ├── tools/                 # Actual tool implementations
│   │   ├── mongodb.ts         # MongoDB operations
│   │   ├── retrieval.ts       # Context retrieval logic
│   │   └── validation.ts      # Output validation
│   ├── lib/                   # Utilities
│   │   ├── prompt-loader.ts   # Load and compose prompts
│   │   ├── db-client.ts       # MongoDB connection
│   │   └── types.ts           # TypeScript interfaces
│   └── app.ts                 # Main application entry
│
├── api/                        # BACKEND API
│   ├── routes/
│   │   ├── stories.ts         # Story CRUD operations
│   │   ├── chapters.ts        # Chapter generation endpoints
│   │   └── canon.ts           # Canon fact management
│   └── server.ts              # Express/Fastify server
│
├── web/                        # FRONTEND
│   ├── src/
│   │   ├── components/        # React/Vue components
│   │   ├── pages/             # Story management UI
│   │   └── api/               # API client
│   └── package.json
│
├── scripts/                    # UTILITIES
│   ├── seed-db.ts             # Initialize MongoDB
│   └── migrate.ts             # Database migrations
│
├── .env.example               # Environment variables template
├── docker-compose.yml         # MongoDB + app containers
├── package.json               # Dependencies
└── tsconfig.json             # TypeScript config
```

## Key Implementation Files

### src/agents/spinner.ts (Example Mastra Agent)
```typescript
import { Agent } from '@mastra/core';
import { loadPrompt } from '../lib/prompt-loader';

export const spinnerAgent = new Agent({
  name: 'spinner',
  model: 'claude-3-opus',
  systemPrompt: loadPrompt('personas/spinner'),
  instructions: [
    loadPrompt('bis/chapter-planning'),
    loadPrompt('bis/pacing-policy')
  ],
  tools: ['searchCanon', 'getRecentChapters'],
  outputSchema: ChapterPlanSchema
});
```

### src/workflows/chapter-cycle.ts (Mastra Workflow)
```typescript
import { Workflow } from '@mastra/core';

export const chapterWorkflow = new Workflow({
  name: 'chapter-cycle',
  steps: [
    { agent: 'spinner', input: 'storyContext' },
    { agent: 'weaver', input: 'chapterPlan' },
    { agent: 'canonist', input: 'chapterDraft' },
    { agent: 'qa', input: 'canonReport' }
  ]
});
```

### prompts/personas/spinner.md (Persona Example)
```markdown
You are the Narrative Spinner - decisive junction strategist.
Priorities: market-fit > arc progress > novelty.

Your role is to plan chapters at narrative junctions.
When given story context and pacing hints, generate a ChapterPlan.

<<INCLUDE:bis/chapter-planning>>
```

### MongoDB Collections
```javascript
// Collections structure
{
  stories: { _id, title, config, marketSpec },
  chapters: { _id, storyId, no, text, plan },
  chapter_summaries: { _id, storyId, no, summary, embed },
  canon_facts: { _id, storyId, subject, statement, validity },
  prompts_versions: { _id, name, version, content } // Optional
}
```

## Minimal Setup Commands
```bash
# Initialize project
npm init -y
npm install @mastra/core mongodb dotenv typescript

# Set up MongoDB
docker-compose up -d mongodb

# Initialize database
npm run seed

# Start development
npm run dev
```

## Environment Variables (.env)
```env
MONGODB_URI=mongodb://localhost:27017/sagaloom
OPENAI_API_KEY=your-key
PORT=3000
```

## Why This Structure Works

1. **Prompts separate from code**: Easy to modify behavior without touching TypeScript
2. **Mastra handles orchestration**: Agents and workflows manage the flow
3. **MongoDB for persistence**: Story state, canon facts, summaries
4. **Clean separation**: Frontend, API, agents, prompts all isolated
5. **Scalable**: Add new agents/BIS by adding files, minimal code changes

## Next Steps to Build This

1. Set up basic Mastra agents that load prompts from files
2. Create MongoDB schemas for the 7 frozen artifacts
3. Implement basic workflow for chapter generation
4. Add simple API endpoints
5. Create minimal UI for story management

This is lean, practical, and actually buildable in a few days. 🔧