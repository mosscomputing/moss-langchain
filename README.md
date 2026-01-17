# moss-langchain

MOSS signing integration for LangChain. **Unsigned output is broken output.**

## Installation

```bash
pip install moss-langchain
```

## Quick Start: Auto-Signing (Recommended)

The easiest way to use MOSS with LangChain is to enable auto-signing:

```python
from moss_langchain import enable_moss

# Enable auto-signing for all LangChain operations
enable_moss("moss:myteam:langchain-agent")

# All subsequent tool calls, chain outputs, and agent actions are signed automatically
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

chain = ChatPromptTemplate.from_template("Summarize: {text}") | ChatOpenAI()
result = chain.invoke({"text": "Long document..."})  # Output is signed!
```

You can also enable auto-signing via environment variable:

```bash
export MOSS_AUTO_ENABLE=true
```

## Manual Usage with Callback Handler

For more control, use the callback handler directly:

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from moss_langchain import SignedCallbackHandler

# Create callback handler
cb = SignedCallbackHandler("moss:bot:summary")

# Create your chain
chain = ChatPromptTemplate.from_template("Summarize: {text}") | ChatOpenAI()

# Invoke with callback
result = chain.invoke(
    {"text": "Long document..."},
    config={"callbacks": [cb]}
)

# Access the signature
envelope = cb.envelope
```

## Verification

```python
from moss import verify

# Verify the output - no network required
result = verify(cb.envelope)

if result.valid:
    print(f"Signed by: {result.subject}")
else:
    print(f"Invalid: {result.reason}")

# Or use envelope.verify() directly
result = envelope.verify()
assert result.valid
```

## Multiple Outputs

The handler tracks all outputs during a session:

```python
cb = SignedCallbackHandler("moss:bot:pipeline")

# Run multiple operations
chain1.invoke(input1, config={"callbacks": [cb]})
chain2.invoke(input2, config={"callbacks": [cb]})

# Access all envelopes
for envelope in cb.envelopes:
    print(f"Seq {envelope.seq}: {envelope.payload_hash}")

# Clear for next session
cb.clear()
```

## Async Chains

```python
from moss_langchain import AsyncSignedCallbackHandler

cb = AsyncSignedCallbackHandler("moss:bot:async")
result = await chain.ainvoke(input, config={"callbacks": [cb]})
```

## Signed Events

The handler signs outputs from:
- `on_llm_end` - LLM generation complete
- `on_chain_end` - Chain execution complete
- `on_tool_end` - Tool execution complete
- `on_agent_finish` - Agent finished
- `on_retriever_end` - Retriever returned documents
