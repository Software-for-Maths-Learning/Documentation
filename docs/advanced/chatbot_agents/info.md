# AI Chatbots - More information

Chatbot agents are AI Assistants that students can chat with to ask for help or further explanations regarding the Question that they are working on. Each Agent has its own personality and approach to assisting the students.

The Chatbots have at their basis a [Large Language Model (LLM)](https://en.wikipedia.org/wiki/Large_language_model) which received information regarding:

- the details of the Question the student is on currently,
- the answer and worked solutions for each Response Area of the Question,
- the student's progress on all parts of the Question,
- the students's current previewed part,
- the guidance time and tips form the lecturer.

---

## Available Chatbots

Currently the students have access to the following AI Chatbots. Many others are in development.

1. Informational Chatbot

This chatbot aims to complete all the relevant tasks the student requests based on the current Question they are working on. The Chatbot is aware of the Question details, answer, worked solution and guidance from the lecturer.

Some technical details:
<pre style="white-space: pre-wrap;">
<code>LLM model: GPT 4o mini [from OpenAI]
response time (on average): 10 seconds

Helping approach: encourages self-discovery of the answer, but will reveal the solution if requested
</code>
</pre>

## AI Chatbot Development

Are you interested in developing your own chatbot? Then please check out the [Quickstart guide](quickstart.md) to develop and deploy your own AI chat agent for Lambda Feedback.