[← Back to README](https://github.com/humanlayer/12-factor-agents/blob/main/README.md)

### 11. Trigger from anywhere, meet users where they are
### 11. 從任何地方觸發，在使用者所在之處與其互動

If you're waiting for the [humanlayer](https://humanlayer.dev) pitch, you made it. If you're doing [factor 6 - launch/pause/resume with simple APIs](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md) and [factor 7 - contact humans with tool calls](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md), you're ready to incorporate this factor.

如果你一直在等 [humanlayer](https://humanlayer.dev) 的宣傳段落，那你等到了。如果你已經在做 [factor 6 - launch/pause/resume with simple APIs](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md) 和 [factor 7 - contact humans with tool calls](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md)，那你就已經準備好納入這個 factor 了。

![1b0-trigger-from-anywhere](https://github.com/humanlayer/12-factor-agents/blob/main/img/1b0-trigger-from-anywhere.png)

Enable users to trigger agents from slack, email, sms, or whatever other channel they want. Enable agents to respond via the same channels.

讓使用者能從 slack、email、sms 或任何他們想用的 channel 觸發 agents，也讓 agents 能透過同樣的 channels 回應。

Benefits:

優點：

- **Meet users where they are**: This helps you build AI applications that feel like real humans, or at the very least, digital coworkers
- **Outer Loop Agents**: Enable agents to be triggered by non-humans, e.g. events, crons, outages, whatever else. They may work for 5, 20, 90 minutes, but when they get to a critical point, they can contact a human for help, feedback, or approval
- **High Stakes Tools**: If you're able to quickly loop in a variety of humans, you can give agents access to higher stakes operations like sending external emails, updating production data and more. Maintaining clear standards gets you auditability and confidence in agents that [perform bigger better things](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md#what-if-llms-get-smarter)

- **Meet users where they are**：在使用者所在之處與他們互動；這能幫助你打造更像真人、至少也像數位同事的 AI applications
- **Outer Loop Agents**：讓 agents 能被非人類觸發，例如 events、crons、outages 或其他來源。它們可能工作 5、20、90 分鐘，但在到達關鍵節點時，可以聯繫人類尋求協助、回饋或核准
- **High Stakes Tools**：如果你能快速把不同的人類拉進流程，就可以讓 agents 取得更高風險的操作權限，例如發送外部 email、更新 production data 等等。維持清楚的標準，能讓你對那些[執行更重大且更有價值工作的](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md#what-if-llms-get-smarter) agents 具備 auditability 與信心

[← Small Focused Agents](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md) | [Stateless Reducer →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-12-stateless-reducer.md)
