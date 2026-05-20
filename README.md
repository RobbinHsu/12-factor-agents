# 12-Factor Agents - Principles for building reliable LLM applications
# 12-Factor Agents - 建構可靠 LLM 應用程式的原則

<div align="center">
<a href="https://www.apache.org/licenses/LICENSE-2.0">
        <img src="https://img.shields.io/badge/Code-Apache%202.0-blue.svg" alt="Code License: Apache 2.0"></a>
<a href="https://creativecommons.org/licenses/by-sa/4.0/">
        <img src="https://img.shields.io/badge/Content-CC%20BY--SA%204.0-lightgrey.svg" alt="Content License: CC BY-SA 4.0"></a>
<a href="https://humanlayer.dev/discord">
    <img src="https://img.shields.io/badge/chat-discord-5865F2" alt="Discord Server"></a>
<a href="https://www.youtube.com/watch?v=8kMaTybvDUw">
    <img src="https://img.shields.io/badge/aidotengineer-conf_talk_(17m)-white" alt="YouTube
Deep Dive"></a>
<a href="https://www.youtube.com/watch?v=yxJDyQ8v6P0">
    <img src="https://img.shields.io/badge/youtube-deep_dive-crimson" alt="YouTube
Deep Dive"></a>
    
</div>

<p></p>

