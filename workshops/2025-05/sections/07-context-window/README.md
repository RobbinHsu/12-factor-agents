# Chapter 7 - Customize Your Context Window
# 第 7 章 - 自訂你的 Context Window

In this section, we'll explore how to customize the context window
of the agent.

在本章節中，我們會探討如何自訂 agent 的 context window。

this is core to [factor 3 - own your context window](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-3-own-your-context-window.md)

這是 [factor 3 - own your context window](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-3-own-your-context-window.md) 的核心內容。


update the agent to pretty-print the Context window for the model

更新 agent，讓它能為 model 將 Context window 以易讀格式輸出。


```diff
src/agent.ts
         // can change this to whatever custom serialization you want to do, XML, etc
         // 你可以把這裡改成任何你想要的自訂序列化方式，例如 XML 等
         // e.g. https://github.com/got-agents/agents/blob/59ebbfa236fc376618f16ee08eb0f3bf7b698892/linear-assistant-ts/src/agent.ts#L66-L105
         // 例如：https://github.com/got-agents/agents/blob/59ebbfa236fc376618f16ee08eb0f3bf7b698892/linear-assistant-ts/src/agent.ts#L66-L105
-        return JSON.stringify(this.events);
+        return JSON.stringify(this.events, null, 2);
     }
 }
```

<details>
<summary>skip this step / 略過此步驟</summary>

    cp ./walkthrough/07-agent.ts src/agent.ts

</details>

Test the formatting

測試格式化結果

    BAML_LOG=info npx tsx src/index.ts 'can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result'

next, let's update the agent to use XML formatting instead 

接著，讓我們更新 agent，改用 XML 格式。

this is a very popular format for passing data to a model,

這是一種非常常見的 model 資料傳遞格式，

among other things, because of the token efficiency of XML.

其中一個原因就是 XML 具有不錯的 token 效率。


```diff
src/agent.ts
 
     serializeForLLM() {
-        // can change this to whatever custom serialization you want to do, XML, etc
-        // 你可以把這裡改成任何你想要的自訂序列化方式，例如 XML 等
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
<summary>skip this step / 略過此步驟</summary>

    cp ./walkthrough/07b-agent.ts src/agent.ts

</details>

let's try it out

讓我們試試看。


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
<summary>skip this step / 略過此步驟</summary>

    cp ./walkthrough/07c-agent.baml baml_src/agent.baml

</details>

check out the updated tests

查看更新後的測試


    npx baml-cli test

