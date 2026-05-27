---
uid: Uno.PlatformStudio.GetStarted
---

# Get Started with Uno Platform Studio

This guide walks you through setting up Uno Platform Studio and running your first app. If you are new to Uno Platform, start with the Uno Platform getting-started content, then return here to continue with Uno Platform Studio.

## First Launch

The **Projects** page is the main entry point for starting work.

Use **New** to create a new app with a prompt.

- Enter a prompt describing the app you want to build, then submit the prompt.
- You can see the process as it happens in the Agent activity panel and view the app as it builds thanks to Hot Reload.
- When the app finishes building, you will see a recap summarizing what was generated.

<!-- TODO: Add screenshots — Projects page (New tab), Agent activity panel during scaffold, Post-build recap panel. -->

### Gallery

Uno Platform Studio also offers a curated set of samples of complete, working Uno Platform applications you can open, explore, and customize immediately.

When you open a sample, Uno Platform Studio:

1. Clones the sample project into a fresh workspace.
2. Builds and starts the live preview.
3. Opens the conversation panel so you can immediately start customizing.

The sample is your own copy — any changes you make do not affect the original.

<!-- TODO: Add screenshot — Projects page with Gallery tab showing available Uno sample templates. -->

## Conversation Panel & AI Agent

The conversation panel is the primary way to interact with the AI agent in Uno Platform Studio. The agent understands natural language and translates your intent into working Uno Platform code.

Enter your prompt in the text box at the bottom of the conversation panel and press **Enter** (or click the send button). The agent responds with an explanation of what it is doing and applies changes to your project.

You can also add image files (PNG or JPEG) into the prompt panel to give the agent additional context.

### Prompting Best Practices

Prompt quality strongly affects the app quality. Better prompts lead to better first results, fewer regeneration loops, and less cleanup after the app is created.

Good prompts:

- make the app goal explicit,
- describe the primary user,
- name the first few screens,
- define the key interactions, and
- constrain the visual direction enough to avoid generic output.

Strong prompts help the model make better decisions about navigation, data shape, layout hierarchy, and the amount of functionality to include in the first pass.

### Examples

#### Better starting prompt

"Create a project planning app for small teams. Include a dashboard, a project list, and a task detail page. The dashboard should show active projects, overdue tasks, and team progress. Use a clean enterprise layout with clear hierarchy and simple status badges."

#### Better refinement prompt

"Keep the same structure, but make the dashboard denser, add a left rail with Projects, Tasks, and Reports, and show task status with color-coded chips. Do not add new screens."

#### Better editing prompt

"On the task detail page, add editable priority, due date, and assignee fields. Keep the layout responsive and preserve the existing navigation structure."

These examples work because they tell the model what to preserve, what to change, and what to avoid.

## Export and IDE Handoff

Export lets you move from Uno Platform Studio to your day-to-day IDE workflow without losing project fidelity. In addition to export, Uno Platform Studio is expected to support more direct cross-shell handoff for moving work between Web and Desktop.

### Recommended Handoff Flow

1. Complete a scaffold or design iteration in Uno Platform Studio.
2. Export the app from Uno Platform Studio.
3. Open the exported project in your preferred IDE.
4. Restore dependencies and run a local build.
5. Commit or continue development in your normal source workflow.
