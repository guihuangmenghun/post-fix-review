# post-fix-review

> **每次 Bug 修复都是一次学习机会。修复解决了眼前的 Bug，自省则阻止整类问题再次发生。**

一个通用 Skill，引导 AI 编程助手在修复 Bug 后进行结构化自省。将单次修复转化为可复用的项目知识 —— 自动化。

[English](README.md) | **中文**

## 解决什么问题

AI 编程助手修 Bug 很快，但**不会从 Bug 中学习**。同类 Bug 跨会话反复出现，原因是：

1. **静默失效无人察觉** —— 校验前置条件不满足时直接跳过，不报错
2. **反模式反复出现** —— 空 catch、单一类型假设、乐观 null 处理
3. **知识不会沉淀** —— 每次会话从零开始，上次的教训丢了
4. **缺乏系统性复盘** —— Agent 修完 Bug 直接跳到下一个任务

结果？同类 Bug 修了三次你才意识到这是个模式。

## 怎么解决

`post-fix-review` 在每次 Bug 修复后强制执行 **5 个结构化问题**：

```
Q1: 根因分析     -> 到底哪里出了问题？（代码级 / 设计级 / 流程级）
Q2: 反模式识别   -> 这属于哪类错误？
Q3: 通用化提炼   -> 能否变成通用编码规则？
Q4: 规范提取     -> 是否该写入 AGENTS.md？
Q5: 记忆更新     -> Agent 该记住什么？
```

每个问题有固定的输出格式，Skill 会产出具体动作：
- **AGENTS.md 补丁** —— 从修复中提炼新编码规范
- **记忆更新** —— 持久化知识供未来会话使用
- **反模式分类** —— 将 Bug 映射到已知类别

## 安装

### 手动安装

```bash
git clone https://github.com/guihuangmenghun/post-fix-review.git
```

将 `SKILL.md` 复制到你 Agent 的 skill/plugin 目录。

### 推荐：在 AGENTS.md 中配置触发规则

在你项目的 `AGENTS.md` 中加入触发规则，让 Agent 知道何时调用这个 Skill：

```markdown
## 修复后自省（强制）

Agent 在修复任何 Bug 后必须调用 /post-fix-review Skill，
完成后才能进入下一个任务。Skill 会引导完成全部自省流程。

**触发条件：**
1. 修复任何 Bug 后（P0-P3）
2. 新增校验/拦截逻辑后
3. 编写工具类/Helper 后
4. 发现静默失效问题后
```

## 使用方式

修复 Bug 后，调用 Skill：

```
/post-fix-review
```

Agent 会依次回答 5 个问题，输出总结表格：

```markdown
## Post-Fix Review Summary

| 问题 | 回答 |
|---|---|
| 根因 | extractAttributes() 只处理 Map 类型，String 输入返回 null |
| 反模式 | 类型假设 —— 工具类假设输入只有一种类型 |
| 通用规则 | 工具类方法必须支持 Map -> String(JSON) -> null 多层适配 |
| 规范级别 | AGENTS.md（新增编码禁令 #1） |
| 记忆操作 | 新建："工具类 JSON 输入类型容错规范" |

### 执行动作
- [x] AGENTS.md 已更新（新增禁令）
- [x] 记忆已创建
- [ ] 检查清单项已添加
```

## 反模式参考库

Skill 内置了 12 种常见反模式，分 4 大类：

### 防御性编程失效
| 模式 | 描述 |
|---|---|
| **单一类型假设** | 工具类只处理一种输入类型，其他类型静默失败 |
| **Null = 跳过** | null 被当作"没事做"而非"错误条件" |
| **默认值掩盖** | 默认值隐藏了数据缺失的真相 |

### 错误处理失效
| 模式 | 描述 |
|---|---|
| **空 catch** | 异常被静默吞掉，无日志 |
| **catch 范围过大** | catch(Exception) 隐藏了具体异常 |
| **日志无上下文** | log.error("failed") 不带参数/ID |

### 校验逻辑失效
| 模式 | 描述 |
|---|---|
| **守卫条件包裹全部逻辑** | if(前置条件){全部校验} —— 条件不满足时全部跳过 |
| **缺少前置检查** | 不验证必要数据是否存在 |
| **乐观默认值** | 无法判断时返回"通过" |

### 数据流失效
| 模式 | 描述 |
|---|---|
| **有损序列化** | JSON -> Object -> JSON 丢失类型信息 |
| **隐式类型转换** | String "60" vs Integer 60 vs Long 60L |
| **快照过期** | 缓存/快照数据不反映当前状态 |

## 真实案例

这个 Skill 诞生于一个真实的生产 Bug：

**Bug：** 71 岁被保人通过了年龄校验，但产品配置的 max_insured_age=60。

**静默失效链：**
```
buildFlowContext() 空 catch 吞掉异常
  -> insured = null（无日志）
    -> ValidateAgeComponent: if(insured != null) 条件不满足
      -> 整个年龄校验被跳过（不报错、不日志）
        -> 71 岁通过了 max_insured_age=60 的校验
```

三个反模式叠加：空 catch + 静默跳过 + 默认值掩盖。

修复后提炼出三条通用编码禁令，现通过此 Skill 强制执行。

## 如何提升 Agent 质量

| 修复前 | 修复后 |
|---|---|
| 修 Bug -> 下一个任务 | 修 Bug -> 5 问自省 -> 知识沉淀 |
| 同类 Bug 复发 | 反模式被识别并预防 |
| 跨会话知识丢失 | 记忆系统保留教训 |
| 编码规范不更新 | AGENTS.md 从真实 Bug 中进化 |


---

## 确保精准触发

> **为了保证每次出现问题或 Bug 时能够精准调用这个 Skill，你需要在 Agent 配置文件中显式写明调用规则：**

**AGENTS.md / .cursorrules / .clinerules：**

```markdown
## 修复后自省（强制）

修复任何 Bug（P0-P3）后，Agent 必须调用 `/post-fix-review` Skill 进行结构化自省，
完成后才能进入下一个任务。
该 Skill 会引导 Agent 完成根因分析、反模式识别、通用化提炼和知识沉淀。
禁止跳过此步骤。
```

5 个自省问题定义在 Skill 内部 —— 你的 Agent 配置文件只需要写明 **"调用它"**，Skill 会引导完成剩余流程。

没有这条显式触发规则，Agent 修完 Bug 就会直接跳到下一个任务 —— 同类问题**一定会**再次出现。

---

## 兼容性

适用于任何支持自定义指令或 Skill 的 AI 编程助手：

- **Qoder CN** —— 原生 Skill 支持（`/post-fix-review`）
- **Cursor** —— 添加到 `.cursorrules` 或自定义命令
- **Windsurf** —— 添加到 `AGENTS.md` 或 Cascade 规则
- **GitHub Copilot** —— 添加到 `.github/copilot-instructions.md`
- **Cline** —— 添加到 `.clinerules`
- **任何 Agent** —— 将 5 个问题直接嵌入你的工作流
- **语言无关** —— 反模式适用于 Java、Python、JavaScript、Go 等
- **项目无关** —— Skill 定义中无任何项目特定假设

## 许可证

AGPL-3.0
