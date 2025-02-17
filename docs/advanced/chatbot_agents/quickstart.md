# Developing Chat Functions: Getting Started

## What is a Chat Function?

It's a function which calls Large Language Models (LLMs) to respond to the student's messages given contextual data:

- question data
- user data such as past responses to the problem

Chat functions host a chatbot agent. Chatbot Agents capture and automate the process of assisting students during their learning process when outside of classroom.

## Getting Setup for Development

1. Get the code on your local machine (Using github desktop or the `git` cli)

	- For new functions: clone the template repo for [chat-function-boilerplate](https://github.com/lambda-feedback/chat-function-boilerplate). **Make sure the new repository is set to public (it needs access to organisation secrets)**. 
	- For existing functions: please make your changes on a new separate branch

2. _If you are creating a new chatbot agent_, can either edit the `scr/agents/base_agent` or copy it and rename it based on your agent's name.
3. You are now ready to start making changes and implementing features by editing each of the main function-logic files:

	1. **`scr/agents/{base_agent}/{base}_agent.py`**: This file contains the main LLM pipeline using [LangGraph](https://langchain-ai.github.io/langgraph/) and [LangChain](https://python.langchain.com/docs/introduction/).

   	- the agent expects the following inputs when it being called:

   	Body with necessary Params:

   	```JSON
   	{
   		"message":"hi",
   		"params":{
   				"conversation_id":"12345Test",
   				"conversation_history": [{"type":"user","content":"hi"}]
   		}
   	}
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

   2. **`scr/agents/{base_agent}/{base}_prompts.py`**: This is where you can write the system prompts that describe how your AI Assistant should behave and respond to the user.

   3. _If you edited the agent file name_, make sure to add your agent `invoke()` function to the `module.py` file.

	 4. Update the `config.json` file with the chatbot's name.

   5. Please add a `README.md` file to describe the use and behaviour of your agent.

4. Changes can be tested locally by running the pipeline tests using:
	```bash
	pytest src/module_test.py
	```
   [Running and Testing Agents Locally](local.md){ .md-button }


5. Merge commits into dev branch will trigger the `dev.yml` workflow, which will build the docker image, push it to a shared `dev` ECR repository and deploy an AWS Lambda function for the `dev` client app to call. In order to make your new chatbot available on the `dev` environment of the Lambda Feedback platform, you will have to get in contact with the ADMINS on the platform.

6. You can now test the deployed chat function using your preferred request client (such as [Insomnia](https://insomnia.rest/) or [Postman](https://www.postman.com/) or simply `curl` from a terminal). `DEV` Functions are made available at:
	```url
	https://57glfwwf7d.execute-api.eu-west-2.amazonaws.com/default/chat/<function name as defined in config.json>
	```

	!!! example "Example Request to chatFunctionBoilerplate-dev"
			curl --location 'https://57glfwwf7d.execute-api.eu-west-2.amazonaws.com/default/chat/chatFunctionBoilerplate-dev' \
			--header 'Content-Type: application/json' \
			--data '{
					"message": "hi",
					"params": {
							"conversation_id": "12345Test",
							"conversation_history": [
									{
											"type": "user",
											"content": "hi"
									}
							]
					}
			}'

6. Once the `dev` chatbot is fully tested, you can merge the code to the default branch (`main`). This will trigger the `main.yml` workflow, which will deploy the `staging` and `prod` versions of your chatbot. Please contact the ADMIN to provide the URLS for the `staging` and `prod` versions of your agent.

6. In order to make your new chatbot available on any of the environments of the Lambda Feedback platform, you will have to get in contact with the ADMINS on the platform.
