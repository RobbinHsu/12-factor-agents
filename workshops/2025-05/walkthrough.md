# Building the 12-factor agent template from scratch
# 從零開始建立 12-factor agent 範本

Steps to start from a bare TS repo and build up a 12-factor agent. This walkthrough will guide you through creating a TypeScript agent that follows the 12-factor methodology.

從一個空白的 TS repo 開始，逐步建立 12-factor agent 的步驟。本 walkthrough 會引導你建立一個遵循 12-factor 方法論的 TypeScript agent。

## Cleanup
## 清理

Make sure you're starting from a clean slate

請確認你是從乾淨的初始狀態開始。

Clean up existing files

清理現有檔案

    rm -rf baml_src/ && rm -rf src/

## Chapter 0 - Hello World
## 第 0 章 - Hello World

Let's start with a basic TypeScript setup and a hello world program.

讓我們先從基本的 TypeScript 設定與 hello world 程式開始。

This guide is written in TypeScript (yes, a python version is coming soon)

本指南以 TypeScript 撰寫（沒錯，python 版本即將推出）。

There are many checkpoints between the every file edit in theworkshop steps, 
so even if you aren't super familiar with typescript,
you should be able to keep up and run each example.

在 workshop 的步驟中，幾乎每次檔案編輯之間都有許多 checkpoint，
因此即使你對 typescript 還不算非常熟悉，
你也應該能夠跟上進度並執行每個範例。

To run this guide, you'll need a relatively recent version of nodejs and npm installed

若要執行本指南，你需要先安裝相對較新的 nodejs 與 npm。

