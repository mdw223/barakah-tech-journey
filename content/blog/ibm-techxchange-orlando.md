# IBM TechXchange Orlando 2025
## Key Learnings & AI Agent Development Guide

---

## Page 1: Core Insights and Industry Trends

### Career Development & Technology Adaptation
**Always learn new technology and stay ahead. Be flexible to change paths.**

The rapidly evolving AI landscape demands continuous learning and adaptability. Success in this field requires staying current with emerging frameworks, tools, and methodologies.

### AI Security
**Security in AI: How to make AI systems resistant to hacking**

As AI systems become more prevalent, securing them against adversarial attacks, prompt injection, and data poisoning becomes critical. This includes implementing proper authentication, input validation, and monitoring systems.

### The Big Opportunity: Infrastructure Integration
**Generative AI Integration into Existing Infrastructure**

The partnership between Anthropic and IBM focuses on this exact challenge: helping enterprises integrate generative AI capabilities into their existing technology stacks without complete overhauls. This represents one of the largest opportunities in the AI space today.

### Tools & Resources
- **Bob in VS Code**: A productivity tool worth exploring
- **IBM Products**: https://www.ibm.com/products

---

## Understanding AI Agents

### Core Definition
**Agent = LLM + Tools**

An AI agent is fundamentally code that combines a large language model with specific tools to process inputs and generate desired outputs in a structured manner.

### Assistant vs Agent

| **Assistant** | **Agent** |
|--------------|----------|
| Handles specific tasks | Achieves goals |
| Answers user prompts | Proactive problem-solving |
| Reactive | Proactive |

### Anatomy of an Agent
The three core functions: **Think → Act → Observe**

**Key Components:**
1. **Perception**: Understanding inputs and environment
2. **Memory**: Storing past interactions and learnings
3. **Tool Calling**: Accessing datasets, APIs, and external resources
4. **Planning**: Using prompts to direct behavior and strategy
5. **Communication**: Interacting with users or other agents
6. **Reasoning**: Making decisions based on perception and memory

---

## IBM Watsonx Orchestrate
A comprehensive platform for building, deploying, and managing AI agents.

**Features:**
- Agent marketplace for pre-built solutions
- **AgentOps**: Tools to manage and monitor agent performance
- Integration with existing workflows

---

## AI Agent Development Lifecycle

### Stages:
1. **Planning**: Define objectives and scope
2. **Design**: Architecture and interaction patterns
3. **Framework, Model & Tool Selection**: Choose appropriate tech stack
4. **Training**: Fine-tune and optimize
5. **Evaluation**: Test performance and accuracy
6. **Deployment & Monitoring**: Production rollout with ongoing observation

---

## Types of AI Agents

### 1. Simple Reflex Agent
- Operates on if-then rules
- No memory or state
- Direct input → output mapping

### 2. Model-Based Reflex Agent
- Maintains internal state for memory
- Still uses if-then logic
- Can track context

### 3. Goal-Based Agent
- Provides multiple solutions to achieve a goal
- Evaluates different paths
- More flexible than reflex agents

### 4. Utility-Based Agent
- Can select the best solution among alternatives
- Optimizes based on utility functions
- More sophisticated decision-making

### 5. Learning Agent
- **Think → Act → Observe** cycle
- Reviews past actions stored in memory
- Continuously improves performance
- Most advanced agent type

**Best Practice**: Try to make the simplest agent that solves your problem. Don't over-engineer.

---

## Agentic Reasoning Strategies

### ReAct (Reasoning + Acting)
- If answer not found, keeps prompting until it does
- Resource-intensive approach
- Requires exit conditions to prevent infinite loops
- Continuous observation and adjustment

### ReWOO (Reasoning Without Observation)
- Doesn't loop back
- More efficient resource usage
- Single-pass reasoning
- Better for straightforward tasks

### Parallelization of Agents
Multiple agents working simultaneously on different aspects of a problem.

