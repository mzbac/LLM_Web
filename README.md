This project demonstrates how to securely host a private chat model using text-generation-inference and chat-ui with self-signed SSL certificate via nginx.

## Prerequisites
- Docker
- Docker Compose
- OpenSSL

## Generate Self-Signed SSL Certificate

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout nginx-selfsigned.key -out nginx-selfsigned.crt
```

## clone chat ui 

```
git clone https://github.com/huggingface/chat-ui.git
```

Add dependency for chat-ui Dockerfile

```
RUN apt-get update && apt-get install -y libvips-dev
```

Change the base path in the svelte.config.js file in chat-ui

```
import adapter from "@sveltejs/adapter-node";
import { vitePreprocess } from "@sveltejs/kit/vite";
import dotenv from "dotenv";

dotenv.config({ path: "./.env.local" });
dotenv.config({ path: "./.env" });

process.env.PUBLIC_VERSION = process.env.npm_package_version;

/** @type {import('@sveltejs/kit').Config} */
const config = {
	// Consult https://kit.svelte.dev/docs/integrations#preprocessors
	// for more information about preprocessors
	preprocess: vitePreprocess(),

	kit: {
		adapter: adapter(),

		paths: {
			base: process.env.APP_BASE || "/chat",
		},
		csrf: {
			// handled in hooks.server.ts, because we can have multiple valid origins
			checkOrigin: false,
		},
	},
};

export default config;
```

Add the `.env.local` file in the chat-ui

```
MONGODB_URL=mongodb://mongodb:27017/
MODELS=`[
    {
        "name": "TheBloke/deepseek-coder-33B-instruct-AWQ",
        "datasetName": "deepseek-ai/deepseek-coder-33b-instruct",
        "description": "A good alternative to ChatGPT",
        "websiteUrl": "https://huggingface.co/deepseek-ai/deepseek-coder-33b-instruct",
        "userMessageToken": "### Instruction: \n",
        "assistantMessageToken": "### Response: \n",
        "userMessageEndToken": "\n",
        "assistantMessageEndToken": "\n",
        "preprompt": "You are an AI programming assistant, utilizing the Deepseek Coder model, developed by Deepseek Company, and you only answer questions related to computer science. For politically sensitive questions, security and privacy issues, and other non-computer science questions, you will refuse to answer.\n",
        "promptExamples": [
            {
                "title": "Write an email from bullet list",
                "prompt": "As a restaurant owner, write a professional email to the supplier to get these products every week: \n\n- Wine (x10)\n- Eggs (x24)\n- Bread (x12)"
                }, {
                "title": "Code a snake game",
                "prompt": "Code a basic snake game in python, give explanations for each step."
                }, {
                "title": "Assist in a task",
                "prompt": "How do I make a delicious lemon cheesecake?"
            }
        ],
        "parameters": {
            "temperature": 0.7,
            "top_p": 0.95,
            "repetition_penalty": 1.1,
            "top_k": 50,
            "truncate": 1512,
            "max_new_tokens": 1024,
            "stop": ["### Instruction:"]
        },
        "endpoints": [{"url": "http://text-generation-inference:80"}]
    }
]`

```

## create volume for mongo db file
```
mkdir mongo
```


## Usage
Start the services:
```
docker-compose up --build
```

