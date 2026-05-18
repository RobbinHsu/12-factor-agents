# Chapter 2 - Add Calculator Tools
# 第 2 章 - 加入 Calculator Tools

Let's add some calculator tools to our agent.

讓我們為 agent 加入一些 calculator tools。

Let's start by adding a tool definition for the calculator

先從為 calculator 加入 tool 定義開始。

These are simpile structured outputs that we'll ask the model to 
return as a "next step" in the agentic loop.

這些是簡單的 structured outputs，我們會要求 model 在 agentic loop 中
將它們作為「next step」回傳。


    cp ./walkthrough/02-tool_calculator.baml baml_src/tool_calculator.baml

<details>
<summary>show file / 顯示檔案</summary>

```rust
// ./walkthrough/02-tool_calculator.baml
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

現在，讓我們更新 agent 的 `DetermineNextStep` 方法，
將 calculator tools 暴露為可能的 next steps。


```diff
baml_src/agent.baml
 function DetermineNextStep(
     thread: string 
-) -> DoneForNow {
+) -> CalculatorTools | DoneForNow {
     client Qwen3
 
```

<details>
<summary>skip this step / 跳過此步驟</summary>

    cp ./walkthrough/02-agent.baml baml_src/agent.baml

</details>

Generate updated BAML client

產生更新後的 BAML client

    npx baml-cli generate

Try out the calculator

試試看 calculator

    npx tsx src/index.ts 'can you add 3 and 4'

You should see a tool call to the calculator

你應該會看到發給 calculator 的 tool call

    {
      intent: 'add',
      a: 3,
      b: 4
    }
