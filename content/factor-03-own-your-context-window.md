[← Back to README](https://github.com/RobbinHsu/12-factor-agents/blob/main/README.md)
[← 返回 README](https://github.com/RobbinHsu/12-factor-agents/blob/main/README.md)

### 3. Own your context window
### 3. 掌握你的 context window

You don't necessarily need to use standard message-based formats for conveying context to an LLM.

你不一定需要使用標準的 message-based 格式，才能把 context 傳達給 LLM。

> #### At any given point, your input to an LLM in an agent is "here's what's happened so far, what's the next step"

> #### 在任一時刻，你在 agent 中提供給 LLM 的輸入其實就是：「到目前為止發生了什麼，下一步是什麼？」

<!-- todo syntax highlighting -->
<!-- ![130-own-your-context-building](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/130-own-your-context-building.png) -->

Everything is context engineering. [LLMs are stateless functions](https://thedataexchange.media/baml-revolution-in-ai-engineering/) that turn inputs into outputs. To get the best outputs, you need to give them the best inputs.

一切都是 context engineering。[LLMs are stateless functions](https://thedataexchange.media/baml-revolution-in-ai-engineering/)，它們會把輸入轉成輸出。若要得到最好的輸出，你就得給它們最好的輸入。

Creating great context means:

打造優秀的 context，意味著你要處理好以下幾件事：

- The prompt and instructions you give to the model
- 你提供給 model 的 prompt 與 instructions
- Any documents or external data you retrieve (e.g. RAG)
- 你擷取回來的任何文件或外部資料（例如 RAG）
- Any past state, tool calls, results, or other history 
- 任何過去的 state、tool calls、results，或其他歷史資訊
- Any past messages or events from related but separate histories/conversations (Memory)
- 來自相關但彼此分離的歷史／對話中的過往訊息或事件（Memory）
- Instructions about what sorts of structured data to output
- 關於應該輸出何種結構化資料的 instructions


![image](https://github.com/user-attachments/assets/0f1f193f-8e94-4044-a276-576bd7764fd0)

### on context engineering
### 關於 context engineering

This guide is all about getting as much as possible out of today's models. Notably not mentioned are:

本指南的重點，是盡可能把當今 models 的能力發揮到最大。特別沒有納入討論的，包括：

- Changes to models parameters like temperature, top_p, frequency_penalty, presence_penalty, etc.
- 調整 models parameters，例如 temperature、top_p、frequency_penalty、presence_penalty 等
- Training your own completion or embedding models
- 訓練你自己的 completion 或 embedding models
- Fine-tuning existing models
- 對既有 models 進行 fine-tuning


Again, I don't know what's the best way to hand context to an LLM, but I know you want the flexibility to be able to try EVERYTHING.

再說一次，我不知道把 context 交給 LLM 的最佳方式是什麼，但我知道你一定希望自己有能力把所有方法都試過一輪。

#### Standard vs Custom Context Formats
#### 標準與自訂的 context formats

Most LLM clients use a standard message-based format like this:

大多數 LLM clients 都使用像下面這樣的標準 message-based 格式：

```yaml
[
  {
    "role": "system",
    "content": "You are a helpful assistant..."
  },
  {
    "role": "user",
    "content": "Can you deploy the backend?"
  },
  {
    "role": "assistant",
    "content": null,
    "tool_calls": [
      {
        "id": "1",
        "name": "list_git_tags",
        "arguments": "{}"
      }
    ]
  },
  {
    "role": "tool",
    "name": "list_git_tags",
    "content": "{\"tags\": [{\"name\": \"v1.2.3\", \"commit\": \"abc123\", \"date\": \"2024-03-15T10:00:00Z\"}, {\"name\": \"v1.2.2\", \"commit\": \"def456\", \"date\": \"2024-03-14T15:30:00Z\"}, {\"name\": \"v1.2.1\", \"commit\": \"abe033d\", \"date\": \"2024-03-13T09:15:00Z\"}]}",
    "tool_call_id": "1"
  }
]
```

While this works great for most use cases, if you want to really get THE MOST out of today's LLMs, you need to get your context into the LLM in the most token- and attention-efficient way you can.

這種做法對大多數 use cases 都很好用；但如果你真的想把當前 LLMs 的能力榨到極致，就必須用最節省 token、也最有效利用 attention 的方式，把 context 放進 LLM。

As an alternative to the standard message-based format, you can build your own context format that's optimized for your use case. For example, you can use custom objects and pack/spread them into one or more user, system, assistant, or tool messages as makes sense.

作為標準 message-based 格式的替代方案，你可以建立專為自己 use case 最佳化的 context format。例如，你可以使用 custom objects，並視情況把它們 pack／spread 到一個或多個 user、system、assistant 或 tool messages 中。

Here's an example of putting the whole context window into a single user message:

以下是一個把整個 context window 放進單一 user message 的範例：

```yaml

[
  {
    "role": "system",
    "content": "You are a helpful assistant..."
  },
  {
    "role": "user",
    "content": |
            Here's everything that happened so far:
        
        <slack_message>
            From: @alex
            Channel: #deployments
            Text: Can you deploy the backend?
        </slack_message>
        
        <list_git_tags>
            intent: "list_git_tags"
        </list_git_tags>
        
        <list_git_tags_result>
            tags:
              - name: "v1.2.3"
                commit: "abc123"
                date: "2024-03-15T10:00:00Z"
              - name: "v1.2.2"
                commit: "def456"
                date: "2024-03-14T15:30:00Z"
              - name: "v1.2.1"
                commit: "ghi789"
                date: "2024-03-13T09:15:00Z"
        </list_git_tags_result>
        
        what's the next step?
    }
]
```

The model may infer that you're asking it `what's the next step` by the tool schemas you supply, but it never hurts to roll it into your prompt template.

model 也許能從你提供的 tool schemas 推斷出你在問它 `what's the next step`，但把這句話直接寫進 prompt template，通常也只有好處沒有壞處。

### code example
### code 範例

We can build this with something like: 

我們可以用像下面這樣的方式來建構：

```python

class Thread:
  events: List[Event]

class Event:
  # could just use string, or could be explicit - up to you
  # 可以只用 string，也可以明確定義——由你決定
  type: Literal["list_git_tags", "deploy_backend", "deploy_frontend", "request_more_information", "done_for_now", "list_git_tags_result", "deploy_backend_result", "deploy_frontend_result", "request_more_information_result", "done_for_now_result", "error"]
  data: ListGitTags | DeployBackend | DeployFrontend | RequestMoreInformation |  
        ListGitTagsResult | DeployBackendResult | DeployFrontendResult | RequestMoreInformationResult | string

def event_to_prompt(event: Event) -> str:
    data = event.data if isinstance(event.data, str) \
           else stringifyToYaml(event.data)

    return f"<{event.type}>\n{data}\n</{event.type}>"


def thread_to_prompt(thread: Thread) -> str:
  return '\n\n'.join(event_to_prompt(event) for event in thread.events)
```

#### Example Context Windows
#### context window 範例

Here's how context windows might look with this approach:

以下展示用這種方法建立出來的 context windows 可能長什麼樣子：

**Initial Slack Request:**
**初始 Slack 請求：**

```xml
<slack_message>
    From: @alex
    Channel: #deployments
    Text: Can you deploy the latest backend to production?
</slack_message>
```

**After Listing Git Tags:**
**列出 Git Tags 之後：**

```xml
<slack_message>
    From: @alex
    Channel: #deployments
    Text: Can you deploy the latest backend to production?
    Thread: []
</slack_message>

<list_git_tags>
    intent: "list_git_tags"
</list_git_tags>

<list_git_tags_result>
    tags:
      - name: "v1.2.3"
        commit: "abc123"
        date: "2024-03-15T10:00:00Z"
      - name: "v1.2.2"
        commit: "def456"
        date: "2024-03-14T15:30:00Z"
      - name: "v1.2.1"
        commit: "ghi789"
        date: "2024-03-13T09:15:00Z"
</list_git_tags_result>
```

**After Error and Recovery:**
**錯誤與復原之後：**

```xml
<slack_message>
    From: @alex
    Channel: #deployments
    Text: Can you deploy the latest backend to production?
    Thread: []
</slack_message>

<deploy_backend>
    intent: "deploy_backend"
    tag: "v1.2.3"
    environment: "production"
</deploy_backend>

<error>
    error running deploy_backend: Failed to connect to deployment service
</error>

<request_more_information>
    intent: "request_more_information_from_human"
    question: "I had trouble connecting to the deployment service, can you provide more details and/or check on the status of the service?"
</request_more_information>

<human_response>
    data:
      response: "I'm not sure what's going on, can you check on the status of the latest workflow?"
</human_response>
```

From here your next step might be: 

從這裡開始，你的下一步可能會是：

```python
nextStep = await determine_next_step(thread_to_prompt(thread))
```

```python
{
  "intent": "get_workflow_status",
  "workflow_name": "tag_push_prod.yaml",
}
```

The XML-style format is just one example - the point is you can build your own format that makes sense for your application. You'll get better quality if you have the flexibility to experiment with different context structures and what you store vs. what you pass to the LLM. 

XML 風格格式只是其中一個例子——重點在於，你可以建立最適合自己 application 的格式。當你能夠彈性實驗不同的 context 結構，以及哪些內容該儲存、哪些內容該傳給 LLM 時，你通常會得到更好的品質。

Key benefits of owning your context window:

自己掌握 context window 的主要好處包括：

1. **Information Density**: Structure information in ways that maximize the LLM's understanding
1. **Information Density**：用最能幫助 LLM 理解的方式組織資訊
2. **Error Handling**: Include error information in a format that helps the LLM recover. Consider hiding errors and failed calls from context window once they are resolved.
2. **Error Handling**：以能幫助 LLM 復原的格式納入錯誤資訊；一旦問題解決，也可以考慮把 errors 與失敗的 calls 從 context window 中隱藏起來
3. **Safety**: Control what information gets passed to the LLM, filtering out sensitive data
3. **Safety**：控制哪些資訊會傳給 LLM，並過濾掉敏感資料
4. **Flexibility**: Adapt the format as you learn what works best for your use case
4. **Flexibility**：隨著你逐步了解什麼最適合自己的 use case，持續調整格式
5. **Token Efficiency**: Optimize context format for token efficiency and LLM understanding
5. **Token Efficiency**：為 token 效率與 LLM 理解能力最佳化 context format


Context includes: prompts, instructions, RAG documents, history, tool calls, memory

Context 包含：prompts、instructions、RAG 文件、history、tool calls、memory

Remember: The context window is your primary interface with the LLM. Taking control of how you structure and present information can dramatically improve your agent's performance.

請記住：context window 是你與 LLM 之間最主要的介面。主動掌控你如何組織與呈現資訊，能大幅提升 agent 的表現。

Example - information density - same message, fewer tokens:

範例——information density：同一則訊息，更少的 tokens：

![Loom Screenshot 2025-04-22 at 09 00 56](https://github.com/user-attachments/assets/5cf041c6-72da-4943-be8a-99c73162b12a)

### Don't take it from me
### 不要只聽我說

About 2 months after 12-factor agents was published, context engineering started to become a pretty popular term.

大約在 12-factor agents 發表兩個月後，context engineering 開始成為一個相當熱門的詞。

<a href="https://x.com/karpathy/status/1937902205765607626"><img width="378" alt="Screenshot 2025-06-25 at 4 11 45 PM" src="https://github.com/user-attachments/assets/97e6e667-c35f-4855-8233-af40f05d6bce" /></a> <a href="https://x.com/tobi/status/1935533422589399127"><img width="378" alt="Screenshot 2025-06-25 at 4 12 59 PM" src="https://github.com/user-attachments/assets/7e6f5738-0d38-4910-82d1-7f5785b82b99" /></a>

There's also a quite good [Context Engineering Cheat Sheet](https://x.com/lenadroid/status/1943685060785524824) from [@lenadroid](https://x.com/lenadroid) from July 2025.

另外，2025 年 7 月 [@lenadroid](https://x.com/lenadroid) 也整理了一份相當不錯的 [Context Engineering Cheat Sheet](https://x.com/lenadroid/status/1943685060785524824)。

<a href="https://x.com/lenadroid/status/1943685060785524824"><img width="256" alt="image" src="https://github.com/user-attachments/assets/cac88aa3-8faf-440b-9736-cab95a9de477" /></a>

Recurring theme here: I don't know what's the best approach, but I know you want the flexibility to be able to try EVERYTHING.

這裡反覆出現的主題是：我不知道哪種方法最好，但我知道你一定希望自己有足夠的彈性，能把所有方法都試過。

[← Own Your Prompts](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-02-own-your-prompts.md) | [Tools Are Structured Outputs →](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs.md)
[← 掌握你的 prompts](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-02-own-your-prompts.md) | [tools 就是結構化輸出 →](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs.md)
