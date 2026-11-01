# 天工 · Skill 骨架搭建实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 按已批准的设计文档，一次性搭出 Tiangong-Skill 的完整目录骨架，并迁移 Gewu 的视觉渲染引擎。

**Architecture:** 写作侧由精简的 `SKILL.md` 做路由，具体方法分散在 `references/`、`workflows/`、`templates/`、`schemas/`；视觉侧原样继承 Gewu 的零依赖 Python 渲染管线，包名改为 `tiangong`。两类内容通过 `visual_assets` 插图计划和 VEM 衔接。

**Tech Stack:** Markdown、YAML、Python 3 标准库；PyYAML 可选（读 YAML），PIL + numpy 仅 `vis_check.py` 需要。

**Spec:** `docs/superpowers/specs/2026-11-01-tiangong-skill-design.md`

## Global Constraints

- 所有面向用户的文档使用简体中文；代码标识符、命令、字段名保持原文。
- 渲染器不引入第三方运行时依赖；PyYAML 缺失时支持 `.json` 形式的 VEM。
- 不手写 SVG 坐标；模型只声明语义关系。
- 字号下限 11px 是强制约束；超限应拆图，不是缩字号。
- CSS 变量必须声明在 `svg` 选择器上，不能写在 `:root`。
- 保留 Apache-2.0 `LICENSE`，README 与 `scripts/tiangong/__init__.py` 注明继承自格物 Gewu。
- 结构化产物默认写入 `<文章目录>/.tiangong/`；SVG/图片按发布平台资源目录放置。
- 文章正文默认只读；只有用户明确授权后才修改。
- 评审与修改分离；事实分「已验证 / 合理推断 / 待确认」三层。
- 本轮不产出示例文章、演示图片或端到端运行记录。
- `SKILL.md` 只负责路由，不写详细写作方法。

## Review Focus

以下五类输入最容易被实现者忽略，每条在对应任务的验证步骤中固定：

1. 用户只贴一段已有文字说「继续写」——应进入协作写作并给提示，不整篇重写。由 Task 7 + evals case 3 固定。
2. 用户说「帮我评审」——应先出报告，不直接改正文。由 Task 7 + evals case 4 固定。
3. 原文没有的技术细节——应标为待确认，不编造。由 Task 6 + evals case 5 固定。
4. Hexo 图片引用带目录前缀或全角冒号——应使用裸文件名并检查产物。由 Task 6 + evals case 7 固定。
5. `--check` 通过被说成「图没问题」——应区分程序化已验与视觉未验。由 Task 9 + 迁移后的 evals case 1 固定。

---

## 文件结构

| 路径 | 职责 |
| --- | --- |
| `SKILL.md` | 入口路由：定位、模式识别、加载表、输出约束、能力边界 |
| `README.md` | 对外介绍、目录、安装、命令、来源与许可 |
| `CLAUDE.md` | 仓库维护指南：架构、命令、硬约束、产物约定 |
| `docs/archive/readme-2026-11-01.md` | 旧 readme 归档 |
| `references/*.md` | 写作原则、结构、风格、视觉指南、发布格式 |
| `references/visual/*.md` | 继承自 Gewu 的视觉引擎文档 |
| `workflows/*.md` | 六条可执行工作流 |
| `templates/*.md` | 五套文章章节骨架 |
| `templates/diagrams/*.yaml` | 五个 VEM 示例 |
| `schemas/*.yaml` | 写作计划、评审报告、插图计划契约 |
| `scripts/tiangong/*.py` | 继承的渲染管线 |
| `tests/test_render.py` | 渲染器回归测试 |
| `evals/evals.json` | 评测用例 |
| `examples/README.md` | 说明示例后续补充 |

---

### Task 1: 仓库基线与旧稿归档

**Files:**
- Move: `readme.md` → `docs/archive/readme-2026-11-01.md`
- Create: `.gitignore`
- Create: `examples/README.md`
- Copy: `../Gewu-skill/LICENSE` → `LICENSE`

