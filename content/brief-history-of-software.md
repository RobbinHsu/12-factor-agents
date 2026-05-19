[← Back to README](https://github.com/RobbinHsu/12-factor-agents/blob/main/README.md)

## The longer version: how we got here
## 較長版本：我們是如何走到今天的

### You don't have to listen to me
### 你不一定要聽我的

Whether you're new to agents or an ornery old veteran like me, I'm going to try to convince you to throw out most of what you think about AI Agents, take a step back, and rethink them from first principles. (spoiler alert if you didn't catch the OpenAI responses launch a few weeks back, but pushing MORE agent logic behind an API ain't it)

無論你是剛接觸 agents，還是像我一樣資深又有點固執的老手，我都想試著說服你：把你對 AI Agents 的大部分既有想法先放下，退一步，從第一原理重新思考它們。（如果你前幾週沒有留意 OpenAI responses 的發布，先劇透一下：把更多 agent logic 塞到 API 背後，並不是答案。）


## Agents are software, and a brief history thereof
## Agents 是軟體，以及一段相關的簡史

let's talk about how we got here

讓我們來談談我們是如何走到今天的。

### 60 years ago
### 60 年前

We're gonna talk a lot about Directed Graphs (DGs) and their Acyclic friends, DAGs. I'll start by pointing out that...well...software is a directed graph. There's a reason we used to represent programs as flow charts.

我們會大量談到 Directed Graphs（DGs）以及它們的 Acyclic 親戚 DAGs。先從一個觀點開始：嗯……軟體本身就是一種 directed graph。過去我們會用流程圖來表示程式，絕非沒有原因。

![010-software-dag](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/010-software-dag.png)

### 20 years ago
### 20 年前

Around 20 years ago, we started to see DAG orchestrators become popular. We're talking classics like [Airflow](https://airflow.apache.org/), [Prefect](https://www.prefect.io/), some predecessors, and some newer ones like ([dagster](https://dagster.io/), [inggest](https://www.inngest.com/), [windmill](https://www.windmill.dev/)). These followed the same graph pattern, with the added benefit of observability, modularity, retries, administration, etc.

大約 20 年前，DAG orchestrators 開始流行起來。像是經典的 [Airflow](https://airflow.apache.org/)、[Prefect](https://www.prefect.io/)，一些更早期的前身，以及較新的工具如 [dagster](https://dagster.io/)、[inggest](https://www.inngest.com/)、[windmill](https://www.windmill.dev/)。它們都遵循同樣的 graph pattern，並額外帶來 observability、modularity、retries、administration 等優勢。

![015-dag-orchestrators](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/015-dag-orchestrators.png)

### 10-15 years ago
### 10-15 年前

When ML models started to get good enough to be useful, we started to see DAGs with ML models sprinkled in. You might imagine steps like "summarize the text in this column into a new column" or "classify the support issues by severity or sentiment".

當 ML models 開始好到足以真正派上用場時，我們開始看到 DAGs 中穿插了 ML model 步驟。你可以想像像是「把這一欄的文字摘要成新的一欄」或「依嚴重程度或情緒對支援問題進行分類」這類步驟。

![020-dags-with-ml](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/020-dags-with-ml.png)

But at the end of the day, it's still mostly the same good old deterministic software.

但歸根究柢，它大多仍然是老派而可靠的 deterministic software。

### The promise of agents
### agents 的承諾

I'm not the first [person to say this](https://youtu.be/Dc99-zTMyMg?si=bcT0hIwWij2mR-40&t=73), but my biggest takeaway when I started learning about agents, was that you get to throw the DAG away. Instead of software engineers coding each step and edge case, you can give the agent a goal and a set of transitions:

我不是第一個[這麼說的人](https://youtu.be/Dc99-zTMyMg?si=bcT0hIwWij2mR-40&t=73)，但我在開始學習 agents 時最大的心得，就是你可以把 DAG 丟掉。你不必再讓 software engineers 為每個步驟與 edge case 寫死邏輯，而是可以給 agent 一個目標和一組 transitions：

![025-agent-dag](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/025-agent-dag.png)

And let the LLM make decisions in real time to figure out the path

並讓 LLM 即時做出決策，找出應該走的路徑。

![026-agent-dag-lines](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/026-agent-dag-lines.png)

The promise here is that you write less software, you just give the LLM the "edges" of the graph and let it figure out the nodes. You can recover from errors, you can write less code, and you may find that LLMs find novel solutions to problems.

這種作法的承諾在於：你可以寫更少的 software，只要把 graph 的「edges」交給 LLM，讓它自己推導節點。你可以從錯誤中恢復、少寫一些程式碼，甚至可能發現 LLM 會替問題找出新穎的解法。

### Agents as loops
### 將 agents 視為迴圈

Put another way, you've got this loop consisting of 3 steps:

換個說法，你會有一個由 3 個步驟組成的迴圈：

1. LLM determines the next step in the workflow, outputting structured json ("tool calling")
2. Deterministic code executes the tool call
3. The result is appended to the context window 
4. repeat until the next step is determined to be "done"

1. LLM 決定 workflow 的下一步，並輸出結構化 json（「tool calling」）
2. Deterministic code 執行該 tool call
3. 將結果附加到 context window
4. 重複，直到下一步被判定為「done」

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

Our initial context is just the starting event (maybe a user message, maybe a cron fired, maybe a webhook, etc),
and we ask the llm to choose the next step (tool) or to determine that we're done.

我們的初始 context 就只是起始事件（可能是使用者訊息、cron 被觸發，或某個 webhook 等等），然後我們要求 llm 選擇下一步（tool），或判定整件事已經完成。

Here's a multi-step example:

以下是一個多步驟範例：

[![027-agent-loop-animation](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/027-agent-loop-animation.gif)](https://github.com/user-attachments/assets/3beb0966-fdb1-4c12-a47f-ed4e8240f8fd)

<details>
<summary><a href="https://github.com/RobbinHsu/12-factor-agents/blob/main/img/027-agent-loop-animation.gif">GIF Version</a></summary>

![027-agent-loop-animation](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/027-agent-loop-animation.gif)

</details>

And the "materialized" DAG that was generated would look something like:

而所產生、被「具體化」的 DAG 大致會長這樣：

![027-agent-loop-dag](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/027-agent-loop-dag.png)

### The problem with this "loop until you solve it" pattern
### 這種「一路迴圈直到解決問題」模式的問題

The biggest problems with this pattern:

這種模式最大的問題是：

- Agents get lost when the context window gets too long - they spin out trying the same broken approach over and over again
- literally thats it, but that's enough to kneecap the approach

- 當 context window 變得太長時，agents 很容易迷失方向——它們會一再打轉，重複嘗試同一個失敗的方法。
- 真的基本上就這一點，但這已經足以讓整個方法跛腳。

Even if you haven't hand-rolled an agent, you've probably seen this long-context problem in working with agentic coding tools. They just get lost after a while and you need to start a new chat.

即使你沒有親手從零打造過一個 agent，你大概也在使用 agentic coding tools 時看過這種長 context 問題。它們用久了就會迷航，最後你只能重新開一個新的 chat。

I'll even perhaps posit something I've heard in passing quite a bit, and that YOU probably have developed your own intuition around:

我甚至想進一步提出一個我常常聽到、而你大概也早已有直覺的觀點：

> ### **Even as models support longer and longer context windows, you'll ALWAYS get better results with a small, focused prompt and context**
>
> ### **即使模型支援越來越長的 context windows，你用小而聚焦的 prompt 與 context，永遠都會得到更好的結果**

Most builders I've talked to **pushed the "tool calling loop" idea to the side** when they realized that anything more than 10-20 turns becomes a big mess that the LLM can't recover from. Even if the agent gets it right 90% of the time, that's miles away from "good enough to put in customer hands". Can you imagine a web app that crashed on 10% of page loads?

我聊過的大多數 builders，在意識到只要超過 10-20 個 turns 就會變成 LLM 無法收拾的大混亂後，便**把「tool calling loop」這個想法先擱到一旁**。即使 agent 有 90% 的時間都做對，離「好到可以交到客戶手上」仍然差得很遠。你能想像一個 web app 在 10% 的頁面載入時直接當掉嗎？

**Update 2025-06-09** - I really like how [@swyx](https://x.com/swyx/status/1932125643384455237) put this:

**更新 2025-06-09** - 我很喜歡 [@swyx](https://x.com/swyx/status/1932125643384455237) 對這件事的表達方式：

<a href="https://x.com/swyx/status/1932125643384455237"><img width="593" alt="Screenshot 2025-07-02 at 11 50 50 AM" src="https://github.com/user-attachments/assets/c7d94042-e4b9-4b87-87fd-55c7ff94bb3b" /></a>

### What actually works - micro agents
### 真正有效的方法：micro agents

One thing that I **have** seen in the wild quite a bit is taking the agent pattern and sprinkling it into a broader more deterministic DAG. 

我**確實**常在實務中看到一種有效做法：把 agent pattern 局部嵌入到更廣泛、也更 deterministic 的 DAG 中。

![micro-agent-dag](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/028-micro-agent-dag.png)

You might be asking - "why use agents at all in this case?" - we'll get into that shortly, but basically, having language models managing well-scoped sets of tasks makes it easy to incorporate live human feedback, translating it into workflow steps without spinning out into context error loops. ([factor 1](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-01-natural-language-to-tool-calls.md), [factor 3](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md) [factor 7](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md)).

你可能會問：「在這種情況下，為什麼還要用 agents？」——我們稍後會談到這點，但核心原因是：讓 language models 管理界線清楚的任務集合，就能更容易納入即時的人類回饋，把它轉換成 workflow 步驟，而不會陷入 context error loop。([factor 1](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-01-natural-language-to-tool-calls.md)、[factor 3](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)、[factor 7](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md))。

> #### having language models managing well-scoped sets of tasks makes it easy to incorporate live human feedback...without spinning out into context error loops
>
> #### 讓 language models 管理界線清楚的任務集合，就能更容易納入即時的人類回饋……而不會陷入 context error loop

### A real life micro agent 
### 一個真實世界的 micro agent

Here's an example of how deterministic code might run one micro agent responsible for handling the human-in-the-loop steps for deployment. 

以下是一個範例，說明 deterministic code 如何執行一個 micro agent，專門負責處理 deployment 中 human-in-the-loop 的步驟。

![029-deploybot-high-level](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/029-deploybot-high-level.png)

* **Human** Merges PR to GitHub main branch
* **Deterministic Code** Deploys to staging env
* **Deterministic Code** Runs end-to-end (e2e) tests against staging
* **Deterministic Code** Hands to agent for prod deployment, with initial context: "deploy SHA 4af9ec0 to production"
* **Agent** calls `deploy_frontend_to_prod(4af9ec0)`
* **Deterministic code** requests human approval on this action
* **Human** Rejects the action with feedback "can you deploy the backend first?"
* **Agent** calls `deploy_backend_to_prod(4af9ec0)`
* **Deterministic code** requests human approval on this action
* **Human** approves the action
* **Deterministic code** executed the backend deployment
* **Agent** calls `deploy_frontend_to_prod(4af9ec0)`
* **Deterministic code** requests human approval on this action
* **Human** approves the action
* **Deterministic code** executed the frontend deployment
* **Agent** determines that the task was completed successfully, we're done!
* **Deterministic code** run the end-to-end tests against production
* **Deterministic code** task completed, OR pass to rollback agent to review failures and potentially roll back

* **Human** 將 PR 合併到 GitHub main branch
* **Deterministic Code** 部署到 staging env
* **Deterministic Code** 對 staging 執行 end-to-end（e2e）tests
* **Deterministic Code** 把 prod deployment 交給 agent，並提供初始 context：「deploy SHA 4af9ec0 to production」
* **Agent** 呼叫 `deploy_frontend_to_prod(4af9ec0)`
* **Deterministic code** 對此動作請求人類核准
* **Human** 以回饋「can you deploy the backend first?」拒絕此動作
* **Agent** 呼叫 `deploy_backend_to_prod(4af9ec0)`
* **Deterministic code** 對此動作請求人類核准
* **Human** 核准此動作
* **Deterministic code** 執行 backend deployment
* **Agent** 呼叫 `deploy_frontend_to_prod(4af9ec0)`
* **Deterministic code** 對此動作請求人類核准
* **Human** 核准此動作
* **Deterministic code** 執行 frontend deployment
* **Agent** 判定任務已成功完成，我們結束了！
* **Deterministic code** 對 production 執行 end-to-end tests
* **Deterministic code** 完成任務，或把失敗交給 rollback agent 審查並視情況回滾

[![033-deploybot-animation](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/033-deploybot.gif)](https://github.com/user-attachments/assets/deb356e9-0198-45c2-9767-231cb569ae13)

<details>
<summary><a href="https://github.com/RobbinHsu/12-factor-agents/blob/main/img/033-deploybot.gif">GIF Version</a></summary>

![033-deploybot-animation](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/033-deploybot.gif)

</details>

This example is based on a real life [OSS agent we've shipped to manage our deployments at Humanlayer](https://github.com/got-agents/agents/tree/main/deploybot-ts) - here is a real conversation I had with it last week:

這個範例來自我們在 Humanlayer 實際推出、用於管理 deployment 的 [OSS agent](https://github.com/got-agents/agents/tree/main/deploybot-ts)——下面是我上週和它的一段真實對話：

![035-deploybot-conversation](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/035-deploybot-conversation.png)


We haven't given this agent a huge pile of tools or tasks. The primary value in the LLM is parsing the human's plaintext feedback and proposing an updated course of action. We isolate tasks and contexts as much as possible to keep the LLM focused on a small, 5-10 step workflow.

我們沒有給這個 agent 一大堆 tools 或 tasks。LLM 的主要價值，在於解析人類的 plaintext 回饋，並提出更新後的行動方向。我們盡可能隔離 tasks 與 contexts，好讓 LLM 專注在一個小型、約 5-10 步的 workflow 上。

Here's another [more classic support / chatbot demo](https://x.com/chainlit_io/status/1858613325921480922).

這裡還有另一個[更經典的 support / chatbot demo](https://x.com/chainlit_io/status/1858613325921480922)。

### So what's an agent really?
### 所以，agent 到底是什麼？

- **prompt** - tell an LLM how to behave, and what "tools" it has available. The output of the prompt is a JSON object that describe the next step in the workflow (the "tool call" or "function call"). ([factor 2](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-02-own-your-prompts.md))
- **switch statement** - based on the JSON that the LLM returns, decide what to do with it. (part of [factor 8](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md))
- **accumulated context** - store the list of steps that have happened and their results ([factor 3](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md))
- **for loop** - until the LLM emits some sort of "Terminal" tool call (or plaintext response), add the result of the switch statement to the context window and ask the LLM to choose the next step. ([factor 8](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md))

- **prompt** - 告訴 LLM 應該如何行為，以及有哪些可用的「tools」。prompt 的輸出是一個 JSON 物件，用來描述 workflow 的下一步（也就是「tool call」或「function call」）。([factor 2](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-02-own-your-prompts.md))
- **switch statement** - 根據 LLM 回傳的 JSON，決定接下來要怎麼處理它。（屬於 [factor 8](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md) 的一部分）
- **accumulated context** - 儲存已經發生過的步驟及其結果列表。([factor 3](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md))
- **for loop** - 在 LLM 發出某種「Terminal」tool call（或 plaintext 回應）之前，持續把 switch statement 的結果加入 context window，並讓 LLM 選擇下一步。([factor 8](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md))

![040-4-components](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/040-4-components.png)

In the "deploybot" example, we gain a couple benefits from owning the control flow and context accumulation:

在「deploybot」這個例子中，掌握 control flow 與 context accumulation 能帶來幾個好處：

- In our **switch statement** and **for loop**, we can hijack control flow to pause for human input or to wait for completion of long-running tasks
- We can trivially serialize the **context** window for pause+resume
- In our **prompt**, we can optimize the heck out of how we pass instructions and "what happened so far" to the LLM

- 在 **switch statement** 和 **for loop** 中，我們可以接管 control flow，讓流程暫停以等待 human input，或等待長時間執行的 tasks 完成。
- 我們可以很輕鬆地把 **context** window 序列化，以支援 pause+resume。
- 在 **prompt** 中，我們可以大幅優化把指令與「目前為止發生了什麼」傳給 LLM 的方式。


[Part II](https://github.com/RobbinHsu/12-factor-agents/blob/main/README.md#12-factor-agents) will **formalize these patterns** so they can be applied to add impressive AI features to any software project, without needing to go all in on conventional implementations/definitions of "AI agent".

[Part II](https://github.com/RobbinHsu/12-factor-agents/blob/main/README.md#12-factor-agents) 會**將這些模式正式化**，讓它們可以被應用到任何軟體專案中，加入令人驚豔的 AI 功能，而不必完全投入傳統對「AI agent」的實作／定義方式。


[Factor 1 - Natural Language to Tool Calls →](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-01-natural-language-to-tool-calls.md)
