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

<details>
<summary>show file / 顯示檔案</summary>

```rust
// ./walkthrough/01-agent.baml
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
    // 將所有參數合併成單一訊息
    const message = args.join(" ");

    // Create a new thread with the user's message as the initial event
    // 建立新的 thread，並以使用者訊息作為初始 event
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
<summary>skip this step / 略過此步驟</summary>

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
// tool call 或回應給人類的 tool
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
// 目前這只會讓 LLM 跑一個回合，但
// we'll update this function to handle all the agent logic
// 我們會更新這個函式來處理所有 agent 邏輯
export async function agentLoop(thread: Thread): Promise<AgentResponse> {
    const nextStep = await b.DetermineNextStep(thread.serializeForLLM());
    return nextStep;
}
```

</details>

The the BAML code is configured to use OPENAI_API_KEY by default

BAML 程式碼預設已設定為使用 `OPENAI_API_KEY`。

As you're testing, you can change the model / provider to something else
as you please

在測試時，你可以依需求把 model / provider 換成其他選項。

        client "openai/gpt-4o"

[Docs on baml clients can be found here](https://docs.boundaryml.com/guide/baml-basics/switching-llms)

[關於 baml client 的文件可在此查看](https://docs.boundaryml.com/guide/baml-basics/switching-llms)

For example, you can configure [gemini](https://docs.boundaryml.com/ref/llm-client-providers/google-ai-gemini) 
or [anthropic](https://docs.boundaryml.com/ref/llm-client-providers/anthropic) as your model provider.

例如，你可以將 [gemini](https://docs.boundaryml.com/ref/llm-client-providers/google-ai-gemini)
或 [anthropic](https://docs.boundaryml.com/ref/llm-client-providers/anthropic) 設定為你的 model provider。

If you want to run the example with no changes, you can set the OPENAI_API_KEY env var to any valid openai key.

如果你想在不做任何變更的情況下執行範例，可以將 `OPENAI_API_KEY` env var 設為任何有效的 openai key。


    export OPENAI_API_KEY=...

Try it out

試試看

    npx tsx src/index.ts hello

you should see a familiar response from the model

你應該會看到 model 傳回熟悉的回應。

    {
  intent: 'done_for_now',
  message: 'Hello! How can I assist you today?'
}

