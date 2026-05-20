[← Back to README](https://github.com/RobbinHsu/12-factor-agents/blob/main/README.md)

### 10. Small, Focused Agents
### 10. 小而聚焦的 Agents

Rather than building monolithic agents that try to do everything, build small, focused agents that do one thing well. Agents are just one building block in a larger, mostly deterministic system.

與其打造什麼都想做的 monolithic agents，不如建立小而聚焦、把單一事情做好的 agents。Agents 只是更大、而且大多為 deterministic 的系統中的一種 building block。

![1a0-small-focused-agents](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/1a0-small-focused-agents.png)

The key insight here is about LLM limitations: the bigger and more complex a task is, the more steps it will take, which means a longer context window. As context grows, LLMs are more likely to get lost or lose focus. By keeping agents focused on specific domains with 3-10, maybe 20 steps max, we keep context windows manageable and LLM performance high.

這裡的關鍵洞見，在於 LLM 的限制：任務越大、越複雜，需要的步驟就越多，也就意味著更長的 context window。隨著 context 增長，LLMs 更容易迷失或失焦。把 agents 限制在特定領域內，控制在 3-10 步、最多也許 20 步之內，就能讓 context windows 保持可控，並維持較高的 LLM 表現。

> #### As context grows, LLMs are more likely to get lost or lose focus
>
> #### 隨著 context 增長，LLMs 更容易迷失或失焦

Benefits of small, focused agents:

小而聚焦的 agents 的優點：

1. **Manageable Context**: Smaller context windows mean better LLM performance
1. **Manageable Context**：較小的 context windows 代表更好的 LLM 表現
2. **Clear Responsibilities**: Each agent has a well-defined scope and purpose
2. **Clear Responsibilities**：每個 agent 都有清楚定義的 scope 與 purpose
3. **Better Reliability**: Less chance of getting lost in complex workflows
3. **Better Reliability**：在複雜 workflows 中迷失的機率更低
4. **Easier Testing**: Simpler to test and validate specific functionality
4. **Easier Testing**：更容易針對特定功能進行測試與驗證
5. **Improved Debugging**: Easier to identify and fix issues when they occur
5. **Improved Debugging**：出現問題時更容易定位與修復


### What if LLMs get smarter? 
### 如果 LLMs 變得更聰明呢？

Do we still need this if LLMs get smart enough to handle 100-step+ workflows?

如果 LLMs 已經聰明到足以處理 100 步以上的 workflows，我們還需要這樣做嗎？

tl;dr yes. As agents and LLMs improve, they **might** naturally expand to be able to handle longer context windows. This means handling MORE of a larger DAG. This small, focused approach ensures you can get results TODAY, while preparing you to slowly expand agent scope as LLM context windows become more reliable. (If you've refactored large deterministic code bases before, you may be nodding your head right now).

簡短來說：需要。隨著 agents 和 LLMs 進步，它們**也許**自然會擴展到能處理更長的 context windows，這代表它們能處理更大 DAG 中的更多部分。這種小而聚焦的方法，能確保你**今天**就得到成果，同時也為未來逐步擴大 agent scope 做好準備，前提是 LLM context windows 會變得越來越可靠。（如果你以前重構過大型 deterministic code bases，你現在大概正在點頭。）

[![gif](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/1a5-agent-scope-grow.gif)](https://github.com/user-attachments/assets/0cd3f52c-046e-4d5e-bab4-57657157c82f
)

<details>
<summary><a href="https://github.com/RobbinHsu/12-factor-agents/blob/main/img/1a5-agent-scope-grow.gif">GIF Version</a></summary>
![gif](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/1a5-agent-scope-grow.gif)
</details>

Being intentional about size/scope of agents, and only growing in ways that allow you to maintain quality, is key here. As the [team that built NotebookLM put it](https://open.substack.com/pub/swyx/p/notebooklm?selection=08e1187c-cfee-4c63-93c9-71216640a5f8&utm_campaign=post-share-selection&utm_medium=web):

有意識地控制 agents 的 size / scope，並且只用能維持品質的方式去擴張，是這裡的關鍵。正如打造 NotebookLM 的[團隊所說](https://open.substack.com/pub/swyx/p/notebooklm?selection=08e1187c-cfee-4c63-93c9-71216640a5f8&utm_campaign=post-share-selection&utm_medium=web)：

> I feel like consistently, the most magical moments out of AI building come about for me when I'm really, really, really just close to the edge of the model capability
>
> 我一直覺得，在 AI building 裡最神奇的時刻，往往發生在我非常、非常、非常接近 model capability 邊界的時候

Regardless of where that boundary is, if you can find that boundary and get it right consistently, you'll be building magical experiences. There are many moats to be built here, but as usual, they take some engineering rigor.

不管那條邊界在哪裡，只要你能找到它，並且持續穩定地把它用對，你就能打造出神奇的體驗。這裡有很多護城河可以建立，但一如往常，它們都需要紮實的 engineering rigor。

[← Compact Errors](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-09-compact-errors.md) | [Trigger From Anywhere →](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-11-trigger-from-anywhere.md)
