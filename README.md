# Feather - A lightweight agent framework

![dy_oIoS8L2SudycKG_Noa_eb99b192c6c84c0cbbd3a68d20076e20](https://github.com/user-attachments/assets/be78639b-6c4b-4143-bff1-b246ec0f70f6)

I don't need layers of abstraction. Base LLMs are very capable. This framework is for lightly defining agents with tools that auto-execute.

Chaining agents together with Feather looks like this:

```typescript
const timelineData = twitterAgent.run("Get 50 tweets from my AI list and summarize it for me")
const videoScript = videoScriptAgent.run('Create me a video script based on todays AI news:' + timelineData)
```

## DEBUG GUI 
Feather comes with an optional GUI that displays detailed log of the agent's system prompt, message history, and LLM requests/responses that went into each assistant message.

<img width="1728" alt="image" src="https://github.com/user-attachments/assets/0bc53f8d-0654-47b7-866a-33c59b642e4f" />

## OPENROUTER
We use OpenRouter for LLM calls, which uses the Openai SDK 1:1. This allows us to use ANY model with ease! While it is a centralized service, it is the easiest, most cost effective way to get access to the latest models instantly and switch models with ease. Also, they accept crypto. If OpenRouter ever goes down, we can easily switch as the base SDK is OpenAI.

https://openrouter.ai/

## OPENPIPE
We use OpenPipe for collecting training data of agents. This is optional, but HIGHLY recommended for any agent that is used in production. Your data is GOLD, make sure to mine it!

https://openpipe.ai/

## CREATING AN AGENT

```typescript
const agent = new Agent({
model: "openai/gpt-4o-mini", // REQUIRED
systemPrompt: "You are a helpful assistant", // REQUIRED
tools: [internetTool], // OPTIONAL, TAKES DOMINANCE OVER STRUCTURED OUTPUT 
cognition: true, // OPTIONAL, ENABLES THINK/PLAN/SPEAK XML TAG PROCESS
debug: true, // OPTIONAL, ENABLES DEBUG GUI
})
```

Running an agent is simpler:

```typescript
const result = agent.run("What is the mitochondria of the cell?")
```

## MODIFYING AN AGENT'S MESSAGE HISTORY
You can modify an agent's message history with the following methods:

```typescript
// Adding messages
agent.addUserMessage("Hello, how are you? Do you like my hat?", {image: "https://example.com/blueHat.jpg"}) //image optional
agent.addAssistantMessage("I am fine, thank you! Nice blue hat! Looks good on you!")

// Extracting current message history
agent.extractHistory() //returns the chat history as an array of messages

// Loading in custom message history
const history = [{role: "user", content: "Hello, how are you? Do you like my hat?", image: "https://example.com/blueHat.jpg"}, {role: "assistant", content: "I am fine, thank you! Nice blue hat! Looks good on you!"}] //array of messages
agent.loadHistory(history) //loads the chat history from an array of messages
```

## COGNITION
Cognition is the process of the agent thinking, planning, and speaking. It is enabled by the cognition property in the agent config. What is does is add forced instructions at the end of the agent's system prompt to use XML tags to think, plan, and speak. These XML tags are parsed and executed by the agent. <think>...</think>, <plan>...</plan>, <speak>...</speak> are the tags used. <speak> tags are parsed and returned as the agent's response.

I find that cognition is a great way to get increased accuracy and consistency with tool usage.

## TOOL USE
Tool calls (also known as function calling) allow you to give an LLM access to external tools.

Feather expects your tool to be defined WITH the function execution and output ready to go. This way, when giving an agent a tool, the agent can execute the tool, get the results saved in its chat history, then re-run itself to provide the user a detailed response with the information from the tool result.

Parallel tool calls are supported.

Setting up a tool function call following OpenAI structure + Excecution
```typescript
const internetTool: ToolDefinition = {
  type: "function",
  function: {
    name: "search_internet",
    description: "Search the internet for up-to-date information using Perplexity AI",
    parameters: {
      type: "object", 
      properties: {
        query: {
          type: "string",
          description: "The search query to look up information about"
        }
      },
      required: ["query"]
    }
  },
  // Execute function to search using Perplexity AI
  async execute(args: Record<string, any>): Promise<{ result: string }> {
    logger.info({ args }, "Executing internet search tool");
    
    try {
      const params = typeof args === 'string' ? JSON.parse(args) : args;
      if (typeof params.query !== 'string') {
        throw new Error("Query must be a valid string");
      }

      // Call Perplexity API to get search results
      const searchResult = await queryPerplexity(params.query);
      
      return { result: searchResult };

    } catch (error) {
      logger.error({ error, args }, "Internet search tool error");
      throw error;
    }
  }
};
```

## STRUCTURED OUTPUT
If you are using structured output instead of tools, the .run() function will return the structured output as a JSON object.