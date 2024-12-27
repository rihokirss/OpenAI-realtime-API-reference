
# OpenAI Realtime API Integration Guide

This document serves as a comprehensive guide to interacting with the OpenAI Realtime API, focusing on proper message structure, event handling, and response protocols. It is tailored for developers implementing function call results and other realtime features.

## Key Concepts

### Supported Event Types
The OpenAI Realtime API recognizes the following event types:

- `session.update`: Modify session configurations (e.g., tool settings).
- `input_audio_buffer.append`: Append audio data to the buffer.
- `input_audio_buffer.commit`: Commit audio data for processing.
- `input_audio_buffer.clear`: Clear the audio buffer.
- `conversation.item.create`: Send a new message or interaction.
- `conversation.item.truncate`: Truncate conversation history.
- `conversation.item.delete`: Delete specific conversation items.
- `response.create`: Create a response for a specific tool call or function call.
- `response.cancel`: Cancel an ongoing response or function.

### Supported `item.type` Values
- `message`: Represents a conversational message.
- `function_call`: Indicates a function call request.
- `function_call_output`: Represents the result of a function call.
- `item_reference`: References a previous item in the conversation.

## Message Structures

### Function Call Results
To send a function call result, adhere to the following structure:

```json
{
  "type": "response.create",
  "response": {
    "items": [
      {
        "type": "function_call_output",
        "call_id": "<call_id_from_model>",
        "output": {
          "result": { "key": "value" } // JSON object representing the function output
        }
      }
    ]
  }
}
```

#### Parameters:
- `type`: Always set to `response.create`.
- `response.items`: Array of items containing function call outputs.
  - `type`: Must be `function_call_output`.
  - `call_id`: The `call_id` from the model's original function call request.
  - `output`: Contains the result data as a JSON object.

### Session Update
To update session settings:

```json
{
  "type": "session.update",
  "session": {
    "tools": [
      {
        "type": "function",
        "name": "tool_name",
        "description": "Tool description.",
        "parameters": {
          "type": "object",
          "properties": {
            "param_name": {
              "type": "string",
              "description": "Parameter description."
            }
          },
          "required": ["param_name"]
        }
      }
    ]
  }
}
```

### Sending a Text Message
To send a text message as part of the conversation:

```json
{
  "type": "conversation.item.create",
  "item": {
    "type": "message",
    "role": "user",
    "content": [
      {
        "type": "input_text",
        "text": "Your message here"
      }
    ]
  }
}
```

### Error Handling
Errors are returned in the following format:

```json
{
  "type": "error",
  "event_id": "unique_event_id",
  "error": {
    "type": "error_type",
    "code": "error_code",
    "message": "Detailed error message.",
    "param": "Parameter causing the error.",
    "event_id": null
  }
}
```

#### Common Error Codes:
- `invalid_request_error`: Issues with the structure or parameters of the request.
- `missing_required_parameter`: A required parameter is not provided.
- `unknown_parameter`: A parameter is not recognized.
- `invalid_value`: A parameter has an unsupported value.

## Example Usage

### Sending Function Call Result
```javascript
async function sendFunctionCallResult(callId, resultData) {
  const event = {
    type: "response.create",
    response: {
      items: [
        {
          type: "function_call_output",
          call_id: callId,
          output: {
            result: resultData
          }
        }
      ]
    }
  };

  if (dataChannel && dataChannel.readyState === "open") {
    dataChannel.send(JSON.stringify(event));
    console.log("Function call result sent:", event);
  } else {
    console.error("Data channel is not open.");
  }
}
```

### Updating Session Settings
```javascript
const sessionUpdateEvent = {
  type: "session.update",
  session: {
    tools: [
      {
        type: "function",
        name: "get_data_from_database",
        description: "Fetches data from the database.",
        parameters: {
          type: "object",
          properties: {
            query: {
              type: "string",
              description: "SQL query to execute."
            }
          },
          required: ["query"]
        }
      }
    ]
  }
};

if (dataChannel && dataChannel.readyState === "open") {
  dataChannel.send(JSON.stringify(sessionUpdateEvent));
  console.log("Session updated:", sessionUpdateEvent);
} else {
  console.error("Data channel is not open.");
}
```

## Debugging Tips
- Always validate JSON structures before sending.
- Log all outgoing and incoming messages for traceability.
- Handle `dataChannel` state changes and reconnect if necessary.
- Use `console.error` to capture errors and take corrective action.

## Conclusion
This guide provides a structured approach to leveraging OpenAI's Realtime API effectively. Ensure all messages follow the prescribed formats to avoid errors and streamline communication with the model.