**Interfaces:**
- Consumes: 无
- Produces: 根目录基线文件；后续所有任务在此之上创建文件

- [ ] **Step 1: 归档旧 readme**

```bash
mkdir -p docs/archive examples
git mv readme.md docs/archive/readme-2026-11-01.md
```

- [ ] **Step 2: 复制 LICENSE**

```bash
cp ../Gewu-skill/LICENSE LICENSE
```

- [ ] **Step 3: 写 `.gitignore`**

内容必须包含：`__pycache__/`、`*.pyc`、`.DS_Store`、`*.png` 仅在临时目录（不全局忽略正式产物）。不要忽略 `.tiangong/`——是否提交由作者决定。

- [ ] **Step 4: 写 `examples/README.md`**

说明：示例文章与示例插图将在后续版本补充；本轮骨架不生成演示产物。

- [ ] **Step 5: 验证**

```bash
test -f docs/archive/readme-2026-11-01.md && test ! -f readme.md
test -f LICENSE && test -f .gitignore && test -f examples/README.md
find . -maxdepth 2 -name '*.md' -empty -not -path './.git/*'
```

Expected: 前两条返回 0；空文件检查无输出。

- [ ] **Step 6: 提交**

```bash
git add -A
git commit -m "chore: establish repo baseline and archive old readme

Co-Authored-By: Claude Code <noreply@anthropic.com>"
```

---

### Task 2: 迁移视觉渲染引擎

**Files:**
- Copy: `../Gewu-skill/scripts/render.py` → `scripts/render.py`
- Copy: `../Gewu-skill/scripts/vis_check.py` → `scripts/vis_check.py`
- Copy: `../Gewu-skill/scripts/gewu/` → `scripts/tiangong/`
- Copy: `../Gewu-skill/tests/test_render.py` → `tests/test_render.py`
- Modify: `scripts/render.py`、`scripts/tiangong/cli.py`、`scripts/tiangong/__init__.py`、`tests/test_render.py`

**Interfaces:**
- Consumes: Task 1 的仓库基线
- Produces: `tiangong.cli.main() -> int`；`tiangong.templates.render(spec) -> str`；`tiangong.templates.TEMPLATES`；命令行 `python3 scripts/render.py`、`python3 scripts/vis_check.py`

- [ ] **Step 1: 复制并重命名包**

```bash
mkdir -p scripts tests
cp -R ../Gewu-skill/scripts/render.py ../Gewu-skill/scripts/vis_check.py scripts/
cp -R ../Gewu-skill/scripts/gewu scripts/tiangong
find scripts -name '__pycache__' -type d -exec rm -rf {} +
cp ../Gewu-skill/tests/test_render.py tests/
```

- [ ] **Step 2: 改写导入**

精确替换三处：

```python
# scripts/render.py
from tiangong.cli import main

# scripts/tiangong/cli.py
from tiangong import templates, validate

# tests/test_render.py
from tiangong import textmetrics as tm, validate
from tiangong.templates import TEMPLATES, render
```

`scripts/tiangong/templates.py`、`layout.py`、`validate.py` 使用相对导入，不改。

- [ ] **Step 3: 更新文档字符串与归属说明**

- `scripts/render.py` 首行说明改为「天工 Tiangong 渲染器入口。详见 tiangong/cli.py」。
- `scripts/tiangong/__init__.py` 说明改为「天工 Tiangong 的 SVG 渲染与校验工具包（继承自格物 Gewu，Apache-2.0）」，模块清单中的包名保持与实际一致。
- `scripts/tiangong/cli.py` 用法示例中的 `scripts/render.py` 保持不变。
- `tests/test_render.py` 注释中的「本 skill」保留。

- [ ] **Step 4: 跑回归测试**

```bash
python3 tests/test_render.py
```

Expected: 输出 `全部通过`，退出码 0（约 74 项断言）。

- [ ] **Step 5: 跑 CLI 自检**

```bash
python3 scripts/render.py --list
python3 scripts/render.py --schema
```

Expected: `--list` 列出 7 种 `visual_type`；`--schema` 打印 VEM 字段说明；均退出码 0。

