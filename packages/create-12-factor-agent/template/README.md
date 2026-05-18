# Chapter 0 - Hello World
# 第 0 章 - Hello World

Let's start with a basic TypeScript setup and a hello world program.

讓我們從基本的 TypeScript 設定與 hello world 程式開始。

This guide is written in TypeScript (yes, a python version is coming soon)

本指南以 TypeScript 撰寫（沒錯，python 版本即將推出）。

There are many checkpoints between the every file edit in theworkshop steps, 
so even if you aren't super familiar with typescript,
you should be able to keep up and run each example.

在 workshop 步驟中，每次檔案編輯之間都有許多檢查點，
因此即使你對 typescript 不算非常熟悉，
仍然應該能跟上並執行每個範例。

To run this guide, you'll need a relatively recent version of nodejs and npm installed

若要執行本指南，你需要先安裝較新的 nodejs 與 npm 版本。

You can use whatever nodejs version manager you want, [homebrew](https://formulae.brew.sh/formula/node) is fine

你可以使用任何喜歡的 nodejs 版本管理工具，[homebrew](https://formulae.brew.sh/formula/node) 也沒問題。


    brew install node@20

You should see the node version

你應該會看到 node 版本。

    node --version

Copy initial package.json

複製初始的 package.json

    cp ./walkthrough/00-package.json package.json

Install dependencies

安裝相依套件

    npm install

Copy tsconfig.json

複製 tsconfig.json

    cp ./walkthrough/00-tsconfig.json tsconfig.json

add .gitignore

加入 .gitignore

    cp ./walkthrough/00-.gitignore .gitignore

Create src folder

建立 src 資料夾

    mkdir -p src

Add a simple hello world index.ts

加入一個簡單的 hello world `index.ts`

    cp ./walkthrough/00-index.ts src/index.ts

Run it to verify

執行它以驗證結果

    npx tsx src/index.ts

You should see:

你應該會看到：

    hello, world!


# Chapter 1 - CLI and Agent Loop
# 第 1 章 - CLI 與 Agent Loop

Now let's add BAML and create our first agent with a CLI interface.

現在讓我們加入 BAML，並建立第一個具有 CLI 介面的 agent。

First, we'll need to install [BAML](https://github.com/boundaryml/baml)
which is a tool for prompting and structured outputs.

首先，我們需要安裝 [BAML](https://github.com/boundaryml/baml)，
它是一個用於 prompting 與 structured outputs 的工具。


    npm install @boundaryml/baml

Initialize BAML

初始化 BAML

    npx baml-cli init

Remove default resume.baml

移除預設的 `resume.baml`

    rm baml_src/resume.baml

Add our starter agent, a single baml prompt that we'll build on

加入我們的起始 agent，也就是之後會持續擴充的單一 baml prompt。

    cp ./walkthrough/01-agent.baml baml_src/agent.baml

Generate BAML client code

產生 BAML client 程式碼

    npx baml-cli generate

Enable BAML logging for this section

為本章節啟用 BAML logging

    export BAML_LOG=debug

Add the CLI interface

加入 CLI 介面

    cp ./walkthrough/01-cli.ts src/cli.ts

Update index.ts to use the CLI

更新 `index.ts` 以使用 CLI

    cp ./walkthrough/01-index.ts src/index.ts

Add the agent implementation

加入 agent 實作

    cp ./walkthrough/01-agent.ts src/agent.ts

The the BAML code is configured to use BASETEN_API_KEY by default

BAML 程式碼預設已設定為使用 `BASETEN_API_KEY`。

To get a Baseten API key and URL, create an account at [baseten.co](https://baseten.co),
and then deploy [Qwen3 32B from the model library](https://www.baseten.co/library/qwen-3-32b/).

若要取得 Baseten API key 與 URL，請先在 [baseten.co](https://baseten.co) 建立帳號，
接著部署 [Qwen3 32B from the model library](https://www.baseten.co/library/qwen-3-32b/)。

```rust 
  function DetermineNextStep(thread: string) -> DoneForNow {
      client Qwen3
      // ...
```

If you want to run the example with no changes, you can set the BASETEN_API_KEY env var to any valid baseten key.

如果你想在不做任何變更的情況下執行範例，可以把 `BASETEN_API_KEY` env var 設為任何有效的 baseten key。

If you want to try swapping out the model, you can change the `client` line.

如果你想嘗試替換 model，可以修改 `client` 那一行。

[Docs on baml clients can be found here](https://docs.boundaryml.com/guide/baml-basics/switching-llms)

[關於 baml client 的文件可在此查看](https://docs.boundaryml.com/guide/baml-basics/switching-llms)

For example, you can configure [gemini](https://docs.boundaryml.com/ref/llm-client-providers/google-ai-gemini) 
or [anthropic](https://docs.boundaryml.com/ref/llm-client-providers/anthropic) as your model provider.

例如，你可以將 [gemini](https://docs.boundaryml.com/ref/llm-client-providers/google-ai-gemini)
或 [anthropic](https://docs.boundaryml.com/ref/llm-client-providers/anthropic) 設定為你的 model provider。

For example, to use openai with an OPENAI_API_KEY, you can do:

例如，若要以 `OPENAI_API_KEY` 使用 openai，可以這樣做：

    client "openai/gpt-4o"


Set your env vars

設定你的 env vars

    export BASETEN_API_KEY=...
export BASETEN_BASE_URL=...

Try it out

試試看

    npx tsx src/index.ts hello

you should see a familiar response from the model

你應該會看到 model 傳回熟悉的回應。

    {
      intent: 'done_for_now',
      message: 'Hello! How can I assist you today?'
    }


# Chapter 2 - Add Calculator Tools
# 第 2 章 - 加入 Calculator Tools

Let's add some calculator tools to our agent.

讓我們為 agent 加入一些 calculator tools。

Let's start by adding a tool definition for the calculator

先從加入 calculator 的 tool 定義開始。

These are simpile structured outputs that we'll ask the model to 
return as a "next step" in the agentic loop.

這些是簡單的 structured outputs，我們會要求 model
在 agentic loop 中把它們當成「next step」回傳。


    cp ./walkthrough/02-tool_calculator.baml baml_src/tool_calculator.baml

Now, let's update the agent's DetermineNextStep method to
expose the calculator tools as potential next steps

接著，讓我們更新 agent 的 `DetermineNextStep` 方法，
把 calculator tools 暴露為可能的 next steps。


    cp ./walkthrough/02-agent.baml baml_src/agent.baml

Generate updated BAML client

產生更新後的 BAML client

    npx baml-cli generate

Try out the calculator

試用 calculator

    npx tsx src/index.ts 'can you add 3 and 4'

You should see a tool call to the calculator

你應該會看到對 calculator 的 tool call。

    {
      intent: 'add',
      a: 3,
      b: 4
    }


# Chapter 3 - Process Tool Calls in a Loop
# 第 3 章 - 在迴圈中處理 Tool Calls

Now let's add a real agentic loop that can run the tools and get a final answer from the LLM.

現在讓我們加入一個真正的 agentic loop，能夠執行 tools 並從 LLM 取得最終答案。

First, lets update the agent to handle the tool call

首先，讓我們更新 agent 以處理 tool call。

    cp ./walkthrough/03-agent.ts src/agent.ts

Now, lets try it out

現在，來試試看。


    npx tsx src/index.ts 'can you add 3 and 4'

you should see the agent call the tool and then return the result

你應該會看到 agent 呼叫 tool，然後回傳結果。

    {
      intent: 'done_for_now',
      message: 'The sum of 3 and 4 is 7.'
    }

For the next step, we'll do a more complex calculation, let's turn off the baml logs for more concise output

下一步我們要做更複雜的計算，先把 baml logs 關掉，讓輸出更精簡。

    export BAML_LOG=off

Try a multi-step calculation

試試多步驟計算

    npx tsx src/index.ts 'can you add 3 and 4, then add 6 to that result'

you'll notice that tools like multiply and divide are not available

你會注意到像 `multiply` 和 `divide` 這類工具目前還不可用。

    npx tsx src/index.ts 'can you multiply 3 and 4'

next, let's add handlers for the rest of the calculator tools

接著，讓我們為其餘的 calculator tools 加上 handlers。


    cp ./walkthrough/03b-agent.ts src/agent.ts

Test subtraction

測試減法

    npx tsx src/index.ts 'can you subtract 3 from 4'

now, let's test the multiplication tool

現在，讓我們測試乘法工具。


    npx tsx src/index.ts 'can you multiply 3 and 4'

finally, let's test a more complex calculation with multiple operations

最後，讓我們測試一個包含多個運算的更複雜計算。


    npx tsx src/index.ts 'can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result'

congratulations, you've taking your first step into hand-rolling an agent loop.

恭喜，你已經踏出手刻 agent loop 的第一步。

from here, we're going to start incorporating some more intermediate and advanced
concepts for 12-factor agents.

接下來，我們會開始納入一些屬於 12-factor agents 的
中階與進階概念。



# Chapter 4 - Add Tests to agent.baml
# 第 4 章 - 為 agent.baml 加入測試

Let's add some tests to our BAML agent.

讓我們為 BAML agent 加入一些測試。

to start, leave the baml logs enabled

一開始請保持 baml logs 為啟用狀態。

    export BAML_LOG=debug

next, let's add some tests to the agent

接著，讓我們為 agent 加入一些測試。

We'll start with a simple test that checks the agent's ability to handle
a basic calculation.

我們會先從一個簡單測試開始，用來檢查 agent
處理基本計算的能力。


    cp ./walkthrough/04-agent.baml baml_src/agent.baml

Run the tests

執行測試

    npx baml-cli test

now, let's improve the test with assertions!

現在，讓我們用 assertions 來強化這個測試！

Assertions are a great way to make sure the agent is working as expected,
and can easily be extended to check for more complex behavior.

Assertions 是確保 agent 行為符合預期的絕佳方式，
也能輕鬆擴充來檢查更複雜的行為。


    cp ./walkthrough/04b-agent.baml baml_src/agent.baml

Run the tests

執行測試

    npx baml-cli test

as you add more tests, you can disable the logs to keep the output clean.
You may want to turn them on as you iterate on specific tests.

隨著你加入更多測試，可以關閉 logs 以保持輸出乾淨。
當你針對特定測試反覆調整時，可能又會想把它們打開。


    export BAML_LOG=off

now, let's add some more complex test cases,
where we resume from in the middle of an in-progress
agentic context window

現在，讓我們加入一些更複雜的測試案例，
模擬從一個進行中的
agentic context window 中途恢復。


    cp ./walkthrough/04c-agent.baml baml_src/agent.baml

let's try to run it

讓我們試著執行它。


    npx baml-cli test


# Chapter 5 - Multiple Human Tools
# 第 5 章 - 多個 Human Tools

In this section, we'll add support for multiple tools that serve to
contact humans.

在本章節中，我們會加入對多個用於
聯絡人類的 tools 的支援。


for this section, we'll disable the baml logs. You can optionally enable them if you want to see more details.

在本章節中，我們會停用 baml logs。若你想查看更多細節，也可以自行啟用。

    export BAML_LOG=off

first, let's add a tool that can request clarification from a human

首先，讓我們加入一個可向人類請求澄清的 tool。

this will be different from the "done_for_now" tool,
and can be used to more flexibly handle different types of human interactions
in your agent.

這會與 `done_for_now` tool 不同，
並可用來更彈性地處理 agent 中
不同類型的人類互動。


    cp ./walkthrough/05-agent.baml baml_src/agent.baml

next, let's re-generate the client code

接著，讓我們重新產生 client 程式碼。

NOTE - if you're using the VSCode extension for BAML,
the client will be regenerated automatically when you save the file
in your editor.

注意：如果你使用的是 BAML 的 VSCode extension，
當你在編輯器中儲存檔案時，client 會自動重新產生。


    npx baml-cli generate

now, let's update the agent to use the new tool

現在，讓我們更新 agent 以使用這個新 tool。


    cp ./walkthrough/05-agent.ts src/agent.ts

next, let's update the CLI to handle clarification requests
by requesting input from the user on the CLI

接著，讓我們更新 CLI，透過在 CLI 上向使用者請求輸入
來處理 clarification requests。


    cp ./walkthrough/05-cli.ts src/cli.ts

let's try it out

讓我們試試看。


    npx tsx src/index.ts 'can you multiply 3 and FD*(#F&& '

next, let's add a test that checks the agent's ability to handle
a clarification request

接著，讓我們加入一個測試，檢查 agent
處理 clarification request 的能力。


    cp ./walkthrough/05b-agent.baml baml_src/agent.baml

and now we can run the tests again

現在我們可以再次執行測試。


    npx baml-cli test

you'll notice the new test passes, but the hello world test fails

你會注意到新的測試通過了，但 hello world 測試失敗了。

This is because the agent's default behavior is to return "done_for_now"

這是因為 agent 的預設行為是回傳 `done_for_now`。


    cp ./walkthrough/05c-agent.baml baml_src/agent.baml

Verify tests pass

確認測試通過

    npx baml-cli test


# Chapter 6 - Customize Your Prompt with Reasoning
# 第 6 章 - 以 Reasoning 自訂 Prompt

In this section, we'll explore how to customize the prompt of the agent
with reasoning steps.

在本章節中，我們會探討如何透過 reasoning steps
自訂 agent 的 prompt。

this is core to [factor 2 - own your prompts](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-2-own-your-prompts.md)

這是 [factor 2 - own your prompts](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-2-own-your-prompts.md) 的核心內容。

there's a deep dive on reasoning on AI That Works [reasoning models versus reasoning steps](https://github.com/hellovai/ai-that-works/tree/main/2025-04-07-reasoning-models-vs-prompts)

若想深入了解 reasoning，可參考 AI That Works 的文章 [reasoning models versus reasoning steps](https://github.com/hellovai/ai-that-works/tree/main/2025-04-07-reasoning-models-vs-prompts)。

for this section, it will be helpful to leave the baml logs enabled

在本章節中，保持 baml logs 啟用會很有幫助。

    export BAML_LOG=debug

update the agent prompt to include a reasoning step

更新 agent prompt，讓它包含一個 reasoning step。


    cp ./walkthrough/06-agent.baml baml_src/agent.baml

generate the updated client

產生更新後的 client

    npx baml-cli generate

now, you can try it out with a simple prompt

現在，你可以用一個簡單的 prompt 來試試看。


    npx tsx src/index.ts 'can you multiply 3 and 4'

you should see output from the baml logs showing the reasoning steps

你應該會在 baml logs 中看到顯示 reasoning steps 的輸出。

#### optional challenge
#### 可選挑戰

add a field to your tool output format that includes the reasoning steps in the output!

在你的 tool output format 中加入一個欄位，讓輸出也包含 reasoning steps！



# Chapter 7 - Customize Your Context Window
# 第 7 章 - 自訂你的 Context Window

In this section, we'll explore how to customize the context window
of the agent.

在本章節中，我們會探討如何自訂 agent 的 context window。

this is core to [factor 3 - own your context window](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-3-own-your-context-window.md)

這是 [factor 3 - own your context window](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-3-own-your-context-window.md) 的核心內容。


update the agent to pretty-print the Context window for the model

更新 agent，讓它能為 model 將 Context window 以易讀格式輸出。


    cp ./walkthrough/07-agent.ts src/agent.ts

Test the formatting

測試格式化結果

    BAML_LOG=info npx tsx src/index.ts 'can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result'

next, let's update the agent to use XML formatting instead

接著，讓我們更新 agent，改用 XML 格式。

this is a very popular format for passing data to a model,

這是一種非常常見的 model 資料傳遞格式，

among other things, because of the token efficiency of XML.

其中一個原因就是 XML 具有不錯的 token 效率。


    cp ./walkthrough/07b-agent.ts src/agent.ts

let's try it out

讓我們試試看。


    BAML_LOG=info npx tsx src/index.ts 'can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result'

lets update our tests to match the new output format

讓我們更新測試，使其符合新的輸出格式。


    cp ./walkthrough/07c-agent.baml baml_src/agent.baml

check out the updated tests

查看更新後的測試


    npx baml-cli test


# Chapter 8 - Adding API Endpoints
# 第 8 章 - 加入 API Endpoints

Add an Express server to expose the agent via HTTP.

加入一個 Express server，透過 HTTP 對外提供 agent。

for this section, we'll disable the baml logs. You can optionally enable them if you want to see more details.

在本章節中，我們會停用 baml logs。若你想查看更多細節，也可以自行啟用。

    export BAML_LOG=off

Install Express and types

安裝 Express 與型別套件

    npm install express && npm install --save-dev @types/express supertest

Add the server implementation

加入 server 實作

    cp ./walkthrough/08-server.ts src/server.ts

Start the server

啟動 server

    npx tsx src/server.ts

Test with curl (in another terminal)

使用 curl 測試（在另一個 terminal 中）

    curl -X POST http://localhost:3000/thread \
  -H "Content-Type: application/json" \
  -d '{"message":"can you add 3 and 4"}'

You should get an answer from the agent which includes the
agentic trace, ending in a message like:

你應該會從 agent 收到一個回答，其中包含
agentic trace，最後會出現像這樣的訊息：


    {"intent":"done_for_now","message":"The sum of 3 and 4 is 7."}


# Chapter 9 - In-Memory State and Async Clarification
# 第 9 章 - In-Memory State 與非同步 Clarification

Add state management and async clarification support.

加入 state management 與非同步 clarification 支援。

for this section, we'll disable the baml logs. You can optionally enable them if you want to see more details.

在本章節中，我們會停用 baml logs。若你想查看更多細節，也可以自行啟用。

    export BAML_LOG=off

Add some simple in-memory state management for threads

為 threads 加入一些簡單的 in-memory state management。

    cp ./walkthrough/09-state.ts src/state.ts

update the server to use the state management

更新 server 以使用 state management。

* Add thread state management using `ThreadStore`
* return thread IDs and response URLs from the /thread endpoint
* implement GET /thread/:id
* implement POST /thread/:id/response

* 使用 `ThreadStore` 加入 thread state management
* 從 `/thread` endpoint 回傳 thread ID 與 response URL
* 實作 `GET /thread/:id`
* 實作 `POST /thread/:id/response`


    cp ./walkthrough/09-server.ts src/server.ts

Start the server

啟動 server

    npx tsx src/server.ts

Test clarification flow

測試 clarification flow

    curl -X POST http://localhost:3000/thread \
  -H "Content-Type: application/json" \
  -d '{"message":"can you multiply 3 and xyz"}'


# Chapter 10 - Adding Human Approval
# 第 10 章 - 加入 Human Approval

Add support for human approval of operations.

加入對操作進行 human approval 的支援。

for this section, we'll disable the baml logs. You can optionally enable them if you want to see more details.

在本章節中，我們會停用 baml logs。若你想查看更多細節，也可以自行啟用。

    export BAML_LOG=off

update the server to handle human approvals

更新 server 以處理 human approvals。

* Import `handleNextStep` to execute approved actions
* Add two payload types to distinguish approvals from responses
* Handle responses and approvals differently in the endpoint
* Show better error messages when things go wrongs

* 匯入 `handleNextStep` 以執行已核准的動作
* 加入兩種 payload type，以區分 approvals 與 responses
* 在 endpoint 中以不同方式處理 responses 與 approvals
* 在發生問題時顯示更好的錯誤訊息


    cp ./walkthrough/10-server.ts src/server.ts

Add a few methods to the agent to handle approvals and responses

在 agent 中加入幾個方法，以處理 approvals 與 responses。

    cp ./walkthrough/10-agent.ts src/agent.ts

Start the server

啟動 server

    npx tsx src/server.ts

Test division with approval

測試需要 approval 的除法流程

    curl -X POST http://localhost:3000/thread \
  -H "Content-Type: application/json" \
  -d '{"message":"can you divide 3 by 4"}'

You should see:

你應該會看到：

    {
      "thread_id": "2b243b66-215a-4f37-8bc6-9ace3849043b",
      "events": [
        {
          "type": "user_input",
          "data": "can you divide 3 by 4"
        },
        {
          "type": "tool_call",
          "data": {
            "intent": "divide",
            "a": 3,
            "b": 4,
            "response_url": "/thread/2b243b66-215a-4f37-8bc6-9ace3849043b/response"
          }
        }
      ]
    }

reject the request with another curl call, changing the thread ID

使用另一個 curl 呼叫拒絕此請求，並替換 thread ID。

    curl -X POST 'http://localhost:3000/thread/{thread_id}/response' \
  -H "Content-Type: application/json" \
  -d '{"type": "approval", "approved": false, "comment": "I dont think thats right, use 5 instead of 4"}'

You should see: the last tool call is now `"intent":"divide","a":3,"b":5`

你應該會看到：最後一個 tool call 現在變成 `"intent":"divide","a":3,"b":5`。

    {
      "events": [
        {
          "type": "user_input",
          "data": "can you divide 3 by 4"
        },
        {
          "type": "tool_call",
          "data": {
            "intent": "divide",
            "a": 3,
            "b": 4,
            "response_url": "/thread/2b243b66-215a-4f37-8bc6-9ace3849043b/response"
          }
        },
        {
          "type": "tool_response",
          "data": "user denied the operation with feedback: \"I dont think thats right, use 5 instead of 4\""
        },
        {
          "type": "tool_call",
          "data": {
            "intent": "divide",
            "a": 3,
            "b": 5,
            "response_url": "/thread/1f1f5ff5-20d7-4114-97b4-3fc52d5e0816/response"
          }
        }
      ]
    }

now you can approve the operation

現在你可以核准這個操作。

    curl -X POST 'http://localhost:3000/thread/{thread_id}/response' \
  -H "Content-Type: application/json" \
  -d '{"type": "approval", "approved": true}'

you should see the final message includes the tool response and final result!

你應該會看到最終訊息包含 tool response 與最終結果！

    ...
    {
      "type": "tool_response",
      "data": 0.5
    },
    {
      "type": "done_for_now",
      "message": "I divided 3 by 6 and the result is 0.5. If you have any more operations or queries, feel free to ask!",
      "response_url": "/thread/2b469403-c497-4797-b253-043aae830209/response"
    }


# Chapter 11 - Human Approvals over email
# 第 11 章 - 透過 Email 進行 Human Approvals

in this section, we'll add support for human approvals over email.

在本章節中，我們會加入透過 email 進行 human approval 的支援。

This will start a little bit contrived, just to get the concepts down -

一開始的設定會稍微有點刻意，目的是先把核心概念建立起來。

We'll start by invoking the workflow from the CLI but approvals for `divide`
and `request_more_information` will be handled over email,
then the final `done_for_now` answer will be printed back to the CLI

我們會先從 CLI 啟動 workflow，但 `divide`
與 `request_more_information` 的 approvals 會透過 email 處理，
接著再把最後的 `done_for_now` 答案輸出回 CLI。

While contrived, this is a great example of the flexibility you get from
[factor 7 - contact humans with tools](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-7-contact-humans-with-tools.md)

雖然這樣的例子有點刻意，但它很好地展示了
[factor 7 - contact humans with tools](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-7-contact-humans-with-tools.md) 所帶來的彈性。


for this section, we'll disable the baml logs. You can optionally enable them if you want to see more details.

在本章節中，我們會停用 baml logs。若你想查看更多細節，也可以自行啟用。

    export BAML_LOG=off

Install HumanLayer

安裝 HumanLayer

    npm install humanlayer

Update CLI to send `divide` and `request_more_information` to a human via email

更新 CLI，讓 `divide` 與 `request_more_information` 透過 email 傳送給人類處理。

    cp ./walkthrough/11-cli.ts src/cli.ts

Run the CLI

執行 CLI

    npx tsx src/index.ts 'can you divide 4 by 5'

The last line of your program should mention human review step

程式最後一行應該會提到 human review 步驟。

    nextStep { intent: 'divide', a: 4, b: 5 }
    HumanLayer: Requested human approval from HumanLayer cloud

go ahead and respond to the email with some feedback:

接著請回覆該 email，提供一些 feedback：

![reject-email](https://github.com/humanlayer/12-factor-agents/blob/main/workshops/2025-05/walkthrough/11-email-reject.png?raw=true)


you should get another email with an updated attempt based on your feedback!

你應該會收到另一封 email，其中包含依據你 feedback 更新後的新嘗試！

You can go ahead and approve this one:

你可以直接核准這一次：

![approve-email](https://github.com/humanlayer/12-factor-agents/blob/main/workshops/2025-05/walkthrough/11-email-approve.png?raw=true)


and your final output will look like

你的最終輸出會長得像這樣：

    nextStep {
     intent: 'done_for_now',
     message: 'The division of 4 by 5 is 0.8. If you have any other calculations or questions, feel free to ask!'
    }
    The division of 4 by 5 is 0.8. If you have any other calculations or questions, feel free to ask!

lets implement the `request_more_information` flow as well

讓我們也實作 `request_more_information` flow。


    cp ./walkthrough/11b-cli.ts src/cli.ts

lets test the require_approval flow as by asking for a calculation
with garbled input:

讓我們透過輸入含有亂碼的計算要求，來測試 `require_approval` flow：


    npx tsx src/index.ts 'can you multiply 4 and xyz'

You should get an email with a request for clarification

你應該會收到一封請求澄清的 email。

    Can you clarify what 'xyz' represents in this context? Is it a specific number, variable, or something else?

you can response with something like

你可以回覆類似以下內容：

    use 8 instead of xyz

you should see a final result on the CLI like

你應該會在 CLI 上看到類似以下的最終結果：

    I have multiplied 4 and xyz, using the value 8 for xyz, resulting in 32.

as a final step, lets explore using a custom html template for the email

作為最後一步，讓我們試著為 email 使用自訂的 html template。


    cp ./walkthrough/11c-cli.ts src/cli.ts

first try with divide:

先從 `divide` 開始試試看：


    npx tsx src/index.ts 'can you divide 4 by 5'

you should see a slightly different email with the custom template

你應該會看到一封使用自訂 template、略有不同的 email。

![custom-template-email](https://github.com/humanlayer/12-factor-agents/blob/main/workshops/2025-05/walkthrough/11-email-custom.png?raw=true)

feel free to run with the flow and then you can try updating the template to your liking

你可以實際跑完整個 flow，然後再試著把 template 改成你喜歡的樣式。

(if you're using cursor, something as simple as highlighting the template and asking to "make it better"
should do the trick)

（如果你使用的是 cursor，只要反白 template 並要求它「make it better」，
通常就能達到效果。）

try triggering "request_more_information" as well!

也試著觸發 `request_more_information` 吧！


thats it - in the next chapter, we'll build a fully email-driven
workflow agent that uses webhooks for human approval

以上就是本章內容——在下一章中，我們會建立一個完全由 email 驅動、
並使用 webhook 進行 human approval 的 workflow agent。



# Chapter XX - HumanLayer Webhook Integration
# 第 XX 章 - HumanLayer Webhook 整合

the previous sections used the humanlayer SDK in "synchronous mode" - that
means every time we wait for human approval, we sit in a loop
polling until the human response if received.

前面的章節使用的是 humanlayer SDK 的「synchronous mode」——這表示
每次我們等待 human approval 時，都會停在一個迴圈中，
持續 polling，直到收到人類回應為止。

That's obviously not ideal, especially for production workloads,
so in this section we'll implement [factor 6 - launch / pause / resume with simple APIs](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-6-launch-pause-resume.md)
by updating the server to end processing after contacting a human, and use webhooks to receive the results.

這顯然不是理想做法，尤其對 production workloads 來說更是如此，
因此在本章節中，我們會透過更新 server，讓它在聯絡人類後先結束處理，並改用 webhook 接收結果，
以實作 [factor 6 - launch / pause / resume with simple APIs](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-6-launch-pause-resume.md)。


add code to initialize humanlayer in the server

在 server 中加入初始化 humanlayer 的程式碼。


    cp ./walkthrough/12-1-server-init.ts src/server.ts

next, lets update the /thread endpoint to

接著，讓我們更新 `/thread` endpoint，使其能夠：

1. handle requests asynchronously, returning immediately
2. create a human contact on request_more_information and done_for_now calls

1. 以非同步方式處理請求，並立即回傳
2. 在 `request_more_information` 與 `done_for_now` 呼叫時建立 human contact


Update the server to be able to handle request_clarification responses

更新 server，使其能夠處理 `request_clarification` 回應。

- remove the old /response endpoint and types
- update the /thread endpoint to run processing asynchronously, return immediately
- send a state.threadId when requesting human responses
- add a handleHumanResponse function to process the human response
- add a /webhook endpoint to handle the webhook response

- 移除舊的 `/response` endpoint 與相關 types
- 更新 `/thread` endpoint，使處理流程以非同步方式執行並立即回傳
- 在請求人類回應時傳送 `state.threadId`
- 新增 `handleHumanResponse` 函式以處理人類回應
- 新增 `/webhook` endpoint 以處理 webhook 回應


    cp ./walkthrough/12a-server.ts src/server.ts

Start the server in another terminal

在另一個 terminal 中啟動 server

    npx tsx src/server.ts

now that the server is running, send a payload to the '/thread' endpoint

現在 server 已經啟動，請傳送一個 payload 到 `/thread` endpoint。


__ do the response step

__ 執行回應步驟

__ now handle approvals for divide

__ 現在處理 `divide` 的 approvals

__ now also handle done_for_now

__ 現在也處理 `done_for_now`
