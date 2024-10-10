# Evaluation Function Specification

## Introduction and Philosophy
Functionality for each evaluation function is split up as follows: 

!!! note ""
    Universal function behaviour applicable to *every* function, such as the ability to run tests, return documentation and execute the evaluation is handled by the [**Base Layer**](#base-layer). This is the docker image which is extended by every developed evaluation function.

!!! abstract ""
    Functionality that may be required in more than one function (but not necessarily all), such as the ability to call already deployed functions and error reporting is handled by the [**evaluation_function_utils**](module.md) python package. This package comes pre-installed in the base layer, and can optionally be imported and called from the *evaluation_function*.

!!! info ""
    Finally, specific comparison logic and handling of bespoke evaluation parameters is done in the custom [**evaluation_function**](#the-evaluation_function), unique to each deployed instance. This is the logic that differenciates each function (comparing numbers, matrices, images, equations, graphs, text, tables, etc ...).
 

## Commands
Commands are handled by the [base layer](#base-layer). They define a unified interface for interacting with all deployed evaluation functions on the web. Practically, these are specified in the "command" request header.

!!! example 
    To execute the `docs-user` command for a function, the following header would be specified alonside the http request made to the endpoint on which the function is made available: 

    ```bash 
    curl --request GET \
    --url https://c1o0u8se7b.execute-api.eu-west-2.amazonaws.com/default/isExactEqual \
    --header 'command: docs-user' 
    ```

### `eval`
This is the default command, used to compare a student's `response` and correct `answer`, given certain `params`. Outputs for this command depend on the success of the execution of the user-defined [`evaluation_function`](#the-evaluation_function). If an error was thrown during execution, it is caught by the main handler and an error block is returned - otherwise, successful execution outputs are supplied under a `result` field. 

!!! success "Output Structure: Successful evaluation"

    ``` { .python .annotate }
    {
        "command": "eval",
        "result": {
            "is_correct": "<bool>",

            # Optional fields added by feedback generation (1)
            "feedback": "<string>",
            "warnings": "<array>"

            # This output can also contain any number of fields given by `evaluation_function`
        }
    }
    ```

    1. See the [Feedback Page](feedback.md) for more information


!!! fail "Output Structure: Error thrown during Execution"

    ``` { .python .annotate }
    {
        "command": "eval",
        "error": {
            "message": "<string>", # Always present

            # This object can contain other number of additional fields
            # passed through by the EvaluationException (1) for debugging e.g.:
            "serialization_errors": [],
            "culprit": "user",
            "detail": "..."
        }
    }
    ```

    1.    This is a custom error class from the [evaluation-function-utils](module.md) package, which developers are encouraged to use in order to output richer errors. See the [Error handling](#error-handling) section for more information.


### `healthcheck`
This command runs and returns a summary three testing suites: requests, responses and evaluation. Request and response tests check that inputs and outputs to the function work correctly, and follow the correct syntax. Evaluation tests are unique to each evaluation function and test the actual comparison logic.

### `docs-user`
Command returns the `docs/user.md` file (base64 encoded)

### `docs-dev`
Command returns the `docs/dev.md` file (base64 encoded)


## Base Layer 


## File Structure
A standard evaluation function repository based on the provided [boilerplate](https://github.com/lambda-feedback/Evaluation-Function-Boilerplate) will have the following file structure:
```bash
app/
    __init__.py
    evaluation.py # Script containing the main evaluation_function
    evaluation_tests.py # Unittests for the main evaluation_function
    requirements.txt # list of packages needed for algorithm.py
    Dockerfile # for building whole image to deploy to AWS

    docs/ # Documentation pages for this function (required)
        dev.md # Developer-oriented documentation
        user.md # LambdaFeedback content author documentation

.github/
    workflows/
        test-and-deploy.yml # Testing and deployment pipeline

config.json # Specify the name of the evaluation function in this file
README.md
.gitignore
```

!!! warning
    
    If you want to split up function logic into different files, these must be added to the `Dockerfile`. This is so they are packaged with the built image when deployed. For example, if `evaluation.py` imports functionality from an `app/utils.py` file, then the following line must be added: 

    ```dockerfile linenums="9" hl_lines="7 8"
    RUN pip3 install -r requirements.txt

    # Copy the evaluation and testing scripts
    COPY evaluation.py ./app/
    COPY evaluation_tests.py ./app/

    # Copy additional files
    COPY utils.py ./app/

    # Copy Documentation
    COPY docs/dev.md ./app/docs/dev.md
    ```


## `evaluation.py`
The entire framework, validation and testing developed around evaluation functions is ultimately used to get to this file, or the `evaluation_function` function within it, to be more precise. 

### The `evaluation_function`

#### Inputs
- `response`: Data input by the user 
- `answer`: Data to compare user input to (could be from a DB of answers, or pre-generated by other functions)
- `params`: Parameters which affect the comparison process (replacements, tolerances, feedbacks, ...)

#### Inputs appended to input `params`
When a student submits a response to a response area the number of previously submitted responses submitted to the same response area byt the same student will be sent to the evaluation function. The following format is used:
    ``` { .python .annotate }
    {
        "submission_context": {
            "submissions_per_student_per_response_area": # non-negative integer that represent the nubmer of previously processed responses
        }
    }
    ```

#### Outputs
The function should output a single JSON-encodable dictionary. Although a large amount of freedom is given to what this dict contains, when utilising the function alongside the [lambdafeedback](https://lambdafeedback.com/) web app, a few values are expected/able to be consumed:

**`is_correct: <bool>`**: Boolean parameter indicate whether the comparison between `response` and `answer` was deemed correct under the parameters. This field is then used by the web app to provide the most simple feedback to the user (green/red).

!!! info 
    *More standardised function outputs that the frontend can consume are to come*

### Error Handling
Error reporting should follow a specific approach for all evaluation functions. **If the `evaluation_function` you've written doesn't throw any errors, then it's output is returned under the `result` field - and assumed to have worked properly**. This means that if you catch an error in your code manually, and simply return it - the frontend will assume everything went fine. Instead, errors can be handled in two ways:

**Letting `evaluation_function` fail**: On the request handler in the [Base Layer](#base-layer), the call to evaluation_function is wrapped in a try/except which catches any exception. This causes the evaluation to stop completely, returning a standard message, and a repr of the exception thrown in the `error.detail` field.

**Custom errors**: If you want to report more detailed errors from your function, use the `EvaluationException` class provided in the [evaluation-function-utils](module.md#errors) package. These are caught before all other standard exceptions, and are dealt with in a different way. These provide a way for your function to throw errors and stop executing safely, while supplying more accurate feedback to the front-end. 

!!! Example
    It is discouraged to do the following in the evaluation code:
    ```python
    if something.bad.happened():
        return {
            "error": {
                "message": "Some important message",
                "other": "details",
            }
        }
    ```

    As this causes the actual function output (by the AWS lambda function) to be:
    ```json 
    {
        "command": "eval",
        "result": {
            "error": {
                "message": "Some important message",
                "other": "details"
            }
        }
    }
    ```

    Instead, use custom exceptions from the [evaluation-function-utils](module.md#errors) package.
    ```python
    if something.bad.happened():
        raise EvaluationException(message="Some important message", other='details')
    ```

    As the actual function output will look like:
    ```json 
    {
        "command": "eval",
        "error": {
            "message": "Some important message",
            "other": "details"
        }
    }
    ```

    This immediately indicates to the frontend client that something has gone wrong, allowing for proper feedback to be displayed.

## `evaluation_tests.py`


## Documentation
Two essential and required documentation files are copied over during the creation of the evaluation function docker image. These are subsequently served by the function under the `docs-dev` and `docs-user` commands, to be accessed by this documentation website, as well as for embedding on LambdaFeedback. For more information about the markdown syntax, please refer to the following sources:

- [MkDocs Documentation](https://www.mkdocs.org/user-guide/writing-your-docs/#writing-your-docs)
- [MkDocs-Material Documentation](https://squidfunk.github.io/mkdocs-material/reference/)

### `docs/dev.md`


### `docs/user.md`