### Orchestrator Pattern
- Central agent coordinates multiple specialized agents
- **Synthesizer agent** produces final output
- Good for complex, multi-step workflows

### Evaluator Pattern (Optimizer)
- Two agents working together iteratively
- Back-and-forth refinement
- Continuous improvement cycle

---

## Agent Communication Protocols

Modern agents need standardized ways to communicate:
- **MCP**: Model Context Protocol
- **A2A**: Agent-to-Agent communication
- **ANP**: Agent Networking Protocol
- **AG UI**: Agent User Interface protocols

---

## Agent Development Frameworks

Popular templates and libraries for building agents:
- **LangChain**: Comprehensive chain-based framework
- **LangGraph**: Graph-based agent workflows (used in our code examples)
- **CrewAI**: Multi-agent collaboration
- **BeeAI**: Behavior-driven agents
- **AutoGen**: Multi-agent conversations
- **LlamaIndex**: Data-focused agent framework

---

## Retrieval Augmented Generation (RAG)

### Traditional RAG Flow:
1. User prompt → Server
2. Server searches for top relevant results
3. Server asks model to process selected data
4. Model generates response with retrieved context

### Agentic RAG (Advanced):
1. User prompt → **Aggregation Agent**
2. Aggregation agent creates a plan
3. Works with 3 specialized agents:
   - Internet search agent
   - API agent
   - Database agent
4. Aggregation agent maintains memory
5. Synthesizes results with LLM for final output

**Advantages of Agentic RAG:**
- More sophisticated data retrieval
- Parallel processing capabilities
- Better context management
- Improved result quality

---

## Opportunities & Action Items

### Career Opportunities
**"Jobs to integrate AI and agents to existing code"**

There's massive demand for developers who can:
- Integrate agents into legacy systems
- Build custom agent solutions
- Optimize agent performance
- Ensure security and reliability

### Personal Projects
**Build some agents for Axiom!**

Apply these learnings to real-world projects. Start simple and iterate.

---

## Page 2: Code Implementation - Building AI Agents with LangGraph

### Overview of Code Files
The provided codebase demonstrates building AI agents using multiple approaches:
1. **LangGraph** (Python framework) - `agent.py`
2. **IBM Watsonx Orchestrate** - `agent.yaml`, `price.py`, `income.py`
3. **Langflow** (no-code) - `langflow.py`

---

## File 1: `agent.py` - LangGraph Implementation

### Purpose
This file implements a chatbot agent using LangGraph with tool-calling capabilities and memory. It demonstrates the core concepts of agent architecture: state management, tool integration, and conversational flow.

### Corrected Code with Comments

