> **🍎 This is an unofficial security-hardened fork of goose** ([SMUGSterling/goose-pony](https://github.com/SMUGSterling/goose-pony)).
> Key differences from upstream: **Manual Approval mode is on by default** (every tool call requires confirmation), and **prompt injection detection is enabled by default**.
> For unattended/headless/subagent use, you must explicitly set `GOOSE_MODE=auto`. See the [headless/subagent tradeoff note](#headless--subagent-use) below.

<div align="center">

# goose-pony

_security-hardened fork of goose — your native open source AI agent_

<p align="center">
  <a href="https://opensource.org/licenses/Apache-2.0"
    ><img src="https://img.shields.io/badge/License-Apache_2.0-blue.svg"></a>
  <a href="https://discord.gg/goose-oss"
    ><img src="https://img.shields.io/discord/1287729918100246654?logo=discord&logoColor=white&label=Join+Us&color=blueviolet" alt="Discord"></a>
  <a href="https://github.com/SMUGSterling/goose-pony/actions/workflows/ci.yml"
     ><img src="https://img.shields.io/github/actions/workflow/status/SMUGSterling/goose-pony/ci.yml?branch=main" alt="CI"></a>
</p>
</div>

goose is a general-purpose AI agent that runs on your machine. Not just for code — use it for research, writing, automation, data analysis, or anything you need to get done.

A native desktop app for macOS, Linux, and Windows. A full CLI for terminal workflows. An API to embed it anywhere. Built in Rust for performance and portability.

goose works with 15+ providers — Anthropic, OpenAI, Google, Ollama, OpenRouter, Azure, Bedrock, and more. Use API keys or your existing Claude, ChatGPT, or Gemini subscriptions via [ACP](https://goose-docs.ai/docs/guides/acp-providers). Connect to 70+ extensions via the [Model Context Protocol](https://modelcontextprotocol.io/) open standard.

goose is part of the [Agentic AI Foundation (AAIF)](https://aaif.io/) at the Linux Foundation.

# Get started

**[Download the desktop app](https://goose-docs.ai/docs/getting-started/installation)** for macOS, Linux, and Windows.

Or install this fork's CLI:

```bash
curl -fsSL https://github.com/SMUGSterling/goose-pony/releases/download/stable/download_cli.sh | bash
```

# Quick links
- [Quickstart](https://goose-docs.ai/docs/quickstart)
- [Installation](https://goose-docs.ai/docs/getting-started/installation)
- [Tutorials](https://goose-docs.ai/docs/category/tutorials)
- [Documentation](https://goose-docs.ai/docs/category/getting-started)
- [Governance](https://github.com/SMUGSterling/goose-pony/blob/main/GOVERNANCE.md)
- [Custom Distributions](https://github.com/SMUGSterling/goose-pony/blob/main/CUSTOM_DISTROS.md) — build your own goose distro with preconfigured providers, extensions, and branding

## Headless / subagent use

This fork defaults to **Manual Approval mode** (`GOOSE_MODE=approve`), which means goose will pause and wait for your confirmation before every tool call. This is intentional for interactive use but **breaks unattended headless and subagent workflows** that expect goose to run without user input.

For CI/CD pipelines, cron jobs, or any scenario where there is no human at the keyboard, set:

```bash
export GOOSE_MODE=auto
```

The same applies to subagent use: subagents are only launched automatically in Autonomous mode. In approve mode (the default in this fork), you must explicitly request them or switch to auto mode first.

## Need help?
- [Diagnostics & Reporting](https://goose-docs.ai/docs/troubleshooting/diagnostics-and-reporting)
- [Known Issues](https://goose-docs.ai/docs/troubleshooting/known-issues)

# a little goose humor 🪿

> Why did the developer choose goose as their AI agent?
> 
> Because it always helps them "migrate" their code to production! 🚀

# goose around with us
- [Discord](https://discord.gg/goose-oss)
- [YouTube](https://www.youtube.com/@goose-oss)
- [LinkedIn](https://www.linkedin.com/company/goose-oss)
- [Twitter/X](https://x.com/goose_oss)
