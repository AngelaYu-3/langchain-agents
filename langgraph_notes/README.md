# [LangGraph Documentation Notes](https://langchain-ai.github.io/langgraph/)

## [Core Capabilities](https://langchain-ai.github.io/langgraph/concepts/durable_execution/)

Persistence

Durable Execution

Memory

Context

Models

Tools

Human-in-the-Loop

Time Travel

Subgraphs

Multi-Agent

MCP

Evaluation

## [Building a Basic Chatbot](https://langchain-ai.github.io/langgraph/tutorials/get-started/1-build-basic-chatbot/)

StateGraph:
- defines the structure of the chatbot as a "state machine". Nodes are added to represent the LLM and functions the chatbot can call and edges to specify how the bot transitions between these functions.
- Each node can handle two key tasks--receive the current State as input and output an update to the state, updates to messages will be appended to the existing list

Nodes:
- represent units of work and are typically regular Python functions
- Node function takes the current State as input and returns a dictionary containing and updated messages list under the key "messages"

Entry Point:
- tells the graph where to start its work each time it's run

Exit Point:
- tells the graph where to finish execution

Compile Graph:
- compile the graph before running it

Visualize / Run the Chatbot:
- use the display() and stream() to visualize and display outputs

## [Add Tools](https://langchain-ai.github.io/langgraph/tutorials/get-started/2-add-tools/)

Create a Function to Run Defined Tools:
- add tools to a new node called BasicToolNode that checks the most recent message in the state and calls tools if the message contains tool_calls

Define Conditional Edges:
- edges routhe the control flow from one node to the next
- conditional edges start from a single node and usually contain "if" statements to route to diff nodes depending on the current graph state--these functions receive the current graph state and return a string list indicating which node(s) to call next

Router Function:
- define route_tools() that checks for tool_calls() in the bot's output--provide this function to the graph by calling add_conditional_edges() which tells the graph that whenever the chatbot node completes to check this function to see where to go next

## [Add Memory](https://langchain-ai.github.io/langgraph/tutorials/get-started/3-add-memory/)

Chatbots cannot remember the context of previous interactions, limiting ability to have coherent multi-turn conversations--LangGraph solves this with persistent checkpointing.

If you provide a checkpointer when compiling the graph and a thread_id when calling your graph, LangGraph automatically saves the state after each step--when you invoke the graph again using the same thread_id, the graph loads its saved state, allowing the bot to pick up where it left off.

