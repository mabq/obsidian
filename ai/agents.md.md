## What is an agent?

> [!Summary]
> Agent = Model + Harness

The _model_ is the underlying LLM being used by the agent, e.g. "Claude Open 5.5" from Anthropic — being one of the components of the agent you can change it anytime.

LLMs can only take text input and produce text output, they cannot act on their environment.

_Harness_ is the term used to refer to the capabilities agents have to interact with their environment and perform tasks autonomously, these can be divided into the following categories:

1. Tools, MCP (Model Context Protocol) and Skills for actions
2. RAG, Context and Memory for knowledge
3. Planning and Reflection for reasoning

### Tools

Allow agents to interact with their environment.

The LLM never directly executes the tool, it only generates the instruction to call it.

Each tool has two essential parts:

  - Metadata (names, descriptions, and schemas) —  what the LLM sees and uses to decide when to call the tool. Because the LLM only sees the metadata about the tool and must understand exactly what format and information to provide. Unclear parameters force the LLM to guess, leading to failures. For example, a parameter named `city` should specify whether it expects a zip code, city name only, or 'City, State' format. Most bugs in tools stem from unclear wording in the tool definitions.
  - Functions — what the automated system actually executes.

Some LLMs have already been trained for tool use (see [models.dev](https://models.dev/)), the main advantage of using LLMs already trained for tool use is their higher reliability in selecting and calling the correct tools, but even LLMs not trained to use tools can use them with proper prompting — you can instruct the LLM in the system prompt to output tool calls in a specific format (for example, `TOOL` followed by the function name and parameters), then use regex to parse that output and execute the tool.

### Model Context Protocol (MCP)

MCP is a standardization for connecting tools and APIs to LLMs — MCP standardizes the transportation and format, not the tool itself, so it will work with any client; e.g. Claude, Copilot, etc. 

It solves the problem of needing a consistent way to connect various tools and APIs to language models, similar to how HTTPS standardized internet communication. It provides a standard protocol for communication between MCP servers (which provide tools) and MCP clients (which communicate with servers to execute tools).  Information comes back to the MCP client and fed back to our LLM.


When companies ship MCP servers, maintaining the connection becomes their responsibility. If an API changes, the company updates their MCP server rather than requiring millions of individual developers to fix their integrations. This centralized maintenance eliminates the need for countless software engineers to update their connections when APIs change.

Created by Anthropic in 2024, it is now part of the [Agentic AI Foundation](https://aaif.io/) which is part of the Linux Foundation.

### Skills

Skills give instructions to your agent — like preferences on how to operate.

Anatomy — folder with required `SKILL.md` and optional sub-folders.

Complements MCP, doesn’t replace it.

Bundling deterministic scripts (such as for form validation or format conversion) ensures that tasks are executed correctly and consistently every time, rather than relying on the model to guess at how to perform them. When the agent runs one of these scripts, only the output enters the context, providing reliable results while managing context usage efficiently.

Skills protect context through selective loading. The skill metadata is always loaded into context (roughly 100 tokens per skill). The instruction layer (recommended under 5,000 tokens) is only loaded when the skill is called. Bundled information can be nested layers deep but remains free until actually needed. Even when a skill is active, only relevant bundled information is loaded — for example, frontend information wouldn't load backend bundled data.

Warning! — skills can contain executable code, so only load skills from trusted sources. When skills bundle executable scripts for deterministic tasks like form validation or format conversion, the agent runs the script and only the output enters the context. This ensures deterministic, correct results every time without requiring the model to attempt these tasks, while minimizing context usage by storing only the result rather than the entire execution process.

Created by Anthropic, skills are now an open standard and used industry wide.

---

Sites like [claude.ai](claude.ai) or [grok.com](grok.com) are actually agents — they can reach out to the world and perform many actions that are not part of the underlying LLM.



## Workflow types

- Basic LLM integration into code
- Single-agent workflows
- Multi-agent workflows

Each type offers different trade-offs in terms of context (the less we need the better), reliability, and cost.