You can use whatever nodejs version manager you want, [homebrew](https://formulae.brew.sh/formula/node) is fine

你可以使用任何你習慣的 nodejs 版本管理工具，[homebrew](https://formulae.brew.sh/formula/node) 也可以。


    brew install node@20

You should see the node version

你應該會看到 node 的版本。

    node --version

Copy initial package.json

複製初始的 package.json

    cp ./walkthrough/00-package.json package.json

<details>
<summary>show file / 顯示檔案</summary>

```json
// ./walkthrough/00-package.json
// ./walkthrough/00-package.json（檔案路徑）
{
    "name": "my-agent",
    "version": "0.1.0",
    "private": true,
    "scripts": {
      "dev": "tsx src/index.ts",
      "build": "tsc"
    },
    "dependencies": {
      "tsx": "^4.15.0",
      "typescript": "^5.0.0"
    },
    "devDependencies": {
      "@types/node": "^20.0.0",
      "@typescript-eslint/eslint-plugin": "^6.0.0",
      "@typescript-eslint/parser": "^6.0.0",
      "eslint": "^8.0.0"
    }
  }
```

</details>

Install dependencies

安裝相依套件

    npm install

Copy tsconfig.json

複製 tsconfig.json

    cp ./walkthrough/00-tsconfig.json tsconfig.json

<details>
<summary>show file / 顯示檔案</summary>

```json
// ./walkthrough/00-tsconfig.json
// ./walkthrough/00-tsconfig.json（檔案路徑）
{
    "compilerOptions": {
      "target": "ES2017",
      "lib": ["esnext"],
      "allowJs": true,
      "skipLibCheck": true,
      "strict": true,
      "noEmit": true,
      "esModuleInterop": true,
      "module": "esnext",
      "moduleResolution": "bundler",
      "resolveJsonModule": true,
      "isolatedModules": true,
      "jsx": "preserve",
      "incremental": true,
      "plugins": [],
      "paths": {
        "@/*": ["./*"]
      }
    },
    "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
    "exclude": ["node_modules", "walkthrough"]
  }
```

</details>

add .gitignore

加入 .gitignore

    cp ./walkthrough/00-.gitignore .gitignore

<details>
<summary>show file / 顯示檔案</summary>

```gitignore
// ./walkthrough/00-.gitignore
// ./walkthrough/00-.gitignore（檔案路徑）
baml_client/
node_modules/
```

</details>

Create src folder

建立 src 資料夾

Add a simple hello world index.ts

加入一個簡單的 hello world `index.ts`

    cp ./walkthrough/00-index.ts src/index.ts

<details>
<summary>show file / 顯示檔案</summary>

```ts
// ./walkthrough/00-index.ts
// ./walkthrough/00-index.ts（檔案路徑）
async function hello(): Promise<void> {
    console.log('hello, world!')
}

async function main() {
    await hello()
}

main().catch(console.error)
```

</details>

Run it to verify

執行它來確認結果

    npx tsx src/index.ts

You should see:

你應該會看到：

    hello, world!

## Chapter 1 - CLI and Agent Loop
## 第 1 章 - CLI 與 Agent Loop

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

移除預設的 resume.baml

    rm baml_src/resume.baml

Add our starter agent, a single baml prompt that we'll build on

加入我們的起始 agent，也就是一個之後會持續擴充的單一 baml prompt。

    cp ./walkthrough/01-agent.baml baml_src/agent.baml

<details>
<summary>show file / 顯示檔案</summary>

```rust
// ./walkthrough/01-agent.baml
// ./walkthrough/01-agent.baml（檔案路徑）
class DoneForNow {
  intent "done_for_now"
  message string 
}

function DetermineNextStep(
    thread: string 
) -> DoneForNow {
    client "openai/gpt-4o"

    prompt #"
        {{ _.role("system") }}

        You are a helpful assistant that can help with tasks.

        {{ _.role("user") }}

        You are working on the following thread:

        {{ thread }}

        What should the next step be?

        {{ ctx.output_format }}
    "#
}

test HelloWorld {
  functions [DetermineNextStep]
  args {
    thread #"
      {
        "type": "user_input",
        "data": "hello!"
      }
    "#
  }
}
```

</details>

Generate BAML client code

產生 BAML client 程式碼

    npx baml-cli generate

Enable BAML logging for this section

為本節啟用 BAML logging

    export BAML_LOG=debug

Add the CLI interface

加入 CLI 介面

    cp ./walkthrough/01-cli.ts src/cli.ts

<details>
<summary>show file / 顯示檔案</summary>

```ts
// ./walkthrough/01-cli.ts
// ./walkthrough/01-cli.ts（檔案路徑）
// cli.ts lets you invoke the agent loop from the command line
// cli.ts 讓你可以從命令列呼叫 agent loop
import { agentLoop, Thread, Event } from "./agent";

export async function cli() {
    // Get command line arguments, skipping the first two (node and script name)
    // 取得命令列參數，略過前兩個項目（node 與腳本名稱）
    const args = process.argv.slice(2);

    if (args.length === 0) {
        console.error("Error: Please provide a message as a command line argument");
        process.exit(1);
    }

    // Join all arguments into a single message
    // 將所有參數合併成一則訊息
    const message = args.join(" ");

    // Create a new thread with the user's message as the initial event
    // 以使用者訊息作為初始 event，建立新的 thread
    const thread = new Thread([{ type: "user_input", data: message }]);

    // Run the agent loop with the thread
    // 使用該 thread 執行 agent loop
    const result = await agentLoop(thread);
    console.log(result);
}
```

</details>

Update index.ts to use the CLI

更新 `index.ts` 以使用 CLI

```diff
src/index.ts
+import { cli } from "./cli"
+
 async function hello(): Promise<void> {
     console.log('hello, world!')
 
 async function main() {
-    await hello()
+    await cli()
 }
 
```

<details>
<summary>skip this step / 跳過此步驟</summary>

    cp ./walkthrough/01-index.ts src/index.ts

</details>

Add the agent implementation

加入 agent 實作

    cp ./walkthrough/01-agent.ts src/agent.ts

<details>
<summary>show file / 顯示檔案</summary>

```ts
// ./walkthrough/01-agent.ts
// ./walkthrough/01-agent.ts（檔案路徑）
import { b } from "../baml_client";

// tool call or a respond to human tool
// tool call，或是回覆 human 的 tool
type AgentResponse = Awaited<ReturnType<typeof b.DetermineNextStep>>;

export interface Event {
    type: string
    data: any;
}

export class Thread {
    events: Event[] = [];

    constructor(events: Event[]) {
        this.events = events;
    }

    serializeForLLM() {
        // can change this to whatever custom serialization you want to do, XML, etc
        // 你可以把這裡改成任何你想要的自訂序列化方式，例如 XML 等
        // e.g. https://github.com/got-agents/agents/blob/59ebbfa236fc376618f16ee08eb0f3bf7b698892/linear-assistant-ts/src/agent.ts#L66-L105
        // 例如：https://github.com/got-agents/agents/blob/59ebbfa236fc376618f16ee08eb0f3bf7b698892/linear-assistant-ts/src/agent.ts#L66-L105
        return JSON.stringify(this.events);
    }
}

// right now this just runs one turn with the LLM, but
// 目前這只會讓 LLM 執行一個回合，但
// we'll update this function to handle all the agent logic
// 我們稍後會更新這個函式，讓它處理所有 agent 邏輯
export async function agentLoop(thread: Thread): Promise<AgentResponse> {
    const nextStep = await b.DetermineNextStep(thread.serializeForLLM());
    return nextStep;
}
```

</details>

The the BAML code is configured to use OPENAI_API_KEY by default

BAML 程式碼預設會設定為使用 `OPENAI_API_KEY`。

As you're testing, you can change the model / provider to something else
as you please

在測試時，你也可以依照需要把 model / provider 改成其他選項。

        client "openai/gpt-4o"

[Docs on baml clients can be found here](https://docs.boundaryml.com/guide/baml-basics/switching-llms)

[可在此查看 baml client 的文件](https://docs.boundaryml.com/guide/baml-basics/switching-llms)

For example, you can configure [gemini](https://docs.boundaryml.com/ref/llm-client-providers/google-ai-gemini) 
or [anthropic](https://docs.boundaryml.com/ref/llm-client-providers/anthropic) as your model provider.

例如，你可以將 [gemini](https://docs.boundaryml.com/ref/llm-client-providers/google-ai-gemini) 
或 [anthropic](https://docs.boundaryml.com/ref/llm-client-providers/anthropic) 設定為你的 model provider。

If you want to run the example with no changes, you can set the OPENAI_API_KEY env var to any valid openai key.

如果你想在不做任何變更的情況下執行此範例，可以將 `OPENAI_API_KEY` 環境變數設為任一有效的 openai key。


    export OPENAI_API_KEY=...

Try it out

試著執行看看

    npx tsx src/index.ts hello

you should see a familiar response from the model

你應該會看到 model 回傳熟悉的回應

    {
  intent: 'done_for_now',
  message: 'Hello! How can I assist you today?'
}

## Chapter 2 - Add Calculator Tools
## 第 2 章 - 新增 Calculator 工具

Let's add some calculator tools to our agent.

讓我們為 agent 新增一些 Calculator 工具。

Let's start by adding a tool definition for the calculator

讓我們先為 calculator 新增一個 tool 定義。

These are simpile structured outputs that we'll ask the model to
return as a "next step" in the agentic loop.

這些是簡單的結構化輸出，我們會要求模型在 agentic loop 中
將它們作為「下一步」回傳。


    cp ./walkthrough/02-tool_calculator.baml baml_src/tool_calculator.baml

<details>
<summary>show file / 顯示檔案</summary>

```rust
// ./walkthrough/02-tool_calculator.baml
// 檔案：./walkthrough/02-tool_calculator.baml
type CalculatorTools = AddTool | SubtractTool | MultiplyTool | DivideTool


class AddTool {
    intent "add"
    a int | float
    b int | float
}

class SubtractTool {
    intent "subtract"
    a int | float
    b int | float
}

class MultiplyTool {
    intent "multiply"
    a int | float
    b int | float
}

class DivideTool {
    intent "divide"
    a int | float
    b int | float
}
```

</details>

Now, let's update the agent's DetermineNextStep method to
expose the calculator tools as potential next steps

現在，讓我們更新 agent 的 DetermineNextStep 方法，
將 calculator 工具作為可能的下一步公開給模型。


```diff
baml_src/agent.baml
 function DetermineNextStep(
     thread: string 
-) -> DoneForNow {
+) -> CalculatorTools | DoneForNow {
     client "openai/gpt-4o"
 
```

<details>
<summary>skip this step / 跳過此步驟</summary>

    cp ./walkthrough/02-agent.baml baml_src/agent.baml

</details>

Generate updated BAML client

產生更新後的 BAML client。

    npx baml-cli generate

Try out the calculator

試用 calculator。

    npx tsx src/index.ts 'can you add 3 and 4'

You should see a tool call to the calculator

你應該會看到對 calculator 的 tool call。

    {
  intent: 'add',
  a: 3,
  b: 4
}

## Chapter 3 - Process Tool Calls in a Loop
## 第 3 章 - 在迴圈中處理 Tool Calls

Now let's add a real agentic loop that can run the tools and get a final answer from the LLM.

現在讓我們加入一個真正的 agentic loop，能夠執行工具並從 LLM 取得最終答案。

First, lets update the agent to handle the tool call

首先，讓我們更新 agent，讓它能處理 tool call。


```diff
src/agent.ts
 }
 
-// right now this just runs one turn with the LLM, but
-// 目前這個版本只會與 LLM 執行一輪互動，
-// we'll update this function to handle all the agent logic
-// 但我們稍後會更新這個函式來處理所有 agent 邏輯
-export async function agentLoop(thread: Thread): Promise<AgentResponse> {
-    const nextStep = await b.DetermineNextStep(thread.serializeForLLM());
-    return nextStep;
+
+
+export async function agentLoop(thread: Thread): Promise<string> {
+
+    while (true) {
+        const nextStep = await b.DetermineNextStep(thread.serializeForLLM());
+        console.log("nextStep", nextStep);
+
+        switch (nextStep.intent) {
+            case "done_for_now":
+                // response to human, return the next step object
+                // 回應人類，並回傳下一步物件
+                return nextStep.message;
+            case "add":
+                thread.events.push({
+                    "type": "tool_call",
+                    "data": nextStep
+                });
+                const result = nextStep.a + nextStep.b;
+                console.log("tool_response", result);
+                thread.events.push({
+                    "type": "tool_response",
+                    "data": result
+                });
+                continue;
+            default:
+                throw new Error(`Unknown intent: ${nextStep.intent}`);
+        }
+    }
 }
 
```

<details>
<summary>skip this step / 跳過此步驟</summary>

    cp ./walkthrough/03-agent.ts src/agent.ts

</details>

Now, lets try it out

現在，讓我們試試看。

    npx tsx src/index.ts 'can you add 3 and 4'

you should see the agent call the tool and then return the result

你應該會看到 agent 呼叫工具，然後回傳結果。

    {
  intent: 'done_for_now',
  message: 'The sum of 3 and 4 is 7.'
}

For the next step, we'll do a more complex calculation, let's turn off the baml logs for more concise output

下一步我們要進行更複雜的計算，先把 baml logs 關掉，讓輸出更精簡。

    export BAML_LOG=off

Try a multi-step calculation

試試多步驟計算。

    npx tsx src/index.ts 'can you add 3 and 4, then add 6 to that result'

you'll notice that tools like multiply and divide are not available

你會注意到 multiply 和 divide 這類工具目前還不能使用。

    npx tsx src/index.ts 'can you multiply 3 and 4'

next, let's add handlers for the rest of the calculator tools

接著，讓我們為其餘的 calculator 工具加入 handler。


```diff
src/agent.ts
-import { b } from "../baml_client";
+import { AddTool, SubtractTool, DivideTool, MultiplyTool, b } from "../baml_client";
 
-// tool call or a respond to human tool
-// tool call，或回應人類的工具
-type AgentResponse = Awaited<ReturnType<typeof b.DetermineNextStep>>;
-
 export interface Event {
     type: string
 }
 
+export type CalculatorTool = AddTool | SubtractTool | MultiplyTool | DivideTool;
 
+export async function handleNextStep(nextStep: CalculatorTool, thread: Thread): Promise<Thread> {
+    let result: number;
+    switch (nextStep.intent) {
+        case "add":
+            result = nextStep.a + nextStep.b;
+            console.log("tool_response", result);
+            thread.events.push({
+                "type": "tool_response",
+                "data": result
+            });
+            return thread;
+        case "subtract":
+            result = nextStep.a - nextStep.b;
+            console.log("tool_response", result);
+            thread.events.push({
+                "type": "tool_response",
+                "data": result
+            });
+            return thread;
+        case "multiply":
+            result = nextStep.a * nextStep.b;
+            console.log("tool_response", result);
+            thread.events.push({
+                "type": "tool_response",
+                "data": result
+            });
+            return thread;
+        case "divide":
+            result = nextStep.a / nextStep.b;
+            console.log("tool_response", result);
+            thread.events.push({
+                "type": "tool_response",
+                "data": result
+            });
+            return thread;
+    }
+}
 
 export async function agentLoop(thread: Thread): Promise<string> {
         console.log("nextStep", nextStep);
 
+        thread.events.push({
+            "type": "tool_call",
+            "data": nextStep
+        });
+
         switch (nextStep.intent) {
             case "done_for_now":
                 return nextStep.message;
             case "add":
-                thread.events.push({
-                    "type": "tool_call",
-                    "data": nextStep
-                });
-                const result = nextStep.a + nextStep.b;
-                console.log("tool_response", result);
-                thread.events.push({
-                    "type": "tool_response",
-                    "data": result
-                });
-                continue;
-            default:
-                throw new Error(`Unknown intent: ${nextStep.intent}`);
+            case "subtract":
+            case "multiply":
+            case "divide":
+                thread = await handleNextStep(nextStep, thread);
         }
     }
```

<details>
<summary>skip this step / 跳過此步驟</summary>

    cp ./walkthrough/03b-agent.ts src/agent.ts

</details>

Test subtraction

測試 subtraction。

    npx tsx src/index.ts 'can you subtract 3 from 4'

now, let's test the multiplication tool

現在，讓我們測試 multiplication 工具。


    npx tsx src/index.ts 'can you multiply 3 and 4'

finally, let's test a more complex calculation with multiple operations

最後，讓我們測試一個包含多個運算的更複雜計算。


    npx tsx src/index.ts 'can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result'

## Chapter 4 - Add Tests to agent.baml
## 第 4 章 - 為 agent.baml 新增測試

Let's add some tests to our BAML agent.

讓我們為 BAML agent 新增一些測試。

to start, leave the baml logs enabled

一開始，先保持 baml logs 為啟用狀態。

    export BAML_LOG=debug

next, let's add some tests to the agent

接著，讓我們為 agent 新增一些測試。

We'll start with a simple test that checks the agent's ability to handle
a basic calculation.

我們先從一個簡單的測試開始，檢查 agent 是否能夠處理
基本計算。


```diff
baml_src/agent.baml
     "#
   }
+test MathOperation {
+  functions [DetermineNextStep]
+  args {
+    thread #"
+      {
+        "type": "user_input",
+        "data": "can you multiply 3 and 4?"
+      }
+    "#
+  }
+}
+
```

<details>
<summary>skip this step / 跳過此步驟</summary>

    cp ./walkthrough/04-agent.baml baml_src/agent.baml

</details>

Run the tests

執行測試。

    npx baml-cli test

now, let's improve the test with assertions!

現在，讓我們用 assertions 來改進這個測試！

Assertions are a great way to make sure the agent is working as expected,
and can easily be extended to check for more complex behavior.

Assertions 是一種很好的方式，可以確保 agent 的運作符合預期，
而且也能輕鬆延伸來檢查更複雜的行為。


```diff
baml_src/agent.baml
     "#
   }
+  @@assert(hello, {{this.intent == "done_for_now"}})
 }
 
     "#
   }
+  @@assert(math_operation, {{this.intent == "multiply"}})
 }
 
```

<details>
<summary>skip this step / 跳過此步驟</summary>

    cp ./walkthrough/04b-agent.baml baml_src/agent.baml

</details>

Run the tests

執行測試。

    npx baml-cli test

as you add more tests, you can disable the logs to keep the output clean. 
You may want to turn them on as you iterate on specific tests.

隨著你加入更多測試，可以把 logs 關掉，讓輸出保持乾淨。
當你在迭代特定測試時，可能會想再把它們打開。


    export BAML_LOG=off

now, let's add some more complex test cases,
where we resume from in the middle of an in-progress
agentic context window

現在，讓我們加入一些更複雜的測試案例，
從進行中的 agentic context window 中途狀態繼續。


```diff
baml_src/agent.baml
     "#
   }
-  @@assert(hello, {{this.intent == "done_for_now"}})
+  @@assert(intent, {{this.intent == "done_for_now"}})
 }
 
     "#
   }
-  @@assert(math_operation, {{this.intent == "multiply"}})
+  @@assert(intent, {{this.intent == "multiply"}})
 }
 
+test LongMath {
+  functions [DetermineNextStep]
+  args {
+    thread #"
+      [
+        {
+          "type": "user_input",
+          "data": "can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result?"
+        },
+        {
+          "type": "tool_call",
+          "data": {
+            "intent": "multiply",
+            "a": 3,
+            "b": 4
+          }
+        },
+        {
+          "type": "tool_response",
+          "data": 12
+        },
+        {
+          "type": "tool_call", 
+          "data": {
+            "intent": "divide",
+            "a": 12,
+            "b": 2
+          }
+        },
+        {
+          "type": "tool_response",
+          "data": 6
+        },
+        {
+          "type": "tool_call",
+          "data": {
+            "intent": "add", 
+            "a": 6,
+            "b": 12
+          }
+        },
+        {
+          "type": "tool_response",
+          "data": 18
+        }
+      ]
+    "#
+  }
+  @@assert(intent, {{this.intent == "done_for_now"}})
+  @@assert(answer, {{"18" in this.message}})
+}
+
```

<details>
<summary>skip this step / 跳過此步驟</summary>

    cp ./walkthrough/04c-agent.baml baml_src/agent.baml

</details>

let's try to run it

讓我們實際執行看看。


    npx baml-cli test

## Chapter 5 - Multiple Human Tools
## 第 5 章 - 多個 Human Tools

In this section, we'll add support for multiple tools that serve to 
contact humans.

在這一節中，我們會加入對多個用於
聯絡人類的工具的支援。


for this section, we'll disable the baml logs. You can optionally enable them if you want to see more details.

在這一節中，我們會停用 baml logs。若你想查看更多細節，也可以自行啟用。

    export BAML_LOG=off

first, let's add a tool that can request clarification from a human 

首先，讓我們加入一個可向人類請求澄清的工具。

this will be different from the "done_for_now" tool,
and can be used to more flexibly handle different types of human interactions
in your agent.

這會與 "done_for_now" 工具不同，
並且可讓你的 agent 更有彈性地處理不同類型的人類互動。


```diff
baml_src/agent.baml
+// human tools are async requests to a human
+// human tools 是對人類發出的非同步請求
+type HumanTools = ClarificationRequest | DoneForNow
+
+class ClarificationRequest {
+  intent "request_more_information" @description("you can request more information from me")
+  message string
+}
+
 class DoneForNow {
   intent "done_for_now"
-  message string 
+
+  message string @description(#"
+    message to send to the user about the work that was done. 
+  "#)
 }
 
 function DetermineNextStep(
     thread: string 
-) -> CalculatorTools | DoneForNow {
+) -> HumanTools | CalculatorTools {
     client "openai/gpt-4o"
 
 }
 
+
```

<details>
<summary>skip this step / 跳過此步驟</summary>

    cp ./walkthrough/05-agent.baml baml_src/agent.baml

</details>

next, let's re-generate the client code

接著，讓我們重新產生 client code。

NOTE - if you're using the VSCode extension for BAML,
the client will be regenerated automatically when you save the file
in your editor.

注意：如果你使用的是 BAML 的 VSCode extension，
當你在編輯器中儲存檔案時，client 會自動重新產生。


    npx baml-cli generate

now, let's update the agent to use the new tool

現在，讓我們更新 agent 以使用這個新工具。


```diff
src/agent.ts
 }
 
-export async function agentLoop(thread: Thread): Promise<string> {
+export async function agentLoop(thread: Thread): Promise<Thread> {
 
     while (true) {
         switch (nextStep.intent) {
             case "done_for_now":
-                // response to human, return the next step object
-                return nextStep.message;
+            case "request_more_information":
+                // response to human, return the thread
+                // 回應人類，回傳 thread
+                return thread;
             case "add":
             case "subtract":
```

<details>
<summary>skip this step / 跳過此步驟</summary>

    cp ./walkthrough/05-agent.ts src/agent.ts

</details>

next, let's update the CLI to handle clarification requests
by requesting input from the user on the CLI

接下來，讓我們更新 CLI，
使它能透過在 CLI 上向使用者請求輸入來處理澄清請求。


```diff
src/cli.ts
 // cli.ts lets you invoke the agent loop from the command line
 // cli.ts 讓你可以從命令列呼叫 agent loop
 
-import { agentLoop, Thread, Event } from "./agent";
+import { agentLoop, Thread, Event } from "../src/agent";
 
+
+
 export async function cli() {
     // Get command line arguments, skipping the first two (node and script name)
     // 取得命令列參數，略過前兩個（node 與 script 名稱）
     // Run the agent loop with the thread
     // 使用 thread 執行 agent loop
     const result = await agentLoop(thread);
-    console.log(result);
+    let lastEvent = result.events.slice(-1)[0];
+
+    while (lastEvent.data.intent === "request_more_information") {
+        const message = await askHuman(lastEvent.data.message);
+        thread.events.push({ type: "human_response", data: message });
+        const result = await agentLoop(thread);
+        lastEvent = result.events.slice(-1)[0];
+    }
+
+    // print the final result
+    // 輸出最終結果
+    // optional - you could loop here too
+    // 可選：你也可以在這裡繼續迴圈
+    console.log(lastEvent.data.message);
+    process.exit(0);
 }
+
+async function askHuman(message: string) {
+    const readline = require('readline').createInterface({
+        input: process.stdin,
+        output: process.stdout
+    });
+
+    return new Promise((resolve) => {
+        readline.question(`${message}\n> `, (answer: string) => {
+            resolve(answer);
+        });
+    });
+}
```

<details>
<summary>skip this step / 跳過此步驟</summary>

    cp ./walkthrough/05-cli.ts src/cli.ts

</details>

let's try it out

讓我們來試試看。


    npx tsx src/index.ts 'can you multiply 3 and FD*(#F&& '

next, let's add a test that checks the agent's ability to handle
a clarification request

接下來，讓我們加入一個測試，用來檢查 agent
處理澄清請求的能力。


```diff
baml_src/agent.baml
 
 
+
+test MathOperationWithClarification {
+  functions [DetermineNextStep]
+  args {
+    thread #"
+          [{"type":"user_input","data":"can you multiply 3 and feee9ff10"}]
+      "#
+  }
+  @@assert(intent, {{this.intent == "request_more_information"}})
+}
+
+test MathOperationPostClarification {
+  functions [DetermineNextStep]
+  args {
+    thread #"
+        [
+        {"type":"user_input","data":"can you multiply 3 and FD*(#F&& ?"},
+        {"type":"tool_call","data":{"intent":"request_more_information","message":"It seems like there was a typo or mistake in your request. Could you please clarify or provide the correct numbers you would like to multiply?"}},
+        {"type":"human_response","data":"lets try 12 instead"},
+      ]
+      "#
+  }
+  @@assert(intent, {{this.intent == "multiply"}})
+  @@assert(a, {{this.b == 12}})
+  @@assert(b, {{this.a == 3}})
+}
+        
+
+
```

<details>
<summary>skip this step / 跳過此步驟</summary>

    cp ./walkthrough/05b-agent.baml baml_src/agent.baml

</details>

and now we can run the tests again

現在我們可以再次執行測試。


    npx baml-cli test

you'll notice the new test passes, but the hello world test fails

你會注意到新的測試通過了，但 hello world 測試失敗了。

This is because the agent's default behavior is to return "done_for_now"

這是因為 agent 的預設行為是回傳 "done_for_now"。


```diff
baml_src/agent.baml
     "#
   }
-  @@assert(intent, {{this.intent == "done_for_now"}})
+  @@assert(intent, {{this.intent == "request_more_information"}})
 }
 
```

<details>
<summary>skip this step / 跳過此步驟</summary>

    cp ./walkthrough/05c-agent.baml baml_src/agent.baml

</details>

Verify tests pass

確認測試已通過。

    npx baml-cli test

## Chapter 6 - Customize Your Prompt with Reasoning
## 第 6 章 - 以推理步驟自訂 Prompt

In this section, we'll explore how to customize the prompt of the agent
with reasoning steps.

在這一節中，我們會探討如何透過推理步驟
自訂 agent 的 prompt。

this is core to [factor 2 - own your prompts](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-2-own-your-prompts.md)

這是 [factor 2 - own your prompts](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-2-own-your-prompts.md) 的核心內容。

there's a deep dive on reasoning on AI That Works [reasoning models versus reasoning steps](https://github.com/hellovai/ai-that-works/tree/main/2025-04-07-reasoning-models-vs-prompts)

在 AI That Works 上有一篇深入探討推理的文章：[reasoning models versus reasoning steps](https://github.com/hellovai/ai-that-works/tree/main/2025-04-07-reasoning-models-vs-prompts)。


for this section, it will be helpful to leave the baml logs enabled

在這一節中，保留啟用 baml logs 會比較有幫助。

    export BAML_LOG=debug

update the agent prompt to include a reasoning step

更新 agent prompt，加入一個推理步驟。


```diff
baml_src/agent.baml
 
         {{ ctx.output_format }}
+
+        First, always plan out what to do next, for example:
+
+        - ...
+        - ...
+        - ...
+
+        {...} // schema
+        // 結構描述
     "#
 }
   @@assert(b, {{this.a == 3}})
 }
-        
-
```

<details>
<summary>skip this step / 跳過此步驟</summary>

    cp ./walkthrough/06-agent.baml baml_src/agent.baml

</details>

generate the updated client

產生更新後的 client。

    npx baml-cli generate

now, you can try it out with a simple prompt

現在，你可以用一個簡單的 prompt 來試試看。


    npx tsx src/index.ts 'can you multiply 3 and 4'

you should see output from the baml logs showing the reasoning steps

你應該會在 baml logs 中看到顯示推理步驟的輸出。

#### optional challenge 
#### 選做挑戰

add a field to your tool output format that includes the reasoning steps in the output!

在你的 tool output format 中加入一個欄位，讓輸出也包含推理步驟！


## Chapter 7 - Customize Your Context Window
## 第 7 章 - 自訂你的 Context Window

In this section, we'll explore how to customize the context window
of the agent.

在這一節中，我們會探討如何自訂 agent 的 context window。

this is core to [factor 3 - own your context window](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-3-own-your-context-window.md)

這是 [factor 3 - own your context window](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-3-own-your-context-window.md) 的核心內容。


update the agent to pretty-print the Context window for the model

更新 agent，讓它為 model 以易讀格式輸出 Context window。


```diff
src/agent.ts
         // can change this to whatever custom serialization you want to do, XML, etc
         // 可以改成你想要的任何自訂序列化方式，例如 XML 等
         // e.g. https://github.com/got-agents/agents/blob/59ebbfa236fc376618f16ee08eb0f3bf7b698892/linear-assistant-ts/src/agent.ts#L66-L105
         // 例如：https://github.com/got-agents/agents/blob/59ebbfa236fc376618f16ee08eb0f3bf7b698892/linear-assistant-ts/src/agent.ts#L66-L105
-        return JSON.stringify(this.events);
+        return JSON.stringify(this.events, null, 2);
     }
 }
```

<details>
<summary>skip this step / 跳過此步驟</summary>

    cp ./walkthrough/07-agent.ts src/agent.ts

</details>

Test the formatting

測試格式化結果。

    BAML_LOG=info npx tsx src/index.ts 'can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result'

next, let's update the agent to use XML formatting instead 

接下來，讓我們改為讓 agent 使用 XML 格式。

this is a very popular format for passing data to a model,

這是將資料傳遞給 model 時非常常見的一種格式，

among other things, because of the token efficiency of XML.

其中一個原因是 XML 在 token 使用上的效率。


```diff
src/agent.ts
 
     serializeForLLM() {
-        // can change this to whatever custom serialization you want to do, XML, etc
-        // 可以改成你想要的任何自訂序列化方式，例如 XML 等
-        // e.g. https://github.com/got-agents/agents/blob/59ebbfa236fc376618f16ee08eb0f3bf7b698892/linear-assistant-ts/src/agent.ts#L66-L105
-        // 例如：https://github.com/got-agents/agents/blob/59ebbfa236fc376618f16ee08eb0f3bf7b698892/linear-assistant-ts/src/agent.ts#L66-L105
-        return JSON.stringify(this.events, null, 2);
+        return this.events.map(e => this.serializeOneEvent(e)).join("\n");
     }
+
+    trimLeadingWhitespace(s: string) {
+        return s.replace(/^[ \t]+/gm, '');
+    }
+
+    serializeOneEvent(e: Event) {
+        return this.trimLeadingWhitespace(`
+            <${e.data?.intent || e.type}>
+            ${
+            typeof e.data !== 'object' ? e.data :
+            Object.keys(e.data).filter(k => k !== 'intent').map(k => `${k}: ${e.data[k]}`).join("\n")}
+            </${e.data?.intent || e.type}>
+        `)
+    }
 }
 
```

<details>
<summary>skip this step / 跳過此步驟</summary>

    cp ./walkthrough/07b-agent.ts src/agent.ts

</details>

let's try it out

讓我們來試試看。


    BAML_LOG=info npx tsx src/index.ts 'can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result'

lets update our tests to match the new output format

讓我們更新測試，使其符合新的輸出格式。


```diff
baml_src/agent.baml
         {{ ctx.output_format }}
 
-        First, always plan out what to do next, for example:
+        Always think about what to do next first, like:
 
         - ...
   args {
     thread #"
-      {
-        "type": "user_input",
-        "data": "hello!"
-      }
+      <user_input>
+        hello!
+      </user_input>
     "#
   }
   args {
     thread #"
-      {
-        "type": "user_input",
-        "data": "can you multiply 3 and 4?"
-      }
+      <user_input>
+        can you multiply 3 and 4?
+      </user_input>
     "#
   }
   args {
     thread #"
-      [
-        {
-          "type": "user_input",
-          "data": "can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result?"
-        },
-        {
-          "type": "tool_call",
-          "data": {
-            "intent": "multiply",
-            "a": 3,
-            "b": 4
-          }
-        },
-        {
-          "type": "tool_response",
-          "data": 12
-        },
-        {
-          "type": "tool_call", 
-          "data": {
-            "intent": "divide",
-            "a": 12,
-            "b": 2
-          }
-        },
-        {
-          "type": "tool_response",
-          "data": 6
-        },
-        {
-          "type": "tool_call",
-          "data": {
-            "intent": "add", 
-            "a": 6,
-            "b": 12
-          }
-        },
-        {
-          "type": "tool_response",
-          "data": 18
-        }
-      ]
+         <user_input>
+    can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result?
+    </user_input>
+
+
+    <multiply>
+    a: 3
+    b: 4
+    </multiply>
+
+
+    <tool_response>
+    12
+    </tool_response>
+
+
+    <divide>
+    a: 12
+    b: 2
+    </divide>
+
+
+    <tool_response>
+    6
+    </tool_response>
+
+
+    <add>
+    a: 6
+    b: 12
+    </add>
+
+
+    <tool_response>
+    18
+    </tool_response>
+
     "#
   }
   args {
     thread #"
-          [{"type":"user_input","data":"can you multiply 3 and feee9ff10"}]
+          <user_input>
+          can you multiply 3 and fe1iiaff10
+          </user_input>
       "#
   }
   args {
     thread #"
-        [
-        {"type":"user_input","data":"can you multiply 3 and FD*(#F&& ?"},
-        {"type":"tool_call","data":{"intent":"request_more_information","message":"It seems like there was a typo or mistake in your request. Could you please clarify or provide the correct numbers you would like to multiply?"}},
-        {"type":"human_response","data":"lets try 12 instead"},
-      ]
+        <user_input>
+        can you multiply 3 and FD*(#F&& ?
+        </user_input>
+
+        <request_more_information>
+        message: It seems like there was a typo or mistake in your request. Could you please clarify or provide the correct numbers you would like to multiply?
+        </request_more_information>
+
+        <human_response>
+        lets try 12 instead
+        </human_response>
       "#
   }
   @@assert(intent, {{this.intent == "multiply"}})
 }
         
```

<details>
<summary>skip this step / 跳過此步驟</summary>

    cp ./walkthrough/07c-agent.baml baml_src/agent.baml

</details>

check out the updated tests

查看更新後的測試。


    npx baml-cli test

## Chapter 8 - Adding API Endpoints
## 第 8 章 - 新增 API 端點

Add an Express server to expose the agent via HTTP.

加入一個 Express server，透過 HTTP 將 agent 對外提供。

for this section, we'll disable the baml logs. You can optionally enable them if you want to see more details.

在這一節中，我們會停用 baml logs。若你想看到更多細節，也可以自行啟用。

    export BAML_LOG=off

Install Express and types

安裝 Express 與型別定義

    npm install express && npm install --save-dev @types/express supertest

Add the server implementation

加入 server 實作

    cp ./walkthrough/08-server.ts src/server.ts

<details>
<summary>show file / 顯示檔案</summary>

```ts
// ./walkthrough/08-server.ts
import express from 'express';
import { Thread, agentLoop } from '../src/agent';

const app = express();
app.use(express.json());
app.set('json spaces', 2);

// POST /thread - Start new thread
// POST /thread - 啟動新的 thread
app.post('/thread', async (req, res) => {
    const thread = new Thread([{
        type: "user_input",
        data: req.body.message
    }]);
    const result = await agentLoop(thread);
    res.json(result);
});

// GET /thread/:id - Get thread status 
// GET /thread/:id - 取得 thread 狀態
app.get('/thread/:id', (req, res) => {
    // optional - add state
    // 可選：加入 state
    res.status(404).json({ error: "Not implemented yet" });
});

const port = process.env.PORT || 3000;
app.listen(port, () => {
    console.log(`Server running on port ${port}`);
});

export { app };
```

</details>

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

你應該會從 agent 取得回應，其中包含
agentic trace，最後會以類似下面的訊息結束：

    {"intent":"done_for_now","message":"The sum of 3 and 4 is 7."}

## Chapter 9 - In-Memory State and Async Clarification
## 第 9 章 - 記憶體內 state 與非同步 clarification

Add state management and async clarification support.

加入 state 管理與非同步 clarification 支援。

for this section, we'll disable the baml logs. You can optionally enable them if you want to see more details.

在這一節中，我們會停用 baml logs。若你想看到更多細節，也可以自行啟用。

    export BAML_LOG=off

Add some simple in-memory state management for threads

為 threads 加入一些簡單的記憶體內 state 管理

    cp ./walkthrough/09-state.ts src/state.ts

<details>
<summary>show file / 顯示檔案</summary>

```ts
// ./walkthrough/09-state.ts
import crypto from 'crypto';
import { Thread } from '../src/agent';


// you can replace this with any simple state management,
// 你可以把這裡替換成任何簡單的 state 管理方式，
// e.g. redis, sqlite, postgres, etc
// 例如 redis、sqlite、postgres 等
export class ThreadStore {
    private threads: Map<string, Thread> = new Map();
    
    create(thread: Thread): string {
        const id = crypto.randomUUID();
        this.threads.set(id, thread);
        return id;
    }
    
    get(id: string): Thread | undefined {
        return this.threads.get(id);
    }
    
    update(id: string, thread: Thread): void {
        this.threads.set(id, thread);
    }
}
```

</details>

update the server to use the state management

更新 server，讓它使用 state 管理

* Add thread state management using `ThreadStore`
* return thread IDs and response URLs from the /thread endpoint
* implement GET /thread/:id 
* implement POST /thread/:id/response

* 使用 `ThreadStore` 新增 thread state 管理
* 從 `/thread` endpoint 回傳 thread ID 與 response URL
* 實作 `GET /thread/:id`
* 實作 `POST /thread/:id/response`

```diff
src/server.ts
 import express from 'express';
 import { Thread, agentLoop } from '../src/agent';
+import { ThreadStore } from '../src/state';
 
 const app = express();
 app.set('json spaces', 2);
 
+const store = new ThreadStore();
+
 // POST /thread - Start new thread
 // POST /thread - 啟動新的 thread
 app.post('/thread', async (req, res) => {
         data: req.body.message
     }]);
-    const result = await agentLoop(thread);
-    res.json(result);
+    
+    const threadId = store.create(thread);
+    const newThread = await agentLoop(thread);
+    
+    store.update(threadId, newThread);
+
+    const lastEvent = newThread.events[newThread.events.length - 1];
+    // If we exited the loop, include the response URL so the client can
+    // 如果我們已離開迴圈，就附上 response URL，讓 client 可以
+    // push a new message onto the thread
+    // 將新的訊息推送到 thread 上
+    lastEvent.data.response_url = `/thread/${threadId}/response`;
+
+    console.log("returning last event from endpoint", lastEvent);
+
+    res.json({ 
+        thread_id: threadId,
+        ...newThread 
+    });
 });
 
 app.get('/thread/:id', (req, res) => {
-    // optional - add state
-    // 可選：加入 state
-    res.status(404).json({ error: "Not implemented yet" });
+    const thread = store.get(req.params.id);
+    if (!thread) {
+        return res.status(404).json({ error: "Thread not found" });
+    }
+    res.json(thread);
 });
 
+// POST /thread/:id/response - Handle clarification response
+// POST /thread/:id/response - 處理 clarification 回應
+app.post('/thread/:id/response', async (req, res) => {
+    let thread = store.get(req.params.id);
+    if (!thread) {
+        return res.status(404).json({ error: "Thread not found" });
+    }
+    
+    thread.events.push({
+        type: "human_response",
+        data: req.body.message
+    });
+    
+    // loop until stop event
+    // 持續迴圈直到 stop event
+    const newThread = await agentLoop(thread);
+    
+    store.update(req.params.id, newThread);
+
+    const lastEvent = newThread.events[newThread.events.length - 1];
+    lastEvent.data.response_url = `/thread/${req.params.id}/response`;
+
+    console.log("returning last event from endpoint", lastEvent);
+    
+    res.json(newThread);
+});
+
 const port = process.env.PORT || 3000;
 app.listen(port, () => {
```

<details>
<summary>skip this step / 跳過此步驟</summary>

    cp ./walkthrough/09-server.ts src/server.ts

</details>

Start the server

啟動 server

    npx tsx src/server.ts

Test clarification flow

測試 clarification 流程

    curl -X POST http://localhost:3000/thread \
  -H "Content-Type: application/json" \
  -d '{"message":"can you multiply 3 and xyz"}'

## Chapter 10 - Adding Human Approval
## 第 10 章 - 新增人工核准

Add support for human approval of operations.

加入對操作進行人工核准的支援。

for this section, we'll disable the baml logs. You can optionally enable them if you want to see more details.

在這一節中，我們會停用 baml logs。若你想看到更多細節，也可以自行啟用。

    export BAML_LOG=off

update the server to handle human approvals

更新 server，讓它能處理人工核准

* Import `handleNextStep` to execute approved actions
* Add two payload types to distinguish approvals from responses
* Handle responses and approvals differently in the endpoint
* Show better error messages when things go wrongs

* 匯入 `handleNextStep` 來執行已核准的動作
* 新增兩種 payload 型別，用來區分 approvals 與 responses
* 在 endpoint 中分別處理 responses 與 approvals
* 出錯時顯示更清楚的錯誤訊息

```diff
src/server.ts
 import express from 'express';
-import { Thread, agentLoop } from '../src/agent';
+import { Thread, agentLoop, handleNextStep } from '../src/agent';
 import { ThreadStore } from '../src/state';
 
 });
 
+
+type ApprovalPayload = {
+    type: "approval";
+    approved: boolean;
+    comment?: string;
+}
+
+type ResponsePayload = {
+    type: "response";
+    response: string;
+}
+
+type Payload = ApprovalPayload | ResponsePayload;
+
 // POST /thread/:id/response - Handle clarification response
 // POST /thread/:id/response - 處理 clarification 回應
 app.post('/thread/:id/response', async (req, res) => {
         return res.status(404).json({ error: "Thread not found" });
     }
+
+    const body: Payload = req.body;
+
+    let lastEvent = thread.events[thread.events.length - 1];
+
+    if (thread.awaitingHumanResponse() && body.type === 'response') {
+        thread.events.push({
+            type: "human_response",
+            data: body.response
+        });
+    } else if (thread.awaitingHumanApproval() && body.type === 'approval' && !body.approved) {
+        // push feedback onto the thread
+        // 將回饋加入 thread 中
+        thread.events.push({
+            type: "tool_response",
+            data: `user denied the operation with feedback: "${body.comment}"`
+        });
+    } else if (thread.awaitingHumanApproval() && body.type === 'approval' && body.approved) {
+        // approved, run the tool, pushing results onto the thread
+        // 已核准，執行 tool，並將結果加入 thread 中
+        await handleNextStep(lastEvent.data, thread);
+    } else {
+        res.status(400).json({
+            error: "Invalid request: " + body.type,
+            awaitingHumanResponse: thread.awaitingHumanResponse(),
+            awaitingHumanApproval: thread.awaitingHumanApproval()
+        });
+        return;
+    }
+
     
-    thread.events.push({
-        type: "human_response",
-        data: req.body.message
-    });
-    
     // loop until stop event
     // 持續迴圈直到 stop event
     const newThread = await agentLoop(thread);
     store.update(req.params.id, newThread);
 
-    const lastEvent = newThread.events[newThread.events.length - 1];
+    lastEvent = newThread.events[newThread.events.length - 1];
     lastEvent.data.response_url = `/thread/${req.params.id}/response`;
 
```

<details>
<summary>skip this step / 跳過此步驟</summary>

    cp ./walkthrough/10-server.ts src/server.ts

</details>

Add a few methods to the agent to handle approvals and responses

為 agent 新增幾個方法，用來處理 approvals 與 responses

```diff
src/agent.ts
         `)
     }
+
+    awaitingHumanResponse(): boolean {
+        const lastEvent = this.events[this.events.length - 1];
+        return ['request_more_information', 'done_for_now'].includes(lastEvent.data.intent);
+    }
+
+    awaitingHumanApproval(): boolean {
+        const lastEvent = this.events[this.events.length - 1];
+        return lastEvent.data.intent === 'divide';
+    }
 }
 
                 // response to human, return the thread
                 // 回傳給人類後，直接回傳 thread
                 return thread;
+            case "divide":
+                // divide is scary, return it for human approval
+                // divide 比較敏感，因此回傳並等待人工核准
+                return thread;
             case "add":
             case "subtract":
             case "multiply":
-            case "divide":
                 thread = await handleNextStep(nextStep, thread);
         }
```

<details>
<summary>skip this step / 跳過此步驟</summary>

    cp ./walkthrough/10-agent.ts src/agent.ts

</details>

Start the server

啟動 server

    npx tsx src/server.ts

Test division with approval

測試需要核准的除法流程

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

用另一個 curl 呼叫拒絕這個請求，並把 thread ID 換成你的實際值

    curl -X POST 'http://localhost:3000/thread/{thread_id}/response' \
  -H "Content-Type: application/json" \
  -d '{"type": "approval", "approved": false, "comment": "I dont think thats right, use 5 instead of 4"}'

You should see: the last tool call is now `"intent":"divide","a":3,"b":5`

你應該會看到：最後一個 tool call 現在會是 `"intent":"divide","a":3,"b":5`

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

現在你可以核准這個操作

    curl -X POST 'http://localhost:3000/thread/{thread_id}/response' \
  -H "Content-Type: application/json" \
  -d '{"type": "approval", "approved": true}'

you should see the final message includes the tool response and final result!

你應該會看到最終訊息包含 tool response 與最後結果！

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

## Chapter 11 - Human Approvals over email
## 第 11 章 - 透過 email 進行人工核准

in this section, we'll add support for human approvals over email.

在這一節中，我們將加入透過 email 進行人工核准的支援。

This will start a little bit contrived, just to get the concepts down - 

一開始的範例會有一點刻意安排，目的是先把核心概念建立起來——

We'll start by invoking the workflow from the CLI but approvals for `divide`
and `request_more_information` will be handled over email,
then the final `done_for_now` answer will be printed back to the CLI

我們會先從 CLI 啟動 workflow，但 `divide`
和 `request_more_information` 的核准會透過 email 處理，
最後再把 `done_for_now` 的結果輸出回 CLI。

While contrived, this is a great example of the flexibility you get from
[factor 7 - contact humans with tools](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-7-contact-humans-with-tools.md)

雖然這個例子有點刻意，但它非常適合用來展示你能從
[factor 7 - contact humans with tools](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-7-contact-humans-with-tools.md)
獲得的彈性。

for this section, we'll disable the baml logs. You can optionally enable them if you want to see more details.

在這一節中，我們會停用 baml logs。若你想查看更多細節，也可以自行重新啟用。

    export BAML_LOG=off

Install HumanLayer

安裝 HumanLayer

    npm install humanlayer

Update CLI to send `divide` and `request_more_information` to a human via email

更新 CLI，讓 `divide` 和 `request_more_information` 透過 email 傳送給真人處理

```diff
src/cli.ts
 // cli.ts lets you invoke the agent loop from the command line
 // cli.ts 讓你可以從命令列啟動 agent 迴圈
 
 +import { humanlayer } from "humanlayer";
  import { agentLoop, Thread, Event } from "../src/agent";
 
 -
 -
  export async function cli() {
      // Get command line arguments, skipping the first two (node and script name)
      // 取得命令列參數，略過前兩個（node 與 script 名稱）
 
      // Run the agent loop with the thread
      // 使用 thread 執行 agent 迴圈
 -    const result = await agentLoop(thread);
 -    let lastEvent = result.events.slice(-1)[0];
 +    let newThread = await agentLoop(thread);
 +    let lastEvent = newThread.events.slice(-1)[0];
 
 -    while (lastEvent.data.intent === "request_more_information") {
 -        const message = await askHuman(lastEvent.data.message);
 -        thread.events.push({ type: "human_response", data: message });
 -        const result = await agentLoop(thread);
 -        lastEvent = result.events.slice(-1)[0];
 +    while (lastEvent.data.intent !== "done_for_now") {
 +        const responseEvent = await askHuman(lastEvent);
 +        thread.events.push(responseEvent);
 +        newThread = await agentLoop(thread);
 +        lastEvent = newThread.events.slice(-1)[0];
      }
 
      // print the final result
      // 印出最終結果
      console.log(lastEvent.data.message);
      process.exit(0);
  }
 
 -async function askHuman(message: string) {
 +async function askHuman(lastEvent: Event): Promise<Event> {
 +    if (process.env.HUMANLAYER_API_KEY) {
 +        return await askHumanEmail(lastEvent);
 +    } else {
 +        return await askHumanCLI(lastEvent.data.message);
 +    }
 +}
 +
 +async function askHumanCLI(message: string): Promise<Event> {
      const readline = require('readline').createInterface({
          input: process.stdin,
      return new Promise((resolve) => {
          readline.question(`${message}\n> `, (answer: string) => {
 -            resolve(answer);
 +            resolve({ type: "human_response", data: answer });
          });
      });
  }
 +
 +export async function askHumanEmail(lastEvent: Event): Promise<Event> {
 +    if (!process.env.HUMANLAYER_EMAIL) {
 +        throw new Error("missing or invalid parameters: HUMANLAYER_EMAIL");
 +    }
 +    const hl = humanlayer({ //reads apiKey from env
 +    // 從環境變數讀取 apiKey
 +        // name of this agent
 +        // 這個 agent 的名稱
 +        runId: "12fa-cli-agent",
 +        verbose: true,
 +        contactChannel: {
 +            // agent should request permission via email
 +            // agent 應透過 email 請求許可
 +            email: {
 +                address: process.env.HUMANLAYER_EMAIL,
 +            }
 +        }
 +    }) 
 +
 +    if (lastEvent.data.intent === "divide") {
 +        // fetch approval synchronously - this will block until reply
 +        // 同步取得核准——在收到回覆之前會持續阻塞
 +        const response = await hl.fetchHumanApproval({
 +            spec: {
 +                fn: "divide",
 +                kwargs: {
 +                    a: lastEvent.data.a,
 +                    b: lastEvent.data.b
 +                }
 +            }
 +        })
 +
 +        if (response.approved) {
 +            const result = lastEvent.data.a / lastEvent.data.b;
 +            console.log("tool_response", result);
 +            return {
 +                "type": "tool_response",
 +                "data": result
 +            };
 +        } else {
 +            return {
 +                "type": "tool_response",
 +                "data": `user denied operation ${lastEvent.data.intent}
 +                with feedback: ${response.comment}`
 +            };
 +        }
 +    }
 +    throw new Error(`unknown tool: ${lastEvent.data.intent}`)
 +}
```

<details>
<summary>skip this step / 跳過此步驟</summary>

    cp ./walkthrough/11-cli.ts src/cli.ts

</details>

Run the CLI

執行 CLI

    npx tsx src/index.ts 'can you divide 4 by 5'

The last line of your program should mention human review step

你程式的最後一行應該會提到 human review 步驟。

    nextStep { intent: 'divide', a: 4, b: 5 }
HumanLayer: Requested human approval from HumanLayer cloud

go ahead and respond to the email with some feedback:

接著請直接回覆那封 email，提供一些 feedback：

![reject-email](https://github.com/RobbinHsu/12-factor-agents/blob/main/workshops/2025-05/walkthrough/11-email-reject.png?raw=true)

you should get another email with an updated attempt based on your feedback!

你應該會再收到另一封 email，內容是根據你的 feedback 更新後的嘗試！

You can go ahead and approve this one:

你現在可以核准這一個：

![approve-email](https://github.com/RobbinHsu/12-factor-agents/blob/main/workshops/2025-05/walkthrough/11-email-approve.png?raw=true)

and your final output will look like

最後你的輸出會長這樣：

    nextStep {
 intent: 'done_for_now',
 message: 'The division of 4 by 5 is 0.8. If you have any other calculations or questions, feel free to ask!'
}
The division of 4 by 5 is 0.8. If you have any other calculations or questions, feel free to ask!

lets implement the `request_more_information` flow as well

我們也來實作 `request_more_information` 這條流程。

```diff
src/cli.ts
     }) 
 
 +    if (lastEvent.data.intent === "request_more_information") {
 +        // fetch response synchronously - this will block until reply
 +        // 同步取得回應——在收到回覆之前會持續阻塞
 +        const response = await hl.fetchHumanResponse({
 +            spec: {
 +                msg: lastEvent.data.message
 +            }
 +        })
 +        return {
 +            "type": "tool_response",
 +            "data": response
 +        }
 +    }
 +    
     if (lastEvent.data.intent === "divide") {
         // fetch approval synchronously - this will block until reply
         // 同步取得核准——在收到回覆之前會持續阻塞
```

<details>
<summary>skip this step / 跳過此步驟</summary>

    cp ./walkthrough/11b-cli.ts src/cli.ts

</details>

lets test the require_approval flow as by asking for a calculation
with garbled input:

讓我們透過輸入混亂的計算內容來測試 require_approval 流程：

    npx tsx src/index.ts 'can you multiply 4 and xyz'

You should get an email with a request for clarification

你應該會收到一封要求釐清內容的 email。

    Can you clarify what 'xyz' represents in this context? Is it a specific number, variable, or something else?

you can response with something like

你可以回覆類似下面的內容：

    use 8 instead of xyz

you should see a final result on the CLI like

你應該會在 CLI 上看到像這樣的最終結果：

    I have multiplied 4 and xyz, using the value 8 for xyz, resulting in 32.

as a final step, lets explore using a custom html template for the email

最後一步，讓我們試著為 email 使用自訂的 html template。

```diff
src/cli.ts
             email: {
                 address: process.env.HUMANLAYER_EMAIL,
 +                // custom email body - jinja
 +                // 自訂 email 內容——jinja
 +                template: `{% if type == 'request_more_information' %}
 +{{ event.spec.msg }}
 +{% else %}
 +agent {{ event.run_id }} is requesting approval for {{event.spec.fn}}
 +with args: {{event.spec.kwargs}}
 +<br><br>
 +reply to this email to approve
 +{% endif %}`
             }
         }
```

<details>
<summary>skip this step / 跳過此步驟</summary>

    cp ./walkthrough/11c-cli.ts src/cli.ts

</details>

first try with divide:

先用 divide 試試看：

    npx tsx src/index.ts 'can you divide 4 by 5'

you should see a slightly different email with the custom template

你應該會看到一封套用了自訂 template、內容稍有不同的 email。

![custom-template-email](https://github.com/RobbinHsu/12-factor-agents/blob/main/workshops/2025-05/walkthrough/11-email-custom.png?raw=true)

feel free to run with the flow and then you can try updating the template to your liking

你可以先完整跑一次這個流程，接著再試著把 template 改成你喜歡的樣子。

(if you're using cursor, something as simple as highlighting the template and asking to "make it better"
should do the trick)

（如果你使用的是 cursor，只要反白 template 並要求它「make it better」，
通常就很夠用了。）

try triggering "request_more_information" as well!

也試著觸發看看 `request_more_information` 吧！

thats it - in the next chapter, we'll build a fully email-driven 
workflow agent that uses webhooks for human approval 

就是這樣——下一章我們會建立一個完全由 email 驅動、
並使用 webhook 進行人工核准的 workflow agent。

## Chapter XX - HumanLayer Webhook Integration
## 第 XX 章 - HumanLayer Webhook 整合

the previous sections used the humanlayer SDK in "synchronous mode" - that 
means every time we wait for human approval, we sit in a loop 
polling until the human response if received.

前面的章節是以「synchronous mode」使用 humanlayer SDK——這表示每次等待人工核准時，
我們都會卡在一個迴圈裡，不斷 polling，直到收到人類的回應為止。

That's obviously not ideal, especially for production workloads,
so in this section we'll implement [factor 6 - launch / pause / resume with simple APIs](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-6-launch-pause-resume.md)
by updating the server to end processing after contacting a human, and use webhooks to receive the results. 

這顯然不是理想做法，尤其對 production workloads 來說更是如此，
所以在這一節中，我們會透過更新 server，讓它在聯絡真人之後先結束處理，
並使用 webhooks 接收結果，藉此實作 [factor 6 - launch / pause / resume with simple APIs](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-6-launch-pause-resume.md)。

add code to initialize humanlayer in the server

加入程式碼，在 server 中初始化 humanlayer

```diff
src/server.ts
 import { Thread, agentLoop, handleNextStep } from '../src/agent';
 import { ThreadStore } from '../src/state';
 +import { humanlayer } from 'humanlayer';
 
 const app = express();
 const store = new ThreadStore();
 
 +const getHumanlayer = () => {
 +    const HUMANLAYER_EMAIL = process.env.HUMANLAYER_EMAIL;
 +    if (!HUMANLAYER_EMAIL) {
 +        throw new Error("missing or invalid parameters: HUMANLAYER_EMAIL");
 +    }
 +
 +    const HUMANLAYER_API_KEY = process.env.HUMANLAYER_API_KEY;
 +    if (!HUMANLAYER_API_KEY) {
 +        throw new Error("missing or invalid parameters: HUMANLAYER_API_KEY");
 +    }
 +    return humanlayer({
 +        runId: `12fa-agent`,
 +        contactChannel: {
 +            email: { address: HUMANLAYER_EMAIL }
 +        }
 +    });
 +}
 +
  // POST /thread - Start new thread
  // POST /thread - 啟動新的 thread
  app.post('/thread', async (req, res) => {
      
      // loop until stop event
      // 持續迴圈直到 stop event
 -    const newThread = await agentLoop(thread);
 +    const result = await agentLoop(thread);
 
 -    store.update(req.params.id, newThread);
 +    store.update(req.params.id, result);
 
 -    lastEvent = newThread.events[newThread.events.length - 1];
 +    lastEvent = result.events[result.events.length - 1];
      lastEvent.data.response_url = `/thread/${req.params.id}/response`;
 
      console.log("returning last event from endpoint", lastEvent);
      
 -    res.json(newThread);
 +    res.json(result);
  });
 
```

<details>
<summary>skip this step / 跳過此步驟</summary>

    cp ./walkthrough/12-1-server-init.ts src/server.ts

</details>

next, lets update the /thread endpoint to  

接著，讓我們把 `/thread` endpoint 更新為：

1. handle requests asynchronously, returning immediately
2. create a human contact on request_more_information and done_for_now calls

1. 以非同步方式處理請求，並立即回傳
2. 在 request_more_information 和 done_for_now 呼叫時建立 human contact

Update the server to be able to handle request_clarification responses

更新 server，使其能夠處理 request_clarification 回應

- remove the old /response endpoint and types
- update the /thread endpoint to run processing asynchronously, return immediately
- send a state.threadId when requesting human responses
- add a handleHumanResponse function to process the human response
- add a /webhook endpoint to handle the webhook response

- 移除舊的 /response endpoint 和型別
- 更新 /thread endpoint，改為非同步執行處理流程並立即回傳
- 在請求人類回應時送出 state.threadId
- 新增 handleHumanResponse 函式來處理人類回應
- 新增 /webhook endpoint 來處理 webhook 回應

```diff
src/server.ts
-import express from 'express';
+import express, { Request, Response } from 'express';
 import { Thread, agentLoop, handleNextStep } from '../src/agent';
 import { ThreadStore } from '../src/state';
-import { humanlayer } from 'humanlayer';
+import { humanlayer, V1Beta2HumanContactCompleted } from 'humanlayer';
 
 const app = express();
     });
 }
-
 // POST /thread - Start new thread
 // POST /thread - 啟動新的 thread
-app.post('/thread', async (req, res) => {
+app.post('/thread', async (req: Request, res: Response) => {
     const thread = new Thread([{
         type: "user_input",
     }]);
     
 -    const threadId = store.create(thread);
 -    const newThread = await agentLoop(thread);
 -    
 -    store.update(threadId, newThread);
 +    // run agent loop asynchronously, return immediately
 +    // 以非同步方式執行 agent 迴圈，並立即回傳
 +    Promise.resolve().then(async () => {
 +        const threadId = store.create(thread);
 +        const newThread = await agentLoop(thread);
 +        
 +        store.update(threadId, newThread);
 
 -    const lastEvent = newThread.events[newThread.events.length - 1];
 -    // If we exited the loop, include the response URL so the client can
 -    // 如果我們跳出迴圈，就附上 response URL，讓 client 可以
 -    // push a new message onto the thread
 -    // 將新的訊息推入這個 thread
 -    lastEvent.data.response_url = `/thread/${threadId}/response`;
 +        const lastEvent = newThread.events[newThread.events.length - 1];
 
 -    console.log("returning last event from endpoint", lastEvent);
 -
 -    res.json({ 
 -        thread_id: threadId,
 -        ...newThread 
 +        if (thread.awaitingHumanResponse()) {
 +            const hl = getHumanlayer();
 +            // create a human contact - returns immediately
 +            // 建立 human contact——會立即回傳
 +            hl.createHumanContact({
 +                spec: {
 +                    msg: lastEvent.data.message,
 +                    state: {
 +                        thread_id: threadId,
 +                    }
 +                }
 +            });
 +        }
      });
 +
 +    res.json({ status: "processing" });
  });
 
  // GET /thread/:id - Get thread status
  // GET /thread/:id - 取得 thread 狀態
 -app.get('/thread/:id', (req, res) => {
 +app.get('/thread/:id', (req: Request, res: Response) => {
      const thread = store.get(req.params.id);
      if (!thread) {
  });
 
 +type WebhookResponse = V1Beta2HumanContactCompleted;
 
 -type ApprovalPayload = {
 -    type: "approval";
 -    approved: boolean;
 -    comment?: string;
 -}
 +const handleHumanResponse = async (req: Request, res: Response) => {
 
 -type ResponsePayload = {
 -    type: "response";
 -    response: string;
  }
 
 -type Payload = ApprovalPayload | ResponsePayload;
 +app.post('/webhook', async (req: Request, res: Response) => {
 +    console.log("webhook response", req.body);
 +    const response = req.body as WebhookResponse;
 
 -// POST /thread/:id/response - Handle clarification response
 -// POST /thread/:id/response - 處理釐清回應
 -app.post('/thread/:id/response', async (req, res) => {
 -    let thread = store.get(req.params.id);
 +    // response is guaranteed to be set on a webhook
 +    // 在 webhook 中可保證 response 已被設定
 +    const humanResponse: string = response.event.status?.response as string;
 +
 +    const threadId = response.event.spec.state?.thread_id;
 +    if (!threadId) {
 +        return res.status(400).json({ error: "Thread ID not found" });
 +    }
 +    const thread = store.get(threadId);
      if (!thread) {
          return res.status(404).json({ error: "Thread not found" });
      }
 
 -    const body: Payload = req.body;
 -
 -    let lastEvent = thread.events[thread.events.length - 1];
 -
 -    if (thread.awaitingHumanResponse() && body.type === 'response') {
 -        thread.events.push({
 -            type: "human_response",
 -            data: body.response
 -        });
 -    } else if (thread.awaitingHumanApproval() && body.type === 'approval' && !body.approved) {
 -        // push feedback onto the thread
 -        // 將 feedback 推入 thread
 -        thread.events.push({
 -            type: "tool_response",
 -            data: `user denied the operation with feedback: "${body.comment}"`
 -        });
 -    } else if (thread.awaitingHumanApproval() && body.type === 'approval' && body.approved) {
 -        // approved, run the tool, pushing results onto the thread
 -        // 核准後執行 tool，並將結果推入 thread
 -        await handleNextStep(lastEvent.data, thread);
 -    } else {
 -        res.status(400).json({
 -            error: "Invalid request: " + body.type,
 -            awaitingHumanResponse: thread.awaitingHumanResponse(),
 -            awaitingHumanApproval: thread.awaitingHumanApproval()
 -        });
 -        return;
 +    if (!thread.awaitingHumanResponse()) {
 +        return res.status(400).json({ error: "Thread is not awaiting human response" });
      }
 
 -    
 -    // loop until stop event
 -    // 持續迴圈直到 stop event
 -    const result = await agentLoop(thread);
 -
 -    store.update(req.params.id, result);
 -
 -    lastEvent = result.events[result.events.length - 1];
 -    lastEvent.data.response_url = `/thread/${req.params.id}/response`;
 -
 -    console.log("returning last event from endpoint", lastEvent);
 -    
 -    res.json(result);
  });
 
```

<details>
<summary>skip this step / 跳過此步驟</summary>

    cp ./walkthrough/12a-server.ts src/server.ts

</details>

Start the server in another terminal

在另一個終端機中啟動 server

    npx tsx src/server.ts

now that the server is running, send a payload to the '/thread' endpoint

現在 server 已經在執行，請將一個 payload 傳送到 `'/thread'` endpoint。

__ do the response step

__ 進行 response 步驟

__ now handle approvals for divide

__ 現在處理 divide 的 approvals

__ now also handle done_for_now

__ 現在也處理 done_for_now
