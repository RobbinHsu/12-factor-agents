# Building the 12-factor agent template from scratch
# 從頭構建 12-factor agent 範本

Steps to start from a bare TS repo and build up a 12-factor agent. This walkthrough will guide you through creating a TypeScript agent that follows the 12-factor methodology.

從裸 TS 儲存庫開始，逐步建立 12-factor agent。本指南將帶領你建立一個遵循 12-factor 方法論的 TypeScript agent。

## Cleanup
## 清理

Make sure you're starting from a clean slate

確保你從乾淨的起始狀態開始

Clean up existing files

清理現有檔案

    rm -rf baml_src/ && rm -rf src/

## Chapter 0 - Hello World
## 第 0 章 - Hello World

Let's start with a basic TypeScript setup and a hello world program.

讓我們從基本的 TypeScript 設定與一個 hello world 程式開始。

This guide is written in TypeScript (yes, a python version is coming soon)

本指南以 TypeScript 撰寫（是的，Python 版本很快就會推出）。

There are many checkpoints between the every file edit in theworkshop steps, 
so even if you aren't super familiar with typescript,
you should be able to keep up and run each example.

在 workshop 步驟中，每次檔案編輯之間都設有許多檢查點，
因此即使你對 TypeScript 不是特別熟悉，
也應該能跟上並執行每個範例。

To run this guide, you'll need a relatively recent version of nodejs and npm installed

要執行本指南，你需要先安裝相對較新的 nodejs 與 npm 版本。

