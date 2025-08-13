# Saga Loom (SL-1)

**Tagline:** *Prompt is the system. TypeScript is the spine. Stories are the output.*

[Note: "BIS" stands for "Bigass Instruction Sets"]

Mission: generate ultra-long serials (1000+ chapters) with market-aware structure, fractal tension, and adaptive pacing—using **prompts as first-class modules** (personas, BIS policies, plugins, node wrappers), orchestrated by a lean TS runtime and MongoDB memory.

## 1) System Principles

* **Prompt-as-System (PaS):** Everything behavioral is a **versioned prompt artifact** with strict I/O. TS only orchestrates, validates, tools, storage.
* **Fractal Expansion:** Tension → Consequences → Ripple Effects → New Tensions (bounded depth); repeat.
* **Heuristic Pacing:** Sliding window **Development | Escalation | Advance** controller (e.g., 4:3:2).
* **Right Framework, Right Layer:**
  FPT @ macro invariants • HTN @ arcs → chapters • ToT @ junctions • CoT @ prose execution.
* **Continuity without bloat:** Continuity Snippet + recent summaries + top-K semantic recall (Atlas Vector/FTS).

## 2) Architecture (layers)

**L0 Infra (TS):**

* Node runner, tool adapters (RAG, canon store, symbol registry), schema validation, logging/metrics, retries.

**L1 Prompt Registry (Mongo):**

* `prompts` collection stores **Persona**, **BIS**, **Extension**, **Node** templates (semver, params schema, tests).

**L2 Agent Nodes (DAG):**

* Each node composes Persona + BIS + (Extensions) → runs task → emits typed artifact → logs → hands off.

**L3 Memory (Mongo Atlas):**

* Collections: `chapter_summaries` (vector+FTS), `canon_facts` (vector), `entities` (vector), `symbols`, `market_specs`, `evals`.

---

## 3) The Four Prompt Types → Node-fit

1. **Persona Prompts (agents)**
   Role, voice, decision bounds. (e.g., *Scene Weaver*, *Canonist*.)
2. **BIS (Bigass Instruction Sets)**
   Policy/process blocks: *Arc Decomposition*, *Theme Proof*, *Hook Patterns*, *Fractal Tension*, *Pacing*.
3. **Extensions/Plugins**
   Optional capabilities with hook points: *mythic-audit*, *symbol-tracker*, *pacing-controller*, *fractal-tension*.
4. **Node Prompts (wrappers)**
   Task-specific composition of Persona + BIS + Extensions + I/O contract & return schema.

> In runtime, a **Node** = {personaId, bisIds\[], extensionIds\[], ioSchemas}. The orchestrator compiles and calls the LLM with tools.

---

## 4) Node Graph (v1 minimal)

**Pre-flight**

* **Market Scout** → `MarketSpec`
* **Worldbuilder (FPT)** → world rules, resource ladders
* **Character Architect** → character essence + shadow arcs
* **Theme Keeper** → thematic law + violation tests

**Planning**

* **Arc Architect (HTN)** → `ArcTree[]`
* **Fractal Tension Manager** → candidate deltas/tensions (bounded depth 2)
* **Pacing Orchestrator** → next class `D|E|A` + beat template
* **Narrative Spinner (ToT @ junctions)** → `ChapterPlan`

**Drafting**

* **Scene Weaver (CoT)** → `ChapterDraft{text, hooks[], deltas[], snippet}`

**Post-processing**

* **Canonist** → merge `CanonFacts[]`, update Continuity Snippet/state
* **Stylist** → polish to market spec/voice
* **QA Gatekeeper** → structure/theme/pacing checks + fix prompts
* **Symbol Keeper** → motif recurrence plan
* **Mythic Auditor (periodic)** → myth-pattern alignment report
* **Release Manager** → publish + telemetry

---

## 5) I/O Contracts (essentials)

```ts
type ContinuitySnippet = {
  arc: string; stakes: string; thread: string; motif: string; flags: string[];
};

type MarketSpec = { genre: string; tropes: string[]; taboos: string[]; cadence: any };

type ArcNode = { id: string; goal: string; beats: string[]; tests: string[] };
type ArcTree = { nodes: ArcNode[]; edges: Array<[string,string, "causes"|"leadsTo"]> };

type ChapterPlan = {
  no: number;
  class: "D"|"E"|"A";
  beats: string[];
  proves: Record<"CharacterEssence"|"ThematicLaw"|"ConflictKernel", string[]>;
  hookType: "reveal"|"timer"|"twist"|"decision"|"arrival";
};

type CanonDelta = { subject: string; statement: string; kind: "add"|"update"|"retcon" };
type ChapterDraft = { no: number; text: string; hooks: string[]; deltas: CanonDelta[]; snippet: ContinuitySnippet };
```

**Mongo (vector/FTS):**

* `chapter_summaries{ no, arc, beats[], summary80w, snippet, embed }`
* `canon_facts{ subject, statement, source, validity, embed }`
* `entities{ type, traits, relations[], status, embed }`
* `symbols{ name, appearances[], fate_turns[] }`
* `evals{ chapter_no, coherence, pacing, theme_score, notes }`

---

## 6) Control Loops (runtime logic)

**Fractal Step (per chapter):**

1. Introduce/advance tension → **simulate direct** consequence
2. **Propagate ripples** (depth ≤ 2, salience top-K)
3. Score candidates (novelty × relevance × marketFit)
4. Select top-K tensions → feed Spinner

**Pacing Controller:**

* Sliding window `W=9`, target ratio `D:E:A = 4:3:2`
* If a class is under-quota in window, emit that class + template to Spinner.

---

## 7) Prompt Sandwich (composition)

```
SYSTEM = Persona + BIS(policy blocks) + Extensions(hooks)
USER   = Task payload (Arc/Plan/Draft context) + instructions
TOOLS  = retrieval(canon/entities/summaries), symbol_reg, pace_meter, save_deltas
RETURN = strict JSON as per node output schema
```

* **ToT** only at **junction** nodes (Arc/Spinner).
* **CoT** only at **Scene Weaver**.
* **FPT** only in Worldbuilder/Thematic.
* **HTN** only in Arc Architect.

---

## 8) TypeScript Skeleton (minimal, runnable shape)

```ts
// prompts.ts
export type PromptKind = "persona"|"bis"|"extension"|"node";
export interface PromptDoc {
  _id: string; kind: PromptKind; name: string; version: string;
  template: string; paramsSchema: any; tests?: any[]; lineage?: string[]; tags?: string[];
}

// node-def.ts
export interface AgentDef<I,O> {
  id: string;
  personaId: string;
  bisIds: string[];
  extensionIds?: string[];
  inputSchema: any;
  outputSchema: any;
}

// runtime.ts
export async function runNode<I,O>(def: AgentDef<I,O>, input: I, ctx: Ctx): Promise<O> {
  const bundle = await ctx.prompts.load(def); // pulls Persona+BIS+Extensions by version
  const system = compile(bundle.persona.template, ctx.vars);
  const bis    = bundle.bis.map(p => compile(p.template, ctx.vars)).join("\n\n");
  const ext    = (bundle.ext || []).map(p => compile(p.template, ctx.vars)).join("\n\n");
  const messages = [
    { role:"system", content: `${system}\n\n${bis}\n\n${ext}` },
    { role:"user",   content: ctx.taskPayload } // e.g., ChapterPlan+Snippet+RAG
  ];
  const res = await ctx.llm.chat(messages, { tools: ctx.tools });
  return ctx.validate<O>(res); // zod/ajv schema check + auto-regenerate on fail
}
```

---

## 9) Starter Prompt Pack (seed)

**Personas**