- [ ] **Step 6: 提交**

```bash
git add scripts tests
git commit -m "feat: migrate Gewu SVG renderer as tiangong package

Co-Authored-By: Claude Code <noreply@anthropic.com>"
```

---

### Task 3: 迁移视觉文档与 VEM 示例

**Files:**
- Copy: `../Gewu-skill/references/{candidate-selection,visual-model-schema,visual-design-system,quality-rules,cover-image,diagram-checklist}.md` → `references/visual/`
- Copy: `../Gewu-skill/templates/*.yaml` → `templates/diagrams/`
- Modify: `references/visual/visual-design-system.md`；`templates/diagrams/*.yaml` 顶部注释

**Interfaces:**
- Consumes: Task 2 的 `scripts/tiangong/`
- Produces: `references/visual/` 六篇文档；`templates/diagrams/` 五个可渲染 VEM 示例，供 Task 7、Task 9 引用

- [ ] **Step 1: 复制文档与模板**

```bash
mkdir -p references/visual templates/diagrams
cp ../Gewu-skill/references/*.md references/visual/
cp ../Gewu-skill/templates/*.yaml templates/diagrams/
```

- [ ] **Step 2: 修正路径与归属**

- `references/visual/visual-design-system.md` 中 `scripts/gewu/tokens.py` → `scripts/tiangong/tokens.py`。
- 每个 `templates/diagrams/*.yaml` 顶部注释里的渲染命令路径改为 `templates/diagrams/<文件名>.yaml`。
- 六篇文档顶部各加一行归属说明：「继承自格物 Gewu（Apache-2.0），命令路径已适配天工。」
- 不删除文档中的技术内容、评分模型、能力边界表。

- [ ] **Step 3: 验证示例可渲染**

```bash
python3 scripts/render.py templates/diagrams/redis-rehash.yaml -o /tmp/tg-rehash.svg --check
```

Expected: 退出码 0，无 error。

- [ ] **Step 4: 检查全部示例**

```bash
for f in templates/diagrams/*.yaml; do
  python3 scripts/render.py "$f" -o /tmp/$(basename "$f" .yaml).svg --check || exit 1
done
```

Expected: 五个示例全部退出码 0。

- [ ] **Step 5: 提交**

```bash
git add references/visual templates/diagrams
git commit -m "feat: import Gewu visual references and VEM examples

Co-Authored-By: Claude Code <noreply@anthropic.com>"
```

---

### Task 4: 迁移并改写 evals

**Files:**
- Create: `evals/evals.json`

**Interfaces:**
- Consumes: Task 2 的渲染命令；Task 3 的 VEM 示例
- Produces: `evals/evals.json`，`skill_name: "tiangong"`，用例 id 0–7

- [ ] **Step 1: 写入评测配置**

顶层字段：`skill_name`、`version`、`eval_run_config`、`evals`、`known_risks`。`eval_run_config` 说明：视觉用例在临时目录运行，不需要 Hexo 站点。

- [ ] **Step 2: 迁移三条纯视觉用例**

从 Gewu 原文件迁移并按顺序重编号：

| 新 id | 来源 | name |
| --- | --- | --- |
| 0 | 原 4 | candidate-selection |
| 1 | 原 5 | render-and-validate |
| 2 | 原 6 | renderer-rejects-bad-geometry |

保留原断言，删除 Hexo 专用断言。case 1 必须保留「交付说明区分已验与未验」「不得把 --check 通过说成图没问题」。

- [ ] **Step 3: 新增五条写作路由用例**

每条含 `id`、`name`、`prompt`、`expected_output`、`assertions`、`rationale`：

