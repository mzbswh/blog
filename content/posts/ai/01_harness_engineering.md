---
title: Harness Engineering
subtitle:
date: 2026-06-02T15:22:27+08:00
slug: harness-engineering
draft: false
author:
  name: mzbswh
  link:
  email:
  avatar:
description: Harness Engineering 是围绕 AI Agent 构建约束、反馈、工作流控制和持续改进循环的工程实践。
keywords:
  - AI Agent
  - Harness Engineering
  - 驾驭工程
  - Context Engineering
license:
comment: false
weight: 0
tags:
  - ai
  - agent
  - harness engineering
categories:
  - ai
collections:
  - ai
hiddenFromHomePage: false
hiddenFromSearch: false
hiddenFromRelated: false
hiddenFromFeed: false
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
toc: true
math: false
lightgallery: false
password:
message:
repost:
  enable: true
  url: https://www.runoob.com/ai-agent/harness-engineering.html

# See details front matter: https://fixit.lruihao.cn/documentation/content-management/introduction/#front-matter
---
<style>
  .harness-page {
    --harness-border: var(--fi-hr-background-color, #e3e3e3);
    --harness-muted: var(--fi-secondary, #666);
    --harness-soft: rgba(23, 114, 238, .08);
    --harness-strong: var(--fi-primary, #1772ee);
  }
  .harness-page * { box-sizing: border-box; }
  .harness-hero,
  .harness-card,
  .harness-panel,
  .harness-quote,
  .harness-metric {
    border: 1px solid var(--harness-border);
    border-radius: 8px;
  }
  .harness-hero {
    padding: 1.4rem;
    margin: 1rem 0 1.4rem;
    background: linear-gradient(135deg, var(--harness-soft), transparent 78%);
  }
  .harness-kicker {
    display: inline-block;
    margin-bottom: .65rem;
    color: var(--harness-strong);
    font-size: .78rem;
    font-weight: 700;
    letter-spacing: .08em;
    text-transform: uppercase;
  }
  .harness-hero h2 { margin: 0 0 .65rem; font-size: 1.65rem; line-height: 1.25; }
  .harness-hero p { margin: 0; color: var(--harness-muted); }
  .harness-source { margin-top: .8rem !important; font-size: .92rem; }
  .harness-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(min(100%, 210px), 1fr));
    gap: clamp(.65rem, 2vw, 1.1rem);
    margin: clamp(.85rem, 2vw, 1.2rem) 0;
  }
  .harness-card,
  .harness-panel,
  .harness-metric { padding: .95rem; background: linear-gradient(180deg, var(--harness-soft), transparent 72%); }
  .harness-card strong,
  .harness-metric strong { display: block; margin-bottom: .35rem; }
  .harness-card p,
  .harness-panel p,
  .harness-metric p { margin: 0; color: var(--harness-muted); }
  .harness-metric strong { color: var(--harness-strong); font-size: 1.4rem; }
  .harness-quote {
    margin: 1rem 0;
    padding: 1rem;
    border-left: 4px solid var(--harness-strong);
    background: var(--harness-soft);
  }
  .harness-quote p { margin: 0; }
  .harness-flow {
    display: flex;
    gap: 1.7rem;
    margin: 1rem 0;
    overflow-x: auto;
    padding: .15rem .1rem .55rem;
    scroll-snap-type: x proximity;
  }
  .harness-flow span {
    position: relative;
    display: inline-flex;
    flex: 0 0 auto;
    align-items: center;
    justify-content: center;
    min-width: 7.5rem;
    border: 1px solid var(--harness-border);
    border-radius: 999px;
    padding: .45rem .7rem;
    text-align: center;
    font-weight: 600;
    background: var(--harness-soft);
    scroll-snap-align: start;
  }
  .harness-flow span:not(:last-child)::after {
    content: "→";
    position: absolute;
    right: -1.25rem;
    color: var(--harness-strong);
    font-weight: 700;
  }
  .harness-table-wrap { overflow-x: auto; margin: .9rem 0 1.2rem; }
  .harness-table-wrap table {
    width: 100%;
    display: table;
    margin: 0;
    border-collapse: collapse;
    border-spacing: 0;
    background: var(--fi-table-background-color, var(--fi-global-background-color, #fff));
  }
  .harness-table-wrap th,
  .harness-table-wrap td {
    border: 1px solid var(--harness-border);
    padding: .62rem .75rem;
    vertical-align: top;
  }
  .harness-table-wrap th {
    white-space: nowrap;
    font-weight: 700;
    background: var(--fi-table-thead-color, var(--harness-soft));
  }
  .harness-table-wrap tbody tr:nth-child(even) {
    background: var(--harness-soft);
  }
  .harness-table-wrap td {
    min-width: 9rem;
  }
  @media (max-width: 680px) {
    .harness-hero { padding: 1rem; }
    .harness-hero h2 { font-size: 1.35rem; }
  }
</style>

<div class="harness-page">
  <section class="harness-hero">
    <span class="harness-kicker">AI Agent / Harness Engineering</span>
    <h2>Harness Engineering：从写 Prompt 到驾驭 Agent</h2>
    <p>AI Agent 的核心挑战，正在从“让模型写得更好”转向“让 Agent 在真实工程系统里稳定、可靠、不失控地工作”。</p>
    <p class="harness-source">来源整理：<a href="https://www.runoob.com/ai-agent/harness-engineering.html">菜鸟教程：Harness Engineering（驾驭工程）</a>。本文完整覆盖原文结构和要点，但采用改写整理，不逐字转载。</p>
  </section>
</div>

## 一、什么是 Harness Engineering？

Harness Engineering（驾驭工程）是一种围绕 AI Agent 构建约束、反馈、工作流控制和持续改进循环的系统工程实践。

它不直接优化模型权重，也不只是调整提示词，而是优化模型运行和执行任务的环境。它的核心哲学可以概括为：人类掌舵，智能体执行。

<div class="harness-page">
  <div class="harness-grid">
    <div class="harness-card"><strong>Human Steer</strong><p>人类负责目标、方向、边界和最终验收。</p></div>
    <div class="harness-card"><strong>Agent Execute</strong><p>Agent 负责执行、调用工具、迭代和修正。</p></div>
    <div class="harness-card"><strong>Harness Govern</strong><p>Harness 负责权限、约束、反馈和长期沉淀。</p></div>
  </div>
  <div class="harness-quote">
    <p>原文把 Harness 类比为马具：不是削弱马的力量，而是通过缰绳、马鞍和护具，让强大的力量能被安全地引导。</p>
  </div>
</div>

按照原文脉络，这个概念的重点不是“更换更强模型”，而是：每当 Agent 犯错时，工程师都应该把这次错误转化为环境改进，让它未来更难犯同类错误。

## 二、为什么需要驾驭工程？真实数据说话

原文用几组数据说明：Agent 已经进入真实工程流程，瓶颈开始从模型能力转向外部基础设施。

<div class="harness-page">
  <div class="harness-grid">
    <div class="harness-metric"><strong>100 万行</strong><p>OpenAI 团队实验中 5 个月产出的代码规模。</p></div>
    <div class="harness-metric"><strong>0 行</strong><p>实验叙述中强调的工程师手写代码量。</p></div>
    <div class="harness-metric"><strong>3-7 人</strong><p>小团队规模，人均每日多个 PR 的协作节奏。</p></div>
    <div class="harness-metric"><strong>30 → 5</strong><p>LangChain 通过改进 Harness 后，Terminal Bench 排名提升。</p></div>
  </div>
</div>

LangChain 的案例尤其能说明问题：底层模型不变，只调整外部环境，例如文档结构、验证回路和追踪系统，编码 Agent 的表现就能明显提升。这说明 Agent 表现不只取决于模型本身，也取决于它所处的工程系统。

<div class="harness-page">
  <div class="harness-table-wrap">
    <table>
      <thead>
        <tr><th>变化</th><th>表现</th><th>没有 Harness 的风险</th></tr>
      </thead>
      <tbody>
        <tr><td>规模扩大</td><td>Agent 高频生成代码、文档和配置</td><td>技术债以更快速度累积</td></tr>
        <tr><td>工具增强</td><td>Agent 可以读写文件、运行命令、提交改动</td><td>权限、边界、回滚不清晰</td></tr>
        <tr><td>任务变长</td><td>Agent 跨多轮、多文件、多会话执行</td><td>上下文丢失、误判完成、遗漏验证</td></tr>
        <tr><td>协作复杂</td><td>多个 Agent 或人与 Agent 并行工作</td><td>状态分裂、重复实现、审查困难</td></tr>
      </tbody>
    </table>
  </div>
</div>

## 三、AI 工程范式的三次跃迁

理解 Harness Engineering，需要先区分三种工程范式。

<div class="harness-page">
  <div class="harness-table-wrap">
    <table>
      <thead>
        <tr><th>范式</th><th>核心问题</th><th>优化对象</th><th>交互模式</th></tr>
      </thead>
      <tbody>
        <tr><td>提示词工程</td><td>怎么把话说清楚</td><td>Prompt 的措辞、格式、示例</td><td>一问一答</td></tr>
        <tr><td>上下文工程</td><td>怎么给 AI 喂信息</td><td>文档、代码片段、历史对话</td><td>信息注入后生成</td></tr>
        <tr><td>驾驭工程</td><td>怎么让 Agent 可靠工作</td><td>约束、反馈回路、控制系统</td><td>人类掌舵，Agent 执行</td></tr>
      </tbody>
    </table>
  </div>
  <div class="harness-grid">
    <div class="harness-card"><strong>Prompt Engineering</strong><p>像对马喊话，重点是指令表达。</p></div>
    <div class="harness-card"><strong>Context Engineering</strong><p>像给马地图，重点是信息供给。</p></div>
    <div class="harness-card"><strong>Harness Engineering</strong><p>像造道路、护栏、限速牌和补给站，重点是可控运行。</p></div>
  </div>
</div>

Prompt 解决单次交互质量；Context 解决背景和知识边界；Harness 解决 Agent 长期执行、可靠交付和系统可持续性。

## 四、Agent 常见失败模式

原文将长时间运行 Agent 的典型问题归纳为三类，并补充了一个更隐蔽的风险：Agent 会复制并放大代码库中的坏模式。

### 失败模式 1：试图一步到位（One-shotting）

Agent 倾向于在一个会话里把所有功能做完。结果是上下文窗口逐渐耗尽，后续状态、未完成事项和设计理由没有被结构化记录，下一个会话只能重新猜测之前发生了什么。

更好的 Harness 设计应该把长任务拆成阶段、检查点和可恢复状态，而不是期待 Agent 一次性完成所有事情。

### 失败模式 2：过早宣布胜利

任务进行到后期时，Agent 可能看到部分功能已经完成，就直接宣布任务完成。但真实项目中，剩余功能、边界条件、验收标准和集成行为可能还没有被覆盖。

解决方式不是简单提醒“不要太早结束”，而是把完成条件变成清单、测试、审查和外部状态。

### 失败模式 3：过早标记功能完成

Agent 写完代码后，如果没有明确要求端到端验证，可能只跑单元测试、curl 或局部检查就标记完成。局部命令通过不代表真实功能可用。

Harness 需要把“完成”定义为可验证状态，例如构建通过、关键测试通过、端到端路径可用、风险被记录。

### 隐藏风险：模式复制

Agent 很擅长模仿已有代码。如果仓库里存在架构漂移、坏抽象、陈旧文档或脆弱测试，它也会把这些模式继续复制和放大。不加约束的 Agent 不只会产生代码，也会高速产生技术债。

## 五、驾驭工程的四大护栏

综合原文提到的 OpenAI、Anthropic、LangChain 和 Martin Fowler 等实践，可以把 Harness 归纳为四根护栏。

<div class="harness-page">
  <div class="harness-grid">
    <div class="harness-card"><strong>上下文工程</strong><p>动态知识注入、AGENTS.md、按需检索、活文档机制。</p></div>
    <div class="harness-card"><strong>架构约束</strong><p>分层依赖模型、自动 Linter、CI 阻断、类型系统。</p></div>
    <div class="harness-card"><strong>反馈循环</strong><p>Agent-to-Agent Review、自动测试、CI 验证、错误信号回传。</p></div>
    <div class="harness-card"><strong>熵管理</strong><p>定期清理、文档园丁、持续小额偿还技术债。</p></div>
  </div>
</div>

### 护栏一：上下文工程（Context Engineering）

上下文工程相当于给新员工准备工作手册。`AGENTS.md` 可以作为 Agent 进入仓库后的第一份指南，但它不能无限膨胀。

上下文窗口是稀缺资源。一个越来越长、越来越旧的说明文件，会挤占任务、代码和相关文档的空间。更合理的做法是提供稳定、小巧的入口，然后让 Agent 根据任务按需检索更多上下文。

原文提到 Ghostty 项目的经验：`AGENTS.md` 中的规则来自历史 Agent 失败案例。也就是说，文档不是静态制品，而是反馈循环的一部分。

### 护栏二：架构约束（Architecture Constraints）

架构约束是 Harness 中的“缰绳”。原文举例提到一种分层依赖模型：

<div class="harness-page">
  <div class="harness-flow">
    <span>Types</span>
    <span>Config</span>
    <span>Repo</span>
    <span>Service</span>
    <span>Runtime</span>
    <span>UI</span>
  </div>
</div>

核心规则是：下层不能反向依赖上层。规则不能只写在文档里，而要编码成 Linter、CI、类型系统或其他自动检查。这样无论代码是人写的还是 AI 写的，违规都会被阻断。

关键细节是：错误信息本身也是上下文工程。好的 Linter 不只是说“违反规则”，还会说明规则为什么存在、正确做法是什么，让 Agent 能读懂错误并自我修正。

### 护栏三：反馈循环（Feedback Loop）

传统代码审查主要依赖人类工程师。Harness 中可以加入 Agent-to-Agent Review：一个 Agent 写代码，另一个 Agent 审查行为、边界、测试和风险。

<div class="harness-page">
  <div class="harness-flow">
    <span>生成</span>
    <span>验证</span>
    <span>审查</span>
    <span>修正</span>
    <span>沉淀</span>
  </div>
</div>

反馈循环不只是跑测试。它还包括把失败信息回传给模型、让模型独立评估自己的代码、检查测试是否真正覆盖 Bug。如果 AI 写出的测试能通过带 Bug 的代码，那么 Harness 应该把测试本身判定为无效，要求重新思考边界。

### 护栏四：熵管理（Entropy Management）

软件系统会自然熵增：重复代码、临时方案、过期文档和架构漂移会不断积累。Agent 让产出速度变快，也会让熵增速度变快。

原文强调的策略是持续小额偿还，而不是等问题严重后集中处理。可以让后台 Agent 定期扫描偏差、更新质量等级、发起重构 PR；也可以设置文档园丁 Agent，自动发现文档和代码不一致并提交修复。

## 六、六大行业共识

原文综合多个独立信息源，总结了六个共识。

<div class="harness-page">
  <div class="harness-table-wrap">
    <table>
      <thead>
        <tr><th>#</th><th>共识</th><th>核心观点</th></tr>
      </thead>
      <tbody>
        <tr><td>1</td><td>瓶颈在基础设施，不只在模型智能</td><td>多个团队得到类似结论：只改变 Harness 和工具格式，也能显著提高 Agent 表现。</td></tr>
        <tr><td>2</td><td>文档必须是活的反馈循环</td><td>静态文档容易过时，动态文档要从失败案例和后台维护中持续更新。</td></tr>
        <tr><td>3</td><td>思考与执行需要分离</td><td>复杂任务难以塞进单个上下文窗口，需要 Orchestrator + Worker 分层和外部状态。</td></tr>
        <tr><td>4</td><td>上下文不是越多越好</td><td>上下文是稀缺资源，巨大指令文件会挤掉真正任务空间，应按需检索。</td></tr>
        <tr><td>5</td><td>约束必须自动化</td><td>人工 Review 是瓶颈，护栏应编码为 Linter、CI 和类型系统。</td></tr>
        <tr><td>6</td><td>工程师角色正在变化</td><td>工程师从代码编写者转向环境建筑师，重点是设计可靠控制系统。</td></tr>
      </tbody>
    </table>
  </div>
</div>

## 七、Harness 与传统框架的关系

Harness 不是 SDK、脚手架或 Agent 框架的替代品，而是位于它们之上的一层。

<div class="harness-page">
  <div class="harness-table-wrap">
    <table>
      <thead>
        <tr><th>层级</th><th>解决的问题</th><th>示例</th></tr>
      </thead>
      <tbody>
        <tr><td>Harness Layer</td><td>约束、反馈循环、上下文工程、熵管理、生命周期管理</td><td>CI、Linter、活文档、权限、审查系统</td></tr>
        <tr><td>Agent Frameworks</td><td>智能体定义、消息路由、任务生命周期、依赖管理</td><td>LangGraph、AutoGen、CrewAI</td></tr>
        <tr><td>SDK / API</td><td>模型调用、工具注册、流式输出</td><td>OpenAI、Anthropic、Gemini SDK</td></tr>
        <tr><td>Foundation Model</td><td>语言理解、推理、生成</td><td>GPT、Claude、Gemini、DeepSeek</td></tr>
      </tbody>
    </table>
  </div>
</div>

传统框架解决“如何构建智能体”，Harness 解决“智能体如何可靠运行”。原文还指出，模型可能逐渐吸收框架中的一部分能力，例如智能体定义、消息路由和生命周期管理；但持久化、确定性重放、成本控制、可观测性、错误恢复等问题，仍然是 Harness 层的重要价值。

## 总结

Harness Engineering 不是某一家公司的实验，而是 AI Agent 进入工程实践后自然出现的范式转移。

它的核心不是给 Agent 更少自由，而是在更强自主性外面配套更严格的运行约束。正因为有护栏，工程团队才敢让 Agent 跑得更快。

<div class="harness-page">
  <div class="harness-table-wrap">
    <table>
      <thead>
        <tr><th>核心组件</th><th>解决的问题</th><th>代表实践</th></tr>
      </thead>
      <tbody>
        <tr><td>上下文工程</td><td>Agent 不知道该看什么、怎么找</td><td>AGENTS.md、活文档、按需检索</td></tr>
        <tr><td>架构约束</td><td>Agent 复制并放大坏模式</td><td>分层依赖、自定义 Linter、CI 阻断</td></tr>
        <tr><td>反馈循环</td><td>Agent 不知道自己做错了</td><td>Agent-to-Agent Review、自动测试套件</td></tr>
        <tr><td>熵管理</td><td>技术债务和文档腐烂</td><td>Doc-gardening Agent、持续垃圾回收</td></tr>
      </tbody>
    </table>
  </div>
</div>

软件开发的未来，可能不只取决于我们能写多快的代码，而取决于我们能否设计出足够鲁棒的系统，来驾驭 AI Agent 的巨大执行能力。工程师的价值也会从单纯执行，转向构建“能够构建产品的工厂”。