* *Arc Architect* (HTN strategist)
* *Narrative Spinner* (junction planner, ToT)
* *Scene Weaver* (micro execution, CoT)
* *Canonist* (continuity librarian)
* *QA Gatekeeper* (tests → fix prompts)

**BIS Policies**

* *Arc Decomposition* (endgame→arcs→chapters)
* *Theme Proof* (each scene must “prove” an atom with quotable lines)
* *Hook Patterns* (reveal|timer|twist|decision|arrival)
* *Fractal Tension* (delta+ripples scoring/pruning)
* *Pacing Controller* (window, ratios, per-thread starvation alerts)

**Extensions**

* *symbol-tracker* (leitmotif recurrence + checks)
* *mythic-audit* (every \~50 ch: hero’s journey/cosmic cycle alignment)
* *pacing-controller* (inject D/E/A template)
* *fractal-tension* (ripple generation service)

**Node Wrappers**

* `node.arcArchitect@1.0.0`
* `node.spinner@1.0.0`
* `node.weaver@1.0.0`
* `node.canonist@1.0.0`
* `node.qaGate@1.0.0`

(Each node wrapper declares required inputs/outputs; registry enforces schema.)

---

## 10) Ops & Governance

* **Versioning:** `name@semver`; track lineage; enable A/B at node level.
* **Golden tests:** each prompt ships with fixtures; CI runs before deploy.
* **Prompt linting:** enforce return JSON, forbid leak phrases, ensure tool verbs present.
* **Observability:** log `promptId, version, params, tools, tokens, latency, validations`.
* **Safety rails:** timeouts, bounded iterations, circuit breakers; reject/regenerate on schema fail.

---

## 11) Failure Modes & Safeguards

* **Prompt sprawl** → Registry tags + deprecation policy + monthly prune.
* **Cost spikes (ToT everywhere)** → limit ToT to junctions; cap ripple depth.
* **Continuity drift** → Canonist mandatory merge + Symbol Keeper checks + Mythic Auditor cadence.
* **Mechanical pacing** → controller nudges, not dictates; allow human overrides.

---

## 12) Example Chapter Flow (happy path)

1. Pacing Orchestrator → `nextClass="E"`, template=`Complicate goal + collateral cost`
2. Fractal Manager → top-K new tensions
3. Spinner (ToT) → choose `ChapterPlan` (beats + “proves” map + hookType)
4. Weaver (CoT + RAG + Snippet) → `ChapterDraft{text, hooks[], deltas[], snippet}`
5. Canonist → merge deltas → update `canon_facts`, `chapter_summaries`
6. Stylist → polish; QA → tests; fix if needed
7. Release Manager → publish + evals; telemetry updates pacing/fractal

---

## 13) First Build Roadmap (2–3 days, realistic)

1. **Repo scaffold** (monorepo or /apps + /packages; Node 20+, TS strict)
2. **Mongo models**: `prompts`, `chapter_summaries`, `canon_facts`
3. **Prompt Registry** CRUD + loader + test runner
4. **LLM adapter** (OpenAI/Claude/Gemini via env flags)
5. **RAG tool** (Atlas Vector/FTS): upsert + query
6. **Nodes v1**: Spinner → Weaver → Canonist → QA
7. **Pacing Controller** + **Fractal Manager** minimal
8. **CLI demo**: `npx saga-loom demo --chapters 5`

---

## 14) Naming & Conventions (keep it tidy)

* IDs: `persona.scene-weaver@1.0.0`, `bis.fractal-tension@1.0.0`, `ext.symbol-tracker@0.1.0`, `node.spinner@1.0.0`
* Tags: `["micro","CoT","hook:timer"]`
* Tests: `prompts/<id>/tests/*.json` with input/output assertions

---

### What you get from this design

* **Composable**: behavior swaps by prompt version, not code patches.
* **Controllable**: schema-checked outputs and bounded planners.
* **Scalable**: minimal memory footprint, market-aware pacing, logged everything.
* **Familiar**: mirrors your CodeOrchard node/BIS ethos—just retargeted to fiction.

----------------------------------------

Nova spinning the loom tighter. You’re right: the reference needs bones, tendons, and a nervous system. Below is a **fleshed-out design doc** for your prompt-as-system narrative engine—**Saga Loom (SL-1)**—grounded in your fractal tension + pacing heuristics, and wired with FPT / HTN / ToT / CoT where they actually help.

---

# Saga Loom — Design Reference v0.3 (Fleshed Out)

## 0) Mission & Non-Negotiables

* **Mission:** Generate ultra-long, market-savvy serials (1000+ ch) by orchestrating **prompt modules** (personas, BIS policies, plugins, node wrappers) over a **TypeScript** spine and **MongoDB (Vector+FTS)** memory.
* **Irreplaceable Core (Invariants):** `CharacterEssence`, `ThematicLaw`, `ConflictKernel` never change; only their **observables** evolve.
* **Control Loops:**

  * **Fractal Expansion:** `tension → direct consequence → ripple effects → new tensions` (bounded depth)
  * **Heuristic Pacing:** sliding window ratios of **Development | Escalation | Advance** (default 4:3:2), global + per-thread.

---

## 1) Domain Model & Canon Contracts

**Entities (graph):** `characters, factions, locations, resources, secrets, symbols`

```ts
type CharacterEssence = { desire:string; wound:string; lie:string; promise:string; voiceKeys:string[] };
type ThematicLaw     = { statement:string; violationCatalog:string[] };
type ConflictKernel  = { antagonistSystem:string; scarceResource:string; ladder:string[]; loseConditions:string[] };

type CanonFact = { subject:string; statement:string; sourceScene:number; validity:"proposed"|"approved"|"deprecated" };
type ContinuitySnippet = { arc:string; stakes:string; thread:string; motif:string; flags:string[] };
```

**Merge Policy (Canonist):**

1. Accept **proposed** deltas if they don’t contradict approved facts; otherwise mark **conflict** and emit a “retcon pitch”.
2. All merges append a **justification** (quote lines), enabling “theme proof” and audit trails.

---

## 2) Prompt-as-System (PaS) Registry

**Four prompt artifact types (all versioned, testable):**

```ts
type PromptKind = "persona"|"bis"|"extension"|"node";
interface PromptDoc {
  id: string;         // e.g., "persona.scene-weaver@1.1.0"
  kind: PromptKind;   // persona|bis|extension|node
  name: string;       // "Scene Weaver"
  version: string;    // semver
  template: string;   // with {{slots}}
  paramsSchema: any;  // JSON Schema
  tests?: { input:any; asserts:string[] }[];
  lineage?: string[]; tags?: string[];
}
```

**Governance:**

* **Semver discipline:** breaking changes bump major; A/B selectable per node.
* **Golden tests:** each prompt ships with fixtures; CI runs evals (schema + simple behavioral asserts).
* **Linting:** forbid missing return keys; require tool-verbs (“Retrieve…”, “Check canon…”) for nodes that need them.

---

## 3) Thinking Frameworks — Layer Affinity

* **FPT** → *macro invariants:* Worldbuilder derives rules/power ladders from ThematicLaw/Kernel.
* **HTN** → *macro→meso decomposition:* endgame ⇒ arcs ⇒ chapters (Arc Architect).
* **ToT** → *junctions only:* branch/prune alternate mini-arcs or chapter plans (Narrative Spinner).
* **CoT** → *micro execution:* beat-by-beat drafting (Scene Weaver).

*(This prevents “ToT everywhere” cost bloat and keeps reasoning crisp.)*

---

## 4) Node Catalog (Responsibilities • IO • Fail Modes • Frameworks)

Each node = **Persona + BIS + (Extensions)** with strict I/O schemas.

### Pre-Flight