You can use whatever nodejs version manager you want, [homebrew](https://formulae.brew.sh/formula/node) is fine

你可以使用任何你喜歡的 nodejs 版本管理工具，[homebrew](https://formulae.brew.sh/formula/node) 也可以。

    brew install node@20

You should see the node version

你應該會看到 node 的版本資訊

    node --version

Copy initial package.json

複製初始的 package.json

    cp ./walkthrough/00-package.json package.json

<details>

<summary>show file / 顯示檔案</summary>

```json
// ./walkthrough/00-package.json
// 檔案路徑：./walkthrough/00-package.json
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
// 檔案路徑：./walkthrough/00-tsconfig.json
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

新增 .gitignore

    cp ./walkthrough/00-.gitignore .gitignore

<details>

<summary>show file / 顯示檔案</summary>

```gitignore
// ./walkthrough/00-.gitignore
// 檔案路徑：./walkthrough/00-.gitignore
baml_client/
node_modules/
```

</details>

Create src folder

建立 src 資料夾

Add a simple hello world index.ts

加入一個簡單的 hello world index.ts

    cp ./walkthrough/00-index.ts src/index.ts

<details>

<summary>show file / 顯示檔案</summary>

```ts
// ./walkthrough/00-index.ts
// 檔案路徑：./walkthrough/00-index.ts
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

執行以驗證

    npx tsx src/index.ts

You should see:

你應該會看到：

    hello, world!

## Chapter 1 - CLI and Agent Loop
## 第 1 章 - CLI 與 Agent Loop

Now let's add BAML and create our first agent with a CLI interface.

現在讓我們加入 BAML，並建立第一個帶有 CLI 介面的 agent。

First, we'll need to install [BAML](https://github.com/boundaryml/baml)
which is a tool for prompting and structured outputs.

首先，我們需要安裝 [BAML](https://github.com/boundaryml/baml)，
它是一個用於 prompt 與結構化輸出的工具。

    npm install @boundaryml/baml

Initialize BAML

初始化 BAML

    npx baml-cli init

Remove default resume.baml

移除預設的 resume.baml

    rm baml_src/resume.baml

Add our starter agent, a single baml prompt that we'll build on

加入我們的起始 agent，也就是一個之後會持續擴充的單一 BAML prompt

    cp ./walkthrough/01-agent.baml baml_src/agent.baml

<details>

<summary>show file / 顯示檔案</summary>

```rust
// ./walkthrough/01-agent.baml
// 檔案路徑：./walkthrough/01-agent.baml
class DoneForNow {
  intent "done_for_now"
  message string 
}

client<llm> Qwen3 {
  provider "openai-generic"
  options {
    base_url env.BASETEN_BASE_URL
    api_key env.BASETEN_API_KEY 
  }
}

function DetermineNextStep(
    thread: string 
) -> DoneForNow {
    client Qwen3
    // client "openai/gpt-4o"
    // 使用 client "openai/gpt-4o"

    // use /nothink for now because the thinking tokens (or streaming thereof) screw with baml (i think (no pun intended))
    // 目前先使用 /nothink，因為 thinking tokens（或其串流輸出）會把 baml 弄壞（我想啦，這不是雙關）
    prompt #"
        {{ _.role("system") }}

        /nothink 

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

為本節啟用 BAML 日誌

    export BAML_LOG=debug

Add the CLI interface

加入 CLI 介面

    cp ./walkthrough/01-cli.ts src/cli.ts

<details>

<summary>show file / 顯示檔案</summary>

```ts
// ./walkthrough/01-cli.ts
// 檔案路徑：./walkthrough/01-cli.ts
// cli.ts lets you invoke the agent loop from the command line
// cli.ts 讓你可以從命令列啟動 agent loop

import { agentLoop, Thread, Event } from "./agent";

export async function cli() {
    // Get command line arguments, skipping the first two (node and script name)
    // 取得命令列參數，略過前兩個（node 與 script 名稱）
    const args = process.argv.slice(2);

    if (args.length === 0) {
        console.error("Error: Please provide a message as a command line argument");
        process.exit(1);
    }

    // Join all arguments into a single message
    // 將所有參數串接成單一訊息
    const message = args.join(" ");

    // Create a new thread with the user's message as the initial event
    // 使用使用者訊息作為初始事件來建立新的 thread
    const thread = new Thread([{ type: "user_input", data: message }]);

    // Run the agent loop with the thread
    // 以該 thread 執行 agent loop
    const result = await agentLoop(thread);
    console.log(result);
}
```

</details>

Update index.ts to use the CLI

更新 index.ts 以使用 CLI

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
// 檔案路徑：./walkthrough/01-agent.ts
import { b } from "../baml_client";

// tool call or a respond to human tool
// tool call 或回覆 human 的 tool
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
        // 可以改成任何你想要的自訂序列化格式，例如 XML 等
        // e.g. https://github.com/got-agents/agents/blob/59ebbfa236fc376618f16ee08eb0f3bf7b698892/linear-assistant-ts/src/agent.ts#L66-L105
        // 例如：https://github.com/got-agents/agents/blob/59ebbfa236fc376618f16ee08eb0f3bf7b698892/linear-assistant-ts/src/agent.ts#L66-L105
        return JSON.stringify(this.events);
    }
}

// right now this just runs one turn with the LLM, but
// 目前這只會執行 LLM 的單一回合，但
// we'll update this function to handle all the agent logic
// 我們會更新這個函式來處理所有 agent 邏輯
export async function agentLoop(thread: Thread): Promise<AgentResponse> {
    const nextStep = await b.DetermineNextStep(thread.serializeForLLM());
    return nextStep;
}
```

</details>

The the BAML code is configured to use BASETEN_API_KEY by default

BAML 程式碼預設會使用 BASETEN_API_KEY。

To get a Baseten API key and URL, create an account at [baseten.co](https://baseten.co),
and then deploy [Qwen3 32B from the model library](https://www.baseten.co/library/qwen-3-32b/).

若要取得 Baseten API 金鑰與 URL，請先在 [baseten.co](https://baseten.co) 建立帳號，
然後從模型庫部署 [Qwen3 32B from the model library](https://www.baseten.co/library/qwen-3-32b/)。

```rust 
  function DetermineNextStep(thread: string) -> DoneForNow {
      client Qwen3
      // ...
      // ……
```

If you want to run the example with no changes, you can set the BASETEN_API_KEY env var to any valid baseten key.

如果你想在不修改範例的情況下直接執行，可以將 BASETEN_API_KEY 環境變數設為任何有效的 Baseten 金鑰。

If you want to try swapping out the model, you can change the `client` line.

如果你想嘗試替換模型，可以修改 `client` 那一行。

[Docs on baml clients can be found here](https://docs.boundaryml.com/guide/baml-basics/switching-llms)

關於 baml clients 的文件可在上述連結找到。

For example, you can configure [gemini](https://docs.boundaryml.com/ref/llm-client-providers/google-ai-gemini) 
or [anthropic](https://docs.boundaryml.com/ref/llm-client-providers/anthropic) as your model provider.

例如，你可以將 [gemini](https://docs.boundaryml.com/ref/llm-client-providers/google-ai-gemini)
或 [anthropic](https://docs.boundaryml.com/ref/llm-client-providers/anthropic) 設定為你的模型提供者。

For example, to use openai with an OPENAI_API_KEY, you can do:

例如，若要搭配 OPENAI_API_KEY 使用 openai，可以這樣做：

    client "openai/gpt-4o"

Set your env vars

設定你的環境變數

    export BASETEN_API_KEY=...
export BASETEN_BASE_URL=...

Try it out

試跑看看

    npx tsx src/index.ts hello

you should see a familiar response from the model

你應該會看到模型給出熟悉的回應

    {
      intent: 'done_for_now',
      message: 'Hello! How can I assist you today?'
    }

## Chapter 2 - Add Calculator Tools
## 第 2 章 - 新增計算器工具

Let's add some calculator tools to our agent.

讓我們為代理新增一些計算器工具。

Let's start by adding a tool definition for the calculator

讓我們從新增計算器的工具定義開始

These are simpile structured outputs that we'll ask the model to 
return as a "next step" in the agentic loop.

這些是簡單的結構化輸出，我們會要求模型在代理循環中將其作為「下一步」返回。

    cp ./walkthrough/02-tool_calculator.baml baml_src/tool_calculator.baml

<details>

<summary>show file / 顯示檔案</summary>

```rust
// ./walkthrough/02-tool_calculator.baml
// 檔案路徑：./walkthrough/02-tool_calculator.baml
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

現在，讓我們更新代理的 DetermineNextStep 方法，
將計算器工具暴露為可能的下一步

```diff
baml_src/agent.baml
 function DetermineNextStep(
     thread: string 
-) -> DoneForNow {
+) -> CalculatorTools | DoneForNow {
     client Qwen3
+
     // client "openai/gpt-4o"
     // 使用 client "openai/gpt-4o"
 
```

<details>

<summary>skip this step / 跳過此步驟</summary>

    cp ./walkthrough/02-agent.baml baml_src/agent.baml

</details>

Generate updated BAML client

生成更新的 BAML client

    npx baml-cli generate

Try out the calculator

嘗試使用計算器

    npx tsx src/index.ts 'can you add 3 and 4'

You should see a tool call to the calculator

你應該會看到對計算器的工具呼叫

    {
      intent: 'add',
      a: 3,
      b: 4
    }

## Chapter 3 - Process Tool Calls in a Loop
## 第 3 章 - 在迴圈中處理工具呼叫

Now let's add a real agentic loop that can run the tools and get a final answer from the LLM.

現在讓我們新增一個真正的 agent 循環，可以執行工具並從 LLM 獲取最終答案。

First, lets update the agent to handle the tool call

首先，讓我們更新代理以處理工具呼叫

```diff
src/agent.ts
 }
 
-// right now this just runs one turn with the LLM, but
-// 目前這只會執行 LLM 的單一回合，但
-// we'll update this function to handle all the agent logic
-// 我們會更新這個函式來處理所有 agent 邏輯
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
+                // 回覆 human，回傳 next step 物件
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

現在，讓我們試看看

    npx tsx src/index.ts 'can you add 3 and 4'

you should see the agent call the tool and then return the result

你應該會看到代理呼叫工具然後返回結果

    {
      intent: 'done_for_now',
      message: 'The sum of 3 and 4 is 7.'
    }

For the next step, we'll do a more complex calculation, let's turn off the baml logs for more concise output

下一步，我們會做更複雜的計算，先關閉 baml 日誌以取得更簡潔的輸出

    export BAML_LOG=off

Try a multi-step calculation

嘗試多步驟的計算

    npx tsx src/index.ts 'can you add 3 and 4, then add 6 to that result'

you'll notice that tools like multiply and divide are not available

你會注意到像 multiply 和 divide 之類的工具尚未可用

    npx tsx src/index.ts 'can you multiply 3 and 4'

next, let's add handlers for the rest of the calculator tools

接著，讓我們為其餘的計算器工具新增處理器

```diff
src/agent.ts
-import { b } from "../baml_client";
+import { AddTool, SubtractTool, DivideTool, MultiplyTool, b } from "../baml_client";
 
-// tool call or a respond to human tool
-// tool call 或回覆 human 的 tool
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

測試減法

    npx tsx src/index.ts 'can you subtract 3 from 4'

now, let's test the multiplication tool

現在，讓我們測試乘法工具

    npx tsx src/index.ts 'can you multiply 3 and 4'

finally, let's test a more complex calculation with multiple operations

最後，讓我們測試一個包含多個運算的複雜計算

    npx tsx src/index.ts 'can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result'

congratulations, you've taking your first step into hand-rolling an agent loop.

恭喜，你已踏出手動構建代理循環的第一步。

from here, we're going to start incorporating some more intermediate and advanced
concepts for 12-factor agents.

從這裡開始，我們將開始融入更多中階與進階的 12-factor agent 概念。

## Chapter 4 - Add Tests to agent.baml
## 第 4 章 - 為 agent.baml 新增測試

Let's add some tests to our BAML agent.

讓我們為 BAML 代理新增一些測試。

to start, leave the baml logs enabled

首先，請保持 baml 日誌啟用

    export BAML_LOG=debug

next, let's add some tests to the agent

接下來，讓我們在代理中新增一些測試

We'll start with a simple test that checks the agent's ability to handle
a basic calculation.

我們將從一個簡單測試開始，檢查代理處理基本計算的能力。

```diff
baml_src/agent.baml
 ) -> CalculatorTools | DoneForNow {
     client Qwen3
-
     // client "openai/gpt-4o"
     // 使用 client "openai/gpt-4o"
 
-    // use /nothink for now because the thinking tokens (or streaming thereof) screw with baml (i think (no pun intended))
-    // 目前先使用 /nothink，因為 thinking tokens（或其串流輸出）會把 baml 弄壞（我想啦，這不是雙關）
     prompt #"
         {{ _.role("system") }}
 
 
         You are a helpful assistant that can help with tasks.
     "#
   }
+
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

執行測試

    npx baml-cli test

now, let's improve the test with assertions!

現在，讓我們用斷言改進測試！

Assertions are a great way to make sure the agent is working as expected,
and can easily be extended to check for more complex behavior.

斷言是確保代理按預期工作的好方法，並且可以輕鬆擴展以檢查更複雜的行為。

```diff
baml_src/agent.baml
 ) -> CalculatorTools | DoneForNow {
     client Qwen3
 
     prompt #"
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

執行測試

    npx baml-cli test

as you add more tests, you can disable the logs to keep the output clean.
You may want to turn them on as you iterate on specific tests.

當你新增更多測試時，可以停用日誌以保持輸出清爽。當你在特定測試上迭代時，可能會希望開啟日誌。

    export BAML_LOG=off

now, let's add some more complex test cases,
where we resume from in the middle of an in-progress
agentic context window

現在，讓我們新增一些更複雜的測試案例，
在這些案例中我們會從一個進行中的代理上下文視窗中間恢復

```diff
baml_src/agent.baml
   }
 }
-
 function DetermineNextStep(
     thread: string 
 ) -> CalculatorTools | DoneForNow {
     client Qwen3
+
     prompt #"
         {{ _.role("system") }}
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

讓我們來執行看看

    npx baml-cli test

## Chapter 5 - Multiple Human Tools
## 第 5 章 - 多種人類互動工具

In this section, we'll add support for multiple tools that serve to
contact humans.

在本節中，我們將新增支援多種用於聯繫人類的工具。

for this section, we'll disable the baml logs. You can optionally enable them if you want to see more details.

在本節中，我們會停用 baml 日誌。你也可以選擇啟用日誌以查看更詳細資訊。

    export BAML_LOG=off

first, let's add a tool that can request clarification from a human

首先，讓我們新增一個可以向人類請求澄清的工具

this will be different from the "done_for_now" tool,
and can be used to more flexibly handle different types of human interactions
in your agent.

這將不同於 "done_for_now" 工具，並可用於更靈活地處理代理中的不同類型人類互動。

```diff
baml_src/agent.baml
+// human tools are async requests to a human
+// human tools 是向人類發出的非同步請求
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
 
   }
 }
+
 function DetermineNextStep(
     thread: string 
-) -> CalculatorTools | DoneForNow {
+) -> HumanTools | CalculatorTools {
     client Qwen3
 
 }
 
+
```

<details>

<summary>skip this step / 跳過此步驟</summary>

    cp ./walkthrough/05-agent.baml baml_src/agent.baml

</details>

next, let's re-generate the client code

接著，讓我們重新產生 client 程式碼

NOTE - if you're using the VSCode extension for BAML,
the client will be regenerated automatically when you save the file
in your editor.

注意 - 如果你使用 BAML 的 VSCode 擴充套件，當你在編輯器中儲存檔案時，client 將會自動重新產生。

    npx baml-cli generate

now, let's update the agent to use the new tool

現在，讓我們更新代理以使用新工具

```diff
src/agent.ts
 }
 
-export async function agentLoop(thread: Thread): Promise<string> {
+export async function agentLoop(thread: Thread): Promise<Thread> {
 
     while (true) {
         switch (nextStep.intent) {
             case "done_for_now":
-                // response to human, return the next step object
-                // 回覆 human，回傳 next step 物件
-                return nextStep.message;
+            case "request_more_information":
+                // response to human, return the thread
+                // 回覆 human，回傳 thread
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

接著，讓我們更新 CLI 以處理澄清請求，方法是在 CLI 上向使用者請求輸入

```diff
src/cli.ts
 // cli.ts lets you invoke the agent loop from the command line
 // cli.ts 讓你可以從命令列啟動 agent loop
 
-import { agentLoop, Thread, Event } from "./agent";
+import { agentLoop, Thread, Event } from "../src/agent";
 
+
+
 export async function cli() {
     // Get command line arguments, skipping the first two (node and script name)
     // 取得命令列參數，略過前兩個（node 與 script 名稱）
     // Run the agent loop with the thread
     // 以該 thread 執行 agent loop
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
+    // 選用 - 這裡也可以繼續迴圈
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

讓我們試看看

    npx tsx src/index.ts 'can you multiply 3 and FD*(#F&& '

next, let's add a test that checks the agent's ability to handle
a clarification request

接下來，讓我們新增一個測試，檢查代理處理澄清請求的能力

```diff
baml_src/agent.baml
 ) -> HumanTools | CalculatorTools {
     client Qwen3
-
     // client "openai/gpt-4o"
     // 使用 client "openai/gpt-4o"
 
 
 
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

現在我們可以再次執行測試

    npx baml-cli test

you'll notice the new test passes, but the hello world test fails

你會注意到新的測試通過，但 hello world 測試失敗

This is because the agent's default behavior is to return "done_for_now"

這是因為代理的預設行為會返回 "done_for_now"

```diff
baml_src/agent.baml
     api_key env.BASETEN_API_KEY 
   }
 
 function DetermineNextStep(
 ) -> HumanTools | CalculatorTools {
     client Qwen3
+
     // client "openai/gpt-4o"
     // 使用 client "openai/gpt-4o"
 
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

驗證測試是否通過

    npx baml-cli test

## Chapter 6 - Customize Your Prompt with Reasoning
## 第 6 章 - 使用推理自訂你的 Prompt

In this section, we'll explore how to customize the prompt of the agent
with reasoning steps.

在本節中，我們將探討如何在 agent 的 prompt 中加入並自訂推理步驟。

this is core to [factor 2 - own your prompts](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-2-own-your-prompts.md)

這與 [factor 2 - own your prompts](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-2-own-your-prompts.md) 密切相關

there's a deep dive on reasoning on AI That Works [reasoning models versus reasoning steps](https://github.com/hellovai/ai-that-works/tree/main/2025-04-07-reasoning-models-vs-prompts)

AI That Works 有對推理進行深入探討，參見 [reasoning models versus reasoning steps](https://github.com/hellovai/ai-that-works/tree/main/2025-04-07-reasoning-models-vs-prompts)

for this section, it will be helpful to leave the baml logs enabled

在本節中，建議開啟 baml 日誌以便觀察細節

    export BAML_LOG=debug

update the agent prompt to include a reasoning step

更新 agent 的 prompt，加入推理解題步驟

```diff
baml_src/agent.baml
     api_key env.BASETEN_API_KEY 
   }
 
 function DetermineNextStep(
 
         {{ ctx.output_format }}
+
+        First, always plan out what to do next, for example:
+
+        - ...
+        - ...
+        - ...
+
+        {...} // schema
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

產生更新後的 client

    npx baml-cli generate

now, you can try it out with a simple prompt

現在你可以用簡單的 prompt 試試看

    npx tsx src/index.ts 'can you multiply 3 and 4'

you should see output from the baml logs showing the reasoning steps

你應該會在 baml 日誌中看到顯示推理步驟的輸出

#### optional challenge
可選挑戰

add a field to your tool output format that includes the reasoning steps in the output!

在你的 tool output 格式中新增一個欄位，將推理步驟包含在輸出中！

## Chapter 7 - Customize Your Context Window
## 第 7 章 - 自訂你的 Context Window

In this section, we'll explore how to customize the context window
of the agent.

在本節中，我們將探討如何自訂 agent 的 context window。

this is core to [factor 3 - own your context window](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-3-own-your-context-window.md)

這與 [factor 3 - own your context window](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-3-own-your-context-window.md) 密切相關

update the agent to pretty-print the Context window for the model

更新 agent 以美化（pretty-print）傳給 model 的 Context window

```diff
src/agent.ts
         // can change this to whatever custom serialization you want to do, XML, etc
         // 可以改成任何你想要的自訂序列化格式，例如 XML 等
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

測試輸出格式化結果

    BAML_LOG=info npx tsx src/index.ts 'can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result'

next, let's update the agent to use XML formatting instead

接下來，讓我們把 agent 改成使用 XML 格式化

this is a very popular format for passing data to a model,

這是一種很常用於傳遞資料給 model 的格式，

among other things, because of the token efficiency of XML.

其中一個原因是 XML 在 token 使用上比較有效率。

```diff
src/agent.ts
 
     serializeForLLM() {
-        // can change this to whatever custom serialization you want to do, XML, etc
-        // 可以改成任何你想要的自訂序列化格式，例如 XML 等
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

讓我們試試看

    BAML_LOG=info npx tsx src/index.ts 'can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result'

lets update our tests to match the new output format

更新測試以配合新的輸出格式

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

檢查更新後的測試

    npx baml-cli test

## Chapter 8 - Adding API Endpoints
## 第 8 章 - 新增 API 端點

Add an Express server to expose the agent via HTTP.

新增一個 Express server，透過 HTTP 將 agent 對外暴露。

for this section, we'll disable the baml logs. You can optionally enable them if you want to see more details.

在本節中，我們會關閉 baml 日誌。若要查看詳細資訊，可選擇開啟它們。

    export BAML_LOG=off

Install Express and types

安裝 Express 與其類型定義

    npm install express && npm install --save-dev @types/express supertest

Add the server implementation

加入 server 的實作

    cp ./walkthrough/08-server.ts src/server.ts

<details>

<summary>show file / 顯示檔案</summary>

```ts
// ./walkthrough/08-server.ts
// 檔案路徑：./walkthrough/08-server.ts
import express from 'express';
import { Thread, agentLoop } from '../src/agent';

const app = express();
app.use(express.json());
app.set('json spaces', 2);

// POST /thread - Start new thread
// POST /thread - 啟動新 thread
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
    // 選用 - 可加入 state
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

使用 curl 測試（在另一個終端機）

    curl -X POST http://localhost:3000/thread \
  -H "Content-Type: application/json" \
  -d '{"message":"can you add 3 and 4"}'

You should get an answer from the agent which includes the
agentic trace, ending in a message like:

你應該會從 agent 得到包含 agentic trace 的回答，最後會有類似以下的訊息：

    {"intent":"done_for_now","message":"The sum of 3 and 4 is 7."}

## Chapter 9 - In-Memory State and Async Clarification
## 第 9 章 - 記憶體內狀態與非同步澄清流程

Add state management and async clarification support.

加入狀態管理與非同步澄清支援。

for this section, we'll disable the baml logs. You can optionally enable them if you want to see more details.

在本節中，我們會關閉 baml 日誌。若想查看更多細節，可選擇開啟。

    export BAML_LOG=off

Add some simple in-memory state management for threads

為 threads 新增簡單的記憶體內狀態管理

    cp ./walkthrough/09-state.ts src/state.ts

<details>

<summary>show file / 顯示檔案</summary>

```ts
// ./walkthrough/09-state.ts
// 檔案路徑：./walkthrough/09-state.ts
import crypto from 'crypto';
import { Thread } from '../src/agent';


// you can replace this with any simple state management,
// 你可以將這裡替換成任何簡單的狀態管理方案，
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

更新 server，使用狀態管理

* Add thread state management using `ThreadStore`
* 使用 `ThreadStore` 加入 thread 狀態管理
* return thread IDs and response URLs from the /thread endpoint
* 從 /thread 端點回傳 thread IDs 與 response URLs
* implement GET /thread/:id
* 實作 GET /thread/:id
* implement POST /thread/:id/response
* 實作 POST /thread/:id/response


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
 // POST /thread - 啟動新 thread
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
+    // 若我們離開了迴圈，將 response URL 附上，方便 client 推送新訊息
+    // push a new message onto the thread
+    // 將新訊息推送到 thread 上
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
-    // 選用 - 可加入 state
-    res.status(404).json({ error: "Not implemented yet" });
+    const thread = store.get(req.params.id);
+    if (!thread) {
+        return res.status(404).json({ error: "Thread not found" });
+    }
+    res.json(thread);
 });
 
+// POST /thread/:id/response - Handle clarification response
+// POST /thread/:id/response - 處理澄清請求的回應
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
+    // 迴圈直到停止事件
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

測試澄清流程

    curl -X POST http://localhost:3000/thread \
  -H "Content-Type: application/json" \
  -d '{"message":"can you multiply 3 and xyz"}'

## Chapter 10 - Adding Human Approval
## 第 10 章 - 新增人工審批

Add support for human approval of operations.

加入人工審批（human approval）操作的支援。

for this section, we'll disable the baml logs. You can optionally enable them if you want to see more details.

在本節中，我們會關閉 baml 日誌。若想查看更多細節，可選擇開啟。

    export BAML_LOG=off

update the server to handle human approvals

更新 server 以處理人工審批

* Import `handleNextStep` to execute approved actions
* 匯入 `handleNextStep` 以執行已核准的動作
* Add two payload types to distinguish approvals from responses
* 新增兩種 payload 類型以區分 approvals 與 responses
* Handle responses and approvals differently in the endpoint
* 在端點中分別處理 responses 與 approvals
* Show better error messages when things go wrongs
* 當錯誤發生時顯示更明確的錯誤訊息


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
 // POST /thread/:id/response - 處理澄清請求的回應
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
+        // 將回饋推送到 thread 上
+        thread.events.push({
+            type: "tool_response",
+            data: `user denied the operation with feedback: "${body.comment}"`
+        });
+    } else if (thread.awaitingHumanApproval() && body.type === 'approval' && body.approved) {
+        // approved, run the tool, pushing results onto the thread
+        // 已核准，執行工具，將結果推送到 thread 上
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
     // 迴圈直到停止事件
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

在 agent 中新增幾個方法以處理審批與回應

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
                 // 回覆 human，回傳 thread
                 return thread;
+            case "divide":
+                // divide is scary, return it for human approval
+                // divide 風險較高，回傳給人工審批
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

測試需審批的除法流程

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

用另一個 curl 呼叫拒絕該請求，並更換 thread ID

    curl -X POST 'http://localhost:3000/thread/{thread_id}/response' \
  -H "Content-Type: application/json" \
  -d '{"type": "approval", "approved": false, "comment": "I dont think thats right, use 5 instead of 4"}'

You should see: the last tool call is now `"intent":"divide","a":3,"b":5`

你應該會看到：最後一次的 tool call 現在為 `"intent":"divide","a":3,"b":5`

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

現在你可以核准該操作

    curl -X POST 'http://localhost:3000/thread/{thread_id}/response' \
  -H "Content-Type: application/json" \
  -d '{"type": "approval", "approved": true}'

you should see the final message includes the tool response and final result!

你應該會看到最後的訊息包含 tool response 與最終結果！

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
## 第 11 章 - 透過 Email 的人工審批

in this section, we'll add support for human approvals over email.

在本節中，我們將加入透過 Email 進行人工審批的支援。

This will start a little bit contrived, just to get the concepts down -

這個範例一開始會有點刻意（contrived），只是用來說明概念 —

We'll start by invoking the workflow from the CLI but approvals for `divide`
and `request_more_information` will be handled over email,
then the final `done_for_now` answer will be printed back to the CLI

我們會從 CLI 呼叫這個流程，`divide` 與 `request_more_information` 的審批會透過 Email 處理，最後的 `done_for_now` 回覆會印回 CLI

While contrived, this is a great example of the flexibility you get from
[factor 7 - contact humans with tools](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-7-contact-humans-with-tools.md)

雖然有點刻意，但這是展示 [factor 7 - contact humans with tools](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-7-contact-humans-with-tools.md) 彈性的好範例

for this section, we'll disable the baml logs. You can optionally enable them if you want to see more details.

在本節中，我們會關閉 baml 日誌。若想看更多細節可選擇開啟。

    export BAML_LOG=off

Install HumanLayer

安裝 HumanLayer

    npm install humanlayer

Update CLI to send `divide` and `request_more_information` to a human via email

更新 CLI，將 `divide` 與 `request_more_information` 發送給人工（透過 Email）

```diff
src/cli.ts
 // cli.ts lets you invoke the agent loop from the command line
 // cli.ts 讓你可以從命令列啟動 agent loop
 
+import { humanlayer } from "humanlayer";
 import { agentLoop, Thread, Event } from "../src/agent";
 
-
-
 export async function cli() {
     // Get command line arguments, skipping the first two (node and script name)
     // 取得命令列參數，略過前兩個（node 與 script 名稱）
 
     // Run the agent loop with the thread
     // 以該 thread 執行 agent loop
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
     // 輸出最終結果
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
+        // name of this agent
+        // 此 agent 的名稱
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
+        // 同步取得審批結果 - 這將會阻塞直到收到回覆
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

你程式的最後一行應該會提到有人類審查的步驟

    nextStep { intent: 'divide', a: 4, b: 5 }
    HumanLayer: Requested human approval from HumanLayer cloud

go ahead and respond to the email with some feedback:

去信箱回覆一些回饋：

![reject-email](https://github.com/RobbinHsu/12-factor-agents/blob/main/workshops/2025-05/walkthrough/11-email-reject.png?raw=true)

you should get another email with an updated attempt based on your feedback!

你應該會另一封基於你回饋的更新嘗試郵件！

You can go ahead and approve this one:

你也可以核准該次請求：

![approve-email](https://github.com/RobbinHsu/12-factor-agents/blob/main/workshops/2025-05/walkthrough/11-email-approve.png?raw=true)

and your final output will look like

最後你的輸出會長這樣

    nextStep {
     intent: 'done_for_now',
     message: 'The division of 4 by 5 is 0.8. If you have any other calculations or questions, feel free to ask!'
    }
    The division of 4 by 5 is 0.8. If you have any other calculations or questions, feel free to ask!

lets implement the `request_more_information` flow as well

接著我們也實作 `request_more_information` 流程

```diff
src/cli.ts
     }) 
 
+    if (lastEvent.data.intent === "request_more_information") {
+        // fetch response synchronously - this will block until reply
+        // 同步取得回應 - 這將會阻塞直到收到回覆
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
         // 同步取得審批結果 - 這將會阻塞直到收到回覆
```

<details>

<summary>skip this step / 跳過此步驟</summary>

    cp ./walkthrough/11b-cli.ts src/cli.ts

</details>

lets test the require_approval flow as by asking for a calculation
with garbled input:

讓我們透過請求一個含雜訊（garbled）輸入的計算來測試 require_approval 流程：

    npx tsx src/index.ts 'can you multiply 4 and xyz'

You should get an email with a request for clarification

你會收到一封請求澄清的郵件

    Can you clarify what 'xyz' represents in this context? Is it a specific number, variable, or something else?

you can response with something like

你可以回覆一個像是以下的內容

    use 8 instead of xyz

you should see a final result on the CLI like

你應該會在 CLI 上看到最終結果，如下：

    I have multiplied 4 and xyz, using the value 8 for xyz, resulting in 32.

as a final step, lets explore using a custom html template for the email

最後一步，我們來探討為郵件使用自訂的 HTML 範本

```diff
src/cli.ts
             email: {
                 address: process.env.HUMANLAYER_EMAIL,
+                // custom email body - jinja
+                // 自訂 email 內容 - jinja
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

你會看到使用自訂範本的不同郵件樣貌

![custom-template-email](https://github.com/RobbinHsu/12-factor-agents/blob/main/workshops/2025-05/walkthrough/11-email-custom.png?raw=true)

feel free to run with the flow and then you can try updating the template to your liking

歡迎實際執行該流程，並隨意按你喜好調整範本

(if you're using cursor, something as simple as highlighting the template and asking to "make it better"
should do the trick)

（如果你用 cursor，像是選取範本並請模型「改進它」就能做到）

try triggering "request_more_information" as well!

也試著觸發 "request_more_information"！

thats it - in the next chapter, we'll build a fully email-driven
workflow agent that uses webhooks for human approval

到此為止 — 在下一章，我們將建立一個完全以 Email 為驅動、並使用 webhook 接收人工審批結果的工作流程 agent

## Chapter XX - HumanLayer Webhook Integration
## 第 XX 章 - HumanLayer Webhook 整合

the previous sections used the humanlayer SDK in "synchronous mode" - that
means every time we wait for human approval, we sit in a loop
polling until the human response if received.

前面的章節使用 HumanLayer SDK 的「同步模式（synchronous mode）」——也就是每次等待人工審批時，我們都會在迴圈中輪詢直到收到人工回覆。

That's obviously not ideal, especially for production workloads,
so in this section we'll implement [factor 6 - launch / pause / resume with simple APIs](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-6-launch-pause-resume.md)
by updating the server to end processing after contacting a human, and use webhooks to receive the results.

這顯然不理想，尤其在生產環境中。因此在本節中我們會依據 [factor 6 - launch / pause / resume with simple APIs](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-6-launch-pause-resume.md) 的指引，更新 server：在聯絡人工後結束處理，並透過 webhook 接收結果。

add code to initialize humanlayer in the server

在 server 中新增初始化 humanlayer 的程式碼

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
 // POST /thread - 啟動新 thread
 app.post('/thread', async (req, res) => {
     
     // loop until stop event
     // 迴圈直到停止事件
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

接下來，我們將更新 /thread 端點以：

1. handle requests asynchronously, returning immediately
1. 非同步處理請求，立即回應
2. create a human contact on request_more_information and done_for_now calls
2. 在 request_more_information 與 done_for_now 呼叫時建立 human contact


Update the server to be able to handle request_clarification responses

更新 server，使其能處理 request_clarification 的回應

- remove the old /response endpoint and types
- 移除舊的 /response 端點與類型
- update the /thread endpoint to run processing asynchronously, return immediately
- 更新 /thread 端點，使處理非同步執行並立即回應
- send a state.threadId when requesting human responses
- 在請求人類回覆時送出 state.threadId
- add a handleHumanResponse function to process the human response
- 新增 handleHumanResponse 函式處理人類回應
- add a /webhook endpoint to handle the webhook response
- 新增 /webhook 端點處理 webhook 回應


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
 // POST /thread - 啟動新 thread
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
+    // 非同步執行 agent loop，立即回應
+    Promise.resolve().then(async () => {
+        const threadId = store.create(thread);
+        const newThread = await agentLoop(thread);
+        
+        store.update(threadId, newThread);
 
-    const lastEvent = newThread.events[newThread.events.length - 1];
-    // If we exited the loop, include the response URL so the client can
-    // 若我們離開了迴圈，將 response URL 附上，方便 client 推送新訊息
-    // push a new message onto the thread
-    // 將新訊息推送到 thread 上
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
+            // 建立 human contact - 立即回傳
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
-// POST /thread/:id/response - 處理澄清請求的回應
-app.post('/thread/:id/response', async (req, res) => {
-    let thread = store.get(req.params.id);
+    // response is guaranteed to be set on a webhook
+    // webhook 中的 response 一定有值
+    const humanResponse: string = response.event.status?.response as string;
+
+    const threadId = response.event.spec.state?.thread_id;
+    if (!threadId) {
+        return res.status(400).json({ error: "Thread ID not found" });
+    }
+
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
-        // 將回饋推送到 thread 上
-        thread.events.push({
-            type: "tool_response",
-            data: `user denied the operation with feedback: "${body.comment}"`
-        });
-    } else if (thread.awaitingHumanApproval() && body.type === 'approval' && body.approved) {
-        // approved, run the tool, pushing results onto the thread
-        // 已核准，執行工具，將結果推送到 thread 上
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
-    // 迴圈直到停止事件
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

在另一個終端機啟動 server

    npx tsx src/server.ts

now that the server is running, send a payload to the '/thread' endpoint

現在 server 已啟動，向 '/thread' 端點送出一個 payload

__ do the response step

__ 執行回應步驟

__ now handle approvals for divide

__ 現在處理 divide 的審批

__ now also handle done_for_now

__ 現在也處理 done_for_now
