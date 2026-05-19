[← Back to README](https://github.com/RobbinHsu/12-factor-agents/blob/main/README.md)
[← 返回 README](https://github.com/RobbinHsu/12-factor-agents/blob/main/README.md)

### 2. Own your prompts
### 2. 掌握你的 prompts

Don't outsource your prompt engineering to a framework. 

不要把你的 prompt engineering 外包給 framework。

![120-own-your-prompts](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/120-own-your-prompts.png)

By the way, [this is far from novel advice:](https://hamel.dev/blog/posts/prompt/)

順帶一提，[這遠遠不是什麼新穎的建議：](https://hamel.dev/blog/posts/prompt/)

![image](https://github.com/user-attachments/assets/575bab37-0f96-49fb-9ce3-9a883cdd420b)

Some frameworks provide a "black box" approach like this:

有些 frameworks 會提供像這樣的「black box」做法：

```python
agent = Agent(
  role="...",
  goal="...",
  personality="...",
  tools=[tool1, tool2, tool3]
)

task = Task(
  instructions="...",
  expected_output=OutputModel
)

result = agent.run(task)
```

This is great for pulling in some TOP NOTCH prompt engineering to get you started, but it is often difficult to tune and/or reverse engineer to get exactly the right tokens into your model.

這對於快速導入一些一流的 prompt engineering 做法、讓你開始動手很有幫助，但若你想把正確的 tokens 精準送進 model，往往很難微調，也很難反向拆解其中的機制。

Instead, own your prompts and treat them as first-class code:

相反地，你應該自己掌握 prompts，並把它們視為一等公民的程式碼：

```rust
function DetermineNextStep(thread: string) -> DoneForNow | ListGitTags | DeployBackend | DeployFrontend | RequestMoreInformation {
  prompt #"
    {{ _.role("system") }}
    
    You are a helpful assistant that manages deployments for frontend and backend systems.
    You work diligently to ensure safe and successful deployments by following best practices
    and proper deployment procedures.
    
    Before deploying any system, you should check:
    - The deployment environment (staging vs production)
    - The correct tag/version to deploy
    - The current system status
    
    You can use tools like deploy_backend, deploy_frontend, and check_deployment_status
    to manage deployments. For sensitive deployments, use request_approval to get
    human verification.
    
    Always think about what to do first, like:
    - Check current deployment status
    - Verify the deployment tag exists
    - Request approval if needed
    - Deploy to staging before production
    - Monitor deployment progress
    
    {{ _.role("user") }}

    {{ thread }}
    
    What should the next step be?
  "#
}
```

(the above example uses [BAML](https://github.com/boundaryml/baml) to generate the prompt, but you can do this with any prompt engineering tool you want, or even just template it manually)

（上面的範例使用 [BAML](https://github.com/boundaryml/baml) 來產生 prompt，但你也可以用任何你喜歡的 prompt engineering 工具來做，甚至手動套用 template 也行。）

If the signature looks a little funny, we'll get to that in [factor 4 - tools are just structured outputs](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs.md)

如果這個 signature 看起來有點奇怪，我們會在 [factor 4 - tools are just structured outputs](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs.md) 進一步說明。

```typescript
function DetermineNextStep(thread: string) -> DoneForNow | ListGitTags | DeployBackend | DeployFrontend | RequestMoreInformation {
```

Key benefits of owning your prompts:

自己掌握 prompts 的主要好處包括：

1. **Full Control**: Write exactly the instructions your agent needs, no black box abstractions
2. **Testing and Evals**: Build tests and evals for your prompts just like you would for any other code
3. **Iteration**: Quickly modify prompts based on real-world performance
4. **Transparency**: Know exactly what instructions your agent is working with
5. **Role Hacking**: take advantage of APIs that support nonstandard usage of user/assistant roles - for example, the now-deprecated non-chat flavor of OpenAI "completions" API. This includes some so-called "model gaslighting" techniques

1. **Full Control**：精準寫出你的 agent 真正需要的指示，不必受 black box abstraction 限制
2. **Testing and Evals**：像對待其他程式碼一樣，為 prompts 建立 tests 與 evals
3. **Iteration**：根據真實世界的表現快速修改 prompts
4. **Transparency**：清楚知道你的 agent 實際在依循哪些指示
5. **Role Hacking**：善用支援非標準 user/assistant roles 用法的 APIs，例如現在已淘汰的 OpenAI 非 chat 版 `completions` API；這也包含某些所謂的「model gaslighting」技巧

Remember: Your prompts are the primary interface between your application logic and the LLM.

請記住：你的 prompts 是 application logic 與 LLM 之間最主要的介面。

Having full control over your prompts gives you the flexibility and prompt control you need for production-grade agents.

完整掌握 prompts，能讓你擁有打造 production-grade agents 所需的彈性與 prompt 控制能力。

I don't know what's the best prompt, but I know you want the flexibility to be able to try EVERYTHING.

我不知道什麼才是最好的 prompt，但我知道你一定希望自己擁有可以嘗試「所有方法」的彈性。

[← Natural Language To Tool Calls](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-01-natural-language-to-tool-calls.md) | [Own Your Context Window →](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)
[← 自然語言轉 tool calls](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-01-natural-language-to-tool-calls.md) | [掌握你的 context window →](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)
