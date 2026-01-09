# Rules Learning Pipeline Plugin

Automated learning pipeline that evaluates model sessions, extracts lessons, detects wasteful patterns, and optimizes CLAUDE.md rules following Anthropic best practices.

## Features

- **Session Evaluation**: Parse action sequences, calculate metrics (F1, Precision, Recall)
- **Wasteful Pattern Detection**: Identify BDN, CIP, Verification Theater, Redundant Tool Chains
- **Dismissive Language Flagging**: Catch "pre-existing", "not related", etc.
- **Scope Classification**: Route lessons to correct rule files
- **Rule Optimization**: Apply peripheral bias, emphasis keywords, line limits
- **Multi-Agent Pipeline**: Coordinated agents with checkpointing and retry logic

## Installation

```bash
# In your project directory
claude

# Install the plugin
/plugin install rules-learning-pipeline
```

Or add to `.claude/settings.json`:

```json
{
  "plugins": ["rules-learning-pipeline"]
}
```

## Commands

### Full Pipeline

```bash
/rules-learning-pipeline:learn-from-session <session-file> [--dry-run] [--verbose]
```

Runs the complete 4-phase pipeline:
1. **Evaluate** (model-evaluator): Parse, metrics, errors, lessons
2. **Review** (critics-reviewer): Validate, audit, catch missed patterns
3. **Optimize** (prompt-optimizer): Route, optimize, write rules
4. **Report**: Generate summary, update changelog

### Individual Phases

```bash
# Phase 1 only
/rules-learning-pipeline:evaluate-model <session-file>

# Phase 2 only
/rules-learning-pipeline:critics-review <lessons-file>

# Phase 3 only
/rules-learning-pipeline:optimize-prompts <lessons-file> [--dry-run]
```

## Agents

| Agent | Purpose |
|-------|---------|
| `model-evaluator` | Parse sessions, calculate metrics, extract lessons |
| `critics-reviewer` | Validate scopes, check specificity, catch missed patterns |
| `prompt-optimizer` | Route lessons, optimize rules, write files |

## Skills

| Skill | Purpose |
|-------|---------|
| `wasteful-pattern-detection` | Detect BDN, CIP, Verification Theater |
| `scope-classification` | Classify lessons into correct scopes |
| `rule-optimization` | Transform lessons into optimized rules |

## Wasteful Patterns Detected

| Pattern | Description |
|---------|-------------|
| **BDN** | Broad check → dismiss → narrow check |
| **CIP** | Check → ignore errors → proceed |
| **Verification Theater** | "verify" → errors → "looks good" |
| **Redundant Tool Chains** | Multiple tools when one suffices |
| **Scope Overkill** | Full suite for single file change |

## Dismissive Language Flagged

- "pre-existing errors" (Critical)
- "not related to my changes" (Critical)
- "already there" (Warning)
- "unrelated issues" (Warning)
- "I only changed X" (Warning)

## Scope Routing

| Scope | Target | Applies To |
|-------|--------|------------|
| user-general | `~/.claude/CLAUDE.md` | Any project |
| cross-project-domain | `~/.claude/rules/*.md` | Framework-specific |
| project | `project/CLAUDE.md` | This project |
| domain | `context/rules/*.md` | Project domain |

## Pipeline Architecture

```
┌──────────────────────────────────────────────────┐
│  INPUT VALIDATION                                 │
│  • Verify file exists                             │
│  • Check content not empty                        │
│  • Create checkpoint directory                    │
└──────────────────────────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────┐
│  PHASE 1: model-evaluator                         │
│  • Parse actions                                  │
│  • Calculate metrics                              │
│  • Detect wasteful patterns                       │
│  • Extract lessons                                │
│  Output: {session}-evaluation.md, lessons-raw.md  │
└──────────────────────────────────────────────────┘
                        │
                        ▼
┌──────────────────────────────────────────────────┐
│  PHASE 2: critics-reviewer                        │
│  • Validate scopes                                │
│  • Check specificity                              │
│  • Catch missed patterns                          │
│  • Return verdict                                 │
│  Output: {session}-critics.md, lessons-validated  │
└──────────────────────────────────────────────────┘
                        │
          ┌─────────────┴─────────────┐
          │                           │
     REJECTED                    APPROVED
     (retry ≤2)                       │
          │                           │
          └───────────┬───────────────┘
                      │
                      ▼
┌──────────────────────────────────────────────────┐
│  PHASE 3: prompt-optimizer                        │
│  • Route by scope                                 │
│  • Apply peripheral bias                          │
│  • Create domain files                            │
│  • Write optimized rules                          │
│  Output: {session}-optimized.md, rule files       │
└──────────────────────────────────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────────────┐
│  PHASE 4: Report                                  │
│  • Generate summary                               │
│  • Update changelog                               │
│  • Clean checkpoints                              │
│  Output: {session}-summary.md                     │
└──────────────────────────────────────────────────┘
```

## Error Recovery

- **Checkpointing**: Each phase saves state to `.claude/checkpoints/`
- **Retry Logic**: Up to 2 retries if critics-reviewer rejects
- **Resume**: Pipeline can resume from last successful checkpoint

## Output Files

| File | Content |
|------|---------|
| `{session}-evaluation.md` | Metrics, errors, root causes |
| `{session}-lessons-raw.md` | Extracted lessons with scopes |
| `{session}-critics.md` | Validation report |
| `{session}-lessons-validated.md` | Corrected lessons |
| `{session}-optimized.md` | Routing decisions |
| `{session}-summary.md` | Final report |

## Requirements

- Claude Code >= 2.0.12
- Task tool access for sub-agent invocation

## License

MIT

## Author

Noreddine