```python
from typing import Annotated
from langgraph.graph import START, END, StateGraph
from langgraph.graph.message import add_messages
from langchain_ibm import ChatWatsonx  # Fixed: was ChatWatson
from dotenv import load_dotenv
import os
from colorama import Fore  # Fixed: was colorma

from langgraph.checkpoint.memory import MemorySaver  # Fixed: was InMemorySaver
from langgraph.prebuilt import ToolNode  # Fixed: was prebuild
from langgraph.prebuilt import tools_condition  # Added: missing import

# Load environment variables from .env file
load_dotenv()

# Initialize the Watson LLM
# You need to set MODEL_ID and PROJECT_ID in your .env file
llm = ChatWatsonx(
    model_id=os.getenv('MODEL_ID', 'ibm/granite-13b-chat-v2'),  # Default model
    project_id=os.getenv('PROJECT_ID'),  # Your IBM Cloud project ID
    params={'max_tokens': 5000}
)

# Define your tools - these are functions the agent can call
# You need to import or define the 'trending' tool
# Example tool definition:
from langchain_core.tools import tool

@tool
def trending(query: str) -> str:
    """Get trending information about a topic."""
    # Your implementation here
    return f"Trending info for: {query}"

# Bind tools to the LLM so it knows what functions it can call
tools = [trending]
llm_with_tools = llm.bind_tools(tools)

# Create the ToolNode - this executes the tools when called
tools_node = ToolNode(tools)

# Define the State class - this holds the conversation history
class State(dict):
    """
    State maintains the conversation context.
    messages: list of all messages in the conversation
    add_messages: LangGraph function that properly merges new messages
    """
    messages: Annotated[list, add_messages]

def chatbot(state: State):
    """
    Main chatbot node - processes messages with the LLM.
    
    Args:
        state: Current conversation state
    
    Returns:
        Dictionary with new messages from the LLM
    """
    print(f"Current state: {state}")
    # Invoke the LLM with the conversation history
    return {'messages': [llm_with_tools.invoke(state['messages'])]}

def router(state: State):
    """
    Router function - decides whether to call tools or end.
    
    Args:
        state: Current conversation state
    
    Returns:
        'tools' if the LLM wants to call a tool, END otherwise
    """
    last_message = state['messages'][-1]
    # Check if the last message contains tool calls
    if hasattr(last_message, 'tool_calls') and last_message.tool_calls:
        return 'tools'
    else:
        return END

# Build the state graph - this defines the agent's workflow
graphbuilder = StateGraph(State)

# Add nodes to the graph
graphbuilder.add_node('chatbot', chatbot)  # LLM processing node
graphbuilder.add_node('tools', tools_node)  # Tool execution node

# Define edges - how the agent flows between nodes
graphbuilder.add_edge(START, 'chatbot')  # Start with the chatbot
graphbuilder.add_edge('tools', 'chatbot')  # After tools, go back to chatbot

# Add conditional edge - decides whether to use tools or end
graphbuilder.add_conditional_edges(
    'chatbot',  # From the chatbot node
    router,  # Use the router function to decide
    {
        'tools': 'tools',  # If router returns 'tools', go to tools node
        END: END  # If router returns END, finish
    }
)

# Initialize memory for conversation history
memory = MemorySaver()

# Compile the graph with memory checkpoint
graph = graphbuilder.compile(checkpointer=memory)

# Main execution loop
if __name__ == '__main__':
    print("Agent is ready! Type your prompts below.")
    
    # Create a thread ID for conversation continuity
    thread_id = "conversation-1"
    
    while True:
        # Get user input
        prompt = input(Fore.LIGHTMAGENTA_EX + 'Pass your prompt here: ' + Fore.RESET)
        
        # Exit condition
        if prompt.lower() in ['exit', 'quit', 'bye']:
            print("Goodbye!")
            break
        
        # Invoke the agent with the user's message
        # The thread_id maintains conversation memory
        response = graph.invoke(
            {'messages': [{'role': 'user', 'content': prompt}]},
            config={"configurable": {"thread_id": thread_id}}
        )
        
        # Print the agent's response
        print(Fore.LIGHTCYAN_EX + response['messages'][-1].content + Fore.RESET)
```

### Key Concepts Explained

**1. State Management**
- The `State` class stores the conversation history
- `add_messages` annotation ensures messages are properly merged
- Memory persists across turns using `MemorySaver`

**2. Tool Binding**
- `llm.bind_tools(tools)` teaches the LLM what functions it can call
- The LLM decides when to call tools based on the conversation
- `ToolNode` executes the actual tool functions

**3. Graph Structure**
```
START → chatbot → (decision) → tools → chatbot → END
```
- Chatbot processes the input
- Router decides if tools are needed
- Tools execute if needed
- Returns to chatbot for final response

**4. Memory**
- `MemorySaver` stores conversation history
- `thread_id` maintains conversation context
- Agent can reference previous messages

---

## File 2: `price.py` - Stock Price Tool

### Purpose
Defines a tool for retrieving stock prices using Yahoo Finance. This tool can be imported into agents built with IBM Watsonx Orchestrate.

### Corrected Code with Comments