- 3 `collaborative-writing-hints`：贴一段 Redis 文字说「这一节不知道怎么展开」；断言给四类提示、不整篇续写。
- 4 `review-without-editing`：要求评审一篇已有文章；断言先输出 P0–P3 报告、正文未被修改、未获授权不改稿。
- 5 `fact-layering`：素材缺少迁移触发条件；断言标为待确认、不编造具体阈值或版本行为。
- 6 `visual-only-routing`：只说「给这段配张图」；断言进入 visual-planning、先出候选清单和插图计划，不生成正文。
- 7 `publishing-hexo-bare-filename`：要求把文章发到 Hexo 博客；断言使用裸文件名、检查 `find public -type f -empty`、检查 `<img>` 标签。

- [ ] **Step 4: 验证 JSON**

```bash
python3 - <<'PY'
import json
p = "evals/evals.json"
d = json.load(open(p, encoding="utf-8"))
assert d["skill_name"] == "tiangong"
ids = [e["id"] for e in d["evals"]]
assert ids == list(range(8)), ids
print("ok", len(d["evals"]))
PY
```

Expected: `ok 8`。

- [ ] **Step 5: 提交**

```bash
git add evals/evals.json
git commit -m "test: add tiangong evals for writing routes and visual engine

Co-Authored-By: Claude Code <noreply@anthropic.com>"
```

---

### Task 5: 定义 schemas

**Files:**
- Create: `schemas/writing-plan.yaml`
- Create: `schemas/review-report.yaml`
- Create: `schemas/visual-assets.yaml`

**Interfaces:**
- Consumes: 无
- Produces: 三个契约文件的字段名，供 Task 6、Task 7、Task 9 引用

- [ ] **Step 1: 写 `writing-plan.yaml`**

文件结构为「`schema_version` + `fields`（字段、类型、必填、说明）+ `example`」。字段集合固定为：

```yaml
schema_version: 1
fields:
  article.working_title: string, required
  article.genre: technical-article | tutorial | knowledge-sharing | product-document | wechat-article, required
  article.audience: string, required
  article.purpose: string, required
  article.platform: markdown | hexo | wechat | html, required
  article.core_claim: string, required
  structure[].id: string, required
  structure[].heading: string, required
  structure[].goal: string, required
  structure[].key_points: string[], required
  structure[].evidence_needed: string[]
  structure[].status: planned | drafting | drafted | reviewed
  facts.verified: string[]
  facts.inferred: string[]
  facts.unconfirmed: string[]
  visual.plan_file: string（固定为 visual-assets.yaml）
```

`example` 用一篇技术博客演示；所有字段有非空示例值。

- [ ] **Step 2: 写 `review-report.yaml`**

字段集合固定为：

```yaml
schema_version: 1
fields:
  article.path: string, required
  article.title: string
  overall.verdict: pass | revise | block
  overall.summary: string
  issues[].id: string, required
  issues[].dimension: completeness | structure | logic | accuracy | readability | expressiveness, required
  issues[].severity: P0 | P1 | P2 | P3, required
  issues[].location: string（章节或段落定位）
  issues[].problem: string, required
  issues[].suggestion: string, required
  issues[].status: open | accepted | declined | fixed
  facts.verified: string[]
  facts.inferred: string[]
  facts.unconfirmed: string[]
  modification_authorized: false, required
```

示例必须体现「评审阶段 `modification_authorized: false`，正文未被修改」。

- [ ] **Step 3: 写 `visual-assets.yaml`**

字段集合固定为：

```yaml
schema_version: 1
fields:
  article.path: string
  generated_at: string
  assets[].id: string, required
  assets[].type: diagram | mermaid | image | screenshot | chart, required
  assets[].title: string, required
  assets[].position.after_section: string
  assets[].purpose: string, required
  assets[].content_points: string[]
  assets[].style: string
  assets[].format: svg | mmd | png | jpg
  assets[].prompt: string（外部图片生成用）
  assets[].caption: string
  assets[].alt: string
  assets[].status: planned | generated | verified | rejected
  assets[].vem_file: string（type=diagram 时指向 `.tiangong/models/<id>.yaml`）
  assets[].output_path: string
```

`diagram` 类型的示例必须包含与 VEM 兼容的 `visual_type` 可选字段说明；其它类型只写计划与 prompt，不要求渲染。

- [ ] **Step 4: 验证 YAML**