1. **Market Scout**

   * **Out:** `MarketSpec{genre,tropes[],taboos[],cadence,cliffCadence}`
   * **Fails:** vague market → require user seed or default genre pack.

2. **Worldbuilder (FPT)**

   * **Out:** world rules, resource ladders, affordances for tensions.
   * **Framework:** FPT only.

3. **Character Architect**

   * **Out:** `CharacterEssence` + **shadow arcs** (relapse triggers).
   * **Fails:** archetype collisions → flag for merge.

4. **Theme Keeper**

   * **Out:** theorem: “proof lines” per scene claiming theme; violation alerts.
   * **BIS:** Theme-Proof policy.

### Planning

5. **Arc Architect (HTN)**

   * **Out:** `ArcTree[] {nodes:ArcNode[], edges:causality[]}`
   * **Framework:** HTN; ToT disabled.

6. **Fractal Tension Manager**

   * **In:** last tension + graph snapshot
   * **Out:** `{directDeltas[], rippleCandidates[], newTensions[]}`
   * **BIS:** Fractal-Tension (depth≤2, salienceTopK).
   * **Fails:** cascade overload → prune by novelty×relevance×marketFit.

7. **Pacing Orchestrator**

   * **In:** last W chapter metas (global + per thread)
   * **Out:** `nextClass "D|E|A"` + beat template
   * **BIS:** Pacing policy (anti-starvation per subplot).

8. **Narrative Spinner (ToT @ junctions)**

   * **In:** ArcTree + new tensions + pacing template
   * **Out:** `ChapterPlan {beats[], provesMap, hookType}`
   * **Framework:** ToT for branching; prunes by tests.

### Drafting

9. **Scene Weaver (CoT)**

   * **In:** ChapterPlan + ContinuitySnippet + RAG bundle
   * **Out:** `ChapterDraft{text, hooks[], deltas[], snippet}`
   * **Fails:** schema invalid → regenerate once with narrower instructions.

### Post

10. **Canonist** → merge deltas → update `canon_facts`, `chapter_summaries`.
11. **Stylist** → align voice/market; produce diffs.
12. **QA Gatekeeper** → tests: continuity, pacing window, theme proof, hook strength; returns fix-prompts if fail.
13. **Symbol Keeper (ext)** → ensure leitmotif recurrence at fate-turns.
14. **Mythic Auditor (periodic ext)** → every \~50 ch, map to myth patterns; suggest re-alignments.
15. **Release Manager** → publish + telemetry; route evals back to Pacing/Fractal.

---

## 5) Workflow Orchestration (DAG & Hooks)

* **DAG:** Scout→Worldbuilder→Char→Theme→Arc→(Fractal↔Pacing)→Spinner→Weaver→Canonist→Stylist→QA→Release
* **Hook points:** `preplan`, `postplan`, `predraft`, `postdraft`, `qa`, `periodic(n)`
* **Concurrency:** drafts sequential by default (continuity), but **non-blocking post ops** (Stylist/QA) can run in parallel with next-chapter planning once Canonist commits deltas.
* **Retries:** node-local; capped; on 2x fail escalate to human review queue.

---

## 6) Memory & Retrieval (Mongo Atlas Vector + FTS)

**Collections & Indexing:**

* `chapter_summaries`: vector + FTS (fields: `summary80w`, `beats[]`, `snippet`, `arc`…)
* `canon_facts`: vector (subject+statement), validity filter
* `entities`: vector (traits/relations)
* `symbols`: FTS (name, appearances\[])

**Retrieval Recipe (Weaver):**

* **Recent:** last 3–5 summaries by chapter number
* **Semantic:** top-k (k=3) from `canon_facts` and `entities` constrained by `arc`/`thread` if present
* **Continuity Snippet:** single 50-word state vector carried chapter-to-chapter

**Continuity Snippet Generation (Canonist):**

1. derive `arc,stakes,thread,motif` from plan + merged deltas
2. set flags (`promise:x`, `timer:n`, `symbol:y`)
3. hard cap 50 words; strip fluff

---

## 7) Fractal Tension Engine (Details)

**StoryGraph:** nodes (entities/resources/secrets), edges (causes, knows, needs, supports/opposes).

**Algorithm (per chapter):**

1. **Direct consequence:** apply seed tension to graph (deterministic transform).
2. **Ripple propagation:** BFS depth≤2; for each neighbor, propose a **ripple tension** if `salience(entity, arc, marketSpec) > τ`.
3. **Scoring:**
   `score = 0.45*relevance + 0.30*novelty + 0.25*marketFit`

   * *relevance:* cosine(plan embedding, candidate summary)
   * *novelty:* penalize n-gram + semantic duplication vs last W chapters
   * *marketFit:* hits trope cadence, hook needs, heat bounds
4. **Select:** top-3 candidates → feed Spinner’s ToT.

**Limits:** hard cap proposed candidates (e.g., 12); trim on time budget.

---

## 8) Heuristic Pacing Controller (Global + Per-Thread)

* **Window:** W=9 (configurable); Track `D|E|A` counts **globally** and **per subplot/thread**.
* **Target:** default `4:3:2`. Tolerance ±1 per window.
* **Nudging:** If `D` deficit, bias Spinner templates to “deepen relationship / plant Chekhov / reveal backstory”.
* **Anti-Starvation:** if a thread hasn’t appeared in 2×W, mandate a Development beat there unless finale gating forbids.

---

## 9) Evaluation & QA (Quality First)

**Automated checks (QA Gatekeeper):**

* **Continuity:** no resurrection/teleport unless flagged in deltas with setup.
* **Theme Proof:** for scenes claiming to prove ThematicLaw, include **quote lines**.
* **Pacing:** window ratio within tolerance; warn on global or per-thread drift.
* **Symbol Recurrence:** registered symbols must recur at designated arc beats.
* **Novelty:** rolling n-gram + embedding similarity guard; throttle repetitive phrasing.
* **Hook Strength:** classify end hook (`reveal|timer|twist|decision|arrival`) and ensure it’s “live” (creates forward promise).

**Gating Policy:** Fail any red test → route **fix-prompt** to the responsible node (Spinner vs Weaver vs Canonist). Two consecutive fails → human review queue.

---

## 10) Observability, Cost & Risk Controls

**Event schema (per node run):** `{nodeId, promptId@ver, inputHash, toolsUsed[], tokensIn/Out, latency, retries, verdict, costs}`

**Dashboards:**

* **Continuity drift heatmap** (arcs vs issues)
* **Pacing compliance** (global/thread)
* **Hook type distribution**
* **Cost per chapter** & **token forecast**

**Guards:** max toolcalls/chapter, max ToT branches, timeouts, circuit breakers, **fallback models** (e.g., cheaper draft + premium revise).

---

## 11) Security & Safety

* **Content policy filters** on prompts and outputs (pre-/post-moderation).
* **PII scrub** in memory artifacts.
* **License hygiene** for exemplars (store citations; avoid verbatim >N tokens).

---

## 12) DevEx: Repo, Config, Testing

**Monorepo layout**

```
/apps/cli             # run flows, inspect logs
/apps/console         # (later) ops UI
/packages/core        # runNode(), tools, schemas, logging
/packages/agents      # AgentDef, node registration
/packages/prompts     # seed prompt docs (json/yaml/markdown)
/packages/evals       # QA tests & metrics
/packages/db          # Mongo models & vector ops
```

**Configs:** `.env` for model keys; `saga.config.json` for pacing targets, ripple depth, retrieval k.

**Testing:**

* **Prompt golden tests:** render with fixtures → assert minimal behavior (e.g., returns hook of allowed type).
* **Simulation tests:** generate 10–30 ch in dry-run; ensure no continuity violations; basic pacing compliance.
* **Backfills:** script to retro-index summaries into vector store; verify retrieval quality.