```python
from ibm_watsonx_orchestrate.agent_builder.tools import tool, ToolPermission
import yfinance as yf

@tool(
    name='stock_prices_florida',  # Unique identifier for the tool
    description='This tool returns a stock\'s last closing price given a stock ticker symbol',  # Fixed typo: trock → stock
    permission=ToolPermission.ADMIN  # Requires admin permission to execute
)
def stock_price(stock_ticker: str) -> str:
    """
    Retrieves the last closing price for a given stock.
    
    Args:
        stock_ticker: Stock symbol (e.g., 'AAPL', 'GOOGL', 'MSFT')
    
    Returns:
        String representation of the last closing price
    
    Example:
        stock_price('AAPL') → '150.25'
    """
    try:
        # Create a Ticker object for the given stock
        ticker = yf.Ticker(stock_ticker)
        
        # Get price history for the last month
        history = ticker.history(period='1mo')
        
        # Return the most recent closing price
        if not history.empty:
            return str(history['Close'].iloc[-1])
        else:
            return f"No data available for ticker: {stock_ticker}"
    
    except Exception as e:
        return f"Error retrieving price for {stock_ticker}: {str(e)}"

# IBM Watsonx Orchestrate CLI Commands:
# 1. Import agent configuration:
#    uv agents import -f agent.yaml
#
# 2. Run the agent:
#    uv run
#
# 3. Additional tool commands:
#    uv tool list  # List all available tools
```

### Function Breakdown

**Tool Decorator**
- `@tool` registers this function with IBM Watsonx Orchestrate
- `name`: How the agent refers to this tool
- `description`: Helps the LLM decide when to use this tool
- `permission`: Controls who can execute this tool

**Implementation**
- Uses `yfinance` library to fetch real-time stock data
- Retrieves 1 month of historical data
- Returns the most recent closing price
- Includes error handling for invalid tickers

---

## File 3: `income.py` - Income Statement Tool

### Purpose
Similar to `price.py`, but intended to retrieve income statement data. Note: Current implementation actually returns stock price (needs correction).

### Corrected Code with Comments

```python
from ibm_watsonx_orchestrate.agent_builder.tools import tool, ToolPermission
import yfinance as yf

@tool(
    name='income_statement_florida',
    description='This tool returns a company\'s income statement financial data',
    permission=ToolPermission.ADMIN
)
def income_statement(stock_ticker: str) -> str:
    """
    Retrieves the income statement for a given company.
    
    Args:
        stock_ticker: Stock symbol (e.g., 'AAPL', 'GOOGL', 'MSFT')
    
    Returns:
        Formatted string containing income statement data
    
    Example:
        income_statement('AAPL') → Returns revenue, expenses, net income, etc.
    """
    try:
        # Create a Ticker object
        ticker = yf.Ticker(stock_ticker)
        
        # Get the income statement
        income_stmt = ticker.income_stmt
        
        # Check if data exists
        if income_stmt is not None and not income_stmt.empty:
            # Convert to string format for the agent to read
            # Get the most recent period (first column)
            latest_data = income_stmt.iloc[:, 0]
            
            # Format key financial metrics
            result = f"Income Statement for {stock_ticker}:\n"
            
            # Extract common metrics if available
            metrics = [
                'Total Revenue',
                'Cost Of Revenue', 
                'Gross Profit',
                'Operating Income',
                'Net Income'
            ]
            
            for metric in metrics:
                if metric in latest_data.index:
                    value = latest_data[metric]
                    result += f"{metric}: ${value:,.0f}\n"
            
            return result
        else:
            return f"No income statement data available for ticker: {stock_ticker}"
    
    except Exception as e:
        return f"Error retrieving income statement for {stock_ticker}: {str(e)}"

# Additional Notes:
# - IBM Watsonx Orchestrate allows you to host a dashboard on localhost
# - Agents use libraries and LLMs to produce structured output
# - You can combine multiple tools to create comprehensive financial analysis agents

# Workflow:
# 1. Define tools (like this one)
# 2. Configure agent in agent.yaml
# 3. Import: uv agents import -f agent.yaml
# 4. Run: uv run
# 5. Test in the Orchestrate dashboard

# Resource: lmsys-leaderboard on Hugging Face shows ranked LLM performance
```

