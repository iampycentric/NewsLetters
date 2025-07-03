# From Code to Conversation: How to Make Your API Talk with Large Language Models 🗣️

![Abstract image representing API integration with Large Language Models, showing code, speech bubbles, and AI symbols.](https://placehold.co/1200x675/1a202c/63b3ed?text=API+and+LLM+Integration)

In the rapidly evolving landscape of software development, the ability to interact with applications through natural language is becoming a game-changer. Imagine being able to "talk" to your API, asking it to perform tasks in plain English. This isn't science fiction; it's a practical reality made possible by integrating Large Language Models (LLMs) with your existing services.

This article will guide you through the process of connecting an LLM to a FastAPI server, turning your structured API into a conversational interface. We'll be using the powerful `pydantic-ai` library alongside `FastMCP` to make this connection seamless.

---

## Why Connect an LLM to Your API? 🤔

The primary advantage is **accessibility**. Instead of needing to understand complex API endpoints and request structures, users (or other developers) can interact with your service using simple, intuitive language. This can:

* **Simplify complex workflows:** Chain multiple API calls into a single natural language command.
* **Lower the barrier to entry:** Allow non-technical users to leverage your API's capabilities.
* **Create innovative user experiences:** Build chatbots, voice-activated commands, and intelligent assistants on top of your existing infrastructure.

---

## The Tools of the Trade 🛠️

* **FastAPI:** A modern, fast (high-performance) web framework for building APIs with Python.
* **pydantic-ai:** A library that simplifies the integration of LLMs into Python applications, particularly those using Pydantic models.
* **FastMCP (Fast Multi-Communication Protocol):** A tool that allows for seamless communication between different services, in our case, between the LLM and our FastAPI application.

---

## Installation

```
uv add fastapi pydantic-ai fastmcp
```

---

## A Practical Example: Step-by-Step

Let's break down the code for connecting an LLM to a local FastAPI server.

### Step 1: Setting up the FastMCP Client

The first step is to make our application aware of the API we want to control. We do this by creating a `FastMCP` instance from the API's OpenAPI specification.

```python
import json
import httpx
from fastmcp import FastMCP


# An asynchronous HTTP client to communicate with our FastAPI server
client = httpx.AsyncClient(base_url="http://localhost:8000")

# Load the OpenAPI spec which describes our API's endpoints
with open("mcp_server/openapi_spec.json") as f:
    openapi_spec = json.load(f)

# Create a FastMCP application instance from the OpenAPI spec.
# This allows FastMCP to act as a client to our API.
# - openapi_spec: The loaded OpenAPI specification.
# - client: The httpx client used to communicate with the API.
# - name: A descriptive name for this FastMCP application.
mcp = FastMCP.from_openapi(
    openapi_spec=openapi_spec, client=client, name="AcademiSource API"
)


if __name__ == "__main__":
    mcp.run(transport="streamable-http", host="127.0.0.1", path="/mcp")
```

In this script, FastMCP introspects the `openapi_spec.json` file to understand all the available API routes, their parameters, and what they do. It then exposes these routes to the LLM.

### Step 2: Connecting the LLM

Now, we create an agent that uses an LLM to interact with the FastMCP server.

```python
import asyncio

from pydantic_ai import Agent
from pydantic_ai.mcp import MCPServerStdio
from pydantic_ai.models.openai import OpenAIModel
from pydantic_ai.providers.openai import OpenAIProvider

local_mcp_server = MCPServerStdio(
    "uv",
    args=["run", "--with", "fastmcp", "fastmcp", "run", "mcp_server/main.py"],
)

instruction = \"\"\"
<system prompt>
\"\"\"

# Running with Gemini
# model = GeminiModel(
#     "gemini-2.5-flash",
#     provider=GoogleGLAProvider(api_key=API_KEY),
# )


# Running a local model using ollama
model = OpenAIModel(
    model_name=<model_that_supports_tools>,
    provider=OpenAIProvider(base_url="http://localhost:11434/v1"),
)

api_agent = Agent(model, instructions=instruction, mcp_servers=[local_mcp_server])


async def main():
    # This context manager handles starting and stopping the server
    async with api_agent.run_mcp_servers():
        # Now that the server is running, you can send requests
        result = await api_agent.run(
            "Hello",
        )
        print(result.output)

        while True:
            user_input = input(">>> ")
            if user_input != "/bye":
                result = await api_agent.run(
                    user_input,
                    message_history=result.new_messages(),
                )
                print(result.output)
            else:
                print("Goodbye")
                break

if __name__ == "__main__":
    asyncio.run(main())
```

### Pre-Requisites to Running Your Script

Start your FastAPI server.

---

## Running Your Agent and MCP

To get started, simply run the agent script from your terminal:

`python agent.py`

Executing this script also initiates the **MCP server**. This makes your API's routes accessible to the Large Language Model. Based on your conversation, the LLM will intelligently decide which API endpoint to call and what parameters to use.

---

## Conclusion

By using **Pydantic-AI** and **FastMCP** together, you can build a powerful bridge connecting Large Language Models to your own APIs. This framework simplifies development by managing the server lifecycle, which lets you concentrate on creating conversational applications. The next time you work on a FastAPI project, think about giving it a voice.

---

## References

* **FastMCP:** [https://gofastmcp.com/servers/openapi](https://gofastmcp.com/servers/openapi)
* **Pydantic-AI:** [https://ai.pydantic.dev/api/mcp/#pydantic_ai.mcp.MCPServerStreamableHTTP](https://ai.pydantic.dev/api/mcp/#pydantic_ai.mcp.MCPServerStreamableHTTP)
* **FastAPI:** [https://fastapi.tiangolo.com/](https://fastapi.tiangolo.com/)
"""
