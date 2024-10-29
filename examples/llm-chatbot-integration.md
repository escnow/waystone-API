# LLM Chatbot Integration with Autotask

This guide demonstrates how to integrate an LLM (Large Language Model) chatbot with Autotask using the Waystone API. The example shows how to create a chatbot that can handle ticket creation, status updates, and basic customer inquiries.

## Prerequisites

- Waystone API credentials
- Python 3.7+
- Required Python packages: `requests`, `openai`

## Implementation Example

```python
import requests
import openai
from typing import Dict, Any

class AutotaskLLMBot:
    def __init__(self, api_key: str, base_url: str):
        self.api_key = api_key
        self.base_url = base_url
        self.headers = {
            'Authorization': f'Bearer {api_key}',
            'Content-Type': 'application/json'
        }
        
        # Configure OpenAI (or your preferred LLM provider)
        openai.api_key = 'your-openai-api-key'

    def process_user_message(self, user_message: str) -> Dict[Any, Any]:
        """
        Process user message and determine appropriate action in Autotask
        """
        # Generate LLM response to understand user intent
        response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",
            messages=[
                {"role": "system", "content": """
                You are a helpful IT support assistant. Analyze user requests and:
                1. Determine if a new ticket needs to be created
                2. Check if this is a status inquiry
                3. Identify if this is a general question
                """},
                {"role": "user", "content": user_message}
            ]
        )
        
        # Extract intent from LLM response
        intent = self._analyze_intent(response.choices[0].message['content'])
        
        # Handle different intents
        if intent['type'] == 'create_ticket':
            return self._create_ticket(intent['details'])
        elif intent['type'] == 'status_check':
            return self._check_ticket_status(intent['ticket_id'])
        else:
            return {'response': response.choices[0].message['content']}

    def _create_ticket(self, details: Dict[str, Any]) -> Dict[Any, Any]:
        """
        Create a new ticket in Autotask
        """
        endpoint = f"{self.base_url}/tickets"
        payload = {
            'title': details['title'],
            'description': details['description'],
            'priority': details['priority'],
            'status': 'New'
        }
        
        response = requests.post(endpoint, headers=self.headers, json=payload)
        return response.json()

    def _check_ticket_status(self, ticket_id: str) -> Dict[Any, Any]:
        """
        Check status of an existing ticket
        """
        endpoint = f"{self.base_url}/tickets/{ticket_id}"
        response = requests.get(endpoint, headers=self.headers)
        return response.json()

    def _analyze_intent(self, llm_response: str) -> Dict[str, Any]:
        """
        Analyze LLM response to determine user intent
        """
        # This is a simplified example - in practice, you'd want more robust intent analysis
        if "create" in llm_response.lower():
            return {
                'type': 'create_ticket',
                'details': {
                    'title': 'New Support Request',
                    'description': llm_response,
                    'priority': 'Medium'
                }
            }
        elif "status" in llm_response.lower():
            return {
                'type': 'status_check',
                'ticket_id': 'extract_ticket_id_from_response'
            }
        else:
            return {'type': 'general_inquiry'}

# Usage Example
def main():
    bot = AutotaskLLMBot(
        api_key='your-waystone-api-key',
        base_url='https://api.waystone.com/v1'
    )
    
    # Example interactions
    responses = [
        bot.process_user_message("I need help with my printer, it's not working"),
        bot.process_user_message("What's the status of my ticket #12345?"),
        bot.process_user_message("Can you tell me your operating hours?")
    ]
    
    for response in responses:
        print(response)

if __name__ == "__main__":
    main()
```

## Key Features

1. **Intent Recognition**: Uses OpenAI's GPT model to understand user requests and determine appropriate actions.
2. **Ticket Management**: Handles ticket creation and status checks through the Waystone API.
3. **Flexible Response Handling**: Can handle both structured (ticket-related) and unstructured (general inquiries) conversations.

## Best Practices

1. **Error Handling**
```python
def safe_api_call(self, func):
    try:
        return func()
    except requests.exceptions.RequestException as e:
        return {
            'error': True,
            'message': f"API Error: {str(e)}",
            'status_code': getattr(e.response, 'status_code', None)
        }
    except openai.error.OpenAIError as e:
        return {
            'error': True,
            'message': f"LLM Error: {str(e)}"
        }
```

2. **Rate Limiting**
```python
from time import sleep
from datetime import datetime, timedelta

class RateLimiter:
    def __init__(self, calls_per_minute=60):
        self.calls_per_minute = calls_per_minute
        self.calls = []
    
    def wait_if_needed(self):
        now = datetime.now()
        self.calls = [call for call in self.calls if call > now - timedelta(minutes=1)]
        
        if len(self.calls) >= self.calls_per_minute:
            sleep(60/self.calls_per_minute)
        
        self.calls.append(now)
```

3. **Conversation Context Management**
```python
class ConversationManager:
    def __init__(self):
        self.conversations = {}
    
    def add_message(self, user_id: str, message: str, role: str):
        if user_id not in self.conversations:
            self.conversations[user_id] = []
        self.conversations[user_id].append({
            'role': role,
            'content': message,
            'timestamp': datetime.now()
        })
    
    def get_context(self, user_id: str, limit: int = 5):
        if user_id in self.conversations:
            return self.conversations[user_id][-limit:]
        return []
```

## Security Considerations

1. Always validate and sanitize user input before processing
2. Use environment variables for sensitive credentials
3. Implement proper authentication and authorization
4. Regular security audits and updates
5. Monitor and log all interactions for security purposes

## Testing

```python
import unittest

class TestAutotaskLLMBot(unittest.TestCase):
    def setUp(self):
        self.bot = AutotaskLLMBot('test-api-key', 'test-url')
    
    def test_ticket_creation(self):
        response = self.bot.process_user_message("My computer won't start")
        self.assertIn('ticket_id', response)
    
    def test_status_check(self):
        response = self.bot.process_user_message("What's the status of ticket #12345?")
        self.assertIn('status', response)
```

## Deployment Recommendations

1. Use containerization (Docker) for consistent environments
2. Implement proper logging and monitoring
3. Set up automated testing and CI/CD pipelines
4. Use environment-specific configuration files
5. Regular backup and disaster recovery plans

Remember to replace placeholder API keys and endpoints with your actual credentials when implementing this solution.
