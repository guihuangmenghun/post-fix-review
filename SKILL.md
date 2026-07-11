---
name: post-fix-review
description: Post-fix structured self-reflection for development agents. MUST be invoked after fixing any bug, adding validation logic, or writing utility/helper classes. Guides the agent through root cause analysis, anti-pattern identification, and experience extraction to prevent recurrence. Trigger words: bug fix, post-fix, self-reflection, lessons learned, anti-pattern, code review after fix.
---

# Post-Fix Review (修复后自省)

## Overview

A structured self-reflection workflow that development agents execute after completing code fixes. Transforms individual bug fixes into reusable project knowledge through five guided questions and concrete output actions.

**Core principle:** Every bug fix is a learning opportunity. The fix solves the immediate problem; the review prevents the entire class of problems from recurring.

## When to Use (Mandatory Triggers)

Agent MUST invoke this skill after:

1. **Fixing any bug** — regardless of severity (P0–P3)
2. **Adding validation/checking logic** — new interceptors, validators, guards
3. **Writing utility or helper classes** — especially those handling JSON/Object/dynamic types
4. **Discovering silent failures** — any case where code failed without visible error
5. **Fixing test false positives** — when test results were wrong, not the code

## The Five Questions

Execute each question in order. For each, provide a concrete answer based on the fix just completed.

### Q1: Root Cause — What actually went wrong?

Analyze the fix to identify the true root cause. Distinguish between:

- **Code-level cause**: The specific line/pattern that produced the bug
- **Design-level cause**: The architectural decision that allowed the bug to exist
- **Process-level cause**: The development practice that didn't catch it earlier

Output format:
```
代码根因: [具体代码模式描述]
设计根因: [架构/设计决策描述]
流程根因: [为什么没在编码/审查阶段发现]
```

### Q2: Anti-Pattern — What class of mistake does this represent?

Map the root cause to a general anti-pattern category:

| Anti-Pattern | Description | Example |
|---|---|---|
| Silent Skip | Validation silently skipped when preconditions unmet | `if (x != null) { validate }` — skips entirely when x is null |
| Type Assumption | Code assumes input is always one specific type | `if (obj instanceof Map)` — returns null for String input |
| Empty Catch | Exception swallowed with no logging | `catch (Exception e) { // ignore }` |
| Default Masking | Default values hide the real problem | Returns default 70 when config says 60, caller can't tell |
| Optimistic Null | Null treated as "not needed" instead of "error" | `if (data != null) { process }` — null data passes through |
| Cascade Failure | One silent failure triggers another | Empty catch → null param → silent skip → validation bypassed |
| Separation of Concerns | Trigger config contains execution logic | Copying skill's 5 questions into AGENTS.md instead of just saying 'call it' |

Output format:
```
反模式类别: [从表中选择]
具体表现: [在本次修复中的具体体现]
```

### Q3: Generalization — Can this be a universal practice?

Take the specific fix and abstract it into a project-agnostic coding rule:

- Remove all project-specific class names, method names, business terms
- Replace with generic terms (entity, context, validator, service)
- Write ✅ correct and ❌ incorrect code examples using generic names
- The rule should be applicable to ANY Java/Spring/Python/JS project

Output format:
```
通用规则: [一句话描述]
✅ 正确示例: [通用化代码片段]
❌ 错误示例: [通用化代码片段]
适用范围: [什么类型的项目/场景适用]
```

### Q4: Specification — Should this become a forced coding standard?

Decide whether the generalized rule should be:

- **AGENTS.md entry**: Added to project-level coding standards (mandatory for all agents)
- **Checklist item**: Added to pre-coding or pre-review checklists
- **Code template**: Added as a reference pattern for similar future code

Criteria for AGENTS.md entry:
- Would this mistake happen again in this project?
- Is it a pattern that's easy to accidentally write?
- Is the consequence hard to detect (silent failure)?

Output format:
```
建议级别: [AGENTS.md / Checklist / 仅记忆]
理由: [为什么选择这个级别]
建议文本: [要写入 AGENTS.md 的具体内容]
```

### Q5: Memory Update — What should the agent remember?

Decide what to store in the memory system for future sessions:

