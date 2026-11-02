# 你好，我是 lefan17

测试方向的工程师。近两年主要在做一件事：把测试工程化——接口、UI、性能、质量门禁、
Agent 编排，都做成能跑在 CI 里、能被别人按 README 复现的东西。

下面六个公开仓库不是零散作品。前四个是同一条线上的四段：先在单个仓里把接口与全链路
自动化做扎实，再把「跑得对不对」推进到「跑得够不够快、门禁拦不拦得住」，再把执行与
归因交给 Agent，最后回答「改了一行到底该重跑哪几条」。后两个是业务项目与这条线的前身。

三个 Python 仓都把结果发布成可以直接打开的页面（GitHub Pages），不用 clone、不用装环境：
[测试报告](https://lefan17.github.io/fullstack-test-framework/) ·
[性能看板](https://lefan17.github.io/perf-baseline-quality-gate/) ·
[Agent 报告](https://lefan17.github.io/agent-test-hub/)。报表由 CI 每轮刷新，不是摆上去的截图。

## 一条测试 / 质量线（按发展顺序）

### 1. [fullstack-test-framework](https://github.com/lefan17/fullstack-test-framework) — 接口 + UI + 业务全链路自动化测试框架
Python · pytest + FastAPI + Playwright + Allure + GitHub Actions

这条线的起点，也是后两个仓的地基。`pytest --collect-only -q` 收 **636** 条用例：
接口自动化、UI 自动化、带本地 Mock 后端的业务全链路（状态机、乐观锁、幂等、并发竞态、
Decimal 金额精度），以及独立成组的契约属性测试（**224** 条，其中 213 passed + 11 xfailed）。
契约那一组扫出 **6 类真实契约缺陷**：DEF-05/06 已修并由正常用例固化，DEF-01~04 拆成
11 条 `xfail(strict=True)` 钉住——修不修是取舍，但**修好的和被钉住的都得说清是哪一类**。
**9** 个 CI job，覆盖率有**两道门**：仓库级 `-m "not ui"` **87%**（分母 8966 行，实测 86.98%）
和 per-file **55%**（22 个文件逐个判，拦仓库级看不见的单点失守）。
测试报告在线可看：<https://lefan17.github.io/fullstack-test-framework/>

两件事值得单独说：**门禁是自洽的**——`scripts/check_gate_policy.py` 拿门禁文档与
`test.yml`、`pytest.ini` 逐条机器校验，改了 workflow 不改文档就红；**耗时不被当成结论**——
`-m api` 与 `-m ui` 打公网，两天里同一套用例量到 60–74s 与 21–34s，所以 README 里只写量级，
门禁与判定从不依赖它。

### 2. [perf-baseline-quality-gate](https://github.com/lefan17/perf-baseline-quality-gate) — 性能与稳定性基线平台 · 质量度量与 CI 门禁
Python · FastAPI + SQLite + 自研压测引擎 + GitHub Actions

它回答第一个仓没有回答的问题：变慢了没有、和谁比、门禁真的拦得住吗。内置被测服务里
刻意保留一处 N+1 报表缺陷，20 000 条物料下同一份 SQL 的两种写法单请求差 **7.81×**
（40.6 ms vs 316.9 ms），并发一拉起来差距扩大到 **49.3×**（本机）/ **81.6×**（GitHub runner）——
同一个缺陷换台机器绝对数字全变，但「它是灾难」这个结论不变。

基线显式建立、劣化按实测出来的容忍带判定（先按经验写 3%，被一轮无改动的重跑打出
+4.1% 假阳性，才改成 10%），**12 条门禁规则**接入 CI，平台自身 **398** 条用例本机实测
**379 passed / 19 skipped**（runner 上传的 CI 日志里是 `[suite] 通过 117 / 失败 0`，那是早一轮时点的快照，原样保留）。
难的不是把门禁写绿，是不让它静默放行：CI 上 R3 一度永远是「跳过」，被当成「比过了
没劣化」，这条已经改成显式的 `[SKIP]` 并补了让门禁真的判过一次、真的红一次的端到端用例。
在线看板：<https://lefan17.github.io/perf-baseline-quality-gate/>（P95 趋势、flaky 名单，
CI 每轮刷新，断网可看）。

### 3. [agent-test-hub](https://github.com/lefan17/agent-test-hub) — Agent 自动测试系统
Python · LangGraph + LangChain · OpenAPI 契约驱动

把「从契约生成用例 → 执行 → 自动重试 → 缺陷归因 → 出可视化报告」交给一条四节点
流水线，LLM 调用失败时降级到确定性的 MockBrain，而不是崩掉整条链路。采集 **361** 条用例
（本机有 Chrome + Key 时 346 passed + 15 xfailed，CI 上无 Key 时 338 passed + 4 skipped + 15 xfailed；
skipped 那 4 条是需要真实 LLM Key 的冒烟，故意不在 PR 门禁里跑），
**分支覆盖实测 91.61%**（分母 1618；仓库级门禁 70% + per-file 60%，
历史 91.22% / 91.78% 是上一棵树 1558 stmts 的值，见仓内 README 对两个数差异的说明）。

除接口与真实浏览器断言外，还把 RAG 检索质量当作被测对象，用 recall@5 / MRR 量化——
同一套 7 条查询，缺陷实现 **0.5714**，正确实现 **1.0000**。这个数字不是拿来好看的：
它的作用是把「RAG 好不好」从主观判断变成一个能被门禁拦住的指标。
报告在线可看：<https://lefan17.github.io/agent-test-hub/>

### 4. [api-regression-selector](https://github.com/lefan17/api-regression-selector) — 变更 → 测试选择（TIA）
Python · pytest + coverage-context + GitHub Actions

它回答前三个仓都没回答的问题：**改了一行，到底该重跑哪几条**。用
`--cov-context=test` 采集「每一行被哪条用例执行过」的索引，再按 `git diff` 的行号
反查受影响的用例。改 `qaplat/registry.py` 一行 → 712 条里选中 **36 条（5.06%）**，
单跑 **2.45s**，全量 `-m "not ui"` 是 **137.41s**。

**这条边界是拿一次自我推翻换来的**：早期对外写的是「改 `mock_server/routers.py`
一行 → 只选中 1 条、2.21s、89 倍」，独立复核发现它测的是**失真的索引**——
被测服务由测试拉起 uvicorn 子进程执行，coverage 的 context 在 spawn 那一刻就固定，
行级数据会失真到「整份文件只关联 10 个用例」，而实际有 423 条用例在打接口。
修完之后**改 `mock_server/**` 一律全量**，宁可多跑不可静默漏跑；
上面那组 36/712 只对同进程执行的代码成立（细节见仓内 `docs/tia-review.md`）。

正确性没靠「看起来对」：把 `normalize_path` 故意改坏后，7 条真红里**旧索引漏检 3 条
（3/7）**——那 3 条来自采集之后才新增的测试文件；补采后复算 **0/7**。这个 3/7 是本仓
最诚实的数字，也是「索引新鲜度」存在的理由：采集时把 HEAD 记进 `.tia-coverage.head`，
`tia select` 发现落后就在 stderr 警告「中间有 N 个提交未被覆盖」，但**只警告、不自动重采**
——重采要 140–190 秒，自动做会把一次 6 秒的查询拖成 3 分钟。

76 条平台层用例护着选择器本身；`--select tia` 默认关闭，只挂在 `workflow_dispatch`
手动跑：进流水门之前要先攒真实 diff 分布，看兜底率与误选率。起点是上面第一个仓的
`dfc9fe6`，之后独立演进——**上游代码与测试一字未改**，两侧唯一的变化是 2026-11-02 在
上游补了 `AGENTS.md` 与「智能体协作」一节（纯文档，因为那 200 条提交的协作过程本来
就记在上游仓里）。

## 这些仓库是怎么写出来的（以及我为什么敢这么说）

四个 Python 仓里，**大部分代码是 AI 编码代理写的**——`git log --format='%an'` 能看到
`dispatcher` / `integ` / `agent` 这几个身份，在 `fullstack-test-framework` 截止
`dfc9fe6` 的 200 条提交里占 100 条。身份没统一是工程疏漏，但这件事本身不该藏着，
因为真正的问题是「AI 写的代码要过什么闸才敢合」，而这个问题有可复核的答案：

| 我主张 | 在哪能验证 |
|---|---|
| 代理真的在提交，不是我署个名 | `git log --format='%an' \| sort \| uniq -c \| sort -rn` |
| 它写下的行没被推翻 | 逐行 `git blame` 归属：AI 新增行 **98.8%** 活到 HEAD（人工 97.2%） |
| 我否决过它的结论 | [api-regression-selector/`docs/tia-review.md`](https://github.com/lefan17/api-regression-selector/blob/main/docs/tia-review.md)：三轮独立复核推翻了「89 倍提速」这个对外数字 |
| 约束留在仓库里，不是留在对话里 | 四仓各一份 `AGENTS.md`，随代码分发的踩坑清单（加密钥路径、编码、缓存目录这类硬约束） |
| 流程约束到底有没有用 | 我做过可控两臂对照实验，结论是**没用**：覆盖率一模一样，没被告知的 3 个变异两臂都漏同一个 |

完整证据链（每条结论带复现命令）写在
[fullstack-test-framework README 的「智能体协作」一节](https://github.com/lefan17/fullstack-test-framework#智能体协作这个仓库是怎么用-ai-编码代理改出来的)。

一句话：**我不主张「AI 让我更快」，我主张我把 AI 的输出做成了可核验的东西**——
过程留在 git 里、约束留在 AGENTS.md 里、否决权留在独立复核里。

## 业务项目与出处

- [buzhihu_project](https://github.com/lefan17/buzhihu_project) — 不止乎：Spring Boot 2 + Vue 2
  前后端分离内容社区（博客 / 活动报名 / 关注流 / 站内通知 / 举报治理 + 后台数据看板）。
  Java + Vue 方向的落地，与上面三个 Python 仓互补。
- [software-testing-practice](https://github.com/lefan17/software-testing-practice) — **已归档**，
  是本系列的前身：契约驱动的接口自动化测试实践（Flask 被测系统 + OpenAPI 契约 +
  27 条登录接口用例）。内容已并入 fullstack-test-framework，这里只留位置。

## 一条共同的取舍

三个仓的 README 里有大量「未实测 / 已作废 / 这条不成立」的记录：Docker daemon 在本机
起不来所以容器化那条只到「配置写完、未跑通」；`pip-audit` 与 `pre-commit` 在受限环境里
装不上所以标了未验证；早期把 UI 用例的 19 条 error 归因成沙箱禁止建命名管道，后来逐条
复现证明是缺浏览器，README 里留着这段错误归因并写明为什么不成立。

**能复现的写数字，不能复现的写边界。** 这是我做这些东西时最重要的一条自我约束——
一个测试项目最容易骗的人是自己。
