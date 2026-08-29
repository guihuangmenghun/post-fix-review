---
name: post-fix-review
description: Post-fix structured self-reflection for development agents. MUST be invoked after fixing any bug, adding validation logic, or writing utility/helper classes. Guides the agent through root cause analysis, anti-pattern identification, and experience extraction to prevent recurrence. Trigger words: bug fix, post-fix, self-reflection, lessons learned, anti-pattern, code review after fix.
---

# Post-Fix Review (修复后自省)

## Overview

A structured self-reflection workflow that development agents execute after completing code fixes. Transforms individual bug fixes into reusable project knowledge through five guided questions and concrete output actions.

**Core principle:** Every bug fix is a learning opportunity. The fix solves the immediate problem; the review prevents the entire class of problems from recurring.

**Output language:** Respond in the user's conversation language. The Chinese labels in the templates below are canonical examples, not a language requirement.

## When to Use (Mandatory Triggers)

Agent MUST invoke this skill after:

1. **Fixing any bug** — regardless of severity (P0–P3)
2. **Adding validation/checking logic** — new interceptors, validators, guards
3. **Writing utility or helper classes** — especially those handling JSON/Object/dynamic types
4. **Discovering silent failures** — any case where code failed without visible error
5. **Fixing test false positives** — when test results were wrong, not the code

**Exemption:** Purely cosmetic changes (typo fixes, formatting, comment-only edits) may skip the full review; state a one-line reason instead. When in doubt, run the review.

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
| Wrapper Bypass | Special path bypasses the unified wrapper, dropping cross-cutting concerns (auth, logging) | Native `fetch(url)` for file preview bypasses request.ts → no Authorization header |
| Self-Consistency Masking | Reader and writer share one definition, so a round-trip test can never falsify it | `write(x) → read() == x` passes while a wrong mapping makes real data read as 0 |
| Asymmetric Filter → False Absence | Two branches that should be equivalent apply different predicates | Language A's lookup requires "value ≈ key", language B's doesn't → "B has no data" concluded wrongly |
| Unchecked Index Arithmetic | `base + i*stride` computed without bounds on read or write | Out-of-range index silently aliases the neighbouring region — and corrupts it on write |
| Unconditional Range Write | A helper rewrites a fixed span without asking who owns it | "Set the 4 amount fields" clears a slot holding unrelated user data parked there |
| Untested Safety Net | A guard is recorded as done without any test that makes it fire | Validation exists but a desynchronised stream raises an opaque error before it runs |

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

**Merge before create:** Always search the memory system for similar existing memories first. Prefer updating/merging over creating a new one — duplicate memories rot faster than they help.

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

```markdown
## Post-Fix Review (Mandatory)

After fixing ANY bug (P0-P3), the agent MUST invoke the /post-fix-review skill
before moving to the next task. Do NOT skip this step.
```

Note: The config is **3 lines**. It says "call the skill" — the skill handles the rest.

### ❌ Anti-Pattern: Copying Skill Content into Config

```markdown
## Post-Fix Review (Mandatory)

After fixing ANY bug, answer these 5 questions:
1. Root Cause: What went wrong?
2. Anti-Pattern: What class of mistake?
3. Generalization: Can this be universal?
4. Specification: Should this become a standard?
5. Memory Update: What to remember?
```

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

## Real-World Case Studies

### Case 1: Silent Age-Validation Bypass (backend, origin of this skill)

A 71-year-old insured passed `max_insured_age=60` validation. Failure chain: empty catch → `insured = null` (no log) → `if (insured != null)` guard skipped the entire validation → silent pass. Anti-patterns: Empty Catch + Silent Skip + Default Masking. The fix produced three universal coding prohibitions.

### Case 2: File Preview Bypasses Auth (frontend, Wrapper Bypass)

**Q1 Root Cause**
- Code: preview logic used native `fetch(url)` / `window.open(url)`, bypassing the unified request wrapper (`request.ts`) — no `Authorization` header attached
- Design: preview is a "browser-consumed content stream" scenario (popup text, new-tab PDF) that doesn't fit the JSON response wrapper; the need for auth headers went unnoticed during development
- Process: preview was tested at the interface-logic level only, never end-to-end in a real logged-in session

