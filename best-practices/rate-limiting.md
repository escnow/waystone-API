# Rate Limiting

This guide covers rate limiting in the Autotask API, including best practices for handling rate limits and implementing proper backoff strategies.

## Rate Limit Overview

The Autotask API implements rate limiting to ensure fair usage and system stability.

### Default Limits

- **600 requests per minute** per API token
- **10 requests per minute** for authentication endpoints
- **Concurrent request limit**: 3 simultaneous requests

### Rate Limit Headers

Every API response includes rate limit information in the headers:

```
X-RateLimit-Limit: 600
X-RateLimit-Remaining: 599
X-RateLimit-Reset: 1612345678
```

## Handling Rate Limits

### Rate Limit Response

When you exceed the rate limit, you'll receive a 429 (Too Many Requests) response:

```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "API rate limit exceeded",
    "retryAfter": 60
  }
}
```

### Implementation Examples

#### Python Implementation

```python
import time
import requests
from requests.exceptions import RequestException

class RateLimitHandler:
    def __init__(self, base_delay=1, max_retries=3):
        self.base_delay = base_delay
        self.max_retries = max_retries

    def make_request(self, url, headers):
        for attempt in range(self.max_retries):
            try:
                response = requests.get(url, headers=headers)
                
                if response.status_code == 200:
                    return response.json()
                    
                if response.status_code == 429:
                    retry_after = int(response.headers.get('Retry-After', self.base_delay))
                    time.sleep(retry_after)
                    continue
                    
                response.raise_for_status()
                
            except RequestException as e:
                if attempt == self.max_retries - 1:
                    raise e
                    
                time.sleep(self.base_delay * (2 ** attempt))
                
        return None
```

#### JavaScript Implementation

```javascript
class RateLimitHandler {
    constructor(baseDelay = 1000, maxRetries = 3) {
        this.baseDelay = baseDelay;
        this.maxRetries = maxRetries;
    }

    async makeRequest(url, headers) {
        for (let attempt = 0; attempt < this.maxRetries; attempt++) {
            try {
                const response = await fetch(url, { headers });
                
                if (response.ok) {
                    return await response.json();
                }
                
                if (response.status === 429) {
                    const retryAfter = parseInt(response.headers.get('Retry-After')) || this.baseDelay;
                    await new Promise(resolve => setTimeout(resolve, retryAfter * 1000));
                    continue;
                }
                
                throw new Error(`HTTP ${response.status}`);
                
            } catch (error) {
                if (attempt === this.maxRetries - 1) {
                    throw error;
                }
                
                await new Promise(resolve => 
                    setTimeout(resolve, this.baseDelay * Math.pow(2, attempt))
                );
            }
        }
        
        return null;
    }
}
```

## Best Practices

### 1. Implement Exponential Backoff

When encountering rate limits, use exponential backoff:

```python
def calculate_backoff(attempt, base_delay=1):
    return min(300, base_delay * (2 ** attempt))  # Max 5 minutes
```

### 2. Batch Requests

Combine multiple operations when possible:

```python
# Instead of multiple single requests
for id in company_ids:
    get_company(id)  # ❌ Bad practice

# Batch requests
get_companies(company_ids)  # ✅ Good practice
```

### 3. Cache Responses

Implement response caching:

```python
from functools import lru_cache
import time

class APICache:
    def __init__(self, ttl=300):  # 5 minutes TTL
        self.cache = {}
        self.ttl = ttl

    def get(self, key):
        if key in self.cache:
            value, timestamp = self.cache[key]
            if time.time() - timestamp < self.ttl:
                return value
            del self.cache[key]
        return None

    def set(self, key, value):
        self.cache[key] = (value, time.time())
```

### 4. Monitor Rate Limits

Track your API usage:

```python
class RateLimitMonitor:
    def __init__(self):
        self.requests = []

    def track_request(self):
        current_time = time.time()
        self.requests.append(current_time)
        
        # Remove requests older than 1 minute
        self.requests = [t for t in self.requests 
                        if current_time - t <= 60]

    def can_make_request(self):
        return len(self.requests) < 600
```

### 5. Implement Request Queuing

Queue requests when approaching limits:

```python
from queue import Queue
import threading

class RequestQueue:
    def __init__(self):
        self.queue = Queue()
        self.worker = threading.Thread(target=self._process_queue)
        self.worker.daemon = True
        self.worker.start()

    def _process_queue(self):
        while True:
            request = self.queue.get()
            if request is None:
                break
            self._execute_request(request)
            self.queue.task_done()

    def _execute_request(self, request):
        # Implement request execution logic
        pass

    def add_request(self, request):
        self.queue.put(request)
```

## Common Pitfalls

1. **Not Checking Rate Limit Headers**
   ```python
   # Bad: Ignoring rate limits
   response = requests.get(url)

   # Good: Check remaining requests
   remaining = int(response.headers['X-RateLimit-Remaining'])
   if remaining < 10:
       time.sleep(1)  # Slow down requests
   ```

2. **Aggressive Retry Logic**
   ```python
   # Bad: Immediate retry
   while True:
       try:
           make_request()
       except RateLimitError:
           continue  # ❌ Too aggressive

   # Good: Implement backoff
   for attempt in range(max_retries):
       try:
           make_request()
       except RateLimitError:
           wait_time = calculate_backoff(attempt)
           time.sleep(wait_time)  # ✅ Proper backoff
   ```

3. **Not Handling Burst Limits**
   ```python
   # Bad: Parallel requests without control
   with ThreadPoolExecutor() as executor:
       executor.map(make_request, urls)  # ❌ No rate control

   # Good: Controlled parallelism
   semaphore = threading.Semaphore(3)  # Limit concurrent requests
   with ThreadPoolExecutor() as executor:
       futures = []
       for url in urls:
           with semaphore:
               futures.append(executor.submit(make_request, url))
   ```

## Monitoring and Alerts

Set up monitoring for rate limit issues:

```python
class RateLimitAlert:
    def __init__(self, threshold=0.8):  # Alert at 80% usage
        self.threshold = threshold
        self.alerts = []

    def check_rate_limit(self, remaining, limit):
        usage = (limit - remaining) / limit
        if usage >= self.threshold:
            self.alert(f"Rate limit threshold exceeded: {usage:.1%}")

    def alert(self, message):
        # Implement alert logic (e.g., logging, email, Slack)
        self.alerts.append({
            'message': message,
            'timestamp': time.time()
        })