*In the spirit of [12 Factor Apps](https://12factor.net/)*.  *The source for this project is public at https://github.com/humanlayer/12-factor-agents, and I welcome your feedback and contributions. Let's figure this out together!*

*本專案秉持 [12 Factor Apps](https://12factor.net/) 的精神。* *本專案原始碼公開於 https://github.com/humanlayer/12-factor-agents，歡迎你提供回饋與貢獻。讓我們一起把這件事搞清楚！*

> [!TIP]
> Missed the AI Engineer World's Fair? [Catch the talk here](https://www.youtube.com/watch?v=8kMaTybvDUw)
> 錯過 AI Engineer World's Fair 了嗎？[點此觀看演講](https://www.youtube.com/watch?v=8kMaTybvDUw)
>
> Looking for Context Engineering? [Jump straight to factor 3](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)
> 想找 Context Engineering？[直接跳到 factor 3](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)
>
> Want to contribute to `npx/uvx create-12-factor-agent` - check out [the discussion thread](https://github.com/humanlayer/12-factor-agents/discussions/61)
> 想為 `npx/uvx create-12-factor-agent` 做出貢獻嗎？請查看[討論串](https://github.com/humanlayer/12-factor-agents/discussions/61)


<img referrerpolicy="no-referrer-when-downgrade" src="https://static.scarf.sh/a.png?x-pxid=2acad99a-c2d9-48df-86f5-9ca8061b7bf9" />

<a href="#visual-nav"><img width="907" alt="Screenshot 2025-04-03 at 2 49 07 PM" src="https://github.com/user-attachments/assets/23286ad8-7bef-4902-b371-88ff6a22e998" /></a>


Hi, I'm Dex. I've been [hacking](https://youtu.be/8bIHcttkOTE) on [AI agents](https://theouterloop.substack.com) for [a while](https://humanlayer.dev). 

嗨，我是 Dex。我已經投入 [AI agents](https://theouterloop.substack.com) 的[研究與實作](https://youtu.be/8bIHcttkOTE) [好一段時間](https://humanlayer.dev)了。


**I've tried every agent framework out there**, from the plug-and-play crew/langchains to the "minimalist" smolagents of the world to the "production grade" langraph, griptape, etc. 

**我幾乎試過市面上所有的 agent framework**，從即插即用的 crew/langchains，到主打「極簡」的 smolagents，再到「production grade」的 langraph、griptape 等等。

**I've talked to a lot of really strong founders**, in and out of YC, who are all building really impressive things with AI. Most of them are rolling the stack themselves. I don't see a lot of frameworks in production customer-facing agents.

**我和許多非常厲害的創辦人聊過**，無論是在 YC 內外，他們都在用 AI 打造非常令人印象深刻的產品。多數人都是自己組裝整個技術堆疊。我在面向客戶的 production agents 中，沒有看到太多 framework 的身影。

**I've been surprised to find** that most of the products out there billing themselves as "AI Agents" are not all that agentic. A lot of them are mostly deterministic code, with LLM steps sprinkled in at just the right points to make the experience truly magical.

**讓我意外的是**，市面上多數自稱為「AI Agents」的產品，其實沒有那麼 agentic。它們很多主要仍是 deterministic code，只是在恰到好處的地方加入一些 LLM 步驟，讓整體體驗看起來真的很神奇。

Agents, at least the good ones, don't follow the ["here's your prompt, here's a bag of tools, loop until you hit the goal"](https://www.anthropic.com/engineering/building-effective-agents#agents) pattern. Rather, they are comprised of mostly just software. 

Agents，至少那些優秀的 agents，並不是遵循那種[「給你一個 prompt，再給你一袋 tools，然後一路 loop 到達成目標」](https://www.anthropic.com/engineering/building-effective-agents#agents)的模式。更精確地說，它們大多仍然是由軟體組成。

So, I set out to answer:

於是，我開始嘗試回答：

> ### **What are the principles we can use to build LLM-powered software that is actually good enough to put in the hands of production customers?**
> ### **我們可以依循哪些原則，來打造真正足以交到 production 客戶手中的 LLM-powered software？**

Welcome to 12-factor agents. As every Chicago mayor since Daley has consistently plastered all over the city's major airports, we're glad you're here.

歡迎來到 12-factor agents。正如 Daley 以來歷任芝加哥市長一直貼滿全市各大機場的那句話一樣，我們很高興你來到這裡。

*Special thanks to [@iantbutler01](https://github.com/iantbutler01), [@tnm](https://github.com/tnm), [@hellovai](https://www.github.com/hellovai), [@stantonk](https://www.github.com/stantonk), [@balanceiskey](https://www.github.com/balanceiskey), [@AdjectiveAllison](https://www.github.com/AdjectiveAllison), [@pfbyjy](https://www.github.com/pfbyjy), [@a-churchill](https://www.github.com/a-churchill), and the SF MLOps community for early feedback on this guide.*

*特別感謝 [@iantbutler01](https://github.com/iantbutler01)、[@tnm](https://github.com/tnm)、[@hellovai](https://www.github.com/hellovai)、[@stantonk](https://www.github.com/stantonk)、[@balanceiskey](https://www.github.com/balanceiskey)、[@AdjectiveAllison](https://www.github.com/AdjectiveAllison)、[@pfbyjy](https://www.github.com/pfbyjy)、[@a-churchill](https://www.github.com/a-churchill) 與 SF MLOps 社群，為本指南提供早期回饋。*

## The Short Version: The 12 Factors
## 短版摘要：12 Factors

Even if LLMs [continue to get exponentially more powerful](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md#what-if-llms-get-smarter), there will be core engineering techniques that make LLM-powered software more reliable, more scalable, and easier to maintain.

即使 LLMs [持續以指數級速度變得更強](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md#what-if-llms-get-smarter)，仍然會有一些核心工程技巧，能讓 LLM-powered software 更可靠、更可擴充，也更容易維護。

- [How We Got Here: A Brief History of Software](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/brief-history-of-software.md)
- [我們如何走到這裡：Software 簡史](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/brief-history-of-software.md)
- [Factor 1: Natural Language to Tool Calls](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-01-natural-language-to-tool-calls.md)
- [Factor 1：自然語言轉 tool calls](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-01-natural-language-to-tool-calls.md)
- [Factor 2: Own your prompts](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-02-own-your-prompts.md)
- [Factor 2：掌握你的 prompts](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-02-own-your-prompts.md)
- [Factor 3: Own your context window](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)
- [Factor 3：掌握你的 context window](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)
- [Factor 4: Tools are just structured outputs](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs.md)
- [Factor 4：Tools 只是 structured outputs](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs.md)
- [Factor 5: Unify execution state and business state](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md)
- [Factor 5：統一 execution state 與 business state](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md)
- [Factor 6: Launch/Pause/Resume with simple APIs](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md)
- [Factor 6：用簡單 API 進行 Launch/Pause/Resume](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md)
- [Factor 7: Contact humans with tool calls](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md)
- [Factor 7：用 tool calls 聯繫人類](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md)
- [Factor 8: Own your control flow](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md)
- [Factor 8：掌握你的 control flow](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md)
- [Factor 9: Compact Errors into Context Window](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-09-compact-errors.md)
- [Factor 9：將 Errors 壓縮進 Context Window](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-09-compact-errors.md)
- [Factor 10: Small, Focused Agents](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md)
- [Factor 10：小而專注的 Agents](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md)
- [Factor 11: Trigger from anywhere, meet users where they are](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-11-trigger-from-anywhere.md)
- [Factor 11：從任何地方觸發，在使用者所在之處與他們相遇](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-11-trigger-from-anywhere.md)
- [Factor 12: Make your agent a stateless reducer](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-12-stateless-reducer.md)
- [Factor 12：讓你的 agent 成為 stateless reducer](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-12-stateless-reducer.md)


### Visual Nav
### 視覺導覽

|    |    |    |
|----|----|-----|
|[![factor 1](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/110-natural-language-tool-calls.png)](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-01-natural-language-to-tool-calls.md) | [![factor 2](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/120-own-your-prompts.png)](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-02-own-your-prompts.md) | [![factor 3](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/130-own-your-context-building.png)](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md) |
|[![factor 4](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/140-tools-are-just-structured-outputs.png)](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs.md) | [![factor 5](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/150-unify-state.png)](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md) | [![factor 6](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/160-pause-resume-with-simple-apis.png)](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md) |
| [![factor 7](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/170-contact-humans-with-tools.png)](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md) | [![factor 8](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/180-control-flow.png)](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md) | [![factor 9](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/190-factor-9-errors-static.png)](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-09-compact-errors.md) |
| [![factor 10](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/1a0-small-focused-agents.png)](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md) | [![factor 11](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/1b0-trigger-from-anywhere.png)](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-11-trigger-from-anywhere.md) | [![factor 12](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/1c0-stateless-reducer.png)](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-12-stateless-reducer.md) |

## How we got here
## 我們如何走到這裡

For a deeper dive on my agent journey and what led us here, check out [A Brief History of Software](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/brief-history-of-software.md) - a quick summary here:

如果你想更深入了解我的 agent 旅程，以及我們如何走到這一步，請閱讀 [A Brief History of Software](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/brief-history-of-software.md)；以下先給你一個快速摘要：

### The promise of agents
### Agents 的承諾

We're gonna talk a lot about Directed Graphs (DGs) and their Acyclic friends, DAGs. I'll start by pointing out that...well...software is a directed graph. There's a reason we used to represent programs as flow charts.

我們會大量談到 Directed Graphs（DGs）以及它們的無環朋友 DAGs。先從一件事開始：嗯……software 本身就是一張 directed graph。這也是為什麼過去我們常用流程圖來表示程式。

![010-software-dag](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/010-software-dag.png)

### From code to DAGs
### 從 code 到 DAGs

Around 20 years ago, we started to see DAG orchestrators become popular. We're talking classics like [Airflow](https://airflow.apache.org/), [Prefect](https://www.prefect.io/), some predecessors, and some newer ones like ([dagster](https://dagster.io/), [inggest](https://www.inngest.com/), [windmill](https://www.windmill.dev/)). These followed the same graph pattern, with the added benefit of observability, modularity, retries, administration, etc.

大約 20 年前，DAG orchestrators 開始流行起來。我們說的是像 [Airflow](https://airflow.apache.org/)、[Prefect](https://www.prefect.io/) 這些經典工具，還有一些前身，以及較新的工具，例如 [dagster](https://dagster.io/)、[inggest](https://www.inngest.com/)、[windmill](https://www.windmill.dev/)。它們遵循相同的 graph 模式，同時額外帶來 observability、modularity、retries、administration 等優勢。

![015-dag-orchestrators](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/015-dag-orchestrators.png)

### The promise of agents
### Agents 的承諾

I'm not the first [person to say this](https://youtu.be/Dc99-zTMyMg?si=bcT0hIwWij2mR-40&t=73), but my biggest takeaway when I started learning about agents, was that you get to throw the DAG away. Instead of software engineers coding each step and edge case, you can give the agent a goal and a set of transitions:

我不是第一個[這樣說的人](https://youtu.be/Dc99-zTMyMg?si=bcT0hIwWij2mR-40&t=73)，但當我開始學習 agents 時，最大的體會就是：你可以把 DAG 丟掉。不必再由 software engineers 為每個步驟與 edge case 寫程式，而是可以給 agent 一個目標與一組 transitions：

![025-agent-dag](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/025-agent-dag.png)

And let the LLM make decisions in real time to figure out the path

然後讓 LLM 即時做決策，找出前進路徑。

![026-agent-dag-lines](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/026-agent-dag-lines.png)

The promise here is that you write less software, you just give the LLM the "edges" of the graph and let it figure out the nodes. You can recover from errors, you can write less code, and you may find that LLMs find novel solutions to problems.

這個願景在於：你可以少寫很多 software，只要把 graph 的「edges」交給 LLM，讓它自己推導出 nodes。你可以從 errors 中恢復、減少程式碼量，甚至可能發現 LLMs 會替問題找出新穎的解法。


### Agents as loops
### 將 Agents 視為 loops

As we'll see later, it turns out this doesn't quite work.

稍後我們會看到，這件事實際上並沒有那麼行得通。

Let's dive one step deeper - with agents you've got this loop consisting of 3 steps:

讓我們再往下一層看——在 agent 中，你會得到一個由 3 個步驟組成的 loop：

1. LLM determines the next step in the workflow, outputting structured json ("tool calling")
1. LLM 決定 workflow 的下一步，並輸出 structured json（「tool calling」）
2. Deterministic code executes the tool call
2. Deterministic code 執行 tool call
3. The result is appended to the context window 
3. 將結果附加到 context window
4. Repeat until the next step is determined to be "done"
4. 重複這個過程，直到下一步被判定為「done」


```python
initial_event = {"message": "..."}
context = [initial_event]
while True:
  next_step = await llm.determine_next_step(context)
  context.append(next_step)

  if (next_step.intent === "done"):
    return next_step.final_answer

  result = await execute_step(next_step)
  context.append(result)
```

Our initial context is just the starting event (maybe a user message, maybe a cron fired, maybe a webhook, etc), and we ask the llm to choose the next step (tool) or to determine that we're done.

最初的 context 只是起始事件（可能是使用者訊息、可能是 cron 觸發，也可能是 webhook 等），然後我們要求 llm 選出下一步（tool），或者判定任務是否已完成。

Here's a multi-step example:

以下是一個多步驟範例：

[![027-agent-loop-animation](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/027-agent-loop-animation.gif)](https://github.com/user-attachments/assets/3beb0966-fdb1-4c12-a47f-ed4e8240f8fd)

<details>
<summary><a href="https://github.com/RobbinHsu/12-factor-agents/blob/main/img/027-agent-loop-animation.gif">GIF Version</a></summary>

![027-agent-loop-animation](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/027-agent-loop-animation.gif)

</details>

## Why 12-factor agents?
## 為什麼是 12-factor agents？

At the end of the day, this approach just doesn't work as well as we want it to.

說到底，這套做法的效果並沒有我們期望的那麼好。

In building HumanLayer, I've talked to at least 100 SaaS builders (mostly technical founders) looking to make their existing product more agentic. The journey usually goes something like:

在打造 HumanLayer 的過程中，我和至少 100 位 SaaS builders（大多是技術背景創辦人）聊過，他們都希望讓既有產品更具 agentic 特性。這段旅程通常大致如下：

1. Decide you want to build an agent
1. 決定你想做一個 agent
2. Product design, UX mapping, what problems to solve
2. 做 product design、UX mapping，定義要解決哪些問題
3. Want to move fast, so grab $FRAMEWORK and *get to building*
3. 想要快速推進，所以抓起 $FRAMEWORK 就 *開始開發*
4. Get to 70-80% quality bar 
4. 做到大約 70-80% 的品質門檻
5. Realize that 80% isn't good enough for most customer-facing features
5. 發現對大多數面向客戶的功能來說，80% 並不夠好
6. Realize that getting past 80% requires reverse-engineering the framework, prompts, flow, etc.
6. 發現要突破 80%，就得反向拆解 framework、prompts、flow 等等
7. Start over from scratch
7. 從頭開始重做


<details>
<summary>Random Disclaimers</summary>

**DISCLAIMER**: I'm not sure the exact right place to say this, but here seems as good as any: **this in BY NO MEANS meant to be a dig on either the many frameworks out there, or the pretty dang smart people who work on them**. They enable incredible things and have accelerated the AI ecosystem. 

**免責聲明**：我不確定最適合在哪裡說這件事，但這裡大概也很合適：**這絕對不是在貶低外面眾多的 frameworks，或是那些在其上工作、非常聰明的人們**。它們確實促成了許多驚人的成果，也加速了整個 AI 生態系的發展。

I hope that one outcome of this post is that agent framework builders can learn from the journeys of myself and others, and make frameworks even better. 

我希望這篇文章能帶來的一個結果是，agent framework builders 能從我和其他人的歷程中學到東西，讓 frameworks 變得更好。

Especially for builders who want to move fast but need deep control.

尤其是對那些想快速前進、但同時又需要深度控制的 builders 而言。

**DISCLAIMER 2**: I'm not going to talk about MCP. I'm sure you can see where it fits in.

**免責聲明 2**：我不打算談 MCP。我相信你看得出它適合放在哪裡。

**DISCLAIMER 3**: I'm using mostly typescript, for [reasons](https://www.linkedin.com/posts/dexterihorthy_llms-typescript-aiagents-activity-7290858296679313408-Lh9e?utm_source=share&utm_medium=member_desktop&rcm=ACoAAA4oHTkByAiD-wZjnGsMBUL_JT6nyyhOh30) but all this stuff works in python or any other language you prefer. 

**免責聲明 3**：我主要使用 TypeScript，出於一些[原因](https://www.linkedin.com/posts/dexterihorthy_llms-typescript-aiagents-activity-7290858296679313408-Lh9e?utm_source=share&utm_medium=member_desktop&rcm=ACoAAA4oHTkByAiD-wZjnGsMBUL_JT6nyyhOh30)，但這些做法在 Python 或你偏好的任何其他語言中都同樣適用。


Anyways back to the thing...

總之，回到正題……

</details>

### Design Patterns for great LLM applications
### 優秀 LLM 應用程式的 Design Patterns

After digging through hundreds of AI libriaries and working with dozens of founders, my instinct is this:

在研究了數百個 AI 函式庫，並與數十位創辦人合作之後，我的直覺結論是：

1. There are some core things that make agents great
1. 有一些核心要素會讓 agents 變得很優秀
2. Going all in on a framework and building what is essentially a greenfield rewrite may be counter-productive
2. 全面投入某個 framework，做一個本質上等於 greenfield rewrite 的專案，可能會適得其反
3. There are some core principles that make agents great, and you will get most/all of them if you pull in a framework
3. 確實有一些核心原則會讓 agents 變得很優秀，而如果你引入 framework，通常能得到其中大部分甚至全部
4. BUT, the fastest way I've seen for builders to get high-quality AI software in the hands of customers is to take small, modular concepts from agent building, and incorporate them into their existing product
4. 但是，就我所見，builders 想把高品質 AI software 交到客戶手中的最快方法，是從 agent building 中抽取小型、模組化的概念，並將它們整合進既有產品
5. These modular concepts from agents can be defined and applied by most skilled software engineers, even if they don't have an AI background
5. 這些來自 agents 的模組化概念，大多數熟練的 software engineers 都能定義並套用，即使他們沒有 AI 背景也一樣


> #### The fastest way I've seen for builders to get good AI software in the hands of customers is to take small, modular concepts from agent building, and incorporate them into their existing product
> #### 就我所見，builders 想把優秀的 AI software 交到客戶手中的最快方法，是從 agent building 中抽取小型、模組化的概念，並將它們整合進既有產品


## The 12 Factors (again)
## 12 Factors（再次列出）


- [How We Got Here: A Brief History of Software](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/brief-history-of-software.md)
- [我們如何走到這裡：Software 簡史](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/brief-history-of-software.md)
- [Factor 1: Natural Language to Tool Calls](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-01-natural-language-to-tool-calls.md)
- [Factor 1：自然語言轉 tool calls](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-01-natural-language-to-tool-calls.md)
- [Factor 2: Own your prompts](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-02-own-your-prompts.md)
- [Factor 2：掌握你的 prompts](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-02-own-your-prompts.md)
- [Factor 3: Own your context window](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)
- [Factor 3：掌握你的 context window](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)
- [Factor 4: Tools are just structured outputs](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs.md)
- [Factor 4：Tools 只是 structured outputs](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs.md)
- [Factor 5: Unify execution state and business state](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md)
- [Factor 5：統一 execution state 與 business state](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md)
- [Factor 6: Launch/Pause/Resume with simple APIs](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md)
- [Factor 6：用簡單 API 進行 Launch/Pause/Resume](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md)
- [Factor 7: Contact humans with tool calls](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md)
- [Factor 7：用 tool calls 聯繫人類](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md)
- [Factor 8: Own your control flow](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md)
- [Factor 8：掌握你的 control flow](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md)
- [Factor 9: Compact Errors into Context Window](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-09-compact-errors.md)
- [Factor 9：將 Errors 壓縮進 Context Window](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-09-compact-errors.md)
- [Factor 10: Small, Focused Agents](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md)
- [Factor 10：小而專注的 Agents](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md)
- [Factor 11: Trigger from anywhere, meet users where they are](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-11-trigger-from-anywhere.md)
- [Factor 11：從任何地方觸發，在使用者所在之處與他們相遇](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-11-trigger-from-anywhere.md)
- [Factor 12: Make your agent a stateless reducer](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-12-stateless-reducer.md)
- [Factor 12：讓你的 agent 成為 stateless reducer](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-12-stateless-reducer.md)


## Honorable Mentions / other advice
## 特別提及 / 其他建議

- [Factor 13: Pre-fetch all the context you might need](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/appendix-13-pre-fetch.md)
- [Factor 13：預先抓取你可能需要的所有 context](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/appendix-13-pre-fetch.md)


## Related Resources
## 相關資源

- Contribute to this guide [here](https://github.com/humanlayer/12-factor-agents)
- [I talked about a lot of this on an episode of the Tool Use podcast](https://youtu.be/8bIHcttkOTE) in March 2025
- I write about some of this stuff at [The Outer Loop](https://theouterloop.substack.com)
- I do [webinars about Maximizing LLM Performance](https://github.com/hellovai/ai-that-works/tree/main) with [@hellovai](https://github.com/hellovai)
- We build OSS agents with this methodology under [got-agents/agents](https://github.com/got-agents/agents)
- We ignored all our own advice and built a [framework for running distributed agents in kubernetes](https://github.com/humanlayer/kubechain)
- Other links from this guide:
  - [12 Factor Apps](https://12factor.net)
  - [Building Effective Agents (Anthropic)](https://www.anthropic.com/engineering/building-effective-agents#agents)
  - [Prompts are Functions](https://thedataexchange.media/baml-revolution-in-ai-engineering/ )
  - [Library patterns: Why frameworks are evil](https://tomasp.net/blog/2015/library-frameworks/)
  - [The Wrong Abstraction](https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction)
  - [Mailcrew Agent](https://github.com/dexhorthy/mailcrew)
  - [Mailcrew Demo Video](https://www.youtube.com/watch?v=f_cKnoPC_Oo)
  - [Chainlit Demo](https://x.com/chainlit_io/status/1858613325921480922)
  - [TypeScript for LLMs](https://www.linkedin.com/posts/dexterihorthy_llms-typescript-aiagents-activity-7290858296679313408-Lh9e)
  - [Schema Aligned Parsing](https://www.boundaryml.com/blog/schema-aligned-parsing)
  - [Function Calling vs Structured Outputs vs JSON Mode](https://www.vellum.ai/blog/when-should-i-use-function-calling-structured-outputs-or-json-mode)
  - [BAML on GitHub](https://github.com/boundaryml/baml)
  - [OpenAI JSON vs Function Calling](https://docs.llamaindex.ai/en/stable/examples/llm/openai_json_vs_function_calling/)
  - [Outer Loop Agents](https://theouterloop.substack.com/p/openais-realtime-api-is-a-step-towards)
  - [Airflow](https://airflow.apache.org/)
  - [Prefect](https://www.prefect.io/)
  - [Dagster](https://dagster.io/)
  - [Inngest](https://www.inngest.com/)
  - [Windmill](https://www.windmill.dev/)
  - [The AI Agent Index (MIT)](https://aiagentindex.mit.edu/)
  - [NotebookLM on Finding Model Capability Boundaries](https://open.substack.com/pub/swyx/p/notebooklm?selection=08e1187c-cfee-4c63-93c9-71216640a5f8)

- 在[這裡](https://github.com/humanlayer/12-factor-agents)為本指南做出貢獻
- 我在 2025 年 3 月的 [Tool Use podcast 某一集](https://youtu.be/8bIHcttkOTE)中談到許多這裡的內容
- 我也在 [The Outer Loop](https://theouterloop.substack.com) 撰寫其中一些主題
- 我會和 [@hellovai](https://github.com/hellovai) 一起舉辦[Maximizing LLM Performance 的 webinars](https://github.com/hellovai/ai-that-works/tree/main)
- 我們依循這套方法論在 [got-agents/agents](https://github.com/got-agents/agents) 中打造 OSS agents
- 我們完全無視自己所有的建議，做了一個[在 kubernetes 上執行 distributed agents 的 framework](https://github.com/humanlayer/kubechain)
- 本指南的其他連結：
  - [12 Factor Apps](https://12factor.net)
  - [Building Effective Agents (Anthropic)](https://www.anthropic.com/engineering/building-effective-agents#agents)
  - [Prompts are Functions](https://thedataexchange.media/baml-revolution-in-ai-engineering/ )
  - [Library patterns: Why frameworks are evil](https://tomasp.net/blog/2015/library-frameworks/)
  - [The Wrong Abstraction](https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction)
  - [Mailcrew Agent](https://github.com/dexhorthy/mailcrew)
  - [Mailcrew Demo Video](https://www.youtube.com/watch?v=f_cKnoPC_Oo)
  - [Chainlit Demo](https://x.com/chainlit_io/status/1858613325921480922)
  - [TypeScript for LLMs](https://www.linkedin.com/posts/dexterihorthy_llms-typescript-aiagents-activity-7290858296679313408-Lh9e)
  - [Schema Aligned Parsing](https://www.boundaryml.com/blog/schema-aligned-parsing)
  - [Function Calling vs Structured Outputs vs JSON Mode](https://www.vellum.ai/blog/when-should-i-use-function-calling-structured-outputs-or-json-mode)
  - [BAML on GitHub](https://github.com/boundaryml/baml)
  - [OpenAI JSON vs Function Calling](https://docs.llamaindex.ai/en/stable/examples/llm/openai_json_vs_function_calling/)
  - [Outer Loop Agents](https://theouterloop.substack.com/p/openais-realtime-api-is-a-step-towards)
  - [Airflow](https://airflow.apache.org/)
  - [Prefect](https://www.prefect.io/)
  - [Dagster](https://dagster.io/)
  - [Inngest](https://www.inngest.com/)
  - [Windmill](https://www.windmill.dev/)
  - [The AI Agent Index (MIT)](https://aiagentindex.mit.edu/)
  - [NotebookLM on Finding Model Capability Boundaries](https://open.substack.com/pub/swyx/p/notebooklm?selection=08e1187c-cfee-4c63-93c9-71216640a5f8)

## Contributors
## 貢獻者

Thanks to everyone who has contributed to 12-factor agents!

感謝所有為 12-factor agents 做出貢獻的人！

[<img src="https://avatars.githubusercontent.com/u/3730605?v=4&s=80" width="80px" alt="dexhorthy" />](https://github.com/dexhorthy) [<img src="https://avatars.githubusercontent.com/u/50557586?v=4&s=80" width="80px" alt="Sypherd" />](https://github.com/Sypherd) [<img src="https://avatars.githubusercontent.com/u/66259401?v=4&s=80" width="80px" alt="tofaramususa" />](https://github.com/tofaramususa) [<img src="https://avatars.githubusercontent.com/u/18105223?v=4&s=80" width="80px" alt="a-churchill" />](https://github.com/a-churchill) [<img src="https://avatars.githubusercontent.com/u/4084885?v=4&s=80" width="80px" alt="Elijas" />](https://github.com/Elijas) [<img src="https://avatars.githubusercontent.com/u/39267118?v=4&s=80" width="80px" alt="hugolmn" />](https://github.com/hugolmn) [<img src="https://avatars.githubusercontent.com/u/1882972?v=4&s=80" width="80px" alt="jeremypeters" />](https://github.com/jeremypeters)

[<img src="https://avatars.githubusercontent.com/u/380402?v=4&s=80" width="80px" alt="kndl" />](https://github.com/kndl) [<img src="https://avatars.githubusercontent.com/u/16674643?v=4&s=80" width="80px" alt="maciejkos" />](https://github.com/maciejkos) [<img src="https://avatars.githubusercontent.com/u/85041180?v=4&s=80" width="80px" alt="pfbyjy" />](https://github.com/pfbyjy) [<img src="https://avatars.githubusercontent.com/u/36044389?v=4&s=80" width="80px" alt="0xRaduan" />](https://github.com/0xRaduan) [<img src="https://avatars.githubusercontent.com/u/7169731?v=4&s=80" width="80px" alt="zyuanlim" />](https://github.com/zyuanlim) [<img src="https://avatars.githubusercontent.com/u/15862501?v=4&s=80" width="80px" alt="lombardo-chcg" />](https://github.com/lombardo-chcg) [<img src="https://avatars.githubusercontent.com/u/160066852?v=4&s=80" width="80px" alt="sahanatvessel" />](https://github.com/sahanatvessel)
 
## License
## 授權

All content and images are licensed under a <a href="https://creativecommons.org/licenses/by-sa/4.0/">CC BY-SA 4.0 License</a>

所有內容與圖片皆採用 <a href="https://creativecommons.org/licenses/by-sa/4.0/">CC BY-SA 4.0 License</a> 授權。

Code is licensed under the <a href="https://www.apache.org/licenses/LICENSE-2.0">Apache 2.0 License</a>

Code 採用 <a href="https://www.apache.org/licenses/LICENSE-2.0">Apache 2.0 License</a> 授權。
