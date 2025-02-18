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


# Output 
(venv) root@INBookX1:/mnt/c/Users/thiru/Downloads/Vaideesh/Courses/GenAI/free-genai-bootcamp-2025-main/free-genai-bootcamp-2025-main/opea-comps/Mega-service# curl http://localhost:8008/api/generate -d '{
  "model": "llama3.2:1b",
  "prompt": "Why is the sky blue?"
}'
{"model":"llama3.2:1b","created_at":"2025-02-18T14:17:57.975556805Z","response":"The","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:17:58.106680049Z","response":" sky","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:17:58.227410899Z","response":" appears","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:17:58.357138381Z","response":" blue","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:17:58.496649087Z","response":" to","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:17:58.590263338Z","response":" us","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:17:58.678036824Z","response":" because","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:17:58.781690912Z","response":" of","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:17:58.869846796Z","response":" a","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:17:58.970033106Z","response":" phenomenon","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:00.329779284Z","response":" called","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:00.495556003Z","response":" Ray","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:00.597351414Z","response":"leigh","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:00.684724088Z","response":" scattering","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:00.822128063Z","response":",","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:00.958233177Z","response":" named","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:01.105073797Z","response":" after","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:01.297454512Z","response":" the","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:01.47491173Z","response":" British","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:01.58372623Z","response":" physicist","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:01.754009847Z","response":" Lord","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:01.907681958Z","response":" Ray","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:02.072519168Z","response":"leigh","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:02.240504322Z","response":".","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:02.433499159Z","response":" He","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:02.760757631Z","response":" discovered","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:03.013748524Z","response":" that","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:03.285657455Z","response":" when","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:06.413027712Z","response":" sunlight","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:08.899853599Z","response":" enters","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:09.759156836Z","response":" Earth","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:10.562855477Z","response":"'s","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:11.973207386Z","response":" atmosphere","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:12.300278881Z","response":",","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:12.48368438Z","response":" it","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:13.152588483Z","response":" encounters","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:13.51572398Z","response":" tiny","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:13.811990623Z","response":" molecules","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:13.974736119Z","response":" of","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:15.232041638Z","response":" gases","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:15.502369585Z","response":" such","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:17.169273241Z","response":" as","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:17.610102532Z","response":" nitrogen","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:17.905573658Z","response":" and","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:18.16995075Z","response":" oxygen","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:18.590223958Z","response":".","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:18.78289406Z","response":" These","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:19.462949107Z","response":" molecules","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:20.187328958Z","response":" scatter","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:20.380165243Z","response":" the","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:20.576202926Z","response":" light","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:21.425862466Z","response":" in","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:21.771735727Z","response":" all","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:21.897741875Z","response":" directions","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:22.063558603Z","response":",","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:22.162582367Z","response":" but","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:22.274751811Z","response":" they","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:22.386637041Z","response":" scatter","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:22.496341638Z","response":" shorter","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:22.64447555Z","response":" (","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:22.754061433Z","response":"blue","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:22.848238281Z","response":")","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:22.944421899Z","response":" wavelengths","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:23.046748792Z","response":" more","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:23.145910967Z","response":" than","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:23.274929594Z","response":" longer","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:23.394868569Z","response":" (","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:23.512322019Z","response":"red","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:23.780332688Z","response":")","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:24.878102463Z","response":" wavelengths","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:24.987744655Z","response":".\n\n","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:25.090203466Z","response":"This","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:25.253037655Z","response":" is","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:25.465318195Z","response":" because","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:25.625578995Z","response":" the","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:25.798951933Z","response":" smaller","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:25.868255649Z","response":" molecules","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:25.981197927Z","response":" are","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:26.132148066Z","response":" more","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:26.315090769Z","response":" effective","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:26.488290404Z","response":" at","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:26.673033006Z","response":" scattering","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:26.847866338Z","response":" the","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:26.956320074Z","response":" blue","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:27.06912738Z","response":" light","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:27.202142931Z","response":",","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:27.36049291Z","response":" which","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:27.550710087Z","response":" has","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:27.956558374Z","response":" a","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:28.11450045Z","response":" lower","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:28.252654608Z","response":" energy","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:28.423814233Z","response":" level","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:28.558815905Z","response":".","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:28.690113362Z","response":" As","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:28.822752592Z","response":" a","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:28.94690151Z","response":" result","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:29.056089158Z","response":",","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:29.220520774Z","response":" the","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:29.329678246Z","response":" blue","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:29.447332662Z","response":" light","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:29.577298832Z","response":" is","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:29.694600378Z","response":" distributed","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:29.796795047Z","response":" throughout","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:29.91285499Z","response":" the","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:30.020881878Z","response":" atmosphere","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:30.126913107Z","response":",","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:30.270798628Z","response":" giving","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:30.371062951Z","response":" the","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:30.501239597Z","response":" sky","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:30.638599533Z","response":" its","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:30.780603053Z","response":" blue","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:30.904985672Z","response":" appearance","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:31.030547659Z","response":".\n\n","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:31.145106979Z","response":"The","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:31.267929889Z","response":" amount","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:31.371472395Z","response":" of","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:31.507232517Z","response":" scattering","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:31.626920312Z","response":" that","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:31.754467081Z","response":" occurs","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:31.878903435Z","response":" depends","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:31.976838966Z","response":" on","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:32.078807706Z","response":" several","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:32.498900763Z","response":" factors","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:32.693731838Z","response":",","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:32.806076351Z","response":" including","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:32.895682072Z","response":" the","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:34.493942474Z","response":" concentration","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:34.666113862Z","response":" of","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:35.012980696Z","response":" atmospheric","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:35.181622906Z","response":" gases","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:35.286746674Z","response":",","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:35.364549283Z","response":" the","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:36.048755028Z","response":" altitude","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:36.491458894Z","response":" of","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:36.567791944Z","response":" the","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:36.657679823Z","response":" sun","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:36.759590663Z","response":",","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:36.895571669Z","response":" and","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:37.048665815Z","response":" the","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:37.159289131Z","response":" angle","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:37.256230816Z","response":" of","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:37.400780292Z","response":" the","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:37.494056431Z","response":" sunlight","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:37.572810296Z","response":".","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:37.741483716Z","response":" At","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:37.924822618Z","response":" sea","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:38.013348633Z","response":" level","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:38.098901665Z","response":",","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:38.207145701Z","response":" the","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:38.296514558Z","response":" sky","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:38.380437335Z","response":" appears","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:38.469147866Z","response":" blue","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:38.553446326Z","response":" because","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:38.849706299Z","response":" the","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:39.180510642Z","response":" sun","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:39.330622611Z","response":"'s","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:39.454089362Z","response":" rays","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:39.722955869Z","response":" travel","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:39.858875576Z","response":" through","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:39.955205471Z","response":" a","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:40.072366689Z","response":" relatively","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:40.176592863Z","response":" clear","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:40.277529649Z","response":" atmosphere","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:40.37167731Z","response":" with","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:40.498540351Z","response":" few","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:40.611600369Z","response":" particles","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:40.695273558Z","response":" to","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:40.793856553Z","response":" scatter","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:40.94234774Z","response":" the","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:41.08410562Z","response":" light","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:41.192882099Z","response":".","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:41.30987217Z","response":" However","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:41.448978687Z","response":",","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:41.551219597Z","response":" as","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:41.674105587Z","response":" you","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:41.794197066Z","response":" go","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:41.898499085Z","response":" higher","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:41.998658401Z","response":" or","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:42.10361428Z","response":" lower","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:42.270076007Z","response":" in","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:42.386009507Z","response":" the","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:42.512200205Z","response":" atmosphere","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:42.63256775Z","response":",","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:42.744228813Z","response":" the","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:42.851985843Z","response":" scattering","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:42.960926967Z","response":" effect","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:43.056519639Z","response":" decreases","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:43.164066322Z","response":",","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:43.256075031Z","response":" and","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:43.350481061Z","response":" the","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:43.458365523Z","response":" sky","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:43.556706031Z","response":" may","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:43.654341223Z","response":" appear","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:43.751971913Z","response":" more","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:43.851817215Z","response":" red","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:43.934816359Z","response":" or","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:44.086601732Z","response":" orange","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:44.296704501Z","response":".\n\n","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:44.389547216Z","response":"It","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:44.545073261Z","response":"'s","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:44.640596006Z","response":" worth","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:44.747933012Z","response":" noting","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:44.871138669Z","response":" that","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:44.951063663Z","response":" the","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:45.039813567Z","response":" color","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:45.125495093Z","response":" of","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:45.210503675Z","response":" the","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:45.309992307Z","response":" sky","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:45.403112531Z","response":" can","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:45.507542457Z","response":" also","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:45.590518195Z","response":" be","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:45.694858144Z","response":" affected","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:45.811064976Z","response":" by","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:45.891623859Z","response":" other","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:45.990698617Z","response":" factors","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:46.10189911Z","response":",","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:46.193238633Z","response":" such","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:46.279450061Z","response":" as","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:46.379326364Z","response":" dust","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:46.510077333Z","response":",","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:46.610711578Z","response":" water","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:46.714239707Z","response":" vapor","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:46.794852733Z","response":",","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:46.894148708Z","response":" and","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:47.008152711Z","response":" pollution","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:47.128932825Z","response":" in","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:47.2248278Z","response":" the","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:47.326346964Z","response":" air","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:47.436458739Z","response":".","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:47.578929635Z","response":" However","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:47.744325545Z","response":",","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:47.833593703Z","response":" Ray","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:47.937125137Z","response":"leigh","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:48.051106253Z","response":" scattering","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:48.144199307Z","response":" is","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:48.232013364Z","response":" the","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:48.33920689Z","response":" primary","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:48.466954255Z","response":" reason","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:48.796598116Z","response":" why","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:48.913556331Z","response":" the","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:49.003918618Z","response":" sky","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:49.104088775Z","response":" appears","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:49.206908868Z","response":" blue","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:49.301002189Z","response":" to","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:49.405703858Z","response":" us","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:49.500240781Z","response":".","done":false}
{"model":"llama3.2:1b","created_at":"2025-02-18T14:18:49.624272533Z","response":"","done":true,"done_reason":"stop","context":[128006,9125,128007,271,38766,1303,33025,2696,25,6790,220,2366,18,271,128009,128006,882,128007,271,10445,374,279,13180,6437,30,128009,128006,78191,128007,271,791,13180,8111,6437,311,603,1606,315,264,25885,2663,13558,64069,72916,11,7086,1306,279,8013,83323,10425,13558,64069,13,1283,11352,430,994,40120,29933,9420,596,16975,11,433,35006,13987,35715,315,45612,1778,439,47503,323,24463,13,4314,35715,45577,279,3177,304,682,18445,11,719,814,45577,24210,320,12481,8,93959,810,1109,5129,320,1171,8,93959,382,2028,374,1606,279,9333,35715,527,810,7524,520,72916,279,6437,3177,11,902,706,264,4827,4907,2237,13,1666,264,1121,11,279,6437,3177,374,4332,6957,279,16975,11,7231,279,13180,1202,6437,11341,382,791,3392,315,72916,430,13980,14117,389,3892,9547,11,2737,279,20545,315,45475,45612,11,279,36958,315,279,7160,11,323,279,9392,315,279,40120,13,2468,9581,2237,11,279,13180,8111,6437,1606,279,7160,596,45220,5944,1555,264,12309,2867,16975,449,2478,19252,311,45577,279,3177,13,4452,11,439,499,733,5190,477,4827,304,279,16975,11,279,72916,2515,43154,11,323,279,13180,1253,5101,810,2579,477,19087,382,2181,596,5922,27401,430,279,1933,315,279,13180,649,1101,387,11754,555,1023,9547,11,1778,439,16174,11,3090,38752,11,323,25793,304,279,3805,13,4452,11,13558,64069,72916,374,279,6156,2944,3249,279,13180,8111,6437,311,603,13],"total_duration":133745833756,"load_duration":73058894279,"prompt_eval_count":31,"prompt_eval_duration":8930000000,"eval_count":247,"eval_duration":51669000000}
(venv) root@INBookX1:/mnt/c/Users/thiru/Downloads/Vaideesh/Courses/GenAI/free-genai-bootcamp-2025-main/free-genai-bootcam
p-2025-main/opea-comps/Mega-service# 