```bash
python3 - <<'PY'
import yaml, pathlib
for p in pathlib.Path("schemas").glob("*.yaml"):
    with open(p, encoding="utf-8") as f:
        d = yaml.safe_load(f)
    assert "fields" in d and "example" in d, p
    print(p, "ok")
PY
```

Expected: 三个文件均输出 `ok`。

- [ ] **Step 5: 提交**

```bash
git add schemas
git commit -m "feat: define writing, review, and visual asset schemas

Co-Authored-By: Claude Code <noreply@anthropic.com>"
```

---

### Task 6: 写五篇写作参考

**Files:**
- Create: `references/writing-principles.md`
- Create: `references/writing-structures.md`
- Create: `references/writing-styles.md`
- Create: `references/visual-guidelines.md`
- Create: `references/publishing-formats.md`

**Interfaces:**
- Consumes: Task 5 的 schema 字段名
- Produces: 被 Task 7 的工作流和 Task 9 的 `SKILL.md` 加载的参考文档

- [ ] **Step 1: 写 `writing-principles.md`**

必须包含七节：作者中心；保留作者风格；事实分层（已验证/合理推断/待确认）；反模板化与反 AI 腔；评审与修改分离；增量协作优先；可验证性。每条给「应当 / 不应」各一个具体例子。

- [ ] **Step 2: 写 `writing-structures.md`**

包含结构选择表（六类：技术博客、原理剖析、教程、知识分享、产品文档、公众号）和每类的章节骨架、适用条件、失败信号。明确「原理剖析」复用 `templates/technical-article.md`。

- [ ] **Step 3: 写 `writing-styles.md`**

包含技术写作、科普/知识、运营/公众号三种语气规范，以及通用禁令（空话、堆砌形容词、伪权威、无信息量的过渡句）。给出「什么时候该保留作者原句」的判断标准。

- [ ] **Step 4: 写 `visual-guidelines.md`**

必须包含：何时需要图；PRD 4.1 的「内容类型 → 视觉表达」映射表；插图计划结构（指向 `schemas/visual-assets.yaml`）；五种视觉类型词表（diagram/mermaid/image/screenshot/chart）；与 `references/visual/` 的衔接（何时进入 VEM）；图注与 Alt Text 写法；风格统一与克制原则；外部图片/图表的接口约定（只输出 prompt 与验收标准，不假装能生成）。

- [ ] **Step 5: 写 `publishing-formats.md`**

必须包含：平台差异表；通用 Markdown 规范；Hexo 配置档（同名资源目录、裸文件名引用、全角冒号导致图片静默消失、`find public -type f -empty` 与 `<img>` 检查）；公众号配置档（排版限制、图片托管、代码块）；HTML 导出；交付前检查清单。

- [ ] **Step 6: 验证交叉引用**

```bash
grep -L "writing-principles\|writing-structures\|writing-styles\|visual-guidelines\|publishing-formats" references/*.md
```

Expected: 不要求每篇互引；用人工检查确认每篇提到的路径都存在：

