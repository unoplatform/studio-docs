---
uid: Uno.PlatformStudio.Troubleshooting
---
# Troubleshooting

This page helps you resolve issues you may run into while working in [Uno Platform Studio App](xref:Uno.PlatformStudio.GetStarted).

## Common Messages and Errors

Uno Platform Studio App surfaces status updates, outcomes, and errors directly in the conversation panel and on the shell. This section lists the messages you are most likely to see, explains what triggers each one, and suggests how to respond.

### Generation Outcomes

After each agent turn, the conversation panel shows an outcome card summarizing how the turn ended.

| Card | Subtitle | Meaning | What to do |
| ---- | -------- | ------- | ---------- |
| **Application Ready** | Your changes have been applied. | The agent finished and your app was rebuilt successfully. | Continue iterating or preview the result. |
| **Could not complete request** | Retry or learn more. | The agent ran into a problem it could not resolve in this turn. | Retry from the card, refine your prompt, or open the linked details. |
| **Awaiting your reply** | No new build ran for this response. Reply to continue. | The agent asked a question or finished without producing a build. | Reply in the prompt box to keep going. |
| **Generation stopped** | Retry or send a new message to continue. | You stopped the generation while it was in progress. | Press the retry button on the card, or send a new message. |
| **Critical failure** | Reporting helps us make Uno Platform Studio better. | The turn ended with an unrecoverable error. | Use the **Report** button on the card to send a feedback bundle. |

### In-Progress Status

While the agent is working, the conversation panel shows lightweight status indicators rather than full outcome cards.

| Status | Meaning |
| ------ | ------- |
| **Thinking…** | The agent is reasoning about the next step. |
| **Generation stopped.** | An inline status shown immediately when you cancel a turn. |
| **Build**, **Read**, **Write**, **Patch**, **List files**, **Screenshot**, **Click element**, **Inspect visual tree**, **Generate API client** | The agent is executing the named tool. |

### Conversation Limits

The agent has a fixed context window. When you reach it, Uno Platform Studio App tries to compact the conversation automatically before asking you to take action.

| Message | Meaning | What to do |
| ------- | ------- | ---------- |
| **Context window full. Compacting conversation and retrying…** | The conversation hit the context limit; Studio is summarizing earlier messages. | Wait. The agent will continue automatically. |
| **Summarizing conversation history…** | Compaction is running. | Wait for it to finish. |
| **Conversation compacted. Earlier context has been summarized.** | Compaction succeeded; earlier turns have been condensed. | Continue normally. |
| **Context still too large. Summarizing conversation history…** | A second compaction is being attempted. | Wait. If this also fails, you will see the next message. |
| **Could not summarize the conversation. Try starting a new conversation.** | Compaction failed. | Start a new conversation from the **Projects** page. |
| **The conversation has grown too large. Please start a new conversation.** | The conversation cannot be compacted further. | Start a new conversation. Export the project first if you want to keep the current state. |
| **The conversation reached the maximum number of actions in a single turn. You can continue by sending another message.** | The agent hit the per-turn action limit. | Send a follow-up prompt to continue the work. |

### Credits and Billing

Uno Platform Studio uses a credit-based model. See [Credits & Usage](xref:Uno.PlatformStudio.GetStarted#credits--usage) for the full breakdown.

| Banner | Meaning | What to do |
| ------ | ------- | ---------- |
| **80% of credits used** | You have consumed 80% of your credits for the current period. | Use **Top up** to add credits, or continue working; generation still functions. |
| **Out of credits** | All credits for the current period are consumed. | Use **Upgrade plan** or wait for the period to reset. The banner shows when credits next reset. |

### Sign-In and License

These messages appear on the login page or when restoring a license.

| Message | Meaning | What to do |
| ------- | ------- | ---------- |
| **License required** | You need a valid Uno Platform license to continue. | Sign in with an account that includes Uno Platform Studio, or import a license file. |
| **Your license has expired.** | Your license is no longer valid. | Use **Purchase Now** from the login page. |
| **Your subscription does not include access to Uno Platform Studio. Please ensure your account has the required license.** | You are signed in, but your subscription does not include Studio. | Check your subscription on the Uno Platform account portal. |
| **Sign in was canceled or failed. Please try again.** | The sign-in flow did not complete. | Retry sign-in. If the issue persists, restart the application. |
| **The imported file could not be validated.** | The license file you imported is invalid or unsupported. | Verify you exported the correct user file from a signed-in installation. |
| **Something went wrong while importing.** | A general error occurred while importing your license file. | Retry, or sign in instead of importing. |
| **An unexpected error occurred. Please retry or restart the application.** | A generic error reached the login page. | Use **Retry**. If the issue persists, restart Uno Platform Studio App. |

### Starting a New Project

When you start a new project while a session is active, Uno Platform Studio App confirms before discarding state.

| Dialog | Shown when | Options |
| ------ | ---------- | ------- |
| **Clear current context?** *This will discard the agent's current state and in-progress work.* | Generation is in progress. | **Clear and continue**, or **Export project first**. |
| **Unsaved work in progress.** *Starting a new project will interrupt the current session and clear all unsaved changes.* | The session has unsaved work. | **Clear and continue**, or **Cancel**. |
| **Clear current context?** *This will clear the current chat history and project content.* | The current session is idle but has content. | **Clear and continue**, or **Export project first**. |

> [!TIP]
> If a message recommends starting a new conversation but you want to keep the current app, **Export** it first from the top action bar. See [Export and IDE Handoff](xref:Uno.PlatformStudio.GetStarted#export-and-ide-handoff).

## Reporting an Issue

If you see a **Critical failure** card, or repeatedly hit an error you cannot work around, use **More > Send feedback** to send a feedback bundle. The bundle includes the conversation, logs, and project state needed for the Uno Platform team to investigate.