### Key Improvements

**Original Issue**: Function was named `stock_price` but tool name was `income_statement_florida`

**Corrections Made**:
- Renamed function to `income_statement` for consistency
- Implemented actual income statement retrieval
- Added formatting for key financial metrics
- Included error handling
- Added comprehensive documentation

---

## Page 3: Additional Implementation Details

## File 4: `agent.yaml` - Agent Configuration

### Purpose
Configuration file for IBM Watsonx Orchestrate agents. Defines the agent's properties, model, and behavior style.

### Corrected YAML with Comments

```yaml
# IBM Watson Agent Configuration
# This file defines the agent's specifications for Watsonx Orchestrate

spec_version: v1  # Configuration specification version

kind: native  # Agent type: native (runs directly) vs. custom

name: TechExchangeFinanceAgent  # Unique agent identifier

description: This agent is great at all things stock research.  # Agent purpose

# LLM Configuration
# Fixed typo: llma → llama
llm: watsonx/meta-llama/llama-3-405b-instruct  # Model to use

# Agent Reasoning Style
style: react  # ReAct pattern (Reasoning + Acting)
# Options: react, rewoo, chain-of-thought, etc.

# Additional Configuration Options (optional):
# tools:  # List of tools this agent can use
#   - stock_prices_florida
#   - income_statement_florida
#
# temperature: 0.7  # Controls randomness (0-1)
# max_tokens: 2000  # Maximum response length
#
# system_prompt: |  # Custom instructions for the agent
#   You are a financial research assistant specializing in stock analysis.
#   Provide accurate, data-driven insights based on the available tools.
```

### Configuration Breakdown

**spec_version**: Version of the configuration schema

**kind**: 
- `native`: Standard agent that runs in Orchestrate
- `custom`: Custom implementation with specific requirements

**llm**: The language model powering the agent
- Format: `provider/model-name`
- Using Meta's Llama 3 405B Instruct model
- One of the most capable open-source models

**style**: Reasoning strategy
- `react`: Best for tasks requiring tool use and iterative reasoning
- Agent will think, act (use tools), and observe results

---

## File 5: `langflow.py` - No-Code Agent Building

### Purpose
Documentation for building agents using Langflow, a visual drag-and-drop interface powered by DataStax.

### Langflow Workflow

```python
# Method 2: Build an agent without code using Langflow (DataStax)
"""
Langflow is a visual interface for building LLM applications and agents.
No coding required - use drag-and-drop components.

STEP-BY-STEP GUIDE:

1. Add Chat Input Component
   - This is where users will type their messages
   - Configure input parameters if needed
   
2. Add Agent Component
   - The core processing unit
   - Configure agent behavior and settings
   
3. Add LLM/Model to the Agent
   - Select your model provider (OpenAI, Anthropic, IBM Watson, etc.)
   - Enter API key in the component settings
   - Configure model parameters:
     * Temperature: Controls randomness
     * Max tokens: Response length
     * System prompt: Agent instructions
   
4. Add Tools to the Agent
   - Example: arXiv tool
     * Searches academic papers
     * Retrieves research information
     * Returns relevant findings
   - Other available tools:
     * Web Search (Google, Bing, DuckDuckGo)
     * Wikipedia
     * Calculator
     * Python REPL
     * Custom API tools
   
5. Add Chat Output Component
   - Displays agent responses to users
   - Configure output formatting
   
6. Connect the Components
   - Draw connections between components:
     Chat Input → Agent → Chat Output
     Model → Agent
     Tools → Agent
   
7. Test in Playground
   - Click the "Playground" button
   - Chat with your agent in real-time
   - Iterate and refine based on results
   
8. View and Customize Code
   - Click "View Code" to see the generated Python code
   - Export code for custom deployment
   - Modify and extend functionality
   - Version control your agent

ADVANTAGES OF LANGFLOW:
- No coding required to get started
- Visual representation of agent architecture
- Quick prototyping and iteration
- Easy tool integration
- Built-in testing environment
- Can export to code when needed

DEPLOYMENT OPTIONS:
- DataStax Langflow Cloud (hosted)
- Self-hosted (Docker, local server)
- Export to custom Python application

USE CASES:
- Research assistants (with arXiv, web search)
- Customer support bots
- Data analysis agents
- Content creation assistants
- Multi-tool orchestration
"""

# Example: What the generated code might look like
from langflow import ChatInput, ChatOutput, Agent
from langflow.components import OpenAI, ArxivTool

# This is automatically generated by Langflow
def build_agent():
    # Input component
    chat_input = ChatInput()
    
    # LLM component
    llm = OpenAI(
        model="gpt-4",
        api_key="your-api-key",
        temperature=0.7
    )
    
    # Tool component
    arxiv = ArxivTool()
    
    # Agent component
    agent = Agent(
        llm=llm,
        tools=[arxiv],
        system_message="You are a research assistant."
    )
    
    # Output component
    chat_output = ChatOutput()
    
    # Connect components
    agent.connect(chat_input, chat_output)
    
    return agent
```