```bash
for f in $(grep -ho '`[a-z][^`]*\.\(md\|yaml\)`' references/*.md | tr -d '`' | sort -u); do test -e "$f" || echo "MISSING: $f"; done
```

Expected: 无 `MISSING`。

- [ ] **Step 7: 提交**

```bash
git add references/*.md
git commit -m "docs: add writing principles, structures, styles, visual and publishing guides

Co-Authored-By: Claude Code <noreply@anthropic.com>"
```

---

### Task 7: 写六条工作流

**Files:**
- Create: `workflows/ideation.md`
- Create: `workflows/outlining.md`
- Create: `workflows/collaborative-writing.md`
- Create: `workflows/visual-planning.md`
- Create: `workflows/review.md`
- Create: `workflows/publishing.md`

**Interfaces:**
- Consumes: Task 5 的 schemas；Task 6 的 references；Task 3 的 `references/visual/` 与 `templates/diagrams/`
- Produces: 六条工作流，供 Task 9 的 `SKILL.md` 路由

每篇统一六节：`适用场景 / 输入 / 步骤 / 输出 / 质量门槛 / 下一步`。步骤编号可执行，不写泛泛建议。

- [ ] **Step 1: 写 `ideation.md`**

步骤：明确主题与边界 → 读者与目的 → 核心观点 → 差异角度 → 素材与事实清单（三层）→ 用户确认。输出：`<文章目录>/.tiangong/writing-plan.yaml` 的 `article` 与 `facts` 部分。质量门槛：核心观点能用一句话复述；未确认信息不得写成结论。

- [ ] **Step 2: 写 `outlining.md`**

步骤：按 `references/writing-structures.md` 选结构 → 章节目标 → 论证路线 → 开头与结尾 → 自检。输出：更新 `writing-plan.yaml` 的 `structure`。质量门槛：每章有目标且服务于核心观点；章节顺序可被论证路线解释。

- [ ] **Step 3: 写 `collaborative-writing.md`**

必须定义四类提示（引导/追问/补充/纠错）及各自触发条件。步骤：读上下文 → 判断当前段落位置 → 给 2–4 条提示 → 等作者选择 → 只展开被选中的部分。明确禁止：未要求时整篇重写。输出：对话中的提示；必要时更新 `writing-plan.yaml` 的章节 `status`。

- [ ] **Step 4: 写 `visual-planning.md`**

步骤：判断插图需求 → 产出 `visual-assets.yaml` → 用户确认 → 对 `diagram` 类型建立 VEM（放 `.tiangong/models/`）→ 用 `scripts/render.py` 渲染 → `--check` → 四层验证 → 写图注/Alt Text → 经用户授权后插入正文。五种类型的处置必须区分：只有 `diagram` 走本地渲染器；`mermaid` 输出 `.mmd` 与嵌入建议；`image`/`screenshot` 输出 prompt/获取说明；`chart` 输出数据与图表规格。

- [ ] **Step 5: 写 `review.md`**

步骤：声明只读 → 六维检查 → 问题分级 P0–P3 → 写 `.tiangong/review-report.yaml` → 输出摘要 → 等用户决定。`modification_authorized` 未置 true 时禁止编辑正文。质量门槛：每个问题有定位、原因、建议；P0/P1 不得省略。

- [ ] **Step 6: 写 `publishing.md`**

步骤：选平台配置档 → 标题/摘要/导语 → 格式转换 → 图片引用检查 → 产物校验 → 交付。Hexo 档必须包含裸文件名、全角冒号陷阱和两条产物检查命令。公众号档说明不能直接复用 HTML 的限制。

- [ ] **Step 7: 验证结构与交叉引用**

```bash
for f in workflows/*.md; do
  for h in 适用场景 输入 步骤 输出 质量门槛 下一步; do
    grep -q "$h" "$f" || echo "MISSING $h in $f"
  done
done
```

Expected: 无 `MISSING`。

- [ ] **Step 8: 提交**

```bash
git add workflows
git commit -m "docs: add six writing workflows

Co-Authored-By: Claude Code <noreply@anthropic.com>"
```

---

### Task 8: 写五套文章模板

**Files:**
- Create: `templates/technical-article.md`
- Create: `templates/tutorial.md`
- Create: `templates/knowledge-sharing.md`
- Create: `templates/product-document.md`
- Create: `templates/wechat-article.md`

**Interfaces:**
- Consumes: Task 6 的 `references/writing-structures.md`
- Produces: 被 Task 7 的 outlining 和 Task 9 的加载表引用

每套模板包含：适用场景、章节骨架、每节写作目标、常见误区、配图机会。模板只给结构和提示，不写成文。

- [ ] **Step 1: `technical-article.md`**

骨架：标题 → 导语 → 问题/背景 → 原理或机制 → 方案与实现 → 权衡与限制 → 结论。同时承担「原理剖析」结构。

- [ ] **Step 2: `tutorial.md`**

骨架：目标与读者 → 前置条件 → 最终效果 → 分步操作 → 验证 → 排错 → 延伸。每步必须可验证。

- [ ] **Step 3: `knowledge-sharing.md`**

骨架：引子 → 核心概念 → 类比或示例 → 主体展开 → 边界与反例 → 总结。明确类比失效点。

- [ ] **Step 4: `product-document.md`**

骨架：产品定位 → 目标用户 → 使用场景 → 能力说明 → 限制与依赖 → 示例 → FAQ。

- [ ] **Step 5: `wechat-article.md`**

骨架：标题 → 导语钩子 → 分段主体（每段一个判断）→ 视觉节奏 → 结论/行动号召。说明手机端段落长度与图片密度。

- [ ] **Step 6: 验证**

```bash
ls templates/*.md | wc -l
```

Expected: `5`。

- [ ] **Step 7: 提交**

```bash
git add templates/*.md
git commit -m "docs: add five article structure templates

Co-Authored-By: Claude Code <noreply@anthropic.com>"
```

---

### Task 9: 写 `SKILL.md`

**Files:**
- Create: `SKILL.md`

**Interfaces:**
- Consumes: Task 3、5、6、7、8 的全部产物
- Produces: skill 入口；frontmatter `name: tiangong`

- [ ] **Step 1: 写 frontmatter**

```yaml
---
name: tiangong
description: 天工是通用写作助手，辅助构思、大纲、协作写作、评审、插图规划、SVG 图表生成与多平台排版。当用户说「我想写一篇…」「这一节不知道怎么展开」「帮我评审这篇文章」「给这段配个图」「排成公众号格式」时使用。它辅助作者创作而不是替代作者，评审与修改分离，不编造技术细节。
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
---
```

- [ ] **Step 2: 写路由主体**

必须包含以下小节：定位；何时使用 / 何时不用；三种工作模式表；阶段路由表（用户信号 → 模式 → 工作流 → 产物）；按需加载表；产物与落盘约定；输出质量约束；视觉引擎命令；能力边界；文件索引。

阶段路由表至少覆盖：

| 用户信号 | 工作流 | 产物 |
| --- | --- | --- |
| 只有一个想法 | ideation → outlining | `.tiangong/writing-plan.yaml` |
| 已有部分文字 | collaborative-writing | 对话提示 + 计划状态 |
| 要求评审 | review | `.tiangong/review-report.yaml` |
| 要求配图 | visual-planning | `.tiangong/visual-assets.yaml` + VEM/SVG |
| 要求排版发布 | publishing | 目标平台格式产物 |

- [ ] **Step 3: 写输出约束与能力边界**

必须明确：评审与修改分离；事实三层；保留作者风格；`--check` 通过不等于图讲对了机制；无法读取图片像素；OCR 只能确认文字渲染；交付时区分已验/未验。

- [ ] **Step 4: 验证加载路径全部存在**

```bash
python3 - <<'PY'
import re, pathlib
text = pathlib.Path("SKILL.md").read_text(encoding="utf-8")
paths = set(re.findall(r'`((?:references|workflows|templates|schemas|scripts)/[^`]+)`', text))
missing = [p for p in paths if not pathlib.Path(p).exists()]
print("paths:", len(paths), "missing:", missing)
assert not missing
PY
```

Expected: `missing: []`。

- [ ] **Step 5: 提交**

```bash
git add SKILL.md
git commit -m "feat: add tiangong skill entry and routing

Co-Authored-By: Claude Code <noreply@anthropic.com>"
```

---

### Task 10: 写 README 与 CLAUDE.md

**Files:**
- Create: `README.md`
- Create: `CLAUDE.md`

**Interfaces:**
- Consumes: 全部前置任务
- Produces: 对外说明与维护指南

- [ ] **Step 1: 写 `README.md`**

必须包含：一句话定位；核心能力（构思、结构、协作写作、视觉、评审、排版）；三种模式；目录树；安装（`ln -sfn ~/company/open-source/Tiangong-Skill ~/.claude/skills/tiangong`）；常用命令（测试、渲染、校验）；来源与许可（视觉引擎继承自格物 Gewu，Apache-2.0）；本轮不包含的内容。

- [ ] **Step 2: 写 `CLAUDE.md`**

必须包含：仓库性质；目录职责；常用命令；架构数据流（VEM → templates.py → layout.py → tokens.py）；硬约束（不手写 SVG、字号 11px、CSS 变量位置、评审/修改分离、事实三层）；产物落盘约定 `.tiangong/`；能力边界；迭代方式；继承来源与许可。

- [ ] **Step 3: 验证命令与实际路径一致**

```bash
grep -n "python3 tests/test_render.py\|scripts/render.py\|scripts/vis_check.py" README.md CLAUDE.md
test -f tests/test_render.py && test -f scripts/render.py && test -f scripts/vis_check.py
```

Expected: grep 有输出；`test` 返回 0。

- [ ] **Step 4: 提交**

```bash
git add README.md CLAUDE.md
git commit -m "docs: add project readme and maintenance guide

Co-Authored-By: Claude Code <noreply@anthropic.com>"
```

---

### Task 11: 最终骨架校验

**Files:**
- 无新增；只验证与修正

**Interfaces:**
- Consumes: 全部前置任务
- Produces: 可交付的骨架与验证记录

- [ ] **Step 1: 检查无空文件**

```bash
find . -type f -empty -not -path './.git/*' -not -name '.gitignore'
```

Expected: 无输出。

- [ ] **Step 2: 检查必需文件齐全**

```bash
for f in SKILL.md README.md CLAUDE.md LICENSE \
  references/writing-principles.md references/writing-structures.md \
  references/writing-styles.md references/visual-guidelines.md \
  references/publishing-formats.md \
  workflows/ideation.md workflows/outlining.md workflows/collaborative-writing.md \
  workflows/visual-planning.md workflows/review.md workflows/publishing.md \
  templates/technical-article.md templates/tutorial.md templates/knowledge-sharing.md \
  templates/product-document.md templates/wechat-article.md \
  schemas/writing-plan.yaml schemas/review-report.yaml schemas/visual-assets.yaml \
  scripts/render.py scripts/vis_check.py tests/test_render.py evals/evals.json \
  examples/README.md; do
  test -f "$f" || echo "MISSING: $f"
done
```

Expected: 无 `MISSING`。

- [ ] **Step 3: 跑回归与渲染校验**

```bash
python3 tests/test_render.py
python3 scripts/render.py templates/diagrams/redis-rehash.yaml -o /tmp/tg-final.svg --check
python3 scripts/render.py --list
```

Expected: 测试 `全部通过`；渲染退出码 0；`--list` 列出 7 种类型。

- [ ] **Step 4: 检查没有示例文章或演示图片**

```bash
find . -type f \( -name '*.png' -o -name '*.jpg' -o -name '*.svg' \) -not -path './.git/*'
```

Expected: 无输出。

- [ ] **Step 5: 查看工作区状态**

```bash
git status --short
```

Expected: 无未跟踪或未提交的骨架文件（临时产物在 `/tmp`）。

- [ ] **Step 6: 提交收尾修正（如有）**

```bash
git add -A
git commit -m "chore: finalize tiangong skill scaffold

Co-Authored-By: Claude Code <noreply@anthropic.com>"
```

---

## Self-Review

- **Spec coverage:** 设计文档第 5 节每个文件都有对应 Task；第 6 节内容契约由 Task 5–10 落实；第 7 节 13 条决定分布在 Global Constraints 与各 Task 中；第 8 节验收对应 Task 11。
- **Step scan:** 每个文档任务给出必含小节与字段名；实现者可据此写出唯一合理内容，不需要猜结构。
- **Type consistency:** schema 字段名在 Task 5 定义，Task 6、7、9 只引用不重定义；渲染器接口在 Task 2 固定，后续任务只调用。
- **Review Focus:** 五条失败模式分别由 Task 6、7、9 与 evals case 1/3/4/5/7 固定。
- **Proportion:** 计划约 480 行，与设计文档规模相当；没有转写文档正文。
