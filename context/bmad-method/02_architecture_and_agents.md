# BMAD Method - Architecture and Agent System

## System Architecture

### Core Framework Design
The BMad Method ecosystem centers around the `bmad-core` directory acting as the system brain, with tools for processing and packaging across different environments.

```
BMad Method Project
├── Core Framework (bmad-core)
│   ├── agents/          # Agent definitions with dependencies
│   ├── agent-teams/     # Bundled agent collections  
│   ├── workflows/       # Prescribed step sequences
│   ├── templates/       # Document generation templates
│   ├── tasks/           # Repeatable action instructions
│   ├── checklists/      # Quality assurance procedures
│   └── data/            # Knowledge base and preferences
├── Tooling
│   └── tools/builders/web-builder.js  # Bundle creation
└── Outputs
    └── dist/            # Generated bundles for web UI
```

## Agent System Architecture

### Core Agents

#### BMad-Master
- **Capability**: Can perform any task of other agents except story implementation
- **Context Management**: Requires conversation compaction for performance
- **Use Case**: Single-agent workflow for simplified interaction
- **Knowledge Access**: Accesses knowledge base to explain BMad Method

#### BMad-Orchestrator  
- **Environment**: Web UI only (heavyweight agent)
- **Purpose**: Facilitates team workflows in web bundles
- **Context**: High context utilization, morphs into other agents
- **Restriction**: Should NOT be used within IDE

#### Planning Agents
- **Analyst**: Brainstorming, market research, competitive analysis, project briefs
- **PM (Product Manager)**: PRD creation from briefs, epic and story definition
- **Architect**: System architecture design from PRD specifications
- **UX Expert**: Front-end specifications, UI prompt generation for tools like Lovable/V0

#### Development Agents
- **PO (Product Owner)**: Document validation, epic/story updates, master checklists, sharding
- **SM (Scrum Master)**: Story drafting with full context from sharded documents
- **Dev**: Sequential task execution, implementation with tests, validation
- **QA**: Senior developer review, active refactoring, code quality improvement

### Agent Dependencies System

#### YAML Structure
Each agent defines its resource dependencies:
```yaml
dependencies:
  templates:
    - prd-template.md
    - user-story-template.md
  tasks:
    - create-doc.md
    - shard-doc.md
  data:
    - bmad-kb.md
```

#### Key Benefits
- **Lean Context**: Agents only load required resources
- **Automatic Resolution**: Dependencies resolved during bundling
- **Resource Sharing**: Consistent templates across agents
- **Modular Design**: Easy to add/remove capabilities

### Agent Teams

#### Team Bundles
- **Purpose**: Pre-configured agent collections for specific domains
- **Examples**: `team-fullstack.yaml`, `team-backend.yaml`
- **Wildcards**: Use `"*"` to include all agents
- **Output**: Single comprehensive bundle for web UI deployment

#### Team Configuration
```yaml
# Example team definition
agents:
  - bmad-master
  - pm
  - architect
  - dev
  - qa
workflows:
  - greenfield-fullstack
```

## Template Processing System

### Core Components

#### template-format.md
- **Purpose**: Foundational markup language specification
- **Features**: Variable substitution (`{{placeholders}}`), AI directives (`[[LLM: instructions]]`), conditional logic
- **Consistency**: Ensures uniform processing across all templates

#### create-doc.md  
- **Role**: Orchestration engine for document generation
- **Functions**: Template selection, user interaction management, validation enforcement
- **Modes**: Incremental vs. rapid generation support

#### advanced-elicitation.md
- **Integration**: Embedded within templates via `[[LLM: instructions]]`
- **Capabilities**: 10 structured brainstorming actions, section review, iterative improvement
- **Purpose**: Interactive refinement layer for content quality

### Template Intelligence
- **Self-Contained**: Templates embed both output format and processing instructions
- **No Separate Tasks**: Many cases require no additional task files
- **AI Processing**: Sophisticated capabilities through embedded intelligence
- **User Transparency**: Processing markup hidden from users

## Build and Delivery Process

### Web Builder (web-builder.js)
1. **Dependency Resolution**: Reads agent/team definitions recursively
2. **Resource Collection**: Finds all dependent tasks, templates, data files  
3. **Content Bundling**: Concatenates files with clear path separators
4. **Output Generation**: Creates single `.txt` file in `dist/` directory

### Environment-Specific Usage

#### IDE Environment
- **Direct Interaction**: Access agents via markdown files in `bmad-core/agents/`
- **IDE Integration**: Native support in Cursor, Claude Code, VS Code extensions
- **Local Resources**: Full access to modular components
- **Development Focus**: Optimized for coding workflows

#### Web UI Environment  
- **Bundle Upload**: Single pre-built file from `dist/`
- **Complete Context**: Entire team and tool knowledge in one file
- **Platform Agnostic**: Works with any web-based AI chat interface
- **Planning Focus**: Optimized for high-level design and strategy

## Configuration Management

### Core Config (bmad-core/core-config.yaml)
```yaml
devLoadAlwaysFiles:
  - docs/architecture/coding-standards.md
  - docs/architecture/tech-stack.md  
  - docs/architecture/project-structure.md
```

#### Purpose
- **Context Control**: Defines files dev agent always loads
- **Consistency**: Ensures uniform rule application
- **Optimization**: Keeps essential context lean and focused
- **Evolution**: Should be reduced as code patterns become consistent

### Technical Preferences Integration
- **Global Influence**: Affects all agent recommendations
- **Project Consistency**: Maintains technical alignment
- **Learning Integration**: Captures experience and preferences
- **Web Bundle Support**: Includes preferences in packaged bundles