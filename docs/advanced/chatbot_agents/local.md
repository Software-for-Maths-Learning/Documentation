# Running and Testing Agents Locally

You can run the Python function for your agent itself by writing a `main()` function, or you can call the [`testbench_prompts.py`](https://github.com/lambda-feedback/lambda-chat/blob/main/src/agents/utils/testbench_prompts.py) script that runs a similar pipeline to the `module.py`.

```bash
python src/agents/utils/testbench_prompts.py
```

You can also use the `test_prompts.py` script to test the agents with example inputs from Lambda Feedback questions and synthetic conversations.
```bash
python src/agents/utils/test_prompts.py
```

## Testing using the Docker Image [:material-docker:](https://www.docker.com/)

You can also build and run the docker pipeline for the agents. The chatbot agents are deployed onto a AWS Lambda serverless cloud function using the docker image. Hence, for final testing of your chatbots, we recommend completing those steps.

#### Build the Docker Image

To build the Docker image, run the following command in the root folder of the project (where the Dockerfile is located):

```bash
docker build -t llm_chat .
```

### Running the Docker Image

To run the Docker image, use the following command:

#### Without .env file:

```bash
docker run -e OPENAI_API_KEY={your key} -e OPENAI_MODEL={your LLM chosen model name} -p 8080:8080 llm_chat
```

#### With container name (for interaction, e.g. copying file from inside the docker container):

```bash
docker run --env-file .env -it --name my-lambda-container -p 8080:8080 llm_chat
```

This will start the evaluation function and expose it on port `8080` and it will be open to be curl:

```bash
curl --location 'http://localhost:8080/2015-03-31/functions/function/invocations' \
--header 'Content-Type: application/json' \
--data '{"body":"{\"message\": \"hi\", \"params\": {\"conversation_id\": \"12345Test\", \"conversation_history\": [{\"type\": \"user\", \"content\": \"hi\"}]}}"}'
```

### Call Docker Container From Postman

POST URL:

```bash
http://localhost:8080/2015-03-31/functions/function/invocations
```

Body:

```JSON
{"body":"{\"message\": \"hi\", \"params\": {\"conversation_id\": \"12345Test\", \"conversation_history\": [{\"type\": \"user\", \"content\": \"hi\"}]}}"}
```

Body with optional Params:
```JSON
{
    "message":"hi",
    "params":{
        "conversation_id":"12345Test",
        "conversation_history":[{"type":"user","content":"hi"}],
        "summary":" ",
        "conversational_style":" ",
        "question_response_details": "",
        "include_test_data": true,
        "agent_type": {agent_name}
    }
}
```