**Q2 Anti-Pattern:** Wrapper Bypass — a Separation-of-Concerns variant. The unified wrapper owns the auth cross-cutting concern; the special path (file-stream preview) bypassed the wrapper and leaked it.

**Q3 Generalized Rule:** Any frontend request that bypasses the unified HTTP wrapper (`fetch`/XHR/`window.open` direct link) must explicitly re-attach auth. When a new-tab direct link cannot carry request headers, switch to "authenticated fetch → blob → open via objectURL".

**Q4 Spec Level:** Memory only (scenario rare in that project; not elevated to AGENTS.md).

**Q5 Memory Action:** New memory created.

### Case 3: Round-Trip Green, Data Wrong (Self-Consistency Masking)

A review pass over a module that maps between a binary/serial form and an in-memory form found
six defects while **every existing test was green** — and one "safety" routine that had been
recorded as verified had never once been observed to fire. Different symptoms, one family of cause.

Instances, stated generically:

- **Direction**: reader and writer shared one hand-maintained mapping table whose order was wrong.
  `write(x) → read() == x` passed, while real input decoded to zero / "absent".
- **Order**: a greedy decomposition iterated that table in *storage* order instead of descending
  magnitude, producing a value above the field's physical capacity — yet "the total round-trips"
  stayed green because the wrong split still summed correctly.
- **False absence**: two branches of an extraction filter that should have been equivalent were not
  (one demanded the value resemble its key, the other had no such rule). "Branch A yields data,
  branch B yields none" was concluded as *the data doesn't exist* — and the work was parked waiting
  for an external input that was never actually needed. The filter was the bug.
- **Index arithmetic**: `base + i*stride` with no bounds check on either side. An out-of-range index
  silently read the neighbouring region, and on write, corrupted it.
- **Range write**: a helper rewrote a fixed span of positions unconditionally, deleting foreign data
  parked inside that span — a pre-existing test exercised the deletion on every single run.
- **Unreachable diagnostic**: an illegal field made the parser consume extra bytes and desynchronise
  the stream, so the guard that existed to explain the problem never ran; the user saw an opaque
  end-of-stream error instead. The guard was "correct" and useless at the same time.

**Q1 Root Cause**
- Code: one shared encode/decode definition plus offset derivation without bounds; a range write with no "do I own this position" precondition; a validity check placed *after* the point where a bad field already shifted the stream.
- Design: correctness was defined as self-consistency (round trip) instead of agreement with a value produced outside the mapping; boundary rules lived in callers rather than in the single place that computes positions.
- Process: earlier "verified ✅" claims were reused as conclusions instead of re-derived; no rule required a guard to be *triggered* by a test, and no rule required a fix to cover the whole transform (direction **and** order **and** capacity) in one pass — so the fix itself needed a second and third round.

**Q2 Anti-Pattern:** Self-Consistency Masking (primary) + Asymmetric Filter → False Absence + Unchecked Index Arithmetic + Unconditional Range Write + Untested Safety Net; Cascade Failure at the diagnostic layer (the guard existed, but a desynchronised stream meant its message never surfaced).

**Q3 Generalized Rule:** A test whose two sides share a definition cannot falsify that definition.
For any mapping / codec / derived-offset / filter-pair code, require at least one expectation whose
value comes from **outside** the code under test (hand-computed from raw bytes, a spec or IDL, a
second independent implementation), plus per-position expectations and physical-capacity limits.
Put bounds in the one function that computes positions. Make range writes refuse-or-preserve foreign
occupants and validate everything before mutating. Treat a guard as unverified until a test makes it
fire with a message naming the offending field. And before concluding "X is absent", diff the
predicates of the branch that found data against the branch that didn't.