- **New memory**: A new pattern/rule that doesn't exist yet
- **Update memory**: An existing memory that needs refinement
- **No action**: The lesson is already captured elsewhere

Output format:
```
记忆操作: [新建 / 更新 / 无需]
记忆标题: [如果新建/更新]
记忆内容摘要: [核心要点]
```

## Output Actions

After completing all five questions, execute the appropriate actions:

1. **If Q4 recommends AGENTS.md entry**: Present the proposed text to the user for approval before writing
2. **If Q5 recommends memory action**: Execute the memory create/update
3. **Always**: Summarize the review in a compact table for the user

## Summary Format

Present the final summary as:

```
## Post-Fix Review Summary

| Question | Answer |
|---|---|
| Root Cause | [one-line summary] |
| Anti-Pattern | [category + one-line] |
| Generalized Rule | [one-line rule] |
| Spec Level | [AGENTS.md / Checklist / Memory only] |
| Memory Action | [Created / Updated / None] |

### Actions Taken
- [ ] AGENTS.md updated (if applicable)
- [ ] Memory created/updated (if applicable)
- [ ] Checklist item added (if applicable)
```


## Integration Guide (How to Wire This Skill)

> **Lesson learned from real usage:** We initially wrote config templates that included all 5 questions, requiring users to copy them into AGENTS.md. This was wrong. The correct separation is:

### Correct Pattern (Separation of Concerns)

| Layer | Responsibility | Content |
|---|---|---|
| **Trigger Config** (AGENTS.md / .cursorrules) | When to call | "Invoke /post-fix-review after fixing bugs" |
| **Skill Content** (this file) | What to do | The 5 questions, anti-pattern library, output format |

The trigger config tells the agent **when** to invoke the skill. The skill itself defines **what** happens during invocation. Never mix these two layers.

### ✅ Correct: Config Template for AGENTS.md

`markdown
## Post-Fix Review (Mandatory)

After fixing ANY bug (P0-P3), the agent MUST invoke the /post-fix-review skill
before moving to the next task.
This skill guides structured self-reflection: root cause analysis, anti-pattern
identification, generalization, and knowledge capture.
Do NOT skip this step.
`

Note: The config is **3 lines**. It says "call the skill" — the skill handles the rest.

### ❌ Anti-Pattern: Copying Skill Content into Config

`markdown
## Post-Fix Review (Mandatory)

After fixing ANY bug, answer these 5 questions:
1. Root Cause: What went wrong?
2. Anti-Pattern: What class of mistake?
3. Generalization: Can this be universal?
4. Specification: Should this become a standard?
5. Memory Update: What to remember?
`

**Why this is wrong:**
- Users copy 15+ lines instead of 3 lines
- Skill content updates don't propagate to user configs
- Violates separation of concerns: trigger config now contains execution logic
- The Skill exists precisely so users don't need to know the 5 questions

### The Meta-Lesson

This anti-pattern is itself an instance of a broader principle:

> **When building tools for others, separate "how to invoke" from "what happens inside".**
> Documentation that leaks internal logic into the caller's configuration creates
> maintenance burden and coupling. The caller should know the interface, not the implementation.

This applies to: Skill configs, API wrappers, CLI tools, middleware setup — any layer where invocation and implementation could be conflated.

---
## Anti-Pattern Reference Library

Common anti-patterns to check against during Q2:

### Defensive Programming Failures
- **Single-type assumption**: Utility only handles one input type
- **Null = skip**: Null treated as "nothing to do" instead of "error condition"
- **Default hiding**: Default values mask the absence of real data

### Error Handling Failures
- **Empty catch**: Exception swallowed silently
- **Catch too broad**: `catch (Exception)` hides specific failures
- **No context in log**: `log.error("failed")` without parameters/IDs

### Validation Failures
- **Guard clause wraps entire logic**: `if (preconditions) { all_validation }` — skips everything
- **Missing precondition check**: No validation that required data exists
- **Optimistic defaults**: Returns "pass" when unable to determine result

### Data Flow Failures
- **Lossy serialization**: JSON → Object → JSON loses type information
- **Implicit type coercion**: String "60" vs Integer 60 vs Long 60L
- **Snapshot staleness**: Cached/snapshotted data doesn't reflect current state
