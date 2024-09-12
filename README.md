# How-To-Connect-OpenAI

Suppose we have two computers: **Computer A** (with access to OpenAI APIs) and **Computer B** (without access due to IP limitations). Computer A does not have a static IP. Here's a simple solution to allow Computer B to make requests via Computer A.

## Cloud Relay Approach

### Step 0: Register with Ngrok
- Register an Ngrok account at [Ngrok Dashboard](https://dashboard.ngrok.com).

### Step 1: Install and Set Up Ngrok
Follow the official Ngrok guide to install it. For macOS, run:
```bash
brew install ngrok/ngrok/ngrok
ngrok config add-authtoken [your-auth-token]
ngrok http http://localhost:8080
```
This command will provide you with a public URL to access Computer A’s port 8080.

### Step 2: Set Up a Flask App on Computer A
Create a simple Flask app to relay OpenAI requests:
```python
from flask import Flask, request, jsonify
import json
import requests

app = Flask(__name__)

def GPT4(prompt, key):
    url = "https://api.openai.com/v1/chat/completions"
    
    payload = json.dumps({
        "model": "gpt-4",
        "messages": [
            {"role": "user", "content": prompt}
        ]
    })
    
    headers = {
        'Accept': 'application/json',
        'Authorization': f'Bearer {key}',
        'Content-Type': 'application/json'
    }

    response = requests.post(url, headers=headers, data=payload)
    return response.json()

@app.route('/gpt4_request', methods=['POST'])
def gpt4_request():
    data = request.json
    prompt = data.get('prompt', '')
    key = data.get('api_key', '')

    response = GPT4(prompt, key)
    return jsonify(response)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```

### Step 3: Make a Request from Computer B
On Computer B, use the following script to make a request to the Flask app running on Computer A:
```python
import requests

# Replace 'xxxx' with the public URL provided by Ngrok
url = 'http://xxxx/gpt4_request'

data = {
    'prompt': 'your prompt here',
    'api_key': 'your-api-key'
}

response = requests.post(url, json=data)
print(response.json())
```
