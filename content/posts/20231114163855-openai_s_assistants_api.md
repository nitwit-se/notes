---
title: "Now we're cooking with GPT"
tags: [GPT,OpenAI,LLM,Agents,Cooking,Recipes,Python]
date: 2023-11-14
---

# Now we're cooking with GPT

### Simple function handling with OpenAI's Assistants API







By Mark Dixon
## Intro



![image](/20231114_192505_Firefly_AI_robot_cooking_a_lasagne._69680.jpg)

Last week OpenAI had their [DevDay](https://devday.openai.com/) where they introduced, amongst a bunch of new stuff, the [Assistants API](https://platform.openai.com/docs/assistants/overview). 

Essentially a direct competitor to [LangChain](https://www.langchain.com/) and Microsoft's [autogen](https://github.com/microsoft/autogen) this new API radically simplifies software development work when trying to build agent based solutions.

So far the documentation is sparse and the examples and sandbox environment leaves a bit to be desired. So I thought I'd write some practical notes for myself while doing some research work at [ClimateView](https://climateview.global).
## Why is this cool?

Support for **function callbacks** in this new API opens up lots of neat ways of adding gpt-4 to your scripting.

Example? Say you wanted to build up a database of recipes for your fancy new cooking app. Now you can get GPT-4 to generate any number of recipes for you. Using function specifications you can define a JSON schema for how you want the recipes to be formatted, and then handle calls to your function to store recipes in a database.

So this is cool because OpenAI now takes care of all the boring details in forcing the output of gpt-4 so that your code only has to deal with well formed JSON objects as defined by your schema. **Anything you can define as a schema, gpt-4 will comply with.**

There's much more to the Assistants API than just this - but this alone is so useful for scripting stuff that I thought I'd dedicate this note to it.

---
## Architecture

The Assistants architecture - and thus the API - consists of only four parts:

- Assistants :: the unique properties of an assistant (agent) - you'd probably
  only have one of these per app
- Threads :: each session or user would get their own thread. Threads are
  meant to be persistent and maintain a unique conversation history
- Messages :: individual messages in the dialogue (output from the model or
  input from the user)
- Runs :: each new input message triggers a run - there can be only one active
  run per thread so while a run is running the user input should be paused until
  the run is evaluated.
### Assistants

A thread must have at least one assistant connected to it so you'd typically start here. At the Assistant level you get to pick:

-  which language model (gpt-4, gpt-3.5-turbo, etc)
-  any functions you have
-  a custom system prompt

On top of this you can define a set of documents for automated retrieval and even define a code interpreter (a Python sandbox) to let the model run code for you. I won't be touching those here.

Assitants can be created using the [OpenAI platform web interface](https://platform.openai.com/assistants) or programmatically in e.g. Python:

```python
from openai import OpenAI
client = OpenAI()

assistant = client.beta.assistants.create(
    name="Recipe Generator",
    instructions="You are a chef, you compose recipes that are easy to follow.",
    tools=[] # here we could have: [{"type": "function"}],
    model="gpt-4-1106-preview"
)
```
### Threads

A thread is the hub for a particular interaction with your newly defined Assistant.

You can give a thread an optional set of messages to store or start with an empty set. Other than that a thread is really just a unique ID for a session, the rest comes from runs.

There isn't much to creating a thread:

```python
thread = client.beta.threads.create()
```
### Messages

Once you have a thread then you can create a user message using the Message object, specifying which thread, a role (e.g. "user") and the message content.

Message objects can also represent responses from the language model. 

Once you have an assistant and an active thread for conversation history then you can create a user message:

```python
message = client.beta.threads.messages.create(
    thread_id=thread.id,
    role="user",
    content="Design a simple easy to cook recipe for vegetarian lasagne. " 
            "Add it to the database then add each ingredient to the recipe "
            "and finally, add each step for cooking to the recipe."
)
```
### Runs

So far everything has been preparation work - the language model hasn't done anything yet. Now you can trigger processing of a message by starting a run. It is essentially here that the language model takes all of the messages and processes them in order to (ultimately) generate a new output message.

It is also at the run level that you specify which assistant to use - so in theory you can be smart about it and use gpt-3.5-turbo for any work where you don't think the cost of gpt-4 is justified.

If you have defined function calls then it is during a run that your functions will be "called". It is important to note here that the Assistants API doesn't actually call your functions - instead you need to poll the status of a run and look for function call requests and process them. This is kind of like a webhook in reverse.

Start by creating a run, with extra instructions to guide the user input:

```python
run = client.beta.threads.runs.create(
    thread_id=thread.id,
    assistant_id=assistant.id,
    instructions="Please address the user Chef. You must respond to any orders given by Chef with 'Yes chef!'"
)
```

And finally you need to loop and periodically check the run status until it is completed (or fails):

```python
import time
while True:
    time.sleep(1)
    run = client.beta.threads.runs.retrieve(
        thread_id=thread.id,
        run_id=run.id
    )
    if run.status in ["completed", "failed"]:
        break
```

Note: we'll improve on this loop below.

---
## Building a recipe generator

So now we have the basic building blocks let's put it all together and show how we can leverage gpt-4 to populate a little database of recipes.
### Get or create an assistant

If we repeatedly run the above code then it will create a new assistant for each run which isn't optimal. We can either clean up afterwards by deleting it or, my preferred option, a classic "get or create" operation:

```python
assistant_name = "Recipe Generator"
assistant = None

assistants = client.beta.assistants.list()
for a in assistants:
    if  a.name == assistant_name:
        assistant = a
        break

if not assistant:
    assistant = client.beta.assistants.create(
        name="Recipe Generator",
        instructions="You are a sous chef, you compose recipes that are easy to follow.",
        tools=[],
        model="gpt-4-1106-preview"
    )
```

If we want to clean up the assistant we can just call delete on it:

```python
client.beta.assistants.delete(assistant_id=assistant.id)
```
### Define some function calls

Now the assistant we created above can't actually do anything more than just answer user input so the next step is to define one or more function calls.

Function calls can either be defined during creation or added later with an update call. Note that calling update essentially "replaces" any existing functions with your new set.

Rather than create a single large JSON schema for a full recipe, I decided to break it down into three function calls:

-  create a new recipe
-  add an ingredient to a recipe
-  add a step to a recipe

I suspect that this works better with chain-of-thought reasoning and we can rely on the language model being smart enough to know which function to call and when.

```python
function_add_recipe = {
    "type": "function",
        "function": {
        "name": "addRecipe",
        "description": "Add a recipe to the database",
        "parameters": {
            "type": "object",
            "properties": {
                "id": {"type": "string", "description": "The unique snake case name of this recipe"},
                "name": {"type": "string", "description": "The unique short name of this recipe"},
                "description": {"type": "string", "description": "A brief description of this recipe"},
            },
            "required": ["id", "name", "description"]
        }
    }
}
function_add_ingredient = {
    "type": "function",
    "function": {
        "name": "addIngredient",
        "description": "Add an ingredient to a recipe",
        "parameters": {
            "type": "object",
            "properties": {
                "id": {"type": "string", "description": "The unique snake case name of this recipe"},
                "ingredient": {"type": "string", "description": "The name of the ingredient"},
                "quantity": {"type": "string", "description": "Measure of how much of this ingredient is required"},
            },
            "required": ["id", "ingredient", "quantity" ]
        }
    }
}
function_add_step = {
    "type": "function",
    "function": {
        "name": "addStep",
        "description": "Add a cooking step to a recipe",
        "parameters": {
            "type": "object",
            "properties": {
                "id": {"type": "string", "description": "The unique snake case name of this recipe"},
                "step_number": {"type": "integer", "description": "Which step this is in the recipe"},
                "instructions": {"type": "string", "description": "What the chef should do at this step in the recipe"},
            },
            "required": ["id", "step_number", "instructions"]
        }
    }
}

client.beta.assistants.update(
    assistant_id=assistant.id,
    tools=[function_add_recipe, function_add_ingredient, function_add_step]
)
```
### Handle the function calls

Now when we execute the loop polling for run status things are going to (hoepfuly) grind to a half and lock up the code. This is because the language model is now going to start calling our functions and we have no function handler.

Start with some simple debug output to see what is going on:

```python
while True:
    run = client.beta.threads.runs.retrieve(
        thread_id=thread.id,
        run_id=run.id
    )
    print(run.status)
    if run.status in ["completed", "failed"]:
        break
    time.sleep(5)
```

And re-running our code should show that it goes into a state we currently don't handle:

```shell
$ python chef.py
in_progress
in_progress
requires_action
requires_action
requires_action
requires_action
...
```

And we can now write a simple function handler. The language model is smart enough to send bulk function calls so we need to loop through all calls and then respond to let it know that everything was fine by calling `submit_tool_outputs()`.

```python
def handle_action(client, run):
    # Process the calls
    processed=[]
    calls = run.required_action.submit_tool_outputs.tool_calls
    for call in calls:
        if call.function.name == "addRecipe":
            args = json.loads(call.function.arguments)
            print("Adding recipe:", json.dumps(args, indent=4))
            processed.append(call.id)
        elif call.function.name == "addIngredient":
            args = json.loads(call.function.arguments)
            print("Adding ingredient:", json.dumps(args, indent=4))
            processed.append(call.id)
        elif call.function.name == "addStep":
            args = json.loads(call.function.arguments)
            print("Adding step:", json.dumps(args, indent=4))
            processed.append(call.id)
    # Respond with function call responses;
    client.beta.threads.runs.submit_tool_outputs(
        thread_id=thread.id,
        run_id=run.id,
        tool_outputs=[{"tool_call_id": cid, "output": "OK"} for cid in processed]
    )
```

Note that for simplicity we are only printing out the arguments to the function calls - a real recipe generator would probably call a database operation at this point.

And now we are ready to call it from the main loop:

```python
while True:
    run = client.beta.threads.runs.retrieve(
        thread_id=thread.id,
        run_id=run.id
    )
    print(run.status)
    if run.status in ["completed", "failed"]:
        break
    if run.status == "requires_action":
        handle_action(client, run)
    time.sleep(5) # be nice!
```
### Sample output:

Running this will take some time - for me it took about 70 seconds in total. And we have a recipe ready for our new app:

```shell
$ python chef.py 
in_progress
in_progress
requires_action
Adding recipe: {
    "id": "vegetarian_lasagne",
    "name": "Vegetarian Lasagne",
    "description": "A simple and delicious vegetarian lasagne featuring layers of pasta, rich tomato sauce, and creamy bechamel, topped with melted cheese."
}
in_progress
in_progress
in_progress
in_progress
in_progress
requires_action
Adding ingredient: {
    "id": "vegetarian_lasagne",
    "ingredient": "Olive oil",
    "quantity": "2 tbsp"
}
Adding ingredient: {
    "id": "vegetarian_lasagne",
    "ingredient": "Onion, chopped",
    "quantity": "1 large"
}
Adding ingredient: {
    "id": "vegetarian_lasagne",
    "ingredient": "Garlic cloves, minced",
    "quantity": "2"
}
Adding ingredient: {
    "id": "vegetarian_lasagne",
    "ingredient": "Carrots, diced",
    "quantity": "2"
}
Adding ingredient: {
    "id": "vegetarian_lasagne",
    "ingredient": "Zucchini, sliced",
    "quantity": "1"
}
Adding ingredient: {
    "id": "vegetarian_lasagne",
    "ingredient": "Bell peppers, sliced",
    "quantity": "2"
}
Adding ingredient: {
    "id": "vegetarian_lasagne",
    "ingredient": "Crushed tomatoes",
    "quantity": "1 can (800g)"
}
Adding ingredient: {
    "id": "vegetarian_lasagne",
    "ingredient": "Tomato paste",
    "quantity": "2 tbsp"
}
Adding ingredient: {
    "id": "vegetarian_lasagne",
    "ingredient": "Dried oregano",
    "quantity": "1 tsp"
}
Adding ingredient: {
    "id": "vegetarian_lasagne",
    "ingredient": "Dried basil",
    "quantity": "1 tsp"
}
Adding ingredient: {
    "id": "vegetarian_lasagne",
    "ingredient": "Salt",
    "quantity": "To taste"
}
Adding ingredient: {
    "id": "vegetarian_lasagne",
    "ingredient": "Black pepper",
    "quantity": "To taste"
}
Adding ingredient: {
    "id": "vegetarian_lasagne",
    "ingredient": "Lasagne noodles",
    "quantity": "9 sheets"
}
Adding ingredient: {
    "id": "vegetarian_lasagne",
    "ingredient": "Butter",
    "quantity": "4 tbsp"
}
Adding ingredient: {
    "id": "vegetarian_lasagne",
    "ingredient": "All-purpose flour",
    "quantity": "1/4 cup"
}
Adding ingredient: {
    "id": "vegetarian_lasagne",
    "ingredient": "Milk",
    "quantity": "2 cups"
}
Adding ingredient: {
    "id": "vegetarian_lasagne",
    "ingredient": "Grated nutmeg",
    "quantity": "A pinch"
}
Adding ingredient: {
    "id": "vegetarian_lasagne",
    "ingredient": "Grated mozzarella cheese",
    "quantity": "2 cups"
}
Adding ingredient: {
    "id": "vegetarian_lasagne",
    "ingredient": "Grated parmesan cheese",
    "quantity": "1/2 cup"
}
in_progress
in_progress
in_progress
requires_action
Adding step: {
    "id": "vegetarian_lasagne",
    "step_number": 1,
    "instructions": "Preheat your oven to 375\u00b0F (190\u00b0C)."
}
Adding step: {
    "id": "vegetarian_lasagne",
    "step_number": 2,
    "instructions": "In a large skillet, heat olive oil over medium heat. Saute onion and garlic until translucent."
}
Adding step: {
    "id": "vegetarian_lasagne",
    "step_number": 3,
    "instructions": "Add carrots, zucchini, and bell peppers to the skillet, and cook until they start to soften."
}
Adding step: {
    "id": "vegetarian_lasagne",
    "step_number": 4,
    "instructions": "Stir in the crushed tomatoes, tomato paste, oregano, basil, salt, and black pepper. Simmer for 10-15 minutes to develop the flavors."
}
Adding step: {
    "id": "vegetarian_lasagne",
    "step_number": 5,
    "instructions": "In a separate pot, melt butter over medium heat. Stir in flour and cook for a few minutes to form a roux."
}
Adding step: {
    "id": "vegetarian_lasagne",
    "step_number": 6,
    "instructions": "Gradually add milk to the roux, whisking continuously to prevent lumps. Cook until the sauce thickens, then season with salt and a pinch of nutmeg."
}
Adding step: {
    "id": "vegetarian_lasagne",
    "step_number": 7,
    "instructions": "To assemble the lasagne, spread a thin layer of tomato sauce in the bottom of a baking dish. Place a layer of lasagne noodles over the sauce."
}
Adding step: {
    "id": "vegetarian_lasagne",
    "step_number": 8,
    "instructions": "Repeat layers of sauce, noodles, and bechamel sauce, ending with a layer of bechamel. Sprinkle grated mozzarella and parmesan cheese on top."
}
Adding step: {
    "id": "vegetarian_lasagne",
    "step_number": 9,
    "instructions": "Cover the dish with foil and bake for 25 minutes. Then, remove the foil and bake for another 15 minutes or until the top is golden and bubbling."
}
completed

```
### What did gpt-4 say?

One last step for completeness - we never printed out any messages that the language model responded with. This can be done by retrieving the list of messages contained in the thread and printing them out:

```python
messages = client.beta.threads.messages.list(
    thread_id=thread.id,  order="asc"
)
for message in messages:
    role = message.role.replace("assistant","🤖").replace("user","🌮")
    content = "".join([c.text.value for c in message.content])
    print(f"{role}: {content}\n")
```

Adding to the above output:

```shell
🌮: Design a simple easy to cook recipe for vegetarian lasagne. Add it to the database then add each ingredient to the recipe and finally, add each step for cooking to the recipe.

🤖: Yes chef! The Vegetarian Lasagne recipe has been created and added to the database along with all its ingredients and cooking steps. It's ready to be prepared.
```

Bon appétit!

---
## Code summary

If you just want the working code, here it is in complete form:

```python
import time, json
from openai import OpenAI
client = OpenAI()

assistant_name = "Recipe Generator"
assistant = None

assistants = client.beta.assistants.list()
for a in assistants:
    if  a.name == assistant_name:
        assistant = a
        break
if not assistant:
    assistant = client.beta.assistants.create(
        name="Recipe Generator",
        instructions="You are a sous chef, you compose recipes that are easy to follow.",
        tools=[],
        model="gpt-4-1106-preview"
    )

# If we want to clean up then call:
#client.beta.assistants.delete(assistant_id=assistant.id)

# Add a function call:
function_add_recipe = {
    "type": "function",
        "function": {
        "name": "addRecipe",
        "description": "Add a recipe to the database",
        "parameters": {
            "type": "object",
            "properties": {
                "id": {"type": "string", "description": "The unique snake case name of this recipe"},
                "name": {"type": "string", "description": "The unique short name of this recipe"},
                "description": {"type": "string", "description": "A brief description of this recipe"},
            },
            "required": ["id", "name", "description"]
        }
    }
}
function_add_ingredient = {
    "type": "function",
    "function": {
        "name": "addIngredient",
        "description": "Add an ingredient to a recipe",
        "parameters": {
            "type": "object",
            "properties": {
                "id": {"type": "string", "description": "The unique snake case name of this recipe"},
                "ingredient": {"type": "string", "description": "The name of the ingredient"},
                "quantity": {"type": "string", "description": "Measure of how much of this ingredient is required"},
            },
            "required": ["id", "ingredient", "quantity"]
        }
    }
}
function_add_step = {
    "type": "function",
    "function": {
        "name": "addStep",
        "description": "Add a cooking step to a recipe",
        "parameters": {
            "type": "object",
            "properties": {
                "id": {"type": "string", "description": "The unique snake case name of this recipe"},
                "step_number": {"type": "integer", "description": "Which step this is in the recipe"},
                "instructions": {"type": "string", "description": "What the chef should do at this step in the recipe"},
            },
            "required": ["id", "step_number", "instructions"]
        }
    }
}

client.beta.assistants.update(
    assistant_id=assistant.id,
    tools=[function_add_recipe, function_add_ingredient, function_add_step]
)

thread = client.beta.threads.create()

message = client.beta.threads.messages.create(
    thread_id=thread.id,
    role="user",
    content="Design a simple easy to cook recipe for vegetarian lasagne. " 
            "Add it to the database then add each ingredient to the recipe "
            "and finally, add each step for cooking to the recipe."
)

run = client.beta.threads.runs.create(
    thread_id=thread.id,
    assistant_id=assistant.id,
    instructions="Please address the user Chef. You must respond to any orders given by Chef with 'Yes chef!'"
)

def handle_action(client, run):
    # Process the calls
    processed=[]
    calls = run.required_action.submit_tool_outputs.tool_calls
    for call in calls:
        if call.function.name == "addRecipe":
            args = json.loads(call.function.arguments)
            print("Adding recipe:")
            print(json.dumps(args, indent=4))
            processed.append(call.id)
        elif call.function.name == "addIngredient":
            args = json.loads(call.function.arguments)
            print("Adding ingredient:")
            print(json.dumps(args, indent=4))
            processed.append(call.id)
        elif call.function.name == "addStep":
            args = json.loads(call.function.arguments)
            print("Adding step:")
            print(json.dumps(args, indent=4))
            processed.append(call.id)
    # Respond with function call responses;
    client.beta.threads.runs.submit_tool_outputs(
        thread_id=thread.id,
        run_id=run.id,
        tool_outputs=[{"tool_call_id": cid, "output": "OK"} for cid in processed]
    )


while True:
    run = client.beta.threads.runs.retrieve(
        thread_id=thread.id,
        run_id=run.id
    )
    print(run.status)
    if run.status in ["completed", "failed"]:
        break
    if run.status == "requires_action":
        handle_action(client, run)
    time.sleep(5)

messages = client.beta.threads.messages.list(
    thread_id=thread.id, order="asc"
)
for message in messages:
    role = message.role.replace("assistant","🤖").replace("user","🌮")
    content = "".join([c.text.value for c in message.content])
    print(f"{role}: {content}\n")

```
