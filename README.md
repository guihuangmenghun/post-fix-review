[**中文**](README.zh-CN.md) | English

# post-fix-review

> **Every bug fix is a learning opportunity. The fix solves the immediate problem; the review prevents the entire class of problems from recurring.**

A [Qoder CN](https://qoder.cn) Skill that guides AI coding agents through structured self-reflection after fixing bugs. Transforms individual fixes into reusable project knowledge — automatically.

## The Problem

AI coding agents fix bugs efficiently, but they **don't learn from them**. The same class of bugs recurs across sessions because:

1. **Silent failures go unnoticed** — validation skipped when preconditions are unmet, no error reported
2. **Anti-patterns repeat** — empty catch blocks, single-type assumptions, optimistic null handling
3. **Knowledge isn't captured** — each session starts fresh, previous lessons are lost
4. **No systematic review** — agents jump to the next task without reflecting on what went wrong

The result? You fix the same type of bug three times before realizing it's a pattern.

## The Solution

`post-fix-review` forces a **structured 5-question review** after every bug fix:

```
Q1: Root Cause      -> What actually went wrong? (code / design / process level)
Q2: Anti-Pattern    -> What class of mistake does this represent?
Q3: Generalization  -> Can this become a universal coding rule?
Q4: Specification   -> Should this be written into AGENTS.md?
Q5: Memory Update   -> What should the agent remember for next time?
```

Each question has a defined output format, and the skill produces concrete actions:
- **AGENTS.md patches** — new coding standards extracted from the fix
- **Memory updates** — persistent knowledge stored for future sessions
- **Anti-pattern classification** — maps the bug to a known category

## Installation

### For Qoder CN Users

```bash
git clone https://github.com/guihuangmenghun/post-fix-review.git ~/.qoder-cn/skills/post-fix-review
```

Or manually copy `SKILL.md` to `~/.qoder-cn/skills/post-fix-review/SKILL.md`.

### Recommended: Add to AGENTS.md

Add this section to your project's `AGENTS.md` to make the skill mandatory:

```markdown
## Post-Fix Review (Mandatory)

After fixing any bug, the agent MUST invoke `/post-fix-review` to perform
structured self-reflection. This ensures every fix produces reusable knowledge.

**Mandatory triggers:**
1. After fixing any bug (P0-P3)
2. After adding validation/checking logic
3. After writing utility/helper classes
4. After discovering silent failures
```

## Usage

After fixing a bug, invoke the skill:

```
/post-fix-review
```

The agent will walk through all 5 questions and produce a summary:

```markdown
## Post-Fix Review Summary

| Question | Answer |
|---|---|
| Root Cause | extractAttributes() only handled Map type, returned null for String |
| Anti-Pattern | Type Assumption - utility assumed single input type |
| Generalized Rule | Utility methods must support Map -> String(JSON) -> null multi-layer adaptation |
| Spec Level | AGENTS.md (added as Coding Prohibition #1) |
| Memory Action | Created: "Utility JSON Input Type Tolerance" |

### Actions Taken
- [x] AGENTS.md updated (new prohibition added)
- [x] Memory created
- [ ] Checklist item added
```

## Anti-Pattern Reference Library

The skill includes a built-in reference library of common anti-patterns:

### Defensive Programming Failures
| Pattern | Description |
|---|---|
| **Single-type assumption** | Utility only handles one input type, silently fails for others |
| **Null = skip** | Null treated as 'nothing to do' instead of 'error condition' |
| **Default hiding** | Default values mask the absence of real data |

### Error Handling Failures
| Pattern | Description |
|---|---|
| **Empty catch** | Exception swallowed silently with no logging |
| **Catch too broad** | catch(Exception) hides specific failures |
| **No context in log** | log.error('failed') without parameters/IDs |

### Validation Failures
| Pattern | Description |
|---|---|
| **Guard clause wraps entire logic** | if(preconditions){all_validation} skips everything when unmet |
| **Missing precondition check** | No validation that required data exists |
| **Optimistic defaults** | Returns 'pass' when unable to determine result |

### Data Flow Failures
| Pattern | Description |
|---|---|
| **Lossy serialization** | JSON -> Object -> JSON loses type information |
| **Implicit type coercion** | String '60' vs Integer 60 vs Long 60L |
| **Snapshot staleness** | Cached data doesn't reflect current state |

## Real-World Origin

This skill was born from a real production bug in an insurance platform:

**The bug:** A 71-year-old insured person passed age validation even though the product configured max_insured_age=60.

**The silent failure chain:**
```
buildFlowContext() empty catch swallowed exception
  -> insured = null (no log)
    -> ValidateAgeComponent: if(insured != null) condition not met
      -> entire age validation skipped (no error, no log)
        -> 71-year-old passed max_insured_age=60 validation
```

Three anti-patterns combined: Empty Catch + Silent Skip + Default Masking.

The fix produced three universal coding prohibitions now enforced via this skill.

## How It Improves Agent Quality

| Before | After |
|---|---|
| Fix bug -> next task | Fix bug -> 5-question review -> knowledge captured |
| Same bug recurs | Anti-pattern recognized and prevented |
| Knowledge lost between sessions | Memory system retains lessons |
| No coding standard updates | AGENTS.md evolves from real bugs |

## Compatibility

- **Qoder CN Desktop IDE** - full support
- **Any Qoder-compatible agent** - works with any agent supporting the Skill protocol
- **Language agnostic** - anti-patterns apply to Java, Python, JavaScript, Go, etc.
- **Project agnostic** - no project-specific assumptions

## License

AGPL-3.0
