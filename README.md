# demo-implement-design

Cursor Agent Skill：在业务仓库里**按设计稿 / 截图 / 本地 demo / 蓝湖说明**，把 UI **1:1 还原**到生产代码。

完整流程与门禁见 [`SKILL.md`](SKILL.md)（Agent 命中本技能时会按该文件执行）。

---

## 适用场景

| 用本技能 | 改用其他 |
|----------|----------|
| 聊天里附截图、本地 png/jpg | 用户提供 **Figma 链接** 且要走 Figma MCP → [`figma-implement-design`](../figma-implement-design/) |
| 仓库内已有 demo 页 / 路由，要对齐到业务页 | 无参考、只要「好看一点」的创意改版（本技能**不**做） |
| 蓝湖标注、色值/间距文字说明（含粘贴） | |

触发词示例：**跟稿实现**、**还原设计稿**、**对齐 demo**、**像素还原**、**蓝湖转代码**、**截图转 UI**。

---

## 技能位置与加载

本目录即技能根：

```text
%USERPROFILE%\.agents\skills\demo-implement-design\
├── SKILL.md      # Agent 工作流（必读）
└── README.md     # 本文件（给人看的使用说明）
```

Cursor 会从用户 skills 目录发现技能（常见路径：`%USERPROFILE%\.agents\skills\` 或 `%USERPROFILE%\.cursor\skills\`）。若你维护的是另一份合集仓库，可用目录联接指向本目录，例如：

```powershell
cmd /c mklink /J "$HOME\.agents\skills\demo-implement-design" `
  "<你的合集路径>\demo-implement-design"
```

**新开 Agent 对话**后，描述里含上述触发词即可让 Agent 自动选用本技能；也可在 Composer **手动附加** `demo-implement-design` 技能。

---

## 如何使用（开发者 / 产品）

### 1. 准备参考源（至少一种）

| 类型 | 你怎么给 Agent |
|------|----------------|
| **截图** | 在对话中上传图片；说明「以这张为主」 |
| **本地 demo** | 给出仓库内路径或路由，例如 `packages/xxx/src/demo/Bar.tsx`、`/demo/bar` |
| **蓝湖** | 粘贴标注（色值、字号、间距、圆角、状态说明）；或说明「以蓝湖某页为准」并附关键参数 |

参考缺失或互相矛盾时，Agent 会**先问清主参考**，不会猜视觉。

### 2. 说清楚目标页

一句话说清：**改哪个仓库、哪个页面/组件、要和谁对齐**。例如：

```text
按 demo-implement-design：用附件截图还原 nursing-plat-web 里「护理计划列表」页，严格对齐间距和按钮样式。
```

```text
把 nis-web-components 里某某 demo 的表单样式迁到 nursing-plat-web 的 XX 页面，本地 demo 路径：src/demos/form-basic.tsx。
```

### 3. 配色 / 圆角冲突时请回复选项

实现前 Agent 会比对**参考稿**与**项目现有主题**（`@primary-color`、antd 默认圆角等）。若不一致，会发出表格请确认，例如：

```text
配色冲突，请确认后再继续实现：
| 用途 | 设计稿/demo | 现有系统 | 建议选项 |
...
请回复 A/B/C（可逐行指定）。
```

```text
圆角冲突，请确认后再继续实现：
| 控件 | 设计稿/demo | 现有系统/组件默认 | 建议选项 |
...
```

**在你回复之前**，Agent 不会写死冲突色值或冲突圆角；可先搭布局与非冲突样式。你应尽快回复，否则相关样式会一直处于「暂停」状态。

常见结论：

- **A 跟稿**：局部覆盖样式 / `border-radius: 0` 等，与参考一致  
- **B 跟系统**：沿用主题色与组件默认圆角  
- **C 折中**：按你补充的说明（指定 hex 或 px）

### 4. 验收时你怎么查

完成前 Agent 会按 [`SKILL.md`](SKILL.md) 中的对照清单自检（布局、字重、前景色、背景、边框、**Input/Button/卡片圆角**、padding/margin、hover/选中态等）。你可：

- 浏览器里与截图或 demo **并排比对**  
- 重点看：**稿上是直角时必须是 `0`**，不能仍是 antd 默认圆角  
- 若 Agent 声明「参考未给，沿用系统」，确认是否与产品预期一致  

---

## Agent 会做什么（简要流程）

与 [`SKILL.md`](SKILL.md) 一致，顺序大致为：

1. **锁定主参考**（Read 截图 / 读 demo 源码与样式 / 整理蓝湖）  
2. **抽出视觉规格**（含圆角：直角记 `0`，不得当「未给」）  
3. **对照项目主题**（less 变量、邻页、组件库默认）  
4. **配色冲突门禁** → 等你确认  
5. **圆角冲突门禁** → 等你确认  
6. **按项目约定实现**（最小 diff、复用现有组件，必要时样式覆盖）  
7. **对照校验**后再声称完成  

---

## 对话模板（复制即用）

**仅截图：**

```text
按设计稿严格 1:1 实现 [页面名]，主参考是附件截图。仓库：[绝对路径或项目名]。
```

**本地 demo → 业务页：**

```text
demo-implement-design：以 [demo 文件路径] 为主参考，把样式对齐到 [业务页面路径]；冲突色和圆角先发我确认。
```

**蓝湖参数已粘贴：**

```text
根据下面蓝湖标注实现 [功能/UI]，主色/圆角以粘贴内容为准；与系统主题冲突时先问我。
```

---

## 与 figma-implement-design 的区别

| | demo-implement-design | figma-implement-design |
|--|----------------------|------------------------|
| 输入 | 截图 / 本地 demo / 蓝湖 | Figma URL + Figma MCP |
| 色与圆角冲突 | **必须先问**，未回复不猜 | 更倾向项目 design system 策略 |
| 典型工具 | Read 图片、读源码、浏览器核对 | `get_design_context` 等 |

同时给了 Figma 与截图时，在对话里说明**以谁为主**；若选 Figma，应改用 `figma-implement-design`。

---

## 常见问题

| 问题 | 说明 |
|------|------|
| Agent 只搭了结构没上色 | 可能在等配色/圆角确认表；请回复选项 |
| 稿是直角，页面仍有圆角 | 应对 Input/Button 显式 `border-radius: 0`；若仍是默认，继续要求按稿覆盖 |
| 只有布局稿没有色值 | 布局跟稿，色可走系统；Agent 会说明 |
| 多份参考不一致 | 指定**主参考**一份 |
| 想改全局主题 token | Agent 可能单独提问；确认后再改全局 less |

---

## 维护说明

- 行为变更：改 [`SKILL.md`](SKILL.md)，并保持与本文「使用方式」一致。  
- 本仓库若启用序号提交：见 [`.cursor/rules/sequential-commit-msg.mdc`](.cursor/rules/sequential-commit-msg.mdc)。
