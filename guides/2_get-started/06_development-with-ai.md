---
description: Documentation for Development With Ai
title: Development With Ai
---

# Development with AI

The Magic Lane SDK for Flutter includes specialized documentation designed for seamless integration with Large Language Models (LLMs) via the [Context7 MCP server](https://context7.com/magiclane/magiclane-maps-sdk-for-flutter-docs-context7). This setup provides LLM-powered tools, such as GitHub Copilot, with direct access to SDK context and documentation, enabling developers to build AI-powered applications more efficiently.

This guide demonstrates how to use the Magic Lane SDK with GitHub Copilot in Visual Studio Code to streamline development and enhance productivity.

## Prerequisites

Ensure that you have the latest version of **Visual Studio Code** installed and the [GitHub Copilot extension](https://code.visualstudio.com/docs/copilot/overview) enabled. Additionally, make sure you are signed in to your **GitHub account** to access Copilot features.

## Install the Context7 MCP server

Open the **Extensions** tab in Visual Studio Code and search for the [Context7 MCP Server](vscode:extension/Upstash.context7-mcp). Click **Install**.  

Once installed, verify that the Context7 MCP Server appears under the **MCP Servers – Installed** section in the **Extensions** tab. Right-click on the Context7 MCP Server and select **Start Server** to activate it.

## Configure the Context7 MCP server

Within the Copilot Chat, click the **Configure Tools...** button. A pop-up will appear showing the available tools. Make sure the `MCP Server: Context7` option is selected.

Ensure that the chat is set to **Agent mode**. Your first prompt should include a variation of the following text:
```
Always use context7 for each request. Do never assume the name of classes or methods.
Use multiple requests to context7 if required.
Check the classes or methods.
Use the Magic Lane Maps SDK for Flutter Documentation docs: https://context7.com/magiclane/magiclane-maps-sdk-for-flutter-docs-context7

[ENTER WHAT YOU NEED TO DO HERE]
```

This prompt can also be used within the Custom Instruction file. More details can be found in the [VS Code docs](https://code.visualstudio.com/docs/copilot/copilot-customization?originUrl=%2Fdocs%2Fcopilot%2Fchat%2Fcopilot-chat-context#_custom-instructions).
Similar workflows are available for other tools.

Grant access to the **Context7 MCP Server** when prompted. Once access is enabled, the LLM will have full access to the **Magic Lane SDK documentation** and can assist you effectively in writing code.

To ensure the LLM utilizes the documentation provided by the Context7 MCP Server, include the `use context7` instruction in all prompts if the response includes non-existing classes or members.

The quality of the generated code will depend on both the complexity of the task and the LLM model used. Experimenting with different prompts and models is recommended to achieve the best results.  

- **GPT-5 mini (preview)**: consistently delivers strong results.  

- **GPT-4.1**: produces satisfactory results for most tasks, with some errors.

## Other tools

The Context7 MCP server can also be used with other tools, including:

- Cursor

- Claude Desktop

- Widsurf
