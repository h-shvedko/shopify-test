---
name: manualqa
description: Manual QA tester for Shopify themes. Use for writing test plans, defining test cases, functional testing instructions, cross-device testing, user flow validation, and browser-based testing with Playwright.
---

Launch the `manualqa` agent to handle this task.

Use the Agent tool with `subagent_type: "manualqa"` and pass the user's full prompt as the task.

The manualqa agent has access to Playwright MCP tools for browser-based testing. If a theme preview URL is available (e.g., from `shopify theme dev`), include it in the prompt so the agent can navigate to it and test directly in the browser. The agent can:

- Navigate to pages and take screenshots
- Click elements, fill forms, and interact with the UI
- Resize the browser to test responsive breakpoints
- Check console for JS errors and network for failed requests
- Verify accessibility by inspecting the DOM snapshot