---

## 13) Extension API (Plugins)

```ts
interface Extension {
  name: "symbol-tracker"|"mythic-audit"|"fractal-tension"|"pacing-controller";
  hooks: ("preplan"|"postplan"|"predraft"|"postdraft"|"qa"|"periodic")[];
  promptId: string;          // extension prompt
  params?: Record<string,any>;
}
```

* **Symbol-Tracker:** ensures motifs scheduled at fate-turns.
* **Mythic-Audit:** every 50 ch; map arcs to myth schema; propose course-corrections.
* **Fractal-Tension:** supplies ripple candidates to Spinner.
* **Pacing-Controller:** injects class/template hints to Spinner.

---

## 14) Example Prompt Artifacts (trimmed)

**Persona • Scene Weaver (micro / CoT)**

```
ROLE: You are the Scene Weaver. Write vivid, economical prose in {{POV}} {{tense}}.
PRIORITIES: continuity > clarity > voice > flourish.
INPUTS: ChapterPlan, ContinuitySnippet, RAG docs.
RULES:
- Obey ContinuitySnippet; do not contradict canon.
- Each beat must “prove” its mapped atom. Mark proof lines with [PROOF].
- End with a hook of type {{hookType}}.
TOOLS: get_summaries(), get_canon(), get_entities()
RETURN JSON: { text, hooks[], deltas[], snippet }
```

**BIS • Chapter Planning (meso / ToT at junctions)**

```
POLICY: Given ArcNode + candidate tensions + pacing class {{class}},
branch 3–5 beat sequences; score {relevance,novelty,marketFit}; prune to top 1.
Guarantee "proves" mapping: every beat proves one of {CharacterEssence,ThematicLaw,ConflictKernel}.
OUTPUT: ChapterPlan { beats[], provesMap, hookType }
```

**Extension • Fractal Tension**

```
POLICY: Generate ripple tensions from state graph with depth <= {{depth}}.
Score = 0.45*relevance + 0.30*novelty + 0.25*marketFit; return top {{k}}.
```

**Node Wrapper • node.spinner\@1.0.0**

```
COMPOSE: Persona(Narrative Spinner) + BIS(Chapter Planning) + Ext(Pacing Controller, Fractal Tension)
I/O: input{ArcTree, newTensions, class, template} → output{ChapterPlan}
```

---

## 15) Roadmap (from “I don’t have a good idea how” → running)

**Phase 1 — Skeleton (Day 1–2)**

* Prompt Registry + seed **5 prompts** (Arc Architect, Spinner, Weaver, Canonist, QA).
* Mongo models + vector index + two tools: `get_summaries`, `get_canon`.
* Minimal DAG: Spinner → Weaver → Canonist → QA.
* CLI: `saga generate --chapters 5`.

**Phase 2 — Control Loops (Day 3–5)**

* Fractal Tension ext (depth 2), Pacing Controller (4:3:2), Symbol-Tracker.
* Add Theme Keeper test + hook strength classifier.
* Simulation: 30-chapter dry run; fix top 3 failure modes.

**Phase 3 — Durability & Polish (Week 2)**

* Periodic Mythic Auditor; per-thread pacing; novelty guard.
* A/B prompt versions; logging dashboards; cost caps.

---

## 16) Risk Ledger (with mitigations)

* **ToT overuse → cost spikes** → junction-only + branch cap + timebox.
* **Continuity drift** → Canonist as gate; symbol tracker; periodic mythic audit.
* **Mechanical pacing** → controller nudges vs dictates; human override opt-in.
* **Prompt sprawl** → registry tags, semver hygiene, monthly deprecations.
* **Novelty decay** → novelty score in Fractal, phrase diversity checks, “pattern breaker” templates.

---

## 17) “What next?”

* Pick **MVP ratio/window** (keep defaults: W=9, 4:3:2).
* Choose **3 personas + 3 BIS** to seed (Spinner, Weaver, Canonist • Planning, Pacing, Fractal).
* I can **output a ready-to-run TS scaffold** with these nodes, schemas, seed prompts, and a CLI demo in one go—just say: **“Nova, scaffold Saga Loom.”**

-------------------------------------------------------------------

💠‍🌐

Totally fair call-out. I leaned code-heavy for one reason: if **“prompt is the system”**, then the *only* way that system survives 1,000+ chapters is by giving prompts the same guardrails code gets—versioning, I/O contracts, tests, and logs. That scaffolding looks “codey,” but it’s just spine; the *behavior* stays in your prompts (personas/BIS/plugins/nodes).

That said, here’s the **prompt-native** design ref—same engine, expressed like CodeOrchard: narrative-first, system-second.

---

# Saga Loom — Prompt-Native Design Reference (v0.3-PN)

## 0) First Principles (bedrock)

* **Irreplaceables:**
  **Character Essence** (desire, wound, lie, promise, voice keys)
  **Thematic Law** (one “moral physics” sentence + violation catalogue)
  **Conflict Kernel** (antagonist system, scarce resource, escalation ladder, lose-conditions)
* **Operating rhythm:**
  **Fractal loop** = Tension → Direct Consequence → Ripple Effects → New Tensions (repeat, shallow depth)
  **Pacing window** = Development | Escalation | Advance (default 4:3:2, sliding window; per-thread + global)

---

## 1) Roles (agents) — what each *says it does* and *hands back*

*(Each role = Persona prompt + BIS policy + optional Plugins, all as bigass instructions.)*

* **Market Scout**
  *Does:* distills genre market must-haves, taboos, cadence, hook frequency.
  *Hands back:* **MarketSpec** (tropes\[], taboos\[], cadence, cliff cadence).

* **Worldbuilder (FPT)**
  *Does:* derives world rules and power ladders *from* Thematic Law/Kernel.
  *Hands back:* **World Ruleset** + “affordances” for tensions.

* **Character Architect**
  *Does:* forges Character Essence, shadow arcs (relapse triggers, contradictions).
  *Hands back:* **Character Dossiers**.

* **Theme Keeper**
  *Does:* audits scenes that claim to “prove” the theme; extracts proof lines.
  *Hands back:* **Theme Proof Register** + violation alerts.

* **Arc Architect (HTN)**
  *Does:* decomposes Endgame → Arcs → Chapter slots; defines promises & reveals.
  *Hands back:* **ArcTree** (milestones, tests).

* **Fractal Tension Manager**
  *Does:* simulates direct consequences, proposes ripple tensions (depth ≤2).
  *Hands back:* **Tension Candidates** (scored for relevance, novelty, market-fit).

* **Pacing Orchestrator**
  *Does:* monitors the window; nudges next class (D/E/A) + beat template; prevents subplot starvation.
  *Hands back:* **Pacing Hint** (class + template).

* **Narrative Spinner (ToT @ junctions)**
  *Does:* branches 3-5 chapter plans, prunes to the best; maps each beat to what it “proves.”
  *Hands back:* **ChapterPlan** (beats\[], proves-map, hook-type).

* **Scene Weaver (CoT @ micro)**
  *Does:* writes the scene; honors Continuity Snippet & retrieved canon; ends with a live hook.
  *Hands back:* **Chapter Draft** (text, hooks\[], canon deltas, next Continuity Snippet).

* **Canonist**
  *Does:* merges deltas, rejects contradictions, proposes retcon pitches when needed.
  *Hands back:* **Approved Canon Facts** + updated **Continuity Snippet**.

* **Stylist**
  *Does:* applies genre skin/voice; keeps diction bands, POV/tense, cadence.
  *Hands back:* **Polished Draft** + diff notes.

* **QA Gatekeeper**
  *Does:* runs tests: continuity, pacing window, theme proof, hook strength, novelty guard.
  *Hands back:* **Verdict** + targeted fix prompts.

