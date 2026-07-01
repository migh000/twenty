# Twenty - AI-DLC v2 Agent Documentation

This project uses AI-DLC (AI-Driven Development Life Cycle) v2 for structured development. Twenty supports three AI DLC harnesses: Claude Code, Kiro IDE, and Kiro CLI. The workspace shell is located in `aidlc/` at the repository root, and each harness has its own configuration directory (`.claude/` or `.kiro/`).

## Prerequisites

### Required: bun Runtime

The AI-DLC framework requires the bun JavaScript runtime for executing TypeScript hooks, tools, and orchestration scripts.

**Installation:**

```bash
# macOS/Linux (standard method)
curl -fsSL https://bun.sh/install | bash

# Add to PATH (automatically done by installer, but verify):
export PATH="$HOME/.bun/bin:$PATH"

# Alternative: If unzip is not available, use Python to extract
mkdir -p ~/.bun/bin
cd ~/.bun/bin
curl -fsSL "https://github.com/oven-sh/bun/releases/latest/download/bun-linux-$(uname -m).zip" -o bun.zip
python3 -c "import zipfile; zipfile.ZipFile('bun.zip').extractall('.')"
mv bun-linux-$(uname -m)/bun ~/.bun/bin/
chmod +x ~/.bun/bin/bun
rm -rf bun.zip bun-linux-$(uname -m)

# Windows
# Use Windows Subsystem for Linux (WSL) or install via package managers:
# - scoop: scoop install bun
# - chocolatey: choco install bun
```

**Verification:**

```bash
bun --version
```

The `bun` executable must be on your PATH for the non-interactive shells that the harness spawns. These shells source `~/.zshenv` (zsh) or `~/.bashrc` (bash), NOT `~/.zshrc`. Ensure your bun PATH export is in the appropriate shell initialization file.

**Note for CI/CD:** If you plan to use AI-DLC in continuous integration environments, ensure bun is installed in your CI pipeline setup steps.

### Required: AWS Bedrock

All AI-DLC harnesses run on AWS Bedrock and require valid credentials with Anthropic model access.

**Setup:**

1. Configure credentials via the standard credential chain (environment variables, credentials file, config file, or IAM role)
2. Ensure your account has Anthropic Claude models enabled in Bedrock
3. Set the region (default: us-east-1) via environment variable if needed

**Verification:**

```bash
# Check credentials are configured
aws sts get-caller-identity

# Check Bedrock access
aws bedrock list-foundation-models --region us-east-1
```

### Harness-Specific Requirements

#### Claude Code

- Claude Code desktop application or web interface
- AI-DLC integration uses `.claude/` configuration directory
- AI-DLC files are in `.claude/aidlc-*` subdirectories

#### Kiro IDE / Kiro CLI

- Kiro CLI >= 2.6 required
- Configuration in `.kiro/` directory
- Check version: `kiro-cli --version`

## Installation Verification

After installing prerequisites, verify your AI-DLC setup:

### Claude Code

Run the doctor command via bun directly:

```bash
export PATH="$HOME/.bun/bin:$PATH"
bun run ./.claude/aidlc-tools/aidlc-utility.ts doctor
```

### Kiro CLI

```bash
kiro-cli chat
/aidlc --doctor
```

### Kiro IDE

In the IDE chat panel:

```
/aidlc --doctor
```

**Sample Output:**

The doctor command performs comprehensive health checks including bun runtime, workspace shell, hooks configuration, AWS Bedrock access, audit integrity, graph validation, and more. A successful run will show mostly passed checks:

```
AI-DLC Health Check
─────────────────────────────────────
✓  bun installed (required for CLI tools and hooks)
✓  settings.json present
✓  workspace shell ready (.claude/ + aidlc/spaces/default/memory/)
✓  Audit locks: none leaked
✓  Cycle detection: 0 cycles
✓  Scope validation: 9 scopes valid
✓  Graph references: 122 artifacts + edges resolved
─────────────────────────────────────
21 passed, 2 failed
```

Note: Some checks may fail in minimal configurations (e.g., missing hooks integration in settings.json) without blocking basic functionality. The doctor command is primarily a diagnostic tool to surface configuration drift and suggest fixes.

## Twenty Agents vs. AI-DLC Agents

This repository contains two complementary agent systems:

### Twenty Meta-Agents (Existing)

Located in `/mnt/workplace/gitproject/agents/`:

- `explore.md` - Read-only codebase explorer
- `implement.md` - Feature implementation for issue-to-PR workflow
- `critique.md` - Code review agent

These orchestrate the Twenty project's SDLC workflow using skills in `/mnt/workplace/gitproject/skills/` and hooks in `/mnt/workplace/gitproject/hooks/`.

### AI-DLC Domain-Expert Agents (New)

Located in `.claude/aidlc-agents/` (Claude Code) and `.kiro/agents/` (Kiro):

1. **aidlc-product-agent** - Product management, intent capture, market research
2. **aidlc-design-agent** - UX/UI design, mockups, accessibility
3. **aidlc-delivery-agent** - Delivery planning, sprint management
4. **aidlc-architect-agent** - Application architecture, system design
5. **aidlc-aws-platform-agent** - Cloud infrastructure, platform services
6. **aidlc-compliance-agent** - Compliance requirements, regulatory adherence
7. **aidlc-devsecops-agent** - Security practices, threat modeling
8. **aidlc-developer-agent** - Code generation, implementation
9. **aidlc-quality-agent** - Testing strategy, quality assurance
10. **aidlc-pipeline-deploy-agent** - CI/CD pipelines, deployment strategies
11. **aidlc-operations-agent** - Observability, monitoring, incident response

