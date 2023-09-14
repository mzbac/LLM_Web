This project demonstrates how to securely host a Hugging Face language model using text-generation-inference and Nginx with a self-signed SSL certificate.

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
        "name": "mzbac/CodeLlama-34b-guanaco-gptq",
        "datasetName": "CodeLlama-34b-guanaco-gptq",
        "description": "A good alternative to ChatGPT",
        "websiteUrl": "https://open-assistant.io",
        "userMessageToken": "### Human:",
        "assistantMessageToken": "### Assistant:",
        "userMessageEndToken": "\n",
        "assistantMessageEndToken": "\n",
        "preprompt": "Below are a series of dialogues between various people and an AI assistant. The AI tries to be helpful, polite, honest, sophisticated, emotionally aware, and humble-but-knowledgeable. The assistant is happy to help with almost anything, and will do its best to understand exactly what is needed. It also tries to avoid giving false or misleading information, and it caveats when it isn't entirely sure about the right answer. That said, the assistant is practical and really does its best, and doesn't let caution get too much in the way of being useful.\n-----\n",
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
            "repetition_penalty": 1.2,
            "top_k": 50,
            "truncate": 1024,
            "max_new_tokens": 2048,
            "stop": ["### Human"]
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

