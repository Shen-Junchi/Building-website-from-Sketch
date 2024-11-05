# Building-website-from-Sketch
This is an example of how to build your own website. As a example from ChatGPT. 

# To build a website, we need to prepare some elements: 
## we focus on design  (HTML, CSS, JavaScript and Python)
### First we start with backend, which is from Python 

```
# Python
from flask import Flask, render_template, request, jsonify
import os
from dotenv import load_dotenv
import dashscope

app = Flask(__name__)
load_dotenv()

# 配置文件路径
CONFIG_FILE = "config.env"

def save_api_key(api_key):
    with open(CONFIG_FILE, "w") as f:
        f.write(f"DASHSCOPE_API_KEY={api_key}")
    load_dotenv(CONFIG_FILE)
    dashscope.api_key = api_key

@app.route("/")
def home():
    return render_template("index.html")

@app.route("/save_config", methods=["POST"])
def save_config():
    data = request.json
    api_key = data.get("api_key")
    if api_key:
        save_api_key(api_key)
        return jsonify({"status": "success"})
    return jsonify({"status": "error", "message": "No API key provided"})

@app.route("/chat", methods=["POST"])
def chat():
    if not dashscope.api_key:
        return jsonify({"error": "Please configure your API key first"})
    
    data = request.json
    message = data.get("message")
    
    try:
        messages = [{'role': 'user', 'content': message}]
        response = dashscope.Generation.call(
            model='qwen-turbo',
            messages=messages,
            result_format='message',  # 设置结果格式为消息
        )
        
        if response.status_code == 200:
            return jsonify({"response": response.output.choices[0].message.content})
        else:
            return jsonify({"error": f"API Error: {response.code}"})
            
    except Exception as e:
        return jsonify({"error": str(e)})

if __name__ == "__main__":
    app.run(debug=True)
```

### Let's seperate it one by one: 

1. import package part： 
  1. os: find accurate location of specific file
  2. flask:  framework for website
  3. dotenv: It is a package to load environment and get data
  4. Dashscope: It is for get request from Qwen.

2. Run the app and load code:
```
app = Flask(__name__)
load_dotenv()
```
The first order is to run the app and the second order is to get the key for Both SQL and API-key
One thing we need to mention here is `__name__`. 
In the next part we will mention
```
def home():
    return render_template('index.html')

if __name__ == '__main__':
    app.run(debug=True)
```
here by this way we find the location of `index.html`, and then find some related sources.
```
CONFIG_FILE = "config.env"
```
Find the location of config files
---
```
def save_api_key(api_key):
    with open(CONFIG_FILE, "w") as f:
        f.write(f"DASHSCOPE_API_KEY={api_key}")
    load_dotenv(CONFIG_FILE)
    dashscope.api_key = api_key
```
This part is to save the api key to tehe `config.env` file. First, we open the CONFIG_FILE and put the api-key into the virtual environment 
Then we load the environment and submit the api-key to variable to be used soon  
---
```
def save_config():
    data = request.json
    api_key = data.get("api_key")
    if api_key:
        save_api_key(api_key)
        return jsonify({"status": "success"})
    return jsonify({"status": "error", "message": "No API key provided"})
```
This part is to receive the request.json file from frontend, like in ChatGPT, you do input `how are you`, then you get a json be like: 
```
{
    "message": "how are you?"
}
```
And this is assign to variable data, then the api_key variable get api-key from data. If api-key exist, then print(I get it )
