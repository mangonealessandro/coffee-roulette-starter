# PRD & Claude.md Generation Skills for Claude Code

Two complementary skills to transform ideas into structured technical specifications for use with Claude Code terminal.

## Quick Start

### Installation for Your Repository

```bash
# Copy the skills directory to your project
cp -r skills .claude/

# Or create symlink (if you want to keep them centralized)
ln -s $(pwd)/skills ~/.claude/skills/prd-tools
```

### Verify Installation

```bash
# Check skills are in place
ls .claude/skills/prd-builder/SKILL.md
ls .claude/skills/stack-claude-md-generator/SKILL.md

# Start Claude Code
claude
```

## Repository Structure

```
your-project/
├── .claude/
│   └── skills/           # Skills for this project
│       ├── prd-builder/
│       │   ├── SKILL.md
│       │   └── EXAMPLES.md
│       └── stack-claude-md-generator/
│           ├── SKILL.md
│           ├── EXAMPLES.md
│           ├── MAPPING-REFERENCE.md
│           └── BEST-PRACTICES.md
```

## Skills Included

### 1. prd-builder

Creates Product Requirements Documents (PRDs) that define:
- **What** to build
- **Why** it matters
- **Who** will use it
- **How** to measure success

**Triggers:**
- "I want to build an app for..."
- "Help me write a PRD"
- "We need to define requirements..."

**Output:** `PRD.md` file with complete product specification

### 2. stack-claude-md-generator

Generates component-specific `claude.md` files from your PRD:
- Backend/API documentation
- Frontend/Web documentation
- Mobile app documentation
- Microservices documentation

**Triggers:**
- "Generate claude.md from the PRD"
- "Tech stack: NestJS, Next.js..."
- Any framework mention after completing PRD

**Output:** Separate `claude.md` for each stack component

## Usage Example

### Step 1: Create PRD

```bash
claude

> I want to build an app to match colleagues for coffee breaks

# Claude activates prd-builder skill and guides you through:
# - Discovery (problem, context, constraints)
# - User personas
# - Feature scoping (MVP vs nice-to-have)
# - Success metrics

# Output: PRD.md file
```

### Step 2: Generate Technical Specs

```
> Generate claude.md files from the PRD.
> Tech stack: NestJS + PostgreSQL backend, Next.js frontend

# Claude activates stack-claude-md-generator and creates:
# - api/claude.md (Backend documentation)
# - web/claude.md (Frontend documentation)
```

### Step 3: Develop

```
> Help me implement the auth module based on api/claude.md

# Claude reads api/claude.md and implements following the specs
```

## What Each Skill Contains

### prd-builder/

**SKILL.md** - Main skill with:
- Discovery workflow
- User persona templates
- Feature scoping patterns
- Success metrics templates
- PRD structure

**EXAMPLES.md** - Complete examples:
- Full Coffee Roulette PRD
- Lite PRD for internal tools

### stack-claude-md-generator/

**SKILL.md** - Main skill with:
- Component identification logic
- PRD to architecture mapping
- claude.md structure templates
- Cross-component consistency checks

**EXAMPLES.md** - Full examples:
- Backend claude.md (NestJS + PostgreSQL)
- Frontend claude.md (Next.js + React)

**MAPPING-REFERENCE.md** - How to map:
- PRD features → Backend modules
- User flows → API endpoints
- Data mentions → Database schema
- Success metrics → Observability

**BEST-PRACTICES.md** - Quality guidelines:
- Essential sections for claude.md
- Component-specific requirements
- Cross-component consistency
- Anti-patterns to avoid

## For Your Training Presentation

### Slide Structure Suggestion

1. **Problem**: Developers need clear specs before coding
2. **Solution**: Automated PRD → claude.md generation
3. **Demo**: Live workflow with Coffee Roulette example
4. **Skills Overview**: What each skill does
5. **Installation**: How to add to projects
6. **Practice**: Attendees create PRD for sample project

### Demo Script

```bash
# Terminal 1: Show skills
ls .claude/skills/

# Terminal 2: Start Claude Code
claude

# Walk through:
1. "I want to build [idea]" → PRD generation
2. "Generate claude.md, stack: [tech]" → Tech specs
3. Show generated files side-by-side
4. "Implement [feature] from api/claude.md" → Code generation
```

## Customization for Your Team

### Add Company Templates

```bash
# Add to prd-builder
nano .claude/skills/prd-builder/COMPANY-TEMPLATE.md

# Reference in SKILL.md
```

### Modify Conventions

```bash
# Edit best practices
nano .claude/skills/stack-claude-md-generator/BEST-PRACTICES.md

# Add your team's naming conventions, code style, etc.
```

## Git Workflow

```bash
# Add skills to repository
git add .claude/skills/
git commit -m "Add PRD and claude.md generation skills"
git push

# Team members get skills automatically
git pull
claude  # Skills are now available
```

## Troubleshooting

### Skills don't activate

**Check installation:**
```bash
ls .claude/skills/prd-builder/SKILL.md
```

**Restart Claude Code:**
```bash
# Exit and restart
exit
claude
```

**Use explicit trigger:**
```
> Use prd-builder skill to create a PRD for [project]
```

### PRD too generic

Provide more specific context:
- Concrete numbers (users, timeline, metrics)
- Named personas with real context
- Specific pain points
- Clear constraints

### claude.md incomplete

Ask Claude to expand:
```
> Add 5 troubleshooting scenarios to api/claude.md
> Make the Boundaries section more specific with examples
> Add database indexing strategy to the schema section
```

## Advanced Patterns

### Multi-Repository Projects

```
> Tech stack:
> - Auth service: NestJS microservice
> - API service: NestJS microservice  
> - Admin UI: React SPA
> - User web: Next.js SSR
> - Mobile: React Native

# Generates 5 claude.md files with consistent API contracts
```

### Monorepo Structure

```
project/
├── .claude/skills/        # Shared skills
├── services/
│   ├── auth/
│   │   └── claude.md
│   └── api/
│       └── claude.md
└── apps/
    ├── web/
    │   └── claude.md
    └── mobile/
        └── claude.md
```

## Benefits

✅ **Consistency**: Everyone uses same PRD and claude.md structure  
✅ **Completeness**: No missing sections (setup, troubleshooting, boundaries)  
✅ **Alignment**: PRD features map directly to technical modules  
✅ **Onboarding**: New devs get complete context from claude.md  
✅ **AI-Ready**: Claude Code reads specs and implements correctly  

## License

MIT - Free to use, modify, and distribute

## Support

- Issues: Check EXAMPLES.md for reference implementations
- Questions: Refer to MAPPING-REFERENCE.md and BEST-PRACTICES.md
- Customization: All .md files are editable

---

Built following [Claude Code Skills best practices](https://code.claude.com/docs/en/skills)
