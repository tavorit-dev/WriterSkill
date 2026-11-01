# 天工 · Skill 初始化设计

- 日期：2026-11-01
- 状态：待确认
- 输入：`docs/prd.md`、`../Gewu-skill/`
- 本轮目标：按 PRD 一次性搭好完整骨架，不实现最小闭环

## 1. 意图

天工（Tiangong）是一个自包含的通用写作 Skill：能够理解创作意图、规划结构、提供写作指导、评审内容、规划插图、自己渲染 SVG 图表，并完成多平台排版。

它不是一个「一次生成几千字」的文章生成器。核心价值在于**辅助作者思考、组织内容、发现问题、改善表达**。

本轮只搭骨架：目录、入口、工作流说明、参考文档、模板、数据结构、继承自 Gewu 的视觉引擎全部就位；不跑一篇完整文章的端到端验证，也不生成示例文章。

## 2. 目标与非目标

### 目标

1. 按 PRD 第六节的模块化架构落地完整目录，不把所有能力塞进一个 `SKILL.md`。
2. 从 `../Gewu-skill/` 直接继承视觉生产线，并完成命名与路径适配。
3. 天工能够独立完成：判断哪里需要图 → 产出结构化插图计划 → 建立 VEM → 渲染 SVG → 校验 → 整合进文章。
4. 写作主线（构思、大纲、协作写作、评审、排版）有可执行的工作流说明和输出契约。
5. 骨架本身可被 Claude Code 作为 skill 加载，克隆后无需额外依赖即可运行视觉引擎。

### 非目标

1. 本轮不验证「从构思到评审一篇技术文章」的完整闭环。
2. 不生成示例文章、示例插图或演示产物。
3. 不实现对外部图片生成、资料检索、事实核查工具的调用，只保留接口约定。
4. 不实现公众号发布 API，只提供排版规则与检查清单。

### 反目标（即使技术可用也算失败）

- 把天工做成「输入主题、输出整篇」的生成器。
- 评审时擅自改写作者观点，或把「评审」和「修改」混成一步。
- 为了让文章或插图看起来完整而编造技术细节、数据、引用。
- 视觉能力变成 Gewu 的重复实现：两边各自维护一套渲染器。

## 3. 职责边界

| 角色 | 负责 |
| --- | --- |
| 天工 | 写作全流程；判断插图需求；插图计划；VEM；SVG 渲染；图文一致性检查；文章排版 |
| 格物（未来形态） | 技术原理的提取、解读与学习辅助；逐步移除绘图与文章编辑能力 |
| 外部工具/Skill | 图片生成、素材搜索、资料检索、事实核查 |

天工对视觉能力是**自包含**的：不依赖格物 skill 是否存在。格物后续删除绘图能力时，天工不受影响。

## 4. 从 Gewu 继承什么、怎么改

| Gewu 资产 | 天工落点 | 改造方式 |
| --- | --- | --- |
| `scripts/render.py`、`vis_check.py` | `scripts/` | 包引用从 `gewu` 改为 `tiangong`；文档字符串改为天工 |
| `scripts/gewu/`（7 个模块） | `scripts/tiangong/` | 目录重命名；内部绝对导入改名；逻辑不改 |
| `templates/*.yaml`（5 个 VEM 示例） | `templates/diagrams/` | 原样复制，避免与文章模板混放 |
| `references/candidate-selection.md` 等 6 篇 | `references/visual/` | 原样复制，仅修正指向天工的路径与名称 |
| `SKILL.md` 中的文章集成章节 | `workflows/publishing.md` + `references/publishing-formats.md` | 从 Hexo 专用扩展为平台分层（Markdown / Hexo / 公众号 / HTML） |
| `tests/test_render.py` | `tests/` | 导入路径改名，断言不变 |
| `evals/evals.json` | `evals/` | 迁移视觉引擎用例并改为天工触发语；新增写作路由用例占位 |
| `LICENSE`（Apache-2.0） | 根目录 | 保留；README 注明视觉引擎来源与许可 |

原则：**能力代码原样继承，只改命名与路径；写作侧是新增内容，不改造 Gewu 的算法。**

## 5. 最终目录

