# Chain-of-Thought (CoT) Reasoning Implementation

This document describes the Chain-of-Thought reasoning feature implemented for the chat-bot application.

## Overview

The CoT feature enables the AI to perform step-by-step reasoning while maintaining control over what is exposed to users. This implementation follows the enhancement plan's P0 priority for safe reasoning mode.

## Features

### Reasoning Modes

1. **Off** (default): Standard response without explicit reasoning
2. **Hidden**: Model performs step-by-step reasoning internally but only shows the final answer
3. **Concise**: Shows a brief, sanitized rationale along with the final answer

### Reasoning Effort Levels

- **Low**: Quick, simple reasoning (2000 max thinking tokens)
- **Medium**: Balanced analysis (5000 max thinking tokens)  
- **High**: Thorough examination (10000 max thinking tokens)

## Backend Implementation

### API Changes

**Endpoint**: `POST /api/v1/chat/conversations/{conversation_id}/messages`

New request parameters:
```json
{
  "content": "user question",
  "reasoning_mode": "off|hidden|concise",
  "reasoning_effort": "low|medium|high",
  // ... other existing parameters
}
```

### SSE Event Types

Added new event type for streaming rationale:
```json
{
  "type": "rationale",
  "content": "Brief reasoning explanation",
  "conversation_id": "...",
  "message_id": null
}
```

### Key Components

1. **chat_schemas.py**: Added reasoning parameters to `ChatMessageRequest`
2. **chat_service.py**: 
   - `_apply_reasoning_mode()`: Configures LLM based on mode/effort
   - `_extract_rationale()`: Extracts concise reasoning from responses
   - `_sanitize_rationale()`: Removes PII from rationales
   - Dynamic prompt templates with reasoning instructions

## Frontend Implementation

### UI Components

1. **Reasoning Mode Selector**: Dropdown to select off/hidden/concise
2. **Effort Level Selector**: Visible when reasoning mode is active
3. **Rationale Display**: Blue info box showing reasoning (concise mode only)

### API Client Updates

Updated `sendChatMessage()` in `lib/api.ts` to include:
- `reasoning_mode` parameter
- `reasoning_effort` parameter

### State Management

New state variables in main page:
- `reasoningMode`: Current reasoning mode
- `reasoningEffort`: Current effort level
- `currentRationale`: Displayed rationale text

## Model Support

### Native Reasoning Models (OpenAI)
- Models with names containing 'o1', 'o3', or 'reasoning'
- Pass reasoning configuration directly to model

### Other Models (Ollama, standard GPT)
- Inject reasoning instructions into system prompts
- Extract rationale from response patterns

## Safety Features

1. **No Raw CoT Exposure**: Never stream or persist full chain-of-thought
2. **PII Redaction**: Automatic removal of phone numbers, emails, SSNs, credit cards
3. **Length Limits**: Rationales capped at 200 characters
4. **Feature Gating**: Concise mode only shows sanitized rationales

## Testing

Run the test script to verify implementation:

```bash
# Start backend
cd langchain_backend
poetry run python -m app.main

# Run tests (in another terminal)
python test_cot.py
```

The test script will:
1. Create a test conversation
2. Test each reasoning mode/effort combination
3. Display responses and rationales

## Usage Example

1. Select a model and provider in the UI
2. Choose reasoning mode:
   - "Off" for standard responses
   - "Hidden" for better quality without visible reasoning
   - "Concise" to see brief rationales
3. Adjust effort level for deeper analysis when needed
4. Send your question

The AI will process according to your settings, showing rationales only in concise mode.

## Future Enhancements

- Token usage tracking for reasoning
- Model-specific reasoning optimizations  
- Reasoning quality metrics
- User preference persistence
- Advanced rationale formatting