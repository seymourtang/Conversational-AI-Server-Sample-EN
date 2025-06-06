# 🌟 Custom LLM Sample Code for Golang

> The Agora Conversational AI Engine supports custom large language model (LLM) functionality. You can refer to this project code to implement custom large language model functionality.

This document provides Golang sample code for implementing custom large language model functionality.

## 🚀 Quick Start

### Environment Preparation

- Golang 1.21+

### Install Dependencies

```bash
go mod tidy
```

### Run Sample Code

```bash
go run custom_llm.go
```

When the server is running, you will see the following output:

```bash
[GIN-debug] Listening and serving HTTP on :8000
```

Use the following command to test the server:

```bash
curl -X POST http://localhost:8000/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_LLM_API_KEY" \
  -d '{"messages": [{"role": "user", "content": "Hello, how are you?"}], "stream": true, "model": "gpt-4o-mini"}'
```

For testing the server, we recommend using a tunneling tool like [ngrok](https://ngrok.com/) to expose your local server to the internet.

## 🔄 Architecture and Flow Diagrams

### System Architecture

```mermaid
flowchart LR
    Client-->|POST Request|Server

    subgraph Server[Custom LLM Server]
        Basic["chat/completions"]
        RAG["rag/chat/completions"]
        Audio["audio/chat/completions"]
    end


    Server-->|SSE Response|Client

    Server-->|API call|OpenAI[OpenAI API]
    OpenAI-->|Stream Response|Server

    subgraph Knowledge
        KB[Knowledge Base]
    end

    RAG-.->|Retrieval|KB
```

For more details about the three endpoints and their request flows, see the [Request Flow Diagrams](#📝-request-flow-diagrams) section.

## 📖 Function Description

### Basic Custom Large Language Model

> To successfully integrate with the Agora Conversational AI Engine, your custom large model service must provide an interface compatible with the OpenAI Chat Completions API.

Refer to the `handleChatCompletion` function for implementation logic.

### Implementing Retrieval-Augmented Custom Large Language Model

> If you want to improve the accuracy and relevance of the agent's responses, you can use the Retrieval-Augmented Generation (RAG) feature. This allows your custom large model to retrieve information from a specific knowledge base and provide the retrieval results as context for the large model to generate answers.

Refer to the `handleRAGChatCompletion` function for implementation logic.

### Implementing Multimodal Custom Large Language Model

Preparation:

- Place a `file.pcm` file in the current directory with a sample rate of 16000, 16-bit, mono, and PCM format.
- Place a `file.txt` file in the current directory containing the text transcription of the above audio file.

Refer to the `handleAudioChatCompletion` function for implementation logic.

## 📝 Request Flow Diagrams

### Basic LLM Request Flow

```mermaid
sequenceDiagram
    participant Client
    participant Server as Custom LLM Server
    participant OpenAI

    Client->>Server: POST /chat/completions
    Note over Client,Server: With messages, model, stream params

    Server->>OpenAI: Create chat.completions stream

    loop For each chunk
        OpenAI->>Server: Streaming chunk
        Server->>Client: SSE data: chunk
    end

    Server->>Client: SSE data: [DONE]
```

### RAG-enhanced LLM Request Flow

```mermaid
sequenceDiagram
    participant Client
    participant Server as Custom LLM Server
    participant KB as Knowledge Base
    participant OpenAI

    Client->>Server: POST /rag/chat/completions
    Note over Client,Server: With messages, model params

    Server->>Client: SSE data: "Waiting message"

    Server->>KB: Perform RAG retrieval
    KB->>Server: Return relevant context

    Server->>Server: Refactor messages with context

    Server->>OpenAI: Create chat.completions stream with context

    loop For each chunk
        OpenAI->>Server: Streaming chunk
        Server->>Client: SSE data: chunk
    end

    Server->>Client: SSE data: [DONE]
```

### Multimodal Audio LLM Request Flow

```mermaid
sequenceDiagram
    participant Client
    participant Server as Custom LLM Server
    participant FS as File System

    Client->>Server: POST /audio/chat/completions
    Note over Client,Server: With messages, model params

    alt Files exist
        Server->>FS: Read text file
        FS->>Server: Return text content

        Server->>FS: Read audio file
        FS->>Server: Return audio data

        Server->>Client: SSE data: transcript

        loop For each audio chunk
            Server->>Client: SSE data: audio chunk
            Note over Server,Client: With small delay between chunks
        end
    else Files not found
        Server->>Server: Generate simulated response
        Server->>Client: SSE data: simulated transcript

        loop For simulated chunks
            Server->>Client: SSE data: random audio data
            Note over Server,Client: With small delay between chunks
        end
    end

    Server->>Client: SSE data: [DONE]
```

## 📚 Resources

- 📖 Check out our [Conversational AI Engine Documentation](https://doc.agora.io/doc/convoai/restful/landing-page) for more details
- 🧩 Visit [Agora SDK Examples](https://github.com/AgoraIO) for more tutorials and example code
- 👥 Explore high-quality repositories managed by the developer community in the [Agora Developer Community](https://github.com/AgoraIO-Community)
- 💬 If you have any questions, feel free to ask on [Stack Overflow](https://stackoverflow.com/questions/tagged/agora.io)

## 💡 Feedback

- 🤖 If you have any problems or suggestions regarding the sample projects, we welcome you to file an issue.

## 📜 License

This project is licensed under the MIT License.