```
Tiangong-Skill/
├── SKILL.md                     # 精简入口：定位、模式识别、工作流路由、加载表
├── README.md                    # 项目介绍、能力、目录、来源与许可
├── CLAUDE.md                    # 仓库维护指南：架构、命令、硬约束
├── LICENSE                      # Apache-2.0（继承自 Gewu）
├── .gitignore
├── docs/
│   ├── prd.md
│   ├── archive/readme-2026-11-01.md   # 旧 readme 归档
│   └── superpowers/specs/2026-11-01-tiangong-skill-design.md
├── references/
│   ├── writing-principles.md    # 写作原则：作者中心、事实分层、反 AI 腔
│   ├── writing-structures.md    # 文章结构库：问题-方案、原理-机制、教程等
│   ├── writing-styles.md        # 风格指南：技术博客、科普、公众号
│   ├── visual-guidelines.md     # 插图判断：何时配图、配什么图、插图计划入口
│   ├── publishing-formats.md    # 平台分层：Markdown / Hexo / 公众号 / HTML
│   └── visual/                  # 继承自 Gewu 的视觉引擎文档
│       ├── candidate-selection.md
│       ├── visual-model-schema.md
│       ├── visual-design-system.md
│       ├── quality-rules.md
│       ├── cover-image.md
│       └── diagram-checklist.md
├── workflows/
│   ├── ideation.md              # 构思与选题
│   ├── outlining.md             # 大纲与结构
│   ├── collaborative-writing.md # 增量式协作写作与写作提示
│   ├── visual-planning.md       # 插图需求 → visual_assets → VEM → SVG
│   ├── review.md                # 六维评审，评审与修改分离
│   └── publishing.md            # 排版、资源引用、产物校验
├── templates/
│   ├── technical-article.md
│   ├── tutorial.md
│   ├── knowledge-sharing.md
│   ├── product-document.md
│   ├── wechat-article.md
│   └── diagrams/                # 5 个可复制的 VEM 示例
├── schemas/
│   ├── writing-plan.yaml        # 文章规划契约
│   ├── review-report.yaml       # 评审报告契约（P0–P3）
│   └── visual-assets.yaml       # 插图计划契约
├── scripts/
│   ├── render.py
│   ├── vis_check.py
│   └── tiangong/                # 继承的渲染管线
│       ├── __init__.py
│       ├── cli.py
│       ├── layout.py
│       ├── templates.py
│       ├── textmetrics.py
│       ├── tokens.py
│       └── validate.py
├── tests/
│   └── test_render.py
├── evals/
│   └── evals.json
└── examples/
    └── README.md                # 说明示例将在后续版本补充
```

## 6. 内容契约（每个文件写什么）

### SKILL.md

只承担路由职责：

1. 定位与适用场景（技术写作、知识创作优先）。
2. 三种工作模式识别：引导创作 / 协作写作 / 内容优化；外加视觉专责（只要求配图）。
3. 当前阶段判断与工作流选择表。
4. 按需加载表：什么情况读哪份 reference / workflow / template / schema。
5. 输出约束：三类结构化产物、六维评审、P0–P3、评审与修改分离、事实分层。
6. 视觉引擎的最小命令入口。

### workflows/

每篇统一结构：`适用场景 / 输入 / 步骤 / 输出 / 质量门槛 / 下一步`。

- `ideation.md`：主题分析、读者与目的、核心观点、差异化角度、素材清单。
- `outlining.md`：结构选型、章节层级、论证路线、开头结尾、大纲自检。
- `collaborative-writing.md`：四类写作提示（引导/追问/补充/纠错）、按段落增量协作、风格保持。
- `visual-planning.md`：插图需求判断 → `visual_assets` 结构化计划 → VEM → 渲染 → 四层验证 → 回填文章。
- `review.md`：六维评审、问题分级、报告结构、修改授权流程。
- `publishing.md`：平台选择、标题摘要、图片引用、产物校验。

### references/

- `writing-principles.md`：作者中心、保留个人风格、事实分层（已验证/合理推断/待确认）、反模板化表达、评审与修改分离。
- `writing-structures.md`：技术博客、原理剖析、教程、知识分享、产品文档、公众号六类结构与选择依据。
- `writing-styles.md`：技术写作、科普、运营文案的语气、句法、术语与篇幅规范。
- `visual-guidelines.md`：内容类型 → 视觉表达的映射；插图必要性判断；与 `references/visual/` 的衔接。
- `publishing-formats.md`：Markdown 通用规范；Hexo 资源目录与裸文件名引用；公众号排版限制；HTML 导出。