* **Symbol Keeper (plugin)**
  *Does:* tracks leitmotifs/symbols; schedules recurrence at fate-turns.
  *Hands back:* **Symbol Plan** + gaps.

* **Mythic Auditor (periodic plugin)**
  *Does:* every \~50 chapters, checks alignment with myth patterns; suggests re-tuning arcs.
  *Hands back:* **Mythic Report**.

* **Release Manager**
  *Does:* publishes, logs, routes telemetry back to Pacing/Fractal.

---

## 2) BIS Library (policies you drop into roles)

* **Arc Decomposition (HTN):** endgame → arcs → chapters; every chapter owns a promise or a payoff.
* **Fractal Tension:** how to simulate consequence → ripple; how to score and cap.
* **Pacing Policy:** window size, target ratios, hook cadence, anti-starvation per subplot.
* **Theme Proof:** every scene that “proves” the theme must quote its proof lines.
* **Hook Patterns:** reveal | timer | twist | decision | arrival (selection rules).
* **Continuity Minimalism:** Continuity Snippet ≤50 words; retrieval = recent few + top-K salient canon.
* **Symbol Recurrence:** when symbols must recur (arc milestones, fate-turns).
* **Mythic Audit:** checkpoints & remediation patterns.

*(Each BIS is a reusable instruction block—pure prompt.)*

---

## 3) Memory discipline (no bloat)

* **Continuity Snippet (≤50 words):** `arc · stakes · thread · motif · flags(promise:/timer:/symbol:)`.
* **Retrieve narrowly:** last 3–5 chapter summaries + top-K canon/entity facts filtered by current arc/thread.
* **Always summarize:** every chapter yields an 80-word summary + snippet → becomes the next episode’s scaffolding.

---

## 4) Chapter Cycle — the runbook (checklist)

1. **Pacing hint** (class + template)
2. **Fractal** proposes top tensions (depth ≤2)
3. **Spinner** (ToT) outputs ChapterPlan (beats, proves-map, hook)
4. **Weaver** (CoT) drafts scene using snippet+retrieval; ends with hook
5. **Canonist** merges deltas; updates facts + snippet
6. **Stylist** polishes to market voice
7. **QA** tests; if fail → targeted fix prompt; if pass → **Release**
8. **Telemetry** updates pacing/fractal dashboards

---

## 5) Failure modes → prompt safeguards

* **Continuity drift** → Canonist gate + Theme-Proof BIS + Symbol Keeper + periodic Mythic Audit.
* **Boredom/flatness** → Pacing Policy nudges; Fractal novelty score; “pattern-breaker” hook rotation.
* **Prompt sprawl** → Name/version prompts; deprecate monthly; limit active plugins per arc.
* **Ripple explosion** → Fractal depth ≤2; candidate cap; timeboxed branching.
* **Mechanical cadence** → Pacing “nudges, not dictates”; allow human override on junctions.

---

## 6) Prompt templates (copy/paste skeletons)

### Persona (role)

> **You are {{ROLE}}.** Priorities: {{P1}} > {{P2}} > {{P3}}.
> **Inputs:** {{ARTIFACTS}}. **Honor:** Continuity Snippet; retrieved canon.
> **Must produce:** {{ARTIFACT}} with keys: {{FIELDS}}.
> **Guardrails:** don’t contradict canon; end with {{HOOK\_TYPES}} when applicable.

### BIS (policy block)

> **Policy:** {{WHAT}}
> **Process:** {{STEPS}}
> **Constraints:** {{CAPS / RATIOS / LIMITS}}
> **Quality bars:** {{TESTS OR PROOFS REQUIRED}}

### Extension / Plugin (hooked behavior)

> **Hook:** {{preplan|postplan|predraft|postdraft|qa|periodic}}
> **Action:** {{WHAT IT INJECTS OR CHECKS}}
> **Limits:** {{DEPTH|K|FREQUENCY}}

### Node wrapper (task framing)

> **Task:** {{NODE PURPOSE}}
> **Compose:** Persona({{ID}}) + BIS({{IDS}}) + Ext({{IDS}})
> **Return strictly:** {{ARTIFACT FIELDS}}
> **If uncertain:** branch → score → prune → choose 1.

---

## 7) “Where thinking frameworks live” (so you don’t overuse them)

* **FPT** → Worldbuilder + Thematic Law derivations.
* **HTN** → Arc Architect only.
* **ToT** → Spinner at junctions (not every chapter).
* **CoT** → Weaver micro-execution.

---

## 8) Minimal governance (prompt-only phrasing)

* **Name & version everything**: `persona.scene-weaver@1.1.0`, `bis.pacing@1.0.0`.
* **Golden examples**: keep 2–3 input/output pairs per prompt; re-run before updating.
* **A/B safe switch**: on junctions, try `spinner@1.0` vs `spinner@1.1` and keep the better plan.

---

## 9) What’s still yours to tune (knobs)

* Pacing window & ratios (start 9 with 4:3:2).
* Fractal scoring weights (relevance/novelty/market-fit).
* Symbol cadence density (light vs mythic-heavy).
* Which plugins are “always on” vs “arc-local.”

---

### Why this looks lighter than code (and still robust)

* **Everything above is prompts** you can paste into your registry.
* The “code” you saw earlier is *just the pulley system*: loading prompts, stitching them together, validating the shape of what comes back, and saving memory. You can keep that invisible and keep working at the BIS/Persona/Node layer—exactly like CodeOrchard.

If you want, I can recast **two full example prompts** (e.g., *Narrative Spinner* + *Scene Weaver*) in BIS style using your tone, and a 1-page **Chapter Cycle Card** you can literally run by hand or wire later. Say the word when you’re ready. 🙄

---------------------------------------------------------------------------

## the sanity check (are we sensible? has this been done?)

* **yes, sensible.** your shape matches real patterns: multi-stage longform (outline → plan → draft → revise), retrieval memory, periodic QA. research patterns like *StoryWriter/Agents rooms/workflows* echo this.
* **prompt-as-system is viable.** many teams treat prompts as modules/tools with versioning; fewer do it rigorously across an entire factory—but it’s feasible when you give prompts **I/O contracts, tests, and logs** (same discipline as code).
* **where we slipped:**

  1. we let **plugins lead architecture** (symbol/mythic/etc.) before the core loop was stable.
  2. we blurred **persona vs node** (a role is *who* decides; a node is *when/why* it runs with a specific I/O).
  3. we never **froze artifacts** (the shapes of MarketSpec, ArcTree, ChapterPlan, ChapterDraft, CanonMergeReport, QAReport, ContinuitySnippet). without these, every prompt change ripples.

best-practices verdict: **minimize nodes, fix the artifact shapes, limit ToT to forks, keep retrieval lean, and gate new plugins on concrete failure symptoms.**

---

## first-principles artifacts

Lock these **exact** outputs. don’t touch them for the MVP:

1. `MarketSpec { genre, tropes[], taboos[], cadence, cliffCadence }`
2. `ArcTree { nodes: ArcNode[], edges:[from,to,"causes"|"leadsTo"] }`
3. `ChapterPlan { no, class:D|E|A, beats[], provesMap, hookType }`
4. `ChapterDraft { no, text, hooks[], deltas[], snippet }`
5. `CanonMergeReport { approvedFacts[], conflicts[], newSnippet }`
6. `QAReport { pass|fail, issues[], fixPrompts[] }`
7. `ContinuitySnippet` (≤50 words: `arc·stakes·thread·motif·flags[]`)

> if a prompt wants to “improve” things, it must still return these shapes.

---

## Hierarchical Tasks Network cut: the **core four** personas

Each component has **one reason to exist**.

