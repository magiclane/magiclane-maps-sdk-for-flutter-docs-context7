---
description: Use LLM-friendly documentation files and AI coding assistants to accelerate
  development with the Magic Lane Maps SDK for Flutter.
title: Coding With Ai
---

  const { siteConfig } = useDocusaurusContext();
  const baseUrl = siteConfig.baseUrl.replace(/\/$/, '');
  const url = `${siteConfig.url}${baseUrl}/${file}`;
  const display = `${siteConfig.url.replace(/^https?:\/\//, '')}${baseUrl}/${file}`;
  return <a href={url}>{display}</a>;
};

  const { siteConfig } = useDocusaurusContext();
  const baseUrl = siteConfig.baseUrl.replace(/\/$/, '');
  return `${siteConfig.url}${baseUrl}/llms-full.txt`;
};

# Coding with AI

Magic Lane provides machine-readable documentation files that integrate with popular AI coding assistants. Use them to get accurate, context-aware answers about the Maps SDK for Flutter while you code.

## Available resources

Two files are generated with every documentation build and published alongside the docs:

| File | Contents | URL |
|------|----------|-----|
| **llms-full.txt** | Developer guides, explanations, and code examples |  |
| **llms.txt** | Concise API overview and page index |  |

These files follow the [llms.txt](https://llmstxt.org/) standard and contain detailed concept explanations, code examples, documentation references, and source file locations for verification.

## Using with AI coding tools

Download both files and place them in your project. Each tool reads them differently - follow the setup for the one you use.

### Cursor

1. Create a `.cursor` directory in your project root.
2. Place `llms-full.txt` and `llms.txt` inside it.
3. Open **Cursor Settings → Rules** and add a new rule with type **Apply Intelligently**.
4. Set the description to: *Use this rule whenever a question is asked about Magic Lane Maps SDK for Flutter.*
5. Reference the files in the rule body. Cursor will automatically use them when relevant.

### GitHub Copilot

1. Create a `.copilot` directory in your project root.
2. Place `llms-full.txt` and `llms.txt` inside it.
3. In Copilot Chat, reference the files by stating:
   
```
   Please use the context from the llms.txt files in the .copilot directory
   when answering questions about the Magic Lane Maps SDK.
   ```

### JetBrains AI Assistant

1. Create a `.idea/ai` directory in your project root.
2. Place `llms-full.txt` and `llms.txt` inside it.
3. Reference the files in AI Assistant chat when asking about the SDK.

### Windsurf

1. Create a `.windsurf` directory in your project root.
2. Place `llms-full.txt` and `llms.txt` inside it.
3. Reference the files in chat for context-aware responses.

### Claude Desktop / Claude Code

Pass the `llms-full.txt` URL directly in your prompt:

<pre><code>{`Use `}{` as context
for the following question about the Magic Lane Maps SDK for Flutter:

[YOUR QUESTION HERE]`}</code></pre>

## Context7 MCP server

For a more integrated experience, use the [Context7 MCP server](https://context7.com/magiclane/magiclane-maps-sdk-for-flutter-docs-context7) which connects AI tools directly to the Magic Lane documentation.

**Requirements:**

- Latest version of Visual Studio Code

- [GitHub Copilot extension](https://code.visualstudio.com/docs/copilot/overview) enabled

- GitHub account sign-in

### Step 1: Install the Context7 MCP server

Open the **Extensions** tab in Visual Studio Code and search for the [Context7 MCP Server](vscode:extension/Upstash.context7-mcp). Click **Install**.

Verify the Context7 MCP Server appears under **MCP Servers - Installed** in the Extensions tab. Right-click on the Context7 MCP Server and select **Start Server**.

### Step 2: Configure in Copilot Chat

In Copilot Chat, click **Configure Tools...**. Select the `MCP Server: Context7` option from the available tools.

Set the chat to **Agent mode**. Your first prompt should include a variation of this text:
```
Always use context7 for each request. Do never assume the name of classes or methods.
Use multiple requests to context7 if required.
Check the classes or methods.
Use the Magic Lane Maps SDK for Flutter Documentation docs: https://context7.com/magiclane/magiclane-maps-sdk-for-flutter-docs-context7

[ENTER WHAT YOU NEED TO DO HERE]
```

You can add this prompt to the Custom Instruction file. See the [VS Code docs](https://code.visualstudio.com/docs/copilot/copilot-customization?originUrl=%2Fdocs%2Fcopilot%2Fchat%2Fcopilot-chat-context#_custom-instructions) for details.

Grant access to the **Context7 MCP Server** when prompted. The LLM will have full access to the Magic Lane SDK documentation.

Context7 also works with Cursor, Claude Desktop, and Windsurf.

## Tips

- Include `use context7` in your prompts if the AI references classes or methods that don't exist.

- Experiment with different prompts and models to get the best results.

- The `llms-full.txt` file is best for general questions and guides. Use `llms.txt` for a quick overview of available pages.

- Always verify generated code against the official [API reference](/docs/flutter/api-reference/).
