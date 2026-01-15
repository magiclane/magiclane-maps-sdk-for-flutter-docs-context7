---
description: Documentation for Development With Ai
title: Development With Ai
---

# Development with AI

Use the Magic Lane SDK for Flutter with AI-powered tools through the [Context7 MCP server](https://context7.com/magiclane/magiclane-maps-sdk-for-flutter-docs-context7). This guide shows how to use GitHub Copilot in Visual Studio Code to streamline development.

**What you need:**

- Latest version of Visual Studio Code

- [GitHub Copilot extension](https://code.visualstudio.com/docs/copilot/overview) enabled

- GitHub account sign-in

---

## Step 1: Install the Context7 MCP server

Open the **Extensions** tab in Visual Studio Code and search for the [Context7 MCP Server](vscode:extension/Upstash.context7-mcp). Click **Install**.

Verify the Context7 MCP Server appears under **MCP Servers – Installed** in the Extensions tab. Right-click on the Context7 MCP Server and select **Start Server**.

---

## Step 2: Configure the Context7 MCP server

In Copilot Chat, click **Configure Tools...**. Select the `MCP Server: Context7` option from the available tools.

Set the chat to **Agent mode**. Your first prompt should include a variation of this text:
```
Always use context7 for each request. Do never assume the name of classes or methods.
Use multiple requests to context7 if required.
Check the classes or methods.
Use the Magic Lane Maps SDK for Flutter Documentation docs: https://context7.com/magiclane/magiclane-maps-sdk-for-flutter-docs-context7

[ENTER WHAT YOU NEED TO DO HERE]
```

You can add this prompt to the Custom Instruction file. See the [VS Code docs](https://code.visualstudio.com/docs/copilot/copilot-customization?originUrl=%2Fdocs%2Fcopilot%2Fchat%2Fcopilot-chat-context#_custom-instructions) for details. Similar workflows are available for other tools.

Grant access to the **Context7 MCP Server** when prompted. The LLM will have full access to the Magic Lane SDK documentation.

Include the `use context7` instruction in all prompts if the response includes non-existing classes or members.

**Model recommendations:**

- **GPT-5 mini (preview)** - consistently delivers strong results

- **GPT-4.1** - produces satisfactory results with occasional errors

Experiment with different prompts and models to achieve the best results.

---

## Use with other tools

The Context7 MCP server works with other tools:

- Cursor

- Claude Desktop

- Windsurf
