# Agent = Model + Harness

## Model

The underlying LLM being used by the agent, for example:
- Claude Opus 5.5 (from Anthropic)
- GPT-6.1 Terra (from OpenAI)
- DeepSeek V4.1 Flash (from Deepseek)

Visit [models.dev](https://models.dev/) to see available models, providers and labs — there you can review the model's:

- Capabilities (usually yes/no flags):
  - Attachment: Whether the model supports file attachments. 
  - github
  - Reasoning: Whether it has reasoning or “thinking” abilities.
  - Tool Call: Whether it supports function/tool calling.
  - Structured Output: Whether it can reliably return output in a specified format such as JSON.
  - Temperature: Whether the temperature sampling parameter is adjustable.
  - Open Weights: Whether the model’s weights are publicly available, as opposed to API-only.

- Limits:
  - Context: The maximum context window, in tokens.
  - Input / Output limits: The maximum tokens accepted in a request and generated in a response.

- Modalities:
  - Input / Output modalities: The data types the model accepts and produces (text, image, audio, video, PDF).

## Harness

_Harness_ is the ability of the agent to perceive the environment and act on it (search the web, query a database, read/create/edit files, run commands, etc.).