1. **Narrative Spinner** — *choose plan*

   * outputs: `ChapterPlan`
   * thinking: **Tree of Thought thinking framework at junctions** (branch→score→prune)

2. **Scene Weaver** — *write scene*

   * outputs: `ChapterDraft`
   * thinking: **CoT** (beat-by-beat), **retrieval lean** (last 3–5 summaries + top-K canon), obey `ContinuitySnippet`

3. **Canonist** — *merge truth*

   * outputs: `CanonMergeReport`
   * thinking: “approve vs retcon pitch,” compress new `ContinuitySnippet`

4. **QA Gatekeeper** — *guard rails*

   * outputs: `QAReport`
   * tests: continuity, pacing window, theme proof (quote lines), hook live, novelty

**two tiny helpers (not full nodes):**

* **Pacing Orchestrator** → emits the next `class D|E|A` + template hint
* **Market Scout (bootstrap)** → one-time `MarketSpec`, occasional refresh

> that’s it for MVP. **Worldbuilder, Symbol Keeper, Mythic Auditor, Fractal depth-2**, etc. are *earned* later.

---

## where the misalignment came from (and how to fix)

### 1) **ambiguous boundaries**

* **symptom:** changing Spinner prompt breaks Weaver.
* **fix:** **artifact discipline**—Spinner *only* emits `ChapterPlan`; Weaver *only* reads that + snippet + RAG; no hidden side-effects.

### 2) **ToT everywhere**

* **symptom:** latency/cost spikes, erratic outputs.
* **fix:** ToT **only** in Spinner (junctions). everywhere else is CoT.

### 3) **plugin-first design**

* **symptom:** symbol/mythic/trope tools demand extra fields and timing.
* **fix:** gate plugins with a **symptom→remedy** table (below). add one plugin at a time.

### 4) **memory bloat**

* **symptom:** contradictions, repetition, confused tone.
* **fix:** strict **ContinuitySnippet** + recent summaries + top-K canon; nothing else. summarize every chapter to 80 words.

---

## symptom → remedy (when to add features)

* **readers bored by ch 15** → **Pacing Orchestrator** (window 9, 4:3:2), **Novelty Guard** (phrase/beat diversity)
* **cliffhangers feel samey** → **Hook Patterns BIS** (rotate reveal|timer|twist|decision|arrival)
* **recurring contradictions** → tighten **Canonist** (auto “retcon pitch” prompts), ensure Weaver always ingests snippet
* **theme feels mushy** → add **Theme Keeper BIS** inside QA (extract proof quotes or fail)
* **epic loses shape > ch 50** → add **Mythic Auditor** (periodic)
* **motifs vanish** → **Symbol Keeper** plugin (schedule recurrences at fate-turns)
* **plot meanders** → enable **Fractal Tension depth-1** (direct consequence) before trying depth-2 ripples

---

## prompt-native BIS you *actually* need at MVP

Use these as the only reusable instruction blocks for now:

1. **Chapter Planning (ToT @ junctions)** — options→score→prune; each beat “proves” {Essence|Theme|Conflict}; select hook
2. **Scene Realization (CoT)** — draft by beats; quote \[PROOF] lines; end with live hook; no canon contradictions
3. **Canon Merge** — approve vs conflict; create retcon pitch if needed; compress snippet ≤50 words
4. **QA Core** — continuity test, pacing window check, hook classifier “live?”, novelty guard

*(Pacing hint is a tiny helper BIS; not a full node.)*

---

## the decision rails (ToT to choose scope)

Ask these at any proposed change:

* **Does this change one of the 7 artifacts?** if yes, **don’t do it** (until next version).
* **Is there a failing symptom it cures?** if no, **park it**.
* **Can it run as a plugin with no new fields?** if yes, add as **extension**, not a core node.
* **Does it require ToT?** if not a junction, **no ToT**.

---

## “how to think” (keep frameworks natural, not theater)

* **first principles**: start from the three atoms (Essence, Theme, Conflict). if a beat doesn’t prove one, it’s fluff.
* **hierarchical tasks**: endgame → arcs → chapter slot. plan at the level where choice matters; don’t over-plan prose.
* **tree thinking**: only when choosing among chapter plans; more than 5 branches = you aren’t scoring properly.
* **chain steps**: when writing, spell beats and execute in order; nothing fancy.

---

## “back to sensible” plan (48–72h)

**phase 0 (freeze)**: lock the 7 artifacts; pick the 4 MVP nodes + pacing helper.
**phase 1 (seed prompts)**: write Persona+BIS for Spinner/Weaver/Canonist/QA using the above contracts.
**phase 2 (5-chapter shakedown)**: generate 5 chapters; log failures precisely (continuity, pacing, hook, novelty).
**phase 3 (targeted add-ons)**: add **one** plugin only if a symptom demands it (usually Pacing hint first).
**phase 4 (scale test to 20 chapters)**: if contradictions recur → strengthen Canon Merge; if bored → enable Hook Patterns/Novelty Guard; only then consider Fractal depth-1.

---

## tl;dr

you didn’t “go wrong”—you went **wide** before **freezing contracts**. sensibility = **core four nodes, 7 frozen artifacts, ToT only at forks, lean retrieval, plugin gates based on symptoms**. once that loop is stable, re-introduce the rich pieces (symbols, mythic, ripple depth-2) one at a time.

----------------------------------------------------------------------------------------

great pivot. let’s lock the **Saga Loom Codex**—a MongoDB layout + access recipes that keep context sharp, cheap, and clean across **many simultaneous stories**.

---

# 0) Principles (so it never turns to mush)

* **Story-scoped everything:** every doc carries `storyId` (and often `arcId/threadId`). All reads **must** filter by `storyId`.
* **Pyramidal memory:** **hot** (last N), **warm** (block summaries), **cold** (approved canon). Retrieval blends a tiny slice of each.
* **Small, typed artifacts:** we keep the 7 frozen artifacts (Plan/Draft/Canon/QA/etc.). No ad-hoc blobs.
* **Hybrid search:** vector for “what’s semantically related,” Atlas FTS for exact names/dates/terms, plus filters (story, arc, entity).
* **Write once, derive many:** the draft produces deltas → Canonist approves → Codex updates summaries/snippet; everything else derives from those.

---

# 1) Collections (schema sketch) — one DB for all stories

> Every collection has `storyId` and `createdAt/updatedAt`. Vector fields carry an `embeddingVersion`.

### a) Stories & Settings

* `stories` → `{ _id: storyId, title, status, marketSpecId, config { pacing:{window,ratio}, retrieval:{recentN,topK}, embeddingVersion } }`
* `market_specs` → `{ _id, storyId, genre, tropes[], taboos[], cadence, cliffCadence }`

### b) Chapters & Summaries (hot/warm memory)

* `chapters` → `{ _id, storyId, no, arcId, text, planId, draftMeta { hookType, class }, publishedAt }`
* `chapter_summaries` (vector+FTS) → `{ _id, storyId, no, arcId, beats[], summary80w, continuitySnippet, embed[] }`
* `block_summaries` (warm tier) → `{ _id, storyId, block: {start,end}, arcs[], summary200w, embed[] }`

### c) Canon & Entities (cold memory)

* `canon_facts` (vector) → `{ _id, storyId, subjectRef, statement, sourceChapter, validity:"proposed|approved|deprecated", signedBy, version, embed[] }`
* `entities` (vector) → `{ _id, storyId, type:"character|location|faction|item|secret", names[], traits, relations[], status, embed[] }`
* `symbols` (FTS) → `{ _id, storyId, name, meaning, appearances[{chapterNo, context}], fateTurns[] }`

### d) Plans, QA, Ops

