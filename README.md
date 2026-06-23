# moss-langchain

Cryptographic signing for LangChain AI agent actions using ML-DSA-44 post-quantum signatures.

[![PyPI](https://img.shields.io/pypi/v/moss-langchain)](https://pypi.org/project/moss-langchain/)
[![License](https://img.shields.io/badge/license-BSL--1.1-blue)](LICENSE)

## Overview

moss-langchain integrates MOSS cryptographic signing into your LangChain workflows. Every tool call, chain output, and AI message gets a tamper-evident signature using ML-DSA-44 (NIST FIPS 204), the post-quantum cryptographic standard. This creates an immutable audit trail for compliance, debugging, and accountability.

## Installation

```bash
pip install moss-langchain
```

## Quick Start

```python
from langchain_openai import ChatOpenAI
from moss_langchain import sign_tool_call, sign_chain_result

# Sign a tool call
tool_call = {"name": "get_weather", "args": {"location": "NYC"}, "id": "call_123"}
result = sign_tool_call(tool_call, agent_id="weather-bot")
print(f"Signed: {result.signature[:20]}...")

# Sign a chain result
chain = ChatOpenAI() | StrOutputParser()
output = chain.invoke("What is 2+2?")
result = sign_chain_result(output, agent_id="math-bot", chain_name="calculator")
```

## Features

- **ML-DSA-44 signatures** - Post-quantum cryptographic standard (NIST FIPS 204)
- **Tool call signing** - Sign every LangChain tool invocation
- **Chain result signing** - Sign outputs from any chain or pipeline
- **Message signing** - Sign individual AI messages
- **Callback handler** - Auto-sign all chain events with `MOSSCallbackHandler`
- **Policy enforcement** - Block high-risk actions with enterprise policies
- **Offline verification** - Verify signatures without network access

## Usage Examples

### Basic Usage

```python
from moss_langchain import sign_tool_call, sign_message, verify_envelope
from langchain_core.messages import AIMessage

# Sign a tool call
result = sign_tool_call(
    {"name": "send_email", "args": {"to": "user@example.com"}, "id": "call_1"},
    agent_id="email-bot"
)

# Sign a message
message = AIMessage(content="The answer is 4")
result = sign_message(message, agent_id="math-bot")

# Verify any envelope
verify_result = verify_envelope(result.envelope)
print(f"Valid: {verify_result.valid}, Subject: {verify_result.subject}")
```

### With Policy Enforcement

```python
import os
os.environ["MOSS_API_KEY"] = "your-api-key"

from moss_langchain import sign_tool_call

result = sign_tool_call(
    {"name": "transfer_funds", "args": {"amount": 50000}, "id": "call_1"},
    agent_id="finance-bot",
    context={"user_id": "u123", "department": "finance"}
)

if result.blocked:
    print(f"Blocked by policy: {result.policy.reason}")
```

### Callback Handler

```python
from moss_langchain import MOSSCallbackHandler
from langchain_openai import ChatOpenAI

handler = MOSSCallbackHandler(
    agent_id="my-agent",
    sign_tools=True,
    sign_chains=True
)

chain = ChatOpenAI() | StrOutputParser()
result = chain.invoke("Hello", config={"callbacks": [handler]})

# Access all signed envelopes
for envelope in handler.envelopes:
    print(f"Signed: {envelope.subject}")
```

## API Reference

| Function | Description |
|----------|-------------|
| `sign_tool_call()` | Sign a LangChain tool call |
| `sign_chain_result()` | Sign chain output |
| `sign_message()` | Sign an AI message |
| `sign_tool_result()` | Sign tool execution result |
| `sign_output()` | Sign any output (generic) |
| `verify_envelope()` | Verify a signed envelope |
| `enterprise_enabled()` | Check if enterprise mode is active |
| `MOSSCallbackHandler` | Auto-signing callback handler |

## Configuration

| Environment Variable | Description |
|---------------------|-------------|
| `MOSS_API_KEY` | API key for enterprise features (policy enforcement, SIEM) |
| `MOSS_API_URL` | Custom API endpoint (default: api.mosscomputing.com) |

## Links

- [Documentation](https://docs.mosscomputing.com/sdks/langchain)
- [Dashboard](https://app.mosscomputing.com)
- [PyPI](https://pypi.org/project/moss-langchain/)

## License

Business Source License 1.1 - Production use requires a [MOSS subscription](https://mosscomputing.com/pricing).
