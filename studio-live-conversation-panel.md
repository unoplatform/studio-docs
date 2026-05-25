---
uid: Uno.StudioLive.ConversationPanel
---

# Conversation Panel & AI Agent

[!INCLUDE [preview banner](includes/sl-important-info.md)]

The conversation panel is the primary way to interact with the AI agent in Studio Live. The agent understands natural language and translates your intent into working Uno Platform code.

## Sending a Message

Type your request in the text box at the bottom of the conversation panel and press **Enter** (or click the send button). The agent responds with an explanation of what it is doing and applies changes to your project.

Examples of effective prompts:

- *"Add a login page with email and password fields."*
- *"Change the primary color to blue and use the Material theme."*
- *"Show a list of items loaded from a REST API."*

## Slash Commands

Slash commands give you direct control over agent behavior without writing a full sentence.

| Command | Description |
|---|---|
| `/undo` | Revert the last set of changes made by the agent. |
| `/reset` | Clear the conversation and start fresh. |
| `/export` | Download the current project as a `.zip`. |

> [!NOTE]
> The list of available slash commands may grow over time. Type `/` in the conversation panel to see the current list.

## Attaching Files

You can drag and drop files (images, sketches, screenshots) onto the conversation panel. The agent uses them as visual context for the next message.

## Tips for Better Results

- Be specific about the UI element you want to change — name the control, page, or section.
- Include the expected behavior, not just the visual appearance (*"when the user taps the button, navigate to the Detail page"*).
- If the result is not what you expected, follow up with a correction rather than starting over.

## Related

- <xref:Uno.StudioLive.Overview>
- <xref:Uno.StudioLive.GetStarted>
