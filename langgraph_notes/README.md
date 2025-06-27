# [LangGraph Documentation Notes](https://langchain-ai.github.io/langgraph/)

### [Building a Basic Chatbot](https://langchain-ai.github.io/langgraph/tutorials/get-started/1-build-basic-chatbot/)

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

### [Add Tools](https://langchain-ai.github.io/langgraph/tutorials/get-started/2-add-tools/)

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


