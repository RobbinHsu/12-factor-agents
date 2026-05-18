# Chapter 1 - CLI and Agent Loop
# 第 1 章 - CLI 與 Agent Loop

Now let's add BAML and create our first agent with a CLI interface.

現在讓我們加入 BAML，並建立第一個帶有 CLI 介面的 agent。

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

加入我們的起始 agent，也就是後續會持續擴充的單一 baml prompt。

    cp ./walkthrough/01-agent.baml baml_src/agent.baml

<details>
<summary>show file / 顯示檔案</summary>

```rust
// ./walkthrough/01-agent.baml
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

    // use /nothink for now because the thinking tokens (or streaming thereof) screw with baml (i think (no pun intended))
    // 目前先使用 /nothink，因為 thinking tokens（或其串流輸出）會把 baml 搞亂（我想是這樣，無意雙關）
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

為本章節啟用 BAML logging

    export BAML_LOG=debug

Add the CLI interface

加入 CLI 介面

    cp ./walkthrough/01-cli.ts src/cli.ts

<details>
<summary>show file / 顯示檔案</summary>

```ts
// ./walkthrough/01-cli.ts
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
    // 將所有參數合併成單一訊息
    const message = args.join(" ");

    // Create a new thread with the user's message as the initial event
    // 使用使用者的訊息作為初始 event，建立新的 thread
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
import { b } from "../baml_client";

// tool call or a respond to human tool
// tool call 或回應 human 的 tool
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
        // 你可以改成任何你想要的自訂序列化方式，例如 XML 等
        // e.g. https://github.com/got-agents/agents/blob/59ebbfa236fc376618f16ee08eb0f3bf7b698892/linear-assistant-ts/src/agent.ts#L66-L105
        // 例如：https://github.com/got-agents/agents/blob/59ebbfa236fc376618f16ee08eb0f3bf7b698892/linear-assistant-ts/src/agent.ts#L66-L105
        return JSON.stringify(this.events);
    }
}

// right now this just runs one turn with the LLM, but
// 目前這只會讓 LLM 執行一個回合，但是
// we'll update this function to handle all the agent logic
// 我們之後會更新這個函式來處理所有 agent 邏輯
export async function agentLoop(thread: Thread): Promise<AgentResponse> {
    const nextStep = await b.DetermineNextStep(thread.serializeForLLM());
    return nextStep;
}
```

</details>

The the BAML code is configured to use BASETEN_API_KEY by default

BAML 程式碼預設已設定為使用 `BASETEN_API_KEY`。

To get a Baseten API key and URL, create an account at [baseten.co](https://baseten.co),
and then deploy [Qwen3 32B from the model library](https://www.baseten.co/library/qwen-3-32b/).

若要取得 Baseten API key 與 URL，請先在 [baseten.co](https://baseten.co) 建立帳號，
然後從 model library 部署 [Qwen3 32B from the model library](https://www.baseten.co/library/qwen-3-32b/)。

```rust 
  function DetermineNextStep(thread: string) -> DoneForNow {
      client Qwen3
      // ...
```

If you want to run the example with no changes, you can set the BASETEN_API_KEY env var to any valid baseten key.

如果你想在不做任何變更的情況下執行範例，可以將 `BASETEN_API_KEY` env var 設定為任何有效的 baseten key。

If you want to try swapping out the model, you can change the `client` line.

如果你想嘗試替換 model，可以修改 `client` 那一行。

[Docs on baml clients can be found here](https://docs.boundaryml.com/guide/baml-basics/switching-llms)

你可以在這裡查看關於 baml client 的文件：[Docs on baml clients can be found here](https://docs.boundaryml.com/guide/baml-basics/switching-llms)

For example, you can configure [gemini](https://docs.boundaryml.com/ref/llm-client-providers/google-ai-gemini) 
or [anthropic](https://docs.boundaryml.com/ref/llm-client-providers/anthropic) as your model provider.

例如，你可以將 [gemini](https://docs.boundaryml.com/ref/llm-client-providers/google-ai-gemini) 
或 [anthropic](https://docs.boundaryml.com/ref/llm-client-providers/anthropic) 設定為你的 model provider。

For example, to use openai with an OPENAI_API_KEY, you can do:

例如，如果要搭配 `OPENAI_API_KEY` 使用 openai，你可以這樣設定：

    client "openai/gpt-4o"


Set your env vars

設定你的 env vars

    export BASETEN_API_KEY=...
    export BASETEN_BASE_URL=...

Try it out

試試看

    npx tsx src/index.ts hello

you should see a familiar response from the model

你應該會看到 model 傳回熟悉的回應

    {
      intent: 'done_for_now',
      message: 'Hello! How can I assist you today?'
    }
