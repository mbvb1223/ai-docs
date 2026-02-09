# Build with AI
> Source: https://angular.dev/ai

## Overview

Angular provides comprehensive support for integrating AI-powered features into web applications. The platform excels at incorporating generative AI through its robust templating APIs, signal-based architecture, and seamless SDK integration capabilities.

## Key Capabilities

Angular facilitates AI integration through three primary strengths:

- **Dynamic UI Composition**: Angular's robust templating APIs enable the creation of dynamic, cleanly composed UIs made from generated content
- **State Management**: Strong signal-based architecture for managing dynamic data
- **SDK Integration**: Seamless compatibility with AI tools and APIs

## Three Integration Pathways

### 1. Genkit Framework

**Best for:** Full-stack applications requiring sophisticated backend logic

Genkit is an open-source toolkit supporting models from Google, OpenAI, Anthropic, and Ollama. It requires a Node-based server environment.

**Key Resources:**
- Agentic Apps starter kit for beginners
- Step-by-step Genkit + Angular + Gemini 2.5 Flash tutorial
- Dynamic Story Generator example with image generation
- Video walkthroughs demonstrating agentic workflows
- Firebase + Google Cloud Barista example (agentic ordering system)
- Server-driven UI generation examples

### 2. Firebase AI Logic

**Best for:** Client-side only or browser-based applications

Provides secure access to Vertex AI and Imagen APIs directly from web applications without exposing credentials.

**Starting Point:**
- Firebase AI Logic x Angular Starter Kit for e-commerce chat agents
- Video tutorials on feature implementation

### 3. Gemini API

**Best for:** Applications needing direct model control

Supports multimodal input (audio, images, video, text) with model optimization for specific use cases.

**Templates Available:**
- AI Text Editor with refinement features
- AI Chatbot with HTTP communication

## Security Best Practices

### API Credential Protection

**Never put your API key in a file that ships to the client, such as `environments.ts`.**

**Recommended approaches:**
- Firebase AI Logic for client-side scenarios (built-in security)
- Proxy servers or Cloud Functions for third-party API keys
- Environment variables for server-side connections
- Firebase secrets manager for latest implementations

### Architecture Considerations

Select integration tools based on application architecture:
- **Client-side**: Firebase AI Logic
- **Server-side**: Environment variables, secrets managers
- **Full-stack**: Proxy servers to protect credentials

## Advanced Patterns

### Tool Calling (Function Calling)

Enables agentic workflows where LLMs request function execution within applications. Developers maintain control over:
- Available tools
- Execution timing
- Function scope exposure

**Example:** E-commerce inventory functions allowing price calculations and stock checks.

### Managing Non-Deterministic Responses

**Strategies:**
- Adjust temperature and model parameters for consistency
- Implement "human in the loop" verification workflows
- Use tool/function calling with schema constraints
- Build graceful degradation for failed responses

**Resilience Pattern Example:**
- Save user responses for retry scenarios
- Alert users to outages without revealing sensitive details
- Resume workflows when services recover

## Prerequisites

Basic Angular knowledge recommended. New developers should explore:
- Essentials guide
- Getting started tutorials

## Additional Resources

- LLM prompts and AI IDE setup guides
- Design patterns for agentic applications
- Angular CLI MCP Server configuration
- Angular AI Tutor tools

---

**Note:** While featuring Google AI products, these tools are model-agnostic, allowing selection of preferred providers. Examples apply to third-party solutions.
