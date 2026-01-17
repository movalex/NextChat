# NextChat - Claude Code Reference

## Overview

NextChat is a multi-provider AI chat application built with Next.js 14, React 18, and TypeScript. It supports 16+ LLM providers and can be deployed as a web app, PWA, or desktop application (via Tauri).

## Project Structure

```
app/
├── api/                    # Server-side API routes for LLM providers
│   └── [provider]/[...path]/ # Dynamic provider routing
├── client/                 # Client-side LLM API implementations
│   ├── platforms/          # Provider-specific clients (openai.ts, anthropic.ts, google.ts, etc.)
│   ├── api.ts              # Abstract LLMApi class & ClientApi router
│   └── controller.ts       # Chat controller pool
├── components/             # React components
│   ├── chat.tsx            # Main chat interface (largest component)
│   ├── settings.tsx        # Settings panel
│   ├── mask.tsx            # Prompt template editor
│   ├── artifacts.tsx       # Code/content display modal
│   ├── markdown.tsx        # Markdown rendering
│   └── home.tsx            # App entry point
├── store/                  # Zustand state stores
│   ├── chat.ts             # Chat sessions & messages
│   ├── config.ts           # App configuration
│   ├── access.ts           # API keys & provider config
│   ├── mask.ts             # Prompt templates
│   └── plugin.ts           # Plugin management
├── config/
│   ├── client.ts           # Client-side config
│   └── server.ts           # Server-side config & env vars
├── mcp/                    # Model Context Protocol support
├── utils/                  # Utility functions
├── locales/                # i18n translations
├── constant.ts             # Global constants & model definitions
└── typing.ts               # TypeScript type definitions
src-tauri/                  # Tauri desktop app (Rust)
```

## Key Technologies

- **Framework**: Next.js 14.1.1, React 18.2.0, TypeScript 5.2.2
- **State**: Zustand 4.3.8 with localStorage/IndexedDB persistence
- **Styling**: SCSS/Sass modules
- **Desktop**: Tauri 2.1.1 (cross-platform)
- **Streaming**: SSE via @fortaine/fetch-event-source

## LLM Provider Architecture

### Adding/Modifying Providers

1. **Client implementation**: `app/client/platforms/<provider>.ts`
   - Implement `LLMApi` interface with `chat()`, `extractMessage()`, `models()` methods

2. **Server route**: `app/api/<provider>.ts`
   - Handle request transformation and API key injection

3. **Config**: Add env vars in `app/config/server.ts`

4. **Models**: Add to `DEFAULT_MODELS` array in `app/constant.ts`

5. **Access store**: Add provider settings in `app/store/access.ts`

### Provider Flow

```
User Input → chat.tsx → useChatStore → ClientApi.chat()
    → LLMApi (provider-specific) → Server API route
    → External LLM API → Streaming response → UI update
```

### Supported Providers

OpenAI, Azure OpenAI, Anthropic Claude, Google Gemini, Baidu Ernie, ByteDance Doubao, Alibaba Qwen, Tencent Hunyuan, Moonshot, Iflytek Spark, DeepSeek, XAI Grok, ChatGLM, SiliconFlow, 302.AI

## Message Handling

### Message Structure (app/store/chat.ts)

```typescript
interface ChatMessage {
  id: string;
  role: "user" | "assistant" | "system";
  content: string;
  date: string;
  streaming?: boolean;
  isError?: boolean;
  model?: string;
  tools?: ChatMessageTool[];  // MCP tool responses
}
```

### Response Parsing

Each provider has an `extractMessage()` method to parse responses. Common issues with malformed output should be fixed here.

Key files for response handling:
- `app/client/platforms/<provider>.ts` - Provider-specific parsing
- `app/utils/stream.ts` - Streaming utilities
- `app/utils/chat.ts` - Message formatting

## Common Development Tasks

### Fix Malformed Model Output

1. Identify the provider in `app/client/platforms/`
2. Check `extractMessage()` method for response parsing
3. Check `chat()` method for streaming handling
4. Look at `app/api/<provider>.ts` for server-side transformations

### Add New Features to Chat

1. UI changes: `app/components/chat.tsx`
2. State changes: `app/store/chat.ts`
3. Message processing: `app/client/platforms/` and `app/utils/chat.ts`

### Modify Model Configuration

1. Model list: `app/constant.ts` → `DEFAULT_MODELS`
2. Model parameters: `app/components/model-config.tsx`
3. Provider settings: `app/store/access.ts`

## Build Commands

```bash
yarn dev          # Development server
yarn build        # Production build (standalone)
yarn export       # Static export (no API routes)
yarn app:dev      # Tauri desktop dev
yarn app:build    # Build desktop app
yarn test         # Run tests
yarn lint         # ESLint
```

## Environment Variables

Key variables (see `app/config/server.ts` for full list):

```
OPENAI_API_KEY, BASE_URL
ANTHROPIC_API_KEY, ANTHROPIC_URL
GOOGLE_API_KEY, GOOGLE_URL
AZURE_URL, AZURE_API_KEY
CODE                        # Access code for app
HIDE_USER_API_KEY          # Hide API key input
ENABLE_BALANCE_QUERY       # Show balance in settings
```

## Important Files Reference

| Purpose | File |
|---------|------|
| Chat UI | `app/components/chat.tsx` |
| Chat state | `app/store/chat.ts` |
| Provider routing | `app/client/api.ts` |
| OpenAI client | `app/client/platforms/openai.ts` |
| Claude client | `app/client/platforms/anthropic.ts` |
| Model definitions | `app/constant.ts` |
| Server config | `app/config/server.ts` |
| TypeScript types | `app/typing.ts` |
| Markdown rendering | `app/components/markdown.tsx` |

## State Management Pattern

Uses Zustand with persistence wrapper:

```typescript
const useStore = createPersistStore(
  { /* initial state */ },
  (set, get) => ({ /* actions */ }),
  { name: StoreKey, version: 1, migrate }
);
```

## Testing

- Test files in `/test/`
- Jest 29.7.0 with React Testing Library
- Run: `yarn test` or `yarn test:ci`
