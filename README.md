# 你好，我是 lefan17

测试方向的工程师。近两年主要在做一件事：把测试工程化——接口、UI、性能、质量门禁、
Agent 编排，都做成能跑在 CI 里、能被别人按 README 复现的东西。

下面五个公开仓库不是零散作品。前三个是同一条线上的三段：先在单个仓里把接口与全链路
自动化做扎实，再把「跑得对不对」推进到「跑得够不够快、门禁拦不拦得住」，最后把执行与
归因交给 Agent。后两个是业务项目与这条线的前身。

## 一条测试 / 质量线（按发展顺序）

### 1. [fullstack-test-framework](https://github.com/lefan17/fullstack-test-framework) — 接口 + UI + 业务全链路自动化测试框架
Python · pytest + FastAPI + Playwright + Allure + GitHub Actions

这条线的起点，也是后两个仓的地基。`pytest --collect-only -q` 收 **577** 条用例：
接口自动化、UI 自动化、带本地 Mock 后端的业务全链路（状态机、乐观锁、幂等、并发竞态、
Decimal 金额精度），以及独立成组的契约属性测试（**224** 条，其中 213 passed + 11 xfailed）。
契约那一组扫出 **6 类真实契约缺陷**：DEF-05/06 已修并由正常用例固化，DEF-01~04 拆成
11 条 `xfail(strict=True)` 钉住——修不修是取舍，但**修好的和被钉住的都得说清是哪一类**。
**9** 个 CI job，`-m "not ui"` 覆盖率 **86%**（分母 8450 行）。

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
+4.1% 假阳性，才改成 10%），**12 条门禁规则**接入 CI，平台自身 **180** 条用例本机实测
全绿（runner 上传的 CI 日志里是 `[suite] 通过 117 / 失败 0`，那是早一轮时点的快照，原样保留）。
难的不是把门禁写绿，是不让它静默放行：CI 上 R3 一度永远是「跳过」，被当成「比过了
没劣化」，这条已经改成显式的 `[SKIP]` 并补了让门禁真的判过一次、真的红一次的端到端用例。

### 3. [agent-test-hub](https://github.com/lefan17/agent-test-hub) — Agent 自动测试系统
Python · LangGraph + LangChain · OpenAPI 契约驱动

把「从契约生成用例 → 执行 → 自动重试 → 缺陷归因 → 出可视化报告」交给一条四节点
流水线，LLM 调用失败时降级到确定性的 MockBrain，而不是崩掉整条链路。采集 **180** 条用例
（含 UI 是 165 passed + 15 xfailed，不含 UI 是 161 passed + 15 xfailed），
**分支覆盖率 76.88%**（门禁 70%）。

除接口与真实浏览器断言外，还把 RAG 检索质量当作被测对象，用 recall@5 / MRR 量化——
同一套 7 条查询，缺陷实现 **0.5714**，正确实现 **1.0000**。这个数字不是拿来好看的：
它的作用是把「RAG 好不好」从主观判断变成一个能被门禁拦住的指标。

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