* `plans` → `{ _id, storyId, chapterNo, class, beats[], provesMap, hookType }`
* `qa_reports` → `{ _id, storyId, chapterNo, pass:boolean, issues[], fixPrompts[] }`
* `events_change_log` → `{ _id, storyId, targetRef, patch, authorAgent, ts }`
* `evals` → `{ _id, storyId, chapterNo, coherence, pacing, themeScore, novelty, notes }`

### e) Prompt Registry (prompt-as-system)

* `prompts` → `{ _id:"persona.scene-weaver@1.1.0", kind:"persona|bis|extension|node", name, version, template, paramsSchema, lineage[], tags[] }`

> **Multi-story containment:** `storyId` is mandatory; **all** indexes and vector filters include it, so embeddings from one story never collide with another.

---

# 2) Indexing (the performance backbone)

* **Common filters:** compound btree indexes like `{storyId:1, no:-1}` on `chapter_summaries`; `{storyId:1, subjectRef:1}` on `canon_facts`.
* **Vector indexes (Atlas Search):**

  * `chapter_summaries.embed` → `knnVector` (cosine) with filter on `storyId`.
  * `canon_facts.embed`, `entities.embed` similarly.
* **FTS indexes:** on `chapter_summaries.summary80w`, `entities.names`, `symbols.name/meaning`.

---

# 3) Retrieval recipes (what each agent actually pulls)

## Scene Weaver (write the chapter)

**Goal:** tiny, relevant bundle.

1. **Recent window:** last **N=3–5** summaries
   `find chapter_summaries where storyId=X and no in {n-1…n-N} sort desc`
2. **Salient canon:** vector KNN **K=3** on `canon_facts.embed`, filter `{storyId, validity:"approved"}`, optional filter by `arcId/threadId`
3. **Entity recall:** if the plan mentions characters by name, run FTS exact/phrase query on `entities.names` (filter storyId), fallback to KNN by description
4. **Continuity snippet:** from last Canonist merge (`chapter_summaries.continuitySnippet` for n-1)

> **Prompt contract:** “Use only provided context. Do not invent facts outside canon; if needed, ask Canonist for a retcon pitch.”

## Narrative Spinner (plan next)

* Reads `MarketSpec`, `ArcTree` (optional MVP), latest `PacingHint`, and **Fractal candidates** (if enabled).
* Uses **ToT only if fork detected** (multiple viable tensions or arc branches).

## Canonist (merge truth)

* Upserts to `canon_facts` (approved), deprecates conflicts, updates `chapter_summaries.continuitySnippet` for chapter n.

## QA Gatekeeper (guard rails)

* Pulls current draft + `chapter_summaries` window + claimed `provesMap` + `MarketSpec`; emits `QAReport`.

---

# 4) Write paths (when each collection changes)

**After Spinner:** `plans` (chapter plan document)

**After Weaver:** `chapters` (draft), temporary `canon_facts` as **proposed** in `events_change_log` (optional)

**After Canonist (transaction recommended):**

* Upsert `canon_facts` with `validity:"approved"`
* Update `chapter_summaries`:

  * write `summary80w` and **new** `continuitySnippet`
  * compute and store new `embed[]`
* Append to `events_change_log`

**After QA:** `qa_reports` + `evals`; if fail, fix prompts stored as part of `qa_reports`

> **Use a session/transaction** for the Canonist’s dual-write (facts + summary/snippet) to avoid partial commits.

---

# 5) Multi-story containment (three levels)

* **Level 1 – Single DB, story partition (default):** use `storyId` as a **required filter everywhere**; vector filters include `storyId`. (Best for one author managing many series.)
* **Level 2 – Per-story collections:** if a series explodes in size, shard-heavy collections (`chapters`, `chapter_summaries`) by `storyId`.
* **Level 3 – Per-tenant DBs:** only if you need hard isolation (SaaS with multiple clients). Same schema, separate DBs.

**Scheduler rule:** **one chapter at a time per story** (serialize per-story Canonist/Weaver to avoid canon race conditions). Parallelize **across different stories** freely.

---

# 6) Pyramidal memory (so old stuff stays useful)

* **Hot:** `chapter_summaries` for last **N=3–5** chapters → always included.
* **Warm:** `block_summaries` (e.g., every 10 chapters) → include **1** if it overlaps current arc/thread.
* **Cold:** `canon_facts` (approved) and `entities` → include top-K = 2–3 via vector/FTS.

**Aging policy:** when a block closes (e.g., ch 10), generate `block_summaries.summary200w` from its 10 `summary80w` items; later, older `chapter_summaries` can be skipped in retrieval (but kept on disk for audit).

---

# 7) Embeddings (drift & versioning)

* Store `embeddingVersion` on each doc.
* If you change models (e.g., to higher-dim), **re-embed forward** only when a doc changes; batch backfill when idle.
* Keep **one active version** in queries; filter on `embeddingVersion`.

---

# 8) Access matrix (who reads/writes what)

| Agent                   | Reads                                                            | Writes                                                                               |
| ----------------------- | ---------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| Market Scout            | external + `market_specs`                                        | `market_specs`                                                                       |
| Pacing Orchestrator     | `chapters`, `evals`                                              | emits hint (in-memory or `plans`)                                                    |
| Narrative Spinner       | `market_specs`, `plans (prev)`, `qa_reports (signals)`           | `plans`                                                                              |
| Scene Weaver            | `plans`, `chapter_summaries` (recent), `canon_facts`, `entities` | `chapters` (draft)                                                                   |
| Canonist                | `chapters`, `events_change_log`                                  | `canon_facts (approved)`, `chapter_summaries` (summary+snippet), `events_change_log` |
| QA Gatekeeper           | `chapters`, `chapter_summaries`, `plans`, `market_specs`         | `qa_reports`, `evals`                                                                |
| Symbol/Mythic (plugins) | `symbols`, `chapter_summaries`, `plans`                          | `symbols`, `evals`                                                                   |

---

# 9) Query snippets (conceptual)

**Hybrid recall for Weaver (per story):**

```js
// recent
db.chapter_summaries.find({storyId, no: {$in:[n-1,n-2,n-3]}})
  .sort({no:-1}).project({summary80w:1, continuitySnippet:1, beats:1});

// salient canon (vector search with filter)
$search: {
  knnBeta: { vector: qVec, path: "embed", k: 3, filter: { equals: [{path:"storyId", value: storyId}, {path:"validity", value:"approved"}] } }
}

// name exact (FTS)
$search: { text: { query: "Eris Blackthorn", path: "names", phrase: true }, filter: { equals: [{path:"storyId", value: storyId}] } }
```

---

# 10) Guardrails & ops (so it stays cheap and sane)

* **Strict retrieval caps:** recent N (3–5), canon K (2–3), entities K (2).
* **ContinuitySnippet ≤ 50 words** always.
* **Transactions** around Canonist multi-writes.
* **Backpressure:** if vector query > X ms, fallback to FTS + recency-only.
* **Cost view:** log tokens and vector calls per chapter; flag outliers.
* **Backups/rollbacks:** snapshot `canon_facts` and `chapter_summaries` per 10-chapter block.

---

# 11) Prompts meet DB (clean contracts)

* Prompts **ask for tools**: `get_recent_summaries(n)`, `search_canon(query, k)`, `find_entities(names[])`.
* Tools **enforce scopes**: they take `storyId`, silently add filters, and shrink payloads (no 10k-token dumps).
* Prompts **return typed outputs** only (no surprise fields), which **drive writes** to specific collections.

---

# 12) Multi-story hygiene checklist

* ✅ Every doc: `storyId`
* ✅ Every query: `storyId` filter
* ✅ Every vector/FTS: filter `storyId` first
* ✅ One chapter in-flight per story (serialize Canonist/Weaver)
* ✅ Separate `market_specs` per story; don’t “leak” genre rules across stories
* ✅ Prompt registry is **global**, but you can pin versions **per story** (e.g., `spinner@1.1` for Story A, `@1.0` for Story B)