### When to Use Langflow

**Pros:**
- Rapid prototyping
- No coding knowledge required
- Visual debugging
- Easy tool integration
- Great for non-technical stakeholders

**Cons:**
- Less control than code-based approaches
- May have limitations for complex logic
- Dependency on Langflow platform

**Best For:**
- Quick proof-of-concepts
- Business users building agents
- Testing different model/tool combinations
- Educational purposes

---

## Additional Resources & Best Practices

### Development Tools

**Excalidraw for Design**
- Use Excalidraw (https://excalidraw.com) for agent architecture diagrams
- Visualize agent workflows before coding
- Share designs with team members
- Great for planning multi-agent systems

### Environment Setup

Create a `.env` file for sensitive data:
```bash
# IBM Watson Configuration
MODEL_ID=ibm/granite-13b-chat-v2
PROJECT_ID=your-project-id-here

# API Keys
WATSONX_API_KEY=your-api-key
IBM_CLOUD_API_KEY=your-cloud-key

# Other configurations
MAX_TOKENS=5000
TEMPERATURE=0.7
```

### Testing Strategy

1. **Unit Testing**: Test individual tools
```python
def test_stock_price():
    result = stock_price('AAPL')
    assert result is not None
    assert isinstance(result, str)
```

2. **Integration Testing**: Test agent workflows
```python
def test_agent_flow():
    response = graph.invoke({
        'messages': [{'role': 'user', 'content': 'What is AAPL stock price?'}]
    })
    assert 'messages' in response
```

3. **End-to-End Testing**: Test complete user scenarios

### Monitoring & Logging

```python
import logging

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def chatbot(state: State):
    logger.info(f"Processing message: {state['messages'][-1]}")
    # ... rest of function
```

### Performance Optimization

1. **Caching**: Store repeated API calls
2. **Token Management**: Monitor and limit token usage
3. **Parallel Processing**: Use async for multiple tool calls
4. **Error Handling**: Graceful degradation when tools fail

---

## Conclusion

This documentation covers:
- ✅ Key learnings from IBM TechXchange 2025
- ✅ AI agent concepts and architecture
- ✅ Multiple implementation approaches (LangGraph, Watsonx, Langflow)
- ✅ Complete, corrected code with detailed comments
- ✅ Best practices and deployment strategies

**Next Steps:**
1. Set up your development environment
2. Choose a framework (start with LangGraph or Langflow)
3. Build a simple agent with one tool
4. Iterate and add complexity
5. Deploy and monitor in production
6. **Build agents for Axiom** - apply these learnings to real projects!

Remember: **Start simple, then scale.** Build the minimum viable agent that solves your problem, then enhance it based on real-world usage.

---

*Document created from IBM TechXchange Orlando 2025 learnings*