These implement the 32-stage AI-DLC methodology across 5 phases: Initialization, Ideation, Inception, Construction, Operation.

### Coexistence Strategy

Both systems are fully functional and complementary:

- Use **Twenty agents** for standard issue-to-PR workflows
- Use **AI-DLC agents** for comprehensive lifecycle tasks requiring domain expertise
- Both can work together on the same codebase

## Usage

### Invoking AI-DLC

```
/aidlc <description>
```

Example:

```
/aidlc add user authentication with OAuth2
```

### AI-DLC Commands

- `/aidlc <description>` - Start or resume a workflow
- `/aidlc --status` - Check progress
- `/aidlc --stage <slug>` - Jump to a specific stage
- `/aidlc --phase <name>` - Jump to a phase
- `/aidlc --doctor` - Validate installation
- `/aidlc --version` - Print framework version
- `/aidlc --help` - Full usage and examples

### Scopes

AI-DLC automatically detects scope or you can specify:

| Scope | Depth | Use Case |
|-------|-------|----------|
| enterprise | Comprehensive | Full enterprise application |
| feature | Standard | New feature with full lifecycle |
| mvp | Standard | Minimum viable product |
| poc | Minimal | Proof of concept |
| bugfix | Minimal | Bug fix implementation |
| refactor | Minimal | Code refactoring |
| infra | Standard | Infrastructure changes |
| security-patch | Minimal | Security vulnerability fix |
| workshop | Standard | Learning/training |

### Phases and Stages

AI-DLC workflows progress through 32 stages across 5 phases:

1. **INITIALIZATION (0.1-0.3)**: 3 stages - Bootstrap, scaffold, detect workspace
2. **IDEATION (1.1-1.7)**: 7 stages - Intent capture, research, feasibility, scope, mockups
3. **INCEPTION (2.1-2.8)**: 8 stages - Requirements, design, planning
4. **CONSTRUCTION (3.1-3.7)**: 7 stages - Design, code, test, CI
5. **OPERATION (4.1-4.7)**: 7 stages - Deploy, observe, optimize

## AI-DLC Structure in This Repository

### Claude Code Harness

- `.claude/aidlc-agents/` - Agent definitions
- `.claude/aidlc-common/` - Common utilities
- `.claude/aidlc-hooks/` - AI-DLC hooks (separate from Twenty hooks)
- `.claude/aidlc-knowledge/` - Methodology reference
- `.claude/aidlc-rules/` - AI-DLC rules
- `.claude/aidlc-skills/` - AI-DLC skills (separate from Twenty skills)
- `.claude/aidlc-tools/` - CLI tools
- `.claude/scopes/` - Scope definition files
- `.claude/sensors/` - Verification manifests
- `.claude/aidlc-settings.json.ref` - AI-DLC settings reference (not active)
- `.claude/AIDLC-CLAUDE.md` - AI-DLC documentation

### Kiro Harnesses

- `.kiro/agents/` - 11 domain-expert agent JSONs
- `.kiro/hooks/` - Framework hooks
- `.kiro/knowledge/` - Methodology reference
- `.kiro/sensors/` - Verification manifests
- `.kiro/settings/cli.json` - Kiro settings
- `.kiro/skills/` - Orchestrator and stage skills
- `.kiro/tools/` - TypeScript CLI tools

### Workspace Shell (Shared)

- `aidlc/` - Workspace shell root (harness-neutral)
- `aidlc/spaces/default/` - Default space
- `aidlc/spaces/default/memory/` - Configuration layers
- `aidlc/spaces/default/knowledge/` - Team knowledge
- `aidlc/spaces/default/intents/` - Intent records and artifacts

## Git Integration

The `aidlc/` workspace tree should be committed. Per-user cursors and runtime files are gitignored.

Additional `.gitignore` entries for AI-DLC:

- `.kiro/*` (Kiro configuration)
- `.claude/aidlc-*` (follows existing .claude/* ignore pattern)

## Version Information

- AI-DLC Framework: v2 (from https://github.com/awslabs/aidlc-workflows v2 branch HEAD)
- Installed: 2026-07-01
- Harnesses: Claude Code, Kiro IDE, Kiro CLI

## Configuration Notes

### Settings Integration

The AI DLC framework files are installed in `.claude/aidlc-*` subdirectories to preserve the existing `.claude/settings.json` configuration. The Twenty repository maintains a minimal settings.json that works with both the Twenty meta-agent orchestrator and AI DLC harnesses.

The doctor command will flag that settings.json doesn't wire AI DLC hooks. This is by design to maintain compatibility with existing workflows. AI DLC functionality is available via direct tool invocation rather than automatic hooks.

### AWS Bedrock Access

AWS Bedrock access is required for AI DLC agents to function:
- Valid credentials must be configured via the standard AWS SDK credential chain
- Anthropic model access must be enabled in your AWS account
- Appropriate IAM permissions are needed for Bedrock operations
- Default region is us-east-1 unless configured otherwise

The doctor command will verify Bedrock connectivity when credentials are available.

### Schema Validation

The doctor command may report a schema validation failure for missing `.claude/agents` directory. This is expected in the Twenty repository structure where agents are organized in `.claude/aidlc-agents/` to coexist with the existing orchestrator agents. This does not impact AI DLC functionality.

## Support

For AI-DLC framework documentation, see the aidlc-workflows repository docs/.

For Twenty project questions, see CLAUDE.md and project documentation.