### templates/

文章模板提供章节骨架、每节写作目标、常见误区，不提供成文内容。

### schemas/

YAML 契约，含字段说明与示例。`visual-assets.yaml` 与 Gewu 的 VEM 字段保持兼容：插图计划里的 `type: diagram` 项可直接展开为 VEM。

### scripts/ 与 tests/、evals/

代码继承并改名，行为不变。验收标准是原有回归测试全部通过。

## 7. 关键实现决定

以下是本轮采用的判断，可在确认时推翻：

1. **包名改为 `tiangong`。** 保留 `gewu` 包名会让两个仓库后续删除/迁移时互相牵连，也不符合天工自包含的定位。
2. **视觉参考文档放 `references/visual/`。** 与 PRD 指定的五篇写作参考分开，一眼能看出哪些是继承资产、哪些是天工新增。
3. **VEM 示例放 `templates/diagrams/`。** `templates/` 根目录只放文章模板，避免两类模板混淆。
4. **发布能力平台分层。** Hexo 的踩坑经验完整保留，但作为 `hexo` 配置档，而不是默认假设。默认档是通用 Markdown。
5. **插图计划与 VEM 分层。** `visual_assets` 回答「文章哪里需要什么图」，VEM 回答「这张图怎么解释」。一层对作者，一层对渲染器。
6. **本轮不执行完整写作闭环。** 骨架的验收是「文件齐全、路由自洽、继承代码测试通过」，不是「写出一篇文章」。
7. **天工承接纯绘图请求。** 用户已明确「绘图也是天工做」；格物并存期间会与天工的触发词重叠，这是过渡态，随格物移除绘图能力而消失。天工不依赖格物存在。
8. **视觉产物类型词表按 PRD 扩展。** `visual-assets.yaml` 支持 `diagram`（SVG）、`mermaid`、`image`（外部生成/素材）、`screenshot`、`chart`；本轮只有 SVG 有本地渲染器，其余类型输出结构化计划与 prompt，接口约定写在 `schemas/visual-assets.yaml` 与 `references/visual-guidelines.md`。
9. **六种结构对应五套模板。** `writing-structures.md` 覆盖技术博客、原理剖析、教程、知识分享、产品文档、公众号六类；其中「原理剖析」复用 `templates/technical-article.md`，不新增第六套模板，以 PRD 列出的五套为准。
10. **evals 只迁移纯视觉用例。** Gewu 的 0–3 是 Hexo 站点流程用例，迁移会与「通用 Markdown 为默认」冲突；只迁移 4–6 的视觉引擎用例，0–3 改写为写作路由占位用例，不保留 Hexo-only 断言。
11. **能力边界继续显式声明。** 「程序化校验不等于图讲对了机制」「无法读取图片像素」「OCR 只能确认英文渲染」三条保留在 `SKILL.md` 与 `references/visual/quality-rules.md`，交付时区分已验/未验。
12. **结构化产物默认落盘到文章旁的 `.tiangong/`。** 写作计划、评审报告、插图计划、VEM 写入 `<文章目录>/.tiangong/`，跨会话可续；SVG 与外部图片按发布平台的资源目录放（如 Hexo 的 post asset folder）；文章正文只在用户明确授权修改后编辑，评审阶段只读。
13. **旧 readme 归档。** 现有小写 `readme.md` 移到 `docs/archive/readme-2026-11-01.md`，新建 `README.md`；正文以 `docs/prd.md` 为准，不沿用旧稿中的推广链接。

## 8. 验收标准

1. 目录与第 5 节完全一致，无空文件；所有 Markdown/YAML 均有实际内容。
2. `SKILL.md` 的加载路径全部存在且相对路径正确。
3. `python3 tests/test_render.py` 全部通过（继承代码无回归）。
4. `python3 scripts/render.py templates/diagrams/redis-rehash.yaml -o /tmp/tg-check.svg --check` 退出码 0。
5. `python3 scripts/render.py --list` 与 `--schema` 正常输出。
6. README 与 CLAUDE.md 的命令、路径、来源说明与实际一致。
7. 不产生示例文章或演示图片。

## 9. 留给构建者的空间

- 各参考文档的具体篇幅与详略。
- 模板中章节数量的具体建议。
- schema 字段的中文命名细节（保持与 Gewu VEM 的键名兼容）。
- evals 写作用例的具体题目。
