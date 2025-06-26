# Q&A Agent

____

# Conceptual Notes

### [Chat Models](https://python.langchain.com/docs/concepts/chat_models/)
Modern LLMs are typically accessed through a chat model interface that takes a list of messages as input and returns a message as output.

Newest generation of chat models offer additional capabilities:

- [Tool Calling:](https://python.langchain.com/docs/concepts/tool_calling/) allows LLMs to interact with external services, APIs, and databases. Tools implement the [Runnable interface](https://python.langchain.com/docs/concepts/runnables/)
- [Structured Output:]() technique to make a model respond in a structured format, such as JSON that matches a given schema
- [Multimodality:]() ability to work with data other than text

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



