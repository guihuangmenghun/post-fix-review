[**中文**](README.zh-CN.md) | English

# post-fix-review

> **Every bug fix is a learning opportunity. The fix solves the immediate problem; the review prevents the entire class of problems from recurring.**

A Skill that guides AI coding agents through structured self-reflection after fixing bugs. Transforms individual fixes into reusable project knowledge — automatically.

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

Built-in workflow guards:
- **Exemption rule** — purely cosmetic changes (typos, formatting, comment-only) may skip the review with a one-line reason
- **Merge before create** — Q5 searches existing memories first and prefers merging over duplicating, preventing memory bloat

## Installation

### Manual Install

```bash
git clone https://github.com/guihuangmenghun/post-fix-review.git
```

Copy `SKILL.md` into your agent's skill/plugin directory.

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

The template lists the four most common triggers; the Skill itself defines the full trigger list (including fixing test false positives) and the exemption rule.

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

### Cross-Cutting Concern Failures
| Pattern | Description |
|---|---|
| **Wrapper Bypass** | Native fetch/XHR/window.open skips the unified HTTP wrapper, dropping auth/logging |
| **New-tab direct link** | URL opened in a new tab where headers can't be attached; use authenticated fetch -> blob -> objectURL |

### Verification-Design Failures
Defects in *how correctness was evidenced* — the suite stays green while behaviour is wrong.

| Pattern | Description |
|---|---|
| **Round-trip-only evidence** | Reader and writer share one definition, so read(write(x)) == x can never falsify it |
| **Aggregate hides decomposition** | Only a total is asserted, so an ordering/capacity error cancels out and still passes |
| **Asymmetric parallel branches** | Branches that should share a predicate don't; the stricter one's 'nothing found' is misread as 'no data exists' |
| **Bounds living in callers** | The offset/position math itself accepts any index; one caller forgets to check (usually the writer) |
| **Unconditional range write** | A helper rewrites a fixed span without checking who owns those positions, destroying parked data |
| **Guard with no triggering test** | A validator counts as 'done' because it exists; nobody has ever seen it fire |
| **Expectation copied from actual** | Golden files/fixtures generated by the code under test freeze the bug along with the behaviour |

## Real-World Origins

### Case 1: Silent Age-Validation Bypass (backend, origin of this skill)

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

### Case 2: File Preview Bypasses Auth (frontend, Wrapper Bypass)

**The bug:** A file-preview feature (popup text, new-tab PDF) called native `fetch(url)` / `window.open(url)`, bypassing the unified request wrapper — requests went out with no `Authorization` header.

**The review outcome:**
- **Anti-pattern:** Wrapper Bypass — the wrapper owns the auth cross-cutting concern; the special path leaked it.
- **Generalized rule:** any frontend request bypassing the unified HTTP wrapper must explicitly re-attach auth; when a new-tab link can't carry headers, use authenticated fetch -> blob -> objectURL.
- **Spec level:** memory only; a new memory was created.

### Case 3: Round-Trip Green, Data Wrong (Self-Consistency Masking)

**The bug:** A review pass over a mapping/codec layer found six defects while **every existing test was green** — real records decoded as zero or 'absent', an out-of-range index silently aliased the neighbouring region, a fixed-span write deleted data parked inside it, and a guard recorded as 'verified' had never once fired.

**The review outcome:**
- **Anti-pattern:** Self-Consistency Masking (plus Asymmetric Filter → False Absence, Unchecked Index Arithmetic, Unconditional Range Write, Untested Safety Net).
- **Generalized rule:** a test whose two sides share a definition cannot falsify it — require at least one expectation sourced *outside* the code under test (raw bytes, spec, second implementation), plus per-position expectations, physical-capacity limits, bounds inside the single offset function, refuse-or-preserve range writes, and a negative test per guard. Before concluding 'X is absent', diff the predicates of the branch that found data against the branch that didn't.
- **Spec level:** checklist + anti-pattern library; the rule also earns a repo-level standard wherever a mapping/codec layer exists.

> **Meta-lesson:** green tests prove the code agrees with itself, not that it agrees with reality.
> The same trap covers fixtures generated by the code under test, expected values copied from actual
> output, and schemas validated against themselves.

## How It Improves Agent Quality

| Before | After |
|---|---|
| Fix bug -> next task | Fix bug -> 5-question review -> knowledge captured |
| Same bug recurs | Anti-pattern recognized and prevented |
| Knowledge lost between sessions | Memory system retains lessons |
| No coding standard updates | AGENTS.md evolves from real bugs |


---

## Ensure This Skill Gets Called

> **To ensure this skill is invoked every time a bug is fixed, add a mandatory trigger rule to your agent configuration:**

**AGENTS.md / .cursorrules / .clinerules:**

```markdown
## Post-Fix Review (Mandatory)

After fixing ANY bug (P0-P3), the agent MUST invoke the `/post-fix-review` skill
before moving to the next task.
This skill guides structured self-reflection: root cause analysis, anti-pattern
identification, generalization, and knowledge capture.
Do NOT skip this step.
```

The 5 reflection questions are defined inside the Skill itself — your agent config only needs to say **"call it"**. The Skill handles the rest.

Without this explicit trigger, agents will fix bugs and immediately move on — the same pattern **will** recur.

---

## Compatibility

Works with any AI coding agent that supports custom instructions or skills:

- **Qoder CN** - native Skill support (`/post-fix-review`)
- **Cursor** - add to `.cursorrules` or custom commands
- **Windsurf** - add to `AGENTS.md` or Cascade rules
- **GitHub Copilot** - add to `.github/copilot-instructions.md`
- **Cline** - add to `.clinerules`
- **Any agent** - paste the 5 questions into your workflow
- **Language agnostic** - anti-patterns apply to Java, Python, JavaScript, Go, etc.
- **Project agnostic** - no project-specific assumptions

## License

AGPL-3.0