[Different Types of Memory](https://langchain-ai.github.io/langgraph/concepts/memory/)

Create a MemorySaver Checkpointer
- memory = MemorySaver()

Compile the Graph
- graph = graph_builder.compile(checkpointer=memory)

## [Add Human-in-the-Loop Controls](https://langchain-ai.github.io/langgraph/tutorials/get-started/4-human-in-the-loop/)

Agents can be unreliable and may need human input to successfully accomplish tasks. Similarly, for some actions, you may want to require human approval before running to ensure that everything is running as intended.

LangGraph's [persistence layer](https://langchain-ai.github.io/langgraph/concepts/persistence/#memory-store) supports human-in-the-loop workflows, allowing execution to pause and resume based on user feedback.

Add Human-Assistance Tool
Compile the Graph

## [Customizing State](https://langchain-ai.github.io/langgraph/tutorials/get-started/5-customize-state/)

Adding additional fields to the State class to define complex behavior without relying on the message list. Chatbot will use its search tool to find specific info and forward them to a human for review.

## [Time Travel](https://langchain-ai.github.io/langgraph/tutorials/get-started/6-time-travel/#learn-more)

What if you want a user to be able to start from a previous response and explore a different outcome? Or what if you want useres to be able to rewind your chatbot's work to fix mistakes or try a diff strategy, something that is common in applications like autonomous software engineers? Use time travel!

Rewind Graph
- get_state_history()

Add Steps
- every step will be checkpointed in its state history

Replay the Full State History
- to_replay()
  
Resume from a CheckPoint
- to_replay()

Load a State from a Moment-in-Time
- to_replay()

## [Workflows and Agents](https://langchain-ai.github.io/langgraph/tutorials/workflows/)

Workflows
- systems where LLMs and tools are orchestrated through predefined code (user-controlled)

Agents
- systems where LLMs dynamically direct their own processes and tool usage, maintaining control over how they accomplish tasks (LLM has free reign)

Prompt Chaining
- decomposes a task into a sequence of steps, where each LLM call processes the output of the previous one
- this workflow is ideal for situations where the task can be easily and cleanly decomposed into fixed subtasks--main goal is to trade off latency for higher accuracy, by making each LLM call an easier task

Parallelization
- LLMs can sometimes work simultaneously on a task and have their outputs aggregated programmaticaallly
- parallelization manifests in two key variations: sectioning and voting
- sectioning is breaking a task into independent subtasks run in parallel
- voting is running the same task multiple times to get diverse outputs
- parallelization is effective when the divided subtasks can be parallelized for speed, or when multiple perspectives or attempts are needed for higher accuracy
- for complex tasks with multiple considerations (voting), LLMs generally perform better when each consideration is handled by a separate LLM call, allowing focused attention on each specific aspect

Routing
- classifies an input and directs it to a specialized followup task
- allows for separation of concerns, and building more specialized prompts--without routing, optimizing for one kind of input can hurt performance on other inputs
- routing works well for complex tasks where there are distinct categories that are better handled separately, and where classification can be handled accurately

Orchestrator-Worker
- a central LLM dynamically breaks down tasks, delegates them to worker LLMs, and synthesizes their results
- well-suited for complex tasks where you can't predict the subtasks needed--subtasks aren't predefined like in parallelization, but determined by the orchestrator based on the specific input

Evaluator-Optimizer
- one LLM call generates a response while another provides evaluation and feedback in a loop
- effective when we have clear evaluation criteria, and when iterative refinement provides measurable value
- signs that this could be a good workflow are: LLM responses can be demonstrably improved when a human articulates their feedback, and that the LLM can provide such feedback
- "iterative writing process"

Agent
- an LLM performing actions (via tool-calling) based on environmental feedback in a loop
- can handle sophisticated tasks, but implementation is fairly straightforward with focus on tools--thus crucial to design toolsets and their documentation clearly and thoughtfully
- agents can be used for open-ended problems where it's difficult or impossible to predict the required number of steps, and where you can't hardcode a fixed path
- agents' autonomy makes them ideal for scaling tasks in trusted environments
____

# Other Conceptual Notes

### [Chat Models](https://python.langchain.com/docs/concepts/chat_models/)
Modern LLMs are typically accessed through a chat model interface that takes a list of messages as input and returns a message as output. LangChain chat models implement the [BaseChatModel](https://python.langchain.com/api_reference/core/language_models/langchain_core.language_models.chat_models.BaseChatModel.html) interface. Because BaseChatModel also implementes the Runnable interface, models support streaming, async programming, optimized batching, etc. Models offer a standard set of [parameters](https://python.langchain.com/docs/concepts/chat_models/#standard-parameters) that can be used to configure the model.

Newest generation of chat models offer additional capabilities:

- [Tool Calling:](https://python.langchain.com/docs/concepts/tool_calling/) allows LLMs to interact with external services, APIs, and databases. Tools implement the [Runnable interface](https://python.langchain.com/docs/concepts/runnables/)
- [Structured Output:](https://python.langchain.com/docs/concepts/structured_outputs/) technique to make a model respond in a structured format, such as JSON that matches a given schema
- [Multimodality:](https://python.langchain.com/docs/concepts/multimodality/) ability to work with data other than text
- Context Window: max size of the input sequence the model can process at one time - in conversational applications, context window determines how much info the model can "remember" throughout the convo. Size of the input is measured in [tokens](https://python.langchain.com/docs/concepts/tokens/)
- Rate-Limiting: imposing a limit on the number of requests that can be made in a given time period - use LangChain's [rate_limiter](https://python.langchain.com/docs/how_to/chat_model_rate_limiting/)
- [Caching](https://python.langchain.com/docs/how_to/chat_model_caching/): effectiveness varies under differing circumstances - semantic caching (but introduces dependencies like embedding models)

Key Methods of a chat model (other methods can be found in [BaseChatMode](https://python.langchain.com/api_reference/core/language_models/langchain_core.language_models.chat_models.BaseChatModel.html):

- invoke: primary method for interacting with a model - takes a list of messages as input and returns a list of messages as output
- stream: allows for streaming the output of a model as it is generated
- batch: allows for batching multiple requests to a model together for more efficient processing
- bind_tools: allows for binding of a tool to a model for use in the model's execution context
- with_structured_output: wrapper around the invoke method for models that natively support structured output

### [Messages](https://python.langchain.com/docs/concepts/messages/)
Messages are the unit of communication in chat models. They are used to represent the input and output of a chat model, as well as any additional context of metadata that may be associated with a conversation. Each message has a role (user, assitant, etc) and content (text, multimodal data, etc) with additional metadata that varies depending on the chat model provider. LangChain provides a unified message format that can be used across chat modles, allowing useres to work with different chat models without worrying about the specific details of the message format used by each model provider.

Role of a message is used to distinguish between different types of messages in a conversation and help the chat model understand how to respond to a given sequence of messages

- system: tell the model how to behave and provide additional context, not supported by all model providers
- user: input from a user interacting with the model
- assistant: response from the model, which can include text or a request to invoke tools
- tool: a message used to pass the results of a tool invocation back to the model after external data or processing has been retrieved

Content of a message is a text or list of dictionaries representing multimodeal data (images, audio, video, etc)

Other data of a message depends on the model, and include data such as (ID, name, metadata, tool calls)

LangChain Messages provide a unified message format that can be used across models, allowing users to work with different models without worrying about the specific details of the message format used by each model

- system: SystemMessage
- user: HumanMessage
- assistant: AIMessage
- assistant (streaming purposes): AIMessageChunk
- tool: ToolMessage

___

# Additional Resources

### [LangChain Documentation](https://python.langchain.com/docs/introduction/)
### [LangGraph Documentation](https://langchain-ai.github.io/langgraph/)
### [LangSmith Documentation](https://docs.smith.langchain.com/?_gl=1*1viy4ju*_gcl_au*MTcyNTAwMDM5OC4xNzUwODAxMDU2*_ga*NTEyNzI1MzE0LjE3NTA4MDA2ODE.*_ga_47WX3HKKY2*czE3NTA5NzA3NjMkbzExJGcxJHQxNzUwOTcxNzE5JGozMCRsMCRoMA..)