```
✅ 正确示例（通用）:
   assert decode(rawSample).count   == 3          // 期望值来自被测代码之外
   assert decode(rawSample).pos[0].kind == KIND_A // 逐位置期望，不只比总量
   assert read(write(x)) == x                     // 往返仍要写，但不能是唯一证据
   assertRaises(() => write(x, overCapacity))     // 超物理上限必须拒绝，而不是写出非法值
   offsetOf(i) { require(0 <= i < count) }        // 边界只存在于唯一一处
   assertRaises(() => parse(corruptInput))        // 安全网必须有"真的会触发"的负向测试
   assert branchA.predicate == branchB.predicate  // 平行分支的判据等价性本身要被断言

❌ 错误示例（通用）:
   assert read(write(x)) == x                     // 表反了也绿
   assert total(decode(sample)) == expectedTotal  // 顺序/容量错误互相抵消，总量照样对
   if (0 <= i < count) write(base + i*stride, v)  // 校验散在调用方，写入函数仍然敞开
   for p in span: write(p, value)                 // 无条件覆写 → 删掉别人停在该位置的数据
   # guard 存在，但没有任何测试让它触发过
   # "B 分支取不到" 被当成 "数据不存在"，从没比较过两个分支的判据
```

适用范围（发散，不限项目形态）：序列化/反序列化、配置读写编码、单位/币种/时区换算、
ORM 与 DTO 映射、多语言与多区域查表、协议与文件格式解析、分页/游标/分片路由的下标计算、
批量"整体保存"接口、ETL 的平行分支、白名单/黑名单与规则引擎，以及任何"由被测代码自己生成测试夹具"的场景。

**Q4 Spec Level:** Checklist + anti-pattern library (this file). Criteria all met: it recurred three
times inside a single session, it is the path of least resistance to write (round-trip assertions are
cheap), and the consequence is silent. Where a repo has a mapping/codec layer, the "external truth +
per-position + capacity" rule also deserves a repo-level standard line.

**Q5 Memory Action:** Update the existing acceptance-gate memory rather than create a new one
(merge-before-create): round-trip cannot prove direction; self-consistency also hides ordering and
capacity errors; unconditional range writes delete data parked in the span; a guard without a
negative test is unverified; and "absent" requires comparing branch predicates first.

**The Meta-Lesson**

> **Green tests prove the code agrees with itself. They do not prove it agrees with reality.**

The same trap appears whenever the assertion and the thing it checks are generated from one source:
fixtures produced by the code under test, expected values copied from actual output, a schema
validated against itself, or a migration tested by round-tripping the same buggy mapping. When the
author of the expectation and the author of the implementation are the same definition, add a second,
independent source of truth — or the test is decoration.

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

### Cross-Cutting Concern Failures
- **Wrapper Bypass**: native fetch/XHR/window.open skips the unified HTTP wrapper, dropping auth/logging
- **New-tab direct link**: URL opened in a new tab where request headers can't be attached; use authenticated fetch → blob → objectURL instead

### Verification-Design Failures
These are defects in *how correctness was evidenced*, not in the production code — which is why the
suite stays green while the behaviour is wrong.

- **Round-trip-only evidence**: reader and writer share one definition, so `read(write(x)) == x`
  can never falsify it. Fix: add an expectation sourced outside the code (raw bytes, spec, second impl).
- **Aggregate hides decomposition**: only a total/sum is asserted, so an ordering or capacity error in
  one component is cancelled out by another. Fix: assert per-position / per-field expectations.
- **Asymmetric parallel branches**: two paths that should apply the same predicate don't, and the
  stricter one reports "nothing found" — which gets recorded as "the data doesn't exist".
  Fix: assert the predicates are equivalent before trusting an absence conclusion.
- **Bounds living in callers**: the offset/position arithmetic itself accepts any index; every caller
  must remember to check, and one forgets (usually the write path). Fix: one checked accessor.
- **Unconditional range write**: a helper rewrites a fixed span without checking who owns those
  positions, destroying parked data — sometimes driven by a long-passing test that performs the
  deletion on every run. Fix: refuse-or-preserve, and validate all before mutating.
- **Guard with no triggering test**: a validator is marked done because it exists. Nobody has ever
  seen it fire, so its placement and its message are both unverified. Fix: negative test per rule.
- **Diagnostic placed downstream of the desynchroniser**: an illegal field shifts the parse position
  before the check runs, so the user gets an opaque generic error instead of the specific one.
  Fix: validate at the point of consumption, not after.
- **Expectation copied from actual output**: golden files / fixtures generated by the code under test,
  which freezes whatever the current behaviour is — including bugs.