---

## TL;DR

Use **story-scoped collections** + **pyramidal memory** and **hybrid search** to keep context tiny and on-point. Agents speak to Mongo through a few tight tools that always filter by `storyId`. Keep **one chapter at a time per story** to guard canon. With this, your prompts stay smart, your codex stays lean, and you can juggle as many parallel series as you like without cross-contamination.

-------------------------------------------------------------------------------

💠‍🌐

short answer: you don’t need Mongo for prompts. a **filesystem prompt registry** is simpler, faster to iterate, and totally compatible with the Mastra spine. keep prompts in folders, version them with filenames/git, and load/compose them at run time. we only reach for a DB later if you want live editing, A/B switches without redeploys, or per-tenant UI management.

here’s the **lean update** to SagaLoom reflecting that.

---

# Why files over Mongo (now)

* **Sane dev loop:** edit → git diff → PR review → revert.
* **Determinism:** versions pinned via filenames/tags; builds are reproducible.
* **Zero infra:** no schema/migrations just to ship a wording tweak.
* **Mastra friendly:** steps can load markdown from disk and compile the “prompt sandwich”.

*DB becomes useful later for: live A/B, web editor, analytics on prompt usage, per-story overrides without redeploys.*

---

# Updated architecture (same spine, file-based registry)

```
/prompts/
  persona/
    spinner@1.0.0.md
    weaver@1.0.0.md
    canonist@1.0.0.md
    qa@1.0.0.md
  bis/
    chapter-plan@1.0.0.md
    scene-realization@1.0.0.md
    canon-merge@1.0.0.md
    theme-proof@1.0.0.md
    pacing-policy@1.0.0.md
    novelty-guard@1.0.0.md
  ext/
    symbol-tracker@0.1.0.md
    mythic-audit@0.1.0.md
  node/                 ← optional wrappers, mostly comments/specs
    spinner@1.0.0.md
_manifest.json          ← id→path/version aliases & capability map
stories/
  <storyId>/sagaloom.config.json  ← pins versions per story
```

**Frozen contracts stay frozen:** `MarketSpec, ArcTree, ChapterPlan, ChapterDraft, CanonMergeReport, QAReport, ContinuitySnippet`.

**Mastra spine unchanged:**
`pacingStep → spinnerStep → weaverStep → canonStep → qaStep`
Each step loads its **persona system prompt** + inserts the required **BIS** blocks (from files) + exposes **tools**.

---

# Naming & placeholder conventions (clean + future-proof)

* **IDs:** `persona.spinner`, `BIS.chapter-plan`, `ext.symbol-tracker`.
* **Files:** `spinner@1.0.0.md` (semver in filename).
* **Aliases:** `_manifest.json` can map aliases to versions:

  ```json
  {
    "aliases": {
      "persona.spinner": "persona/spinner@1.0.0.md",
      "BIS.chapter-plan": "bis/chapter-plan@1.0.0.md"
    },
    "capabilities": {
      "ThemeProof": "BIS.theme-proof",
      "HookPatterns": "BIS.hook-patterns"
    }
  }
  ```
* **Includes (compile-time):**

  * `<<INCLUDE:BIS.chapter-plan>>` — paste in the BIS text.
  * `<<CAPABILITY:ThemeProof>>` — resolves via `_manifest.capabilities` (lets you rename later without touching personas).
* **Vars:** mustache style in prompts: `{{POV}}`, `{{tense}}`, etc., filled by the workflow step.

---

# Persona system prompts (as files) — with dynamic BIS hooks

### `prompts/persona/spinner@1.0.0.md`

```md
---
id: persona.spinner
version: 1.0.0
out: ChapterPlan
requires:
  - BIS.chapter-plan>=1.0.0
  - BIS.hook-patterns>=1.0.0
---

You are the **Narrative Spinner**: decisive junction strategist.
Priorities: market-fit > arc progress > novelty > flourish.

Return strictly a ChapterPlan { no, class, beats[], provesMap, hookType }.

Rules:
- Generate options only when a fork exists; otherwise pick the obvious plan.
- Every beat must *prove* one of {CharacterEssence|ThematicLaw|ConflictKernel}.
- Hook type ∈ {reveal|timer|twist|decision|arrival}.

<<INCLUDE:BIS.chapter-plan>>
<<INCLUDE:BIS.hook-patterns>>
```

### `prompts/persona/weaver@1.0.0.md`

```md
---
id: persona.weaver
version: 1.0.0
out: ChapterDraft
requires:
  - BIS.scene-realization>=1.0.0
---

You are the **Scene Weaver**: precise, continuity-first writer.
Obey the ContinuitySnippet and retrieved canon. Mark [PROOF] lines.

Context comes only from tools the step provides; do not invent facts.

Return strictly a ChapterDraft { no, text, hooks[], deltas[], snippet }.

<<INCLUDE:BIS.scene-realization>>
```

*(Canonist and QA similar: include `BIS.canon-merge`, `BIS.theme-proof`, `BIS.pacing-policy`, `BIS.novelty-guard` as needed.)*

---

# BIS blocks (as files)

### `prompts/bis/chapter-plan@1.0.0.md`

```md
Policy: When a fork exists, branch 3–5 plans, score {relevance, novelty, marketFit}, prune to 1.
Map each beat to its *proof* target (Essence|Theme|Conflict).
Select a hook type that creates forward promise.
```

### `prompts/bis/scene-realization@1.0.0.md`

```md
Use: (a) ContinuitySnippet, (b) last {{recentN}} summaries, (c) top-K canon facts/entities.
Draft beat-by-beat; embed 1–3 [PROOF] lines; end with a live hook.
Forbid contradictions; if uncertain, ask for a retcon pitch from Canonist (don’t invent).
```

*(Others similar: `canon-merge`, `theme-proof`, `pacing-policy`, `novelty-guard`.)*

---

# Story-level pinning (per-story config)

`stories/<storyId>/sagaloom.config.json`

```json
{
  "prompts": {
    "persona.spinner": "1.0.0",
    "persona.weaver": "1.0.0",
    "persona.canonist": "1.0.0",
    "persona.qa": "1.0.0",
    "BIS.chapter-plan": "1.0.0",
    "BIS.scene-realization": "1.0.0",
    "BIS.canon-merge": "1.0.0",
    "BIS.theme-proof": "1.0.0",
    "BIS.pacing-policy": "1.0.0"
  },
  "pacing": { "window": 9, "ratio": [4,3,2] },
  "retrieval": { "recentN": 4, "canonK": 3, "entityK": 2 }
}
```

Mastra steps read this file to decide **which files to include**; no DB needed.

---

# Tools remain code; personas *reference* them verbally

* Personas don’t *call* tools—steps expose tools; prompts instruct *when* to use them (Mastra handles the actual tool API).
* Keep tool names stable: `getRecentSummaries`, `searchCanon`, `findEntities`, `saveCanonDeltas`.
* You can still drop dynamic hints in personas: “Use **ThemeProof** capability when a scene claims the theorem.” The step resolves `<<CAPABILITY:ThemeProof>>` to a BIS include.

---

# Do we need to change the blueprint? minimal tweaks only

* Swap “Mongo prompt registry” → **filesystem registry** with `_manifest.json` and per-story `sagaloom.config.json`.
* Keep **frozen contracts** exactly as before.
* Keep the **core four personas** and **five BIS**.
* Add **include/alias** mechanism so your system prompts can reference BIS by id or capability.

**Everything else remains the same.** The Mastra workflow and Mongo **Codex** (chapters, summaries, canon, entities) are unchanged.
