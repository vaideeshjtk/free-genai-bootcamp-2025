# How The ChatBot works 
Here’s how your Docker Compose and Python code interact to run your application:

Step 1: Start the Ollama Server
When you run docker-compose up, the ollama-server container starts.

The Ollama server listens on port 11434 inside the container, which is mapped to port 8008 on your host machine.

Step 2: Run Your Python Application
Your Python application (ExampleService) starts and initializes the ServiceOrchestrator.

The add_remote_service method adds the embedding and llm microservices.

The llm microservice is configured to communicate with the Ollama server at http://host_ip:8008.

Step 3: Handle User Requests
User Sends a Request:

A user sends a POST request to your application’s endpoint (/v1/example-service) with a prompt (e.g., "Explain quantum physics").

Request Processing:

The handle_request method formats the request for Ollama:

python
Copy
ollama_request = {
    "model": "llama3.2:1b",
    "messages": [{"role": "user", "content": "Explain quantum physics"}],
    "stream": False
}
Send Request to Ollama:

The request is sent to the Ollama server via the ServiceOrchestrator:

python
Copy
result = await self.megaservice.schedule(ollama_request)
Ollama Generates a Response:

The Ollama server processes the request and generates a response (e.g., "Quantum physics is the study of...").

Return Response to User:

The response is extracted, formatted, and sent back to the user:

python
Copy
response = ChatCompletionResponse(
    model="llama3.2:1b",
    choices=[
        ChatCompletionResponseChoice(
            index=0,
            message=ChatMessage(role="assistant", content="Quantum physics is the study of..."),
            finish_reason="stop"
        )
    ],
    usage=UsageInfo(prompt_tokens=0, completion_tokens=0, total_tokens=0)
)
4. Background Workflow
Here’s a high-level overview of how everything works in the background:

User Interaction:

The user interacts with your application (e.g., via a web interface or API).

Request Flow:

The request is sent to your Python application.

The application processes the request and sends it to the Ollama server.

Ollama Processing:

The Ollama server generates a response using the specified model (e.g., llama3.2:1b).

Response Flow:

The response is sent back to your Python application.

The application formats the response and sends it back to the user.

5. Example Workflow
Let’s say a user asks, "What is AI?" Here’s what happens:

User Request:

The user sends a POST request with the prompt: {"messages": "What is AI?"}.

Python Application:

The handle_request method formats the request and sends it to Ollama.

Ollama Server:

Ollama generates the response: "AI is the simulation of human intelligence by machines."

Response to User:

The Python application sends the response back to the user.

6. How to Build and Run Your Application
Start Ollama Server:

Run docker-compose up to start the Ollama server.

Run Python Application:

Run your Python code (ExampleService). It will start listening for requests.

Send Requests:

Use a tool like curl or Postman to send POST requests to your application’s endpoint (/v1/example-service).

View Responses:

The application will process the requests, interact with Ollama, and return the responses.

7. Summary
Docker Compose: Runs the Ollama server in a container.

Python Code: Manages microservices, processes requests, and interacts with Ollama.

Interaction:

User → Python Application → Ollama Server → Python Application → User.

## Running Ollama Third party service container
### This document describes how to run the Ollama third party service container

### Prerequisites
- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- [Ollama] (http://ollama.com/library/)
- [IP] run (apt install net-tools)

### ENV
LLM_ENDPOINT_PORT=8008 \
no_proxy=localhost,127.0.0.1 \
http_proxy=http://your-proxy:8080 \
https_proxy=https://your-proxy:8443 \
LLM_MODEL_ID=ollama run llama3.2:1b \
host_ip=192.168.1.100 \
docker compose up -d

### Ollama API
once the ollama server is running we can make calls to the ollama API 
[Ollama API](https://github.com/ollama/ollama/blob/main/docs/api.md)

### Pull Request
```
sh
curl http://localhost:9000/api/pull -d '{ "model": "llama3.2:1b" }'
```
### Generate a request 
```
sh
curl http://localhost:8008/api/generate -d '{
  "model": "llama3.2:1b",
  "prompt": "Why is the sky blue?"
}'
```


### Technical Uncertainity
Q 1. Does bridge mode means we can access Ollama API with another docker container or model with docker compose? 
A: No, the host machine can access API

Q 2. To get port details?
A. Docker ps reveals 8008->11434/tcp
Hence 8008 is beeing used

Q 3. If we pass the Ollama Model Id  will it download?
A. We need call the model it manually not on start

Q 4.Will the model be donwloaded in the container ? So will the model be deleted when the contianer be deleted.
A.A: Yes, the model will be downloaded inside the container. When you pull a model using the Ollama API, it is stored within the container's filesystem. Therefore, if the container is deleted, the model will also be deleted along with it. To persist the model data, you should consider using Docker volumes to store the model outside the container's lifecycle. This way, even if the container is deleted, the model data will remain intact.

Here is an example of how you can modify your docker-compose.yml to use a volume for storing the model data:

```
sh
services:
  ollama-server:
    image: ollama/ollama
    container_name: ollama-server
    ports:
      - ${LLM_ENDPOINT_PORT:-8008}:11434
    environment:
      no_proxy: ${no_proxy}
      http_proxy: ${http_proxy}
      https_proxy: ${https_proxy}
      LLM_MODEL_ID: ${LLM_MODEL_ID}
      host_ip: ${host_ip}
    volumes:
      - ollama-models:/path/to/model/storage

volumes:
  ollama-models:
```
### Pull a Model
POST /api/pull
Download a model from the ollama library. Cancelled pulls are resumed from where they left off, and multiple calls will share the same download progress.

#### Parameters
model: name of the model to pull
insecure: (optional) allow insecure connections to the library. Only use this if you are pulling from your own library during development.
stream: (optional) if false the response will be returned as a single response object, rather than a stream of objects
#### Examples
##### Request
```
sh
curl http://localhost:8008/api/pull -d '{
  "model": "llama3.2:1b"
}'
```

Q5: The Ollama, vLLM and TGI use same API?
A: Yes same API Endpoint


