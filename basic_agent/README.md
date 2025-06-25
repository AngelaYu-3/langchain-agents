# Basic Agent
Building a complete agentic AI system using only free, local models (GPT-2). Faced with the constraint of not having access to powerful models like GPT-4, I implemented the fundamental logic patterns manually that sophisticated models handle automatically - revealing how tool integration, memory persistence, and state management actually work under the hood.

### Table of Contents
* [Tool Integration](#tool-integration)
* [Building Agentic Workflows](#building-agentic-workflows)
* [Memory Persistance & State Management](#memory-persistance--state-management)
* [Key Takeaways](#key-takeaways)
* [Attributions](#attributions)
___

### Tool Integration
Integrated [Tavily](https://www.tavily.com/) search tool. In this rudimentary implementation, Tavily was only used if certain keywords like "search" or "weather" were matched. However, more powerful models like GPT-4 which have advance reasoning capabilities make tool usage more sophisticated with approaches such as:

- intelligent intent recognition
- context-aware tool selection
- multi-tool reasoning
- tool chaining & dependencies
- parameter extraction & refinement
- dynamic tool discovery

In short, with more advanced models, tools become extensions of reasoning not just keyword triggers. Advanced models add intelligence (understanding intent, context, chaining), multi-step problem solving (orchestrating multiple tools), and context awareness (same query but different tools based on situation).

The basic tool integration framework is the same, but the intelligence to use this framework effectively comes from the model's reasoning capabilities!

___

### Building Agentic Workflows

    # build LangGraph workflow
    workflow = StateGraph(MessagesState)        # like blueprint of agentic model
    workflow.add_node("agent", agent_node)      # add one processing unit
    workflow.set_entry_point("agent")           # start here
    workflow.set_finish_point("agent")          # end here
    
    # add memory persistence--separate from nodes!
    memory = MemorySaver()
    app = workflow.compile(checkpointer=memory) # connect memory to workflow

This is the basic building block of an agentic model with a single agent. Workflow is like the blueprint of the agentic model, and no matter how many agents or tools the model has, there is only one workflow. Nodes are the individual agents themselves. MemorySaver() and StateGraph(MessagesState) are responsible for Memory Persistance & State Management

Here is a more complex agentic model with multiple nodes:

    # ONE StateGraph for the entire system:
      workflow = StateGraph(MessagesState)  # The entire workflow blueprint

    # Then add multiple agents as nodes:
    workflow.add_node("researcher", research_node)    # Agent 1
    workflow.add_node("writer", writer_node)          # Agent 2  
    workflow.add_node("critic", critic_node)          # Agent 3
    workflow.add_node("coordinator", coord_node)      # Agent 4 `

And here is an even more complex agentic model that demonstrates the power of Langgraph over Langchain (Langgraph allows for complex agent relationships beyond just chaining):

    def create_writing_team():
      workflow = StateGraph(MessagesState)
    
    # Add all the agents (nodes)
    workflow.add_node("coordinator", coordinator_node)
    workflow.add_node("researcher", researcher_node)  
    workflow.add_node("writer", writer_node)
    workflow.add_node("critic", critic_node)
    workflow.add_node("editor", editor_node)
    
    # Set the flow control
    workflow.set_entry_point("coordinator")  # Always start with coordinator
    
    # Smart conditional routing
    workflow.add_conditional_edges(
        "coordinator",
        coordinator_decides,
        {
            "research": "researcher",
            "write": "writer", 
            "review": "critic",
            "edit": "editor",
            "done": END
        }
    )
    
    # After research, go back to coordinator
    workflow.add_edge("researcher", "coordinator")
    
    # After writing, go to critic
    workflow.add_edge("writer", "critic")
    
    # After criticism, go to editor or coordinator
    workflow.add_conditional_edges(
        "critic",
        critic_decides,
        {
            "needs_editing": "editor",
            "good_enough": END,
            "start_over": "coordinator"
        }
    )
    
    # After editing, back to critic
    workflow.add_edge("editor", "critic")
    
    return workflow.compile(checkpointer=MemorySaver())

    def coordinator_decides(state):
      # AI looks at conversation and decides what to do next
      last_message = state["messages"][-1].content
      if "research" in last_message.lower():
          return "research"
      elif "write" in last_message.lower():
          return "write"
      elif "review" in last_message.lower():
          return "review"
      else:
          return "done"

___

### Memory Persistance & State Management

The agent node is the "processor", the model that is actually generating output. For each workflow, there is only ONE *StateGraph(MessagesState)*. However, there can be as many *MemorySaver()*(s) to dictate what agents can see what parts of the workflow's memory. This allows for memory access control.

For instance:

    # One workflow orchestrator
    workflow = StateGraph(MessagesState)

    # Multiple memory partitions
    user_memory = MemorySaver()      # User conversation history
    admin_memory = MemorySaver()     # Admin-only conversations  
    work_memory = MemorySaver()      # Work-related discussions
    agent_workspace = MemorySaver()  # Agent's private working memory

    # Different agents see different memory partitions
    researcher_app = workflow.compile(checkpointer=work_memory)
    admin_app = workflow.compile(checkpointer=admin_memory)
    user_app = workflow.compile(checkpointer=user_memory)

    def create_multi_memory_system():
    # Same StateGraph for all
    workflow = StateGraph(MessagesState)
    workflow.add_node("researcher", researcher_node)
    workflow.add_node("admin", admin_node)
    workflow.add_node("coordinator", coordinator_node)
    
    # Different memory access per role
    return {
        "public_agent": workflow.compile(checkpointer=public_memory),
        "private_agent": workflow.compile(checkpointer=private_memory),
        "admin_agent": workflow.compile(checkpointer=admin_memory)
    }

    # Usage:
    agents = create_multi_memory_system()

    # Public agent only sees public conversations
    public_response = agents["public_agent"].invoke(
        {"messages": [HumanMessage("Hello")]},
        config={"configurable": {"thread_id": "user_123"}}
    )

    # Admin agent sees different memory space entirely
    admin_response = agents["admin_agent"].invoke(
        {"messages": [HumanMessage("Show system status")]},
        config={"configurable": {"thread_id": "admin_session"}}
    )

We can think of StateGraph as a memory management tool and MemorySaver as a partioned dictionary.

**StateGraph:**

A memory management tool to update a particular thread's state (thread is a conversation boundary identifier--can represent users, sessions, topics etc--threads can NOT see the state of other threads) as demonstrated in the experiement ran in the code above with Alice's and Bob's conversations)

A typical StateGraph flow could look like:
- load state from memory
- combine old and new messages
- process through agent node
- append AI response to state
- save updated state to memory

**MemorySaver:**

There can be as many MemorySaver(s) as needed to partition memory. Stores state using StateGraph for each thread. Threads cannot access each other's states.

Here is an example of how multiple MemorySaver(s) and threads may interact in an Extended Multi-User, Multi-Mode System:

    class MultiModeMultiUserAgent:
      def __init__(self):
          self.workflow = StateGraph(MessagesState)
        
          # Different memory stores for different agent modes
          self.casual_memory = MemorySaver()        # Casual conversations
          self.professional_memory = MemorySaver()  # Professional conversations  
    
      def get_agent_in_mode(self, mode):
          memory_map = {
              "casual": self.casual_memory,
              "professional": self.professional_memory,
          }
          return self.workflow.compile(checkpointer=memory_map[mode])

    # Create the system
    agent_system = MultiModeMultiUserAgent()

    # Get different agent modes
    casual_bot = agent_system.get_agent_in_mode("casual")
    professional_bot = agent_system.get_agent_in_mode("professional")


How memory is partitioned:

    # Show how memory is partitioned
    memory_structure = {
      "casual_memory": {
        "alice_casual_chat": f"{len(alice_casual_2['messages'])} messages",
        "bob_casual_chat": f"{len(bob_casual['messages'])} messages", 
        "charlie_casual_chat": f"{len(charlie_casual['messages'])} messages"
      },
      "professional_memory": {
        "alice_professional_chat": f"{len(alice_mode_switch['messages'])} messages",
        "bob_professional_chat": f"{len(bob_professional['messages'])} messages"
      },
    }
    
**Key Insights from Code Above**

User-mode combinations thanks to threads
- same user can have multiple threads per MemorySaver (bot mode)
- different users have completely separate threads
- each thread maintains independent conversation history
- no cross-contamination between threads (users) or MemorySaver (bot modes)
- alice_casual_chat ≠ alice_professional_chat
- bob_casual_chat ≠ alice_casual_chat

Memory isolation by MemoryState (bot mode)
- casual bot only sees casual_memory threads
- professional bot only sees professional_memory threads  
- technical bot only sees technical_memory threads

___

### Key Takeaways

While this implementation is very rudimentary, the inherent logic behind tool integration and memory persistence and state management is present. A more sophisticated LLM could better utilize tool integration and use memory persistance more effectively in terms of utilizing all the past messages instead of just the last message.

The existence of threads, design choices of deciding how many MemoryStates to include in a workflow allow for more complex agentic models. Additionally, Langgraph's sophisticated ability to architect complex relationships between agents beyond just chaining allow for more powerful agentic models.

This project was really fun because it helped me better understand the foundations behind tool integration and more importantly memory persistance & state management. I definitely have a deep appreciation of how memory persistance & state management are implemented, and hopefully can now take this knowledge to design agentic models in a purposeful manner.

___

### Attributions
This project loosely follows [Lanchain's Build An Agent Tutorial](https://python.langchain.com/docs/tutorials/agents/)


