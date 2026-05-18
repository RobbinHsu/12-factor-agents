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
<summary>skip this step / 略過此步驟</summary>

    cp ./walkthrough/06-agent.baml baml_src/agent.baml

</details>

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


