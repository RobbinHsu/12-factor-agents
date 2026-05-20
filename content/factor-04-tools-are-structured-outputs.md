[← Back to README](https://github.com/RobbinHsu/12-factor-agents/blob/main/README.md)
[← 返回 README](https://github.com/RobbinHsu/12-factor-agents/blob/main/README.md)

### 4. Tools are just structured outputs
### 4. tools 只是結構化輸出

Tools don't need to be complex. At their core, they're just structured output from your LLM that triggers deterministic code.

Tools 不必很複雜。從核心來看，它們只是 LLM 輸出的結構化資料，用來觸發 deterministic code。

![140-tools-are-just-structured-outputs](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/140-tools-are-just-structured-outputs.png)

For example, lets say you have two tools `CreateIssue` and `SearchIssues`. To ask an LLM to "use one of several tools" is just to ask it to output JSON we can parse into an object representing those tools.

例如，假設你有兩個 tools：`CreateIssue` 與 `SearchIssues`。要請 LLM「使用多個 tools 之一」，本質上就是要求它輸出我們可以解析成對應物件的 JSON。

```python

class Issue:
  title: str
  description: str
  team_id: str
  assignee_id: str

class CreateIssue:
  intent: "create_issue"
  issue: Issue

class SearchIssues:
  intent: "search_issues"
  query: str
  what_youre_looking_for: str
```

The pattern is simple:

這個模式很簡單：

1. LLM outputs structured JSON
1. LLM 輸出結構化 JSON
3. Deterministic code executes the appropriate action (like calling an external API)
3. Deterministic code 執行對應的動作（例如呼叫外部 API）
4. Results are captured and fed back into the context
4. 將結果擷取後再餵回 context


This creates a clean separation between the LLM's decision-making and your application's actions. The LLM decides what to do, but your code controls how it's done. Just because an LLM "called a tool" doesn't mean you have to go execute a specific corresponding function in the same way every time.

這會在 LLM 的決策與 application 的實際動作之間，建立清楚的分界。LLM 決定要做什麼，但怎麼做則由你的程式碼掌控。即使 LLM「呼叫了一個 tool」，也不代表你每次都一定得用同一種方式去執行某個固定對應的 function。

If you recall our switch statement from above

如果你還記得上面的 switch statement：

```python
if nextStep.intent == 'create_payment_link':
    stripe.paymentlinks.create(nextStep.parameters)
    return # or whatever you want, see below
    # 或其他你想要的處理，見下文
elif nextStep.intent == 'wait_for_a_while': 
    # do something monadic idk
    # 做一些 monadic 的處理，我也不知道
else: #... the model didn't call a tool we know about
    # model 沒有呼叫我們已知的 tool
    # do something else
    # 做其他處理
```

**Note**: there has been a lot said about the benefits of "plain prompting" vs. "tool calling" vs. "JSON mode" and the performance tradeoffs of each. We'll link some resources to that stuff soon, but not gonna get into it here. See [Prompting vs JSON Mode vs Function Calling vs Constrained Generation vs SAP](https://www.boundaryml.com/blog/schema-aligned-parsing), [When should I use function calling, structured outputs, or JSON mode?](https://www.vellum.ai/blog/when-should-i-use-function-calling-structured-outputs-or-json-mode#:~:text=We%20don%27t%20recommend%20using%20JSON,always%20use%20Structured%20Outputs%20instead) and [OpenAI JSON vs Function Calling](https://docs.llamaindex.ai/en/stable/examples/llm/openai_json_vs_function_calling/).

**注意**：關於「plain prompting」、「tool calling」與「JSON mode」各自的優點，以及它們在效能上的取捨，外界已有很多討論。我們之後會補上一些相關資源，但這裡先不深入展開。可參考 [Prompting vs JSON Mode vs Function Calling vs Constrained Generation vs SAP](https://www.boundaryml.com/blog/schema-aligned-parsing)、[When should I use function calling, structured outputs, or JSON mode?](https://www.vellum.ai/blog/when-should-i-use-function-calling-structured-outputs-or-json-mode#:~:text=We%20don%27t%20recommend%20using%20JSON,always%20use%20Structured%20Outputs%20instead) 與 [OpenAI JSON vs Function Calling](https://docs.llamaindex.ai/en/stable/examples/llm/openai_json_vs_function_calling/)。

The "next step" might not be as atomic as just "run a pure function and return the result". You unlock a lot of flexibility when you think of "tool calls" as just a model outputting JSON describing what deterministic code should do. Put this together with [factor 8 own your control flow](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md).

所謂的「next step」，不一定要原子化到只剩下「執行一個 pure function 然後回傳結果」。當你把「tool calls」視為 model 輸出的 JSON，用來描述 deterministic code 應該做什麼時，你就能解鎖大量彈性。把這個觀念再與 [factor 8 own your control flow](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md) 搭配起來看。

[← Own Your Context Window](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md) | [Unify Execution State →](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md)
[← 掌握你的 context window](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md) | [統一 execution state →](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md)
