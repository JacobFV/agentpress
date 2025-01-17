# AgentPress: Building Blocks for AI Agents

AgentPress is a collection of _simple, but powerful_ utilities that serve as building blocks for creating AI agents. *Plug, play, and customize.*

- **Threads**: Simple message thread handling utilities
- **Tools**: Flexible tool definition and automatic execution
- **State Management**: Simple JSON key-value state management
- **LLM Integration**: Provider-agnostic LLM calls via LiteLLM

## Installation & Setup

1. Install the package:
```bash
pip install agentpress
```

2. Initialize AgentPress in your project:
```bash
agentpress init
```
Creates a `agentpress` directory with all the core utilities.
Check out [File Overview](#file-overview) for explanations of the generated util files.

3. If you selected the example agent during initialization:
   - Creates an `agent.py` file with a web development agent example
   - Creates a `tools` directory with example tools:
     - `files_tool.py`: File operations (create/update files, read directory and load into state)
     - `terminal_tool.py`: Terminal command execution
   - Creates a `workspace` directory for the agent to work in


## Quick Start

1. Set up your environment variables (API keys, etc.) in a `.env` file.
- OPENAI_API_KEY, ANTHROPIC_API_KEY, GROQ_API_KEY, etc... Whatever LLM you want to use, we use LiteLLM (https://litellm.ai) (Call 100+ LLMs using the OpenAI Input/Output Format) – set it up in your `.env` file.. Also check out the agentpress/llm.py and modify as needed to support your wanted LLM.

2. Create a calculator_tool.py 
```python
from agentpress.tool import Tool, ToolResult, tool_schema

class CalculatorTool(Tool):
    @tool_schema({
        "name": "add",
        "description": "Add two numbers",
        "parameters": {
            "type": "object",
            "properties": {
                "a": {"type": "number"},
                "b": {"type": "number"}
            },
            "required": ["a", "b"]
        }
    })
    async def add(self, a: float, b: float) -> ToolResult:
        try:
            result = a + b
            return self.success_response(f"The sum is {result}")
        except Exception as e:
            return self.fail_response(f"Failed to add numbers: {str(e)}")
```

3. Use the Thread Manager, create a new thread – or access an existing one. Then Add the Calculator Tool, and run the thread. It will automatically use & execute the python function associated with the tool:
```python
import asyncio
from agentpress.thread_manager import ThreadManager
from calculator_tool import CalculatorTool

async def main():
    # Initialize thread manager and add tools
    manager = ThreadManager()
    manager.add_tool(CalculatorTool)

    # Create a new thread
    # Alternatively, you could use an existing thread_id like:
    # thread_id = "existing-thread-uuid" 
    thread_id = await manager.create_thread()
    
    # Add your custom logic here
    await manager.add_message(thread_id, {
        "role": "user", 
        "content": "What's 2 + 2?"
    })
    
    response = await manager.run_thread(
        thread_id=thread_id,
        system_message={
            "role": "system", 
            "content": "You are a helpful assistant with calculation abilities."
        },
        model_name="gpt-4",
        use_tools=True,
        execute_model_tool_calls=True
    )
    print("Response:", response)

asyncio.run(main())
```

4. Autonomous Web Developer Agent (the standard example)

When you run `agentpress init` and select the example agent – you will get code for a simple implementation of an AI Web Developer Agent that leverages architecture similar to platforms like our own [Softgen](https://softgen.ai/) Platform. 

- **Files Tool**: Allows the agent to create, read, update, and delete files within the workspace.
- **Terminal Tool**: Enables the agent to execute terminal commands.
- **State Workspace Management**: The agent has access to a workspace whose state is stored and sent on every request. This state includes all file contents, ensuring the agent knows what it is editing.
- **User Interaction via CLI**: After each action, the agent pauses and allows the user to provide further instructions through the CLI.

You can find the complete implementation in our [example-agent](agentpress/examples/example-agent/agent.py) directory.

5. Thread Viewer 

Run the thread viewer to view messages of threads in a stylised web UI:
```bash
streamlit run agentpress/thread_viewer_ui.py
```


## File Overview

### agentpress/llm.py
Core LLM API interface using LiteLLM. Supports 100+ LLMs using the OpenAI Input/Output Format. Easy to extend for custom model configurations and API endpoints. `make_llm_api_call()` can be imported to make LLM calls.

### agentpress/thread_manager.py
Orchestrates conversations between users, LLMs, and tools. Manages message history and automatically handles tool execution when LLMs request them. Tools registered here become available for LLM function calls.

### agentpress/tool.py
Base infrastructure for LLM-compatible tools. Inherit from `Tool` class and use `@tool_schema` decorator to create tools that are automatically registered for LLM function calling. Returns standardized `ToolResult` responses.

### agentpress/tool_registry.py
Central registry for tool management. Keeps track of available tools and their schemas, allowing selective function registration. Works with `thread_manager.py` to expose tools to LLMs.

### agentpress/state_manager.py
Simple key-value based state persistence using JSON files. For maintaining environment state, settings, or other persistent data.


## Philosophy
- **Plug & Play**: Start with our defaults, then customize to your needs.
- **Agnostic**: Built on LiteLLM, supporting any LLM provider. Minimal opinions, maximum flexibility.
- **Simplicity**: Clean, readable code that's easy to understand and modify.
- **No Lock-in**: Take full ownership of the code. Copy what you need directly into your codebase.

## Contributing

We welcome contributions! Feel free to:
- Submit issues for bugs or suggestions
- Fork the repository and send pull requests
- Share how you've used AgentPress in your projects

## Development

1. Clone:
```bash
git clone https://github.com/kortix-ai/agentpress
cd agentpress
```

2. Install dependencies:
```bash
pip install poetry
poetry install
```

3. Build the package:
```bash
poetry build
```
It will return the built package name with the version number.

4. Install the package with the correct version number, here for example its 0.1.3 `agentpress-0.1.3-py3-none-any.whl`:
```bash
pip install /Users/markokraemer/Projects/agentpress/dist/agentpress-0.1.3-py3-none-any.whl --force-reinstall
```
Then you can test that version.


## License

[MIT License](LICENSE)

Built with ❤️ by [Kortix AI Corp](https://kortix.ai)