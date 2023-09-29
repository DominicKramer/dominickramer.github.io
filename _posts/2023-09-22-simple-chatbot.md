---
layout: post
title: A Simple Chatbot
date: 2023-09-22 11:59:00-0400
description:
tags:
categories:
giscus_comments: true
related_posts: false
thumbnail:
featured: false
---

In this post we will go over how to create a simple chatbot using OpenAI's [Create chat completion](https://platform.openai.com/docs/api-reference/chat/create) API endpoint.

> The code for this blog can be found at [artificial-brilliance/simple_chatbot](https://github.com/artificial-brilliance/simple_chatbot).

## What is a chatbot?

Before beginning, it is important to know what we are actually making, i.e. what is a chatbot?  We are going to make a command line tool that when runs has a prompt.  You can enter anything you want, press enter, and get a response from the computer as if you are talking with another person.

Here is an example of what the final result will look like:
```yaml
$ pdm run start
You are chatting with a helpful assistant.  To stop the conversation press Ctrl+C.
[You] Hi
[Assistant] Hello! How can I assist you today?
[You] Can I ask you a question
[Assistant] Of course! Feel free to ask me anything you'd like.
[You] What is a Corgi dog?
[Assistant] A Corgi is a type of dog breed that originated in Wales, United Kingdom. There are two main breeds of Corgis: the Pembroke Welsh Corgi and the Cardigan Welsh Corgi.

Pembroke Welsh Corgis are known for their short legs, long bodies, and pointy ears. They have a friendly and outgoing personality, and they are often described as intelligent and affectionate. They are also known for their distinctive "smiling" faces.

Cardigan Welsh Corgis, on the other hand, have a similar body shape but with a longer tail. They have rounded ears and are known for their intelligence, loyalty, and versatility. They are often described as being more reserved and cautious compared to Pembroke Welsh Corgis.

Both breeds are herding dogs and were traditionally used to help farmers in Wales with herding cattle and other livestock. Nowadays, Corgis are popular pets known for their playful nature, loyalty, and adorable appearance.
[You] Thanks.  How do you feel?
[Assistant] As an AI, I don't have feelings in the same way humans do. However, I'm here to assist you and provide information to the best of my abilities. Is there anything else I can help you with?
[You] That makes sense.
[Assistant] I'm glad it makes sense to you! If you have any more questions or need further assistance, feel free to ask. I'm here to help!
[You]
```
Note that the computers responses can vary.

## How is this possible?

Not too long ago, the above would have looked magical.  What allows it to be created today: Large Language Models (LLMs).

At a very high (and hand-wavy) level, you can think of an LLM as a machine where you give it an input sentance and it returns the most likely next word to follow the sentance.  Essentially, LLMs are machines that are very good at playing fill in the blank.

For example, if someone said "After waking up, the person drank a cup of _____", the next word they would probably say is "coffee".  However, they could also say "water" or, less likely, even say "orange juice".  From this, you can see that the completion of a sentance can vary, with some completions more likely than others.

LLMs are designed so that they can be shown enormous amounts of text, for example all of Wikipedia, and "learn" relationships between words such that when given an input sentance, they do a good job of giving a resonable next word for the sentance (based on the text they were trained on).

## How does this relate to a chatbot?

From the perspective of an LLM, suppose it has been trained on a lot of text that consists of conversations.  That is, it was shown a lot of text of the form "Hi. How are you? I'm doing good ..." so that it could learn the relationship between words.

In that case, if you gave it an input sentance such as "You are a helpful assistant: Hi." it may complete that input with "Hello how are you doing?".  This is because, based on the text it has been trained on, and seeing the the context that it is a helpful assistant and the word "Hi", it is the most likely way to complete the input as "Hello how are you doing?".

## How does the LLM build a phrase to return?

Technically, the LLM proposes one word at a time, and different techniques (which I won't get into here) are used to keep proposing words until a complete phrase is said.

That is, if the OpenAI APIs are asked to complete "You are a helpful assistant: Hi.", at a very high (hand-wavy) level, the LLM would first suggest the completion "You are a helpful assistant: Hi. Hello".  That is, with one additional word added to the text, which is what the model proposes as the next word.

Then when given back the text "You are a helpful assistant: Hi. Hello" the model will return "You are a helpful assistant: Hi. Hello how".  Again when given back the text "You are a helpful assistant: Hi. Hello how" it will return "You are a helpful assistant: Hi. Hello how are".

That is, the sentance is being built up by the model word by word until an entire phrase is constructed.  How that phrase is constructed (that is, determining when to stop building the phrase) isn't going to be discussed here, but for now just note that when you ask the OpenAI chat completion API to complete an input message, it returns the entire phrase.

## What does the conversation look like for the LLM?

Now lets walk through very-roughly what the LLM does when creating the chat in the example above.

First the following input text is given to the LLM:

```
System:
You are a helpful assistant.

User:
Hi
```

The LLM completes this text as:

```
System:
You are a helpful assistant.

User:
Hi

Assistant:
Hello! How can I assist you today?
```

The user is then given this last response from the LLM ("Hello! How can I assist you today?") and is asked to provide input.  In the case of the example, the user provides "Can I ask you a question".

As such, this is appened to the end of the message being built up and is given to the LLM:

```
System:
You are a helpful assistant.

User:
Hi

Assistant:
Hello! How can I assist you today?

User:
Can I ask you a question
```

Note the new `User:` entry at the end of the message and that the message continues to grow.

After the above message is given to the LLM, it returns with a completion of:

```
System:
You are a helpful assistant.

User:
Hi

Assistant:
Hello! How can I assist you today?

User:
Can I ask you a question

Assistant:
Of course! Feel free to ask me anything you'd like.
```

This process then continues for the whole conversation.

## Utilizing OpenAI's API

OpenAI's [Create chat completion](https://platform.openai.com/docs/api-reference/chat/create) endpoint is a REST endpoint that accepts many parameters.  The one that is of interest to us in the `messages` parameter that is a list of entries of the form:

```typescript
{
  role: "system"|"user"|"assistant"|"function",
  content: string,
}
```

That is, the messages represent a conversation, like the one we built above, with the role of *system*, *user*, or *assistant* being something that is explicitly stated.  (Do not worry about the function role for this post).

For this post we are going to access the Create chat completion API using the official OpenAI Python client, [openai](https://github.com/openai/openai-python).  It's usage looks like the following:

```python
import openai

completion = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[{
      "role": "system",
      "content": "You are a helpful assistant."
    }, {
      "role": "user",
      "content": "Hi"
    }],
)
response = completion.choices[0].message.content
# in the example above, response would be the string
# "Hello! How can I assist you today?"
```

Don't worry about the `model` parameter which is used to describe which OpenAI LLM to use.  Instead, notice the messages align with the long text we built up in the previous section of the form:

```
System:
...

User:
...

Assistant:
...
```

However, we specify each message separately and explicitly set the role.

Thus, if we explicitly wrote out each step of the conversation we built up above in the previous section, the code could look like the following:


```python
import openai

completion1 = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[{
      "role": "system",
      "content": "You are a helpful assistant."
    }, {
      "role": "user",
      "content": "Hi"
    }],
)

# Hello! How can I assist you today?
response1 = completion.choices[0].message.content

completion2 = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[{
      "role": "system",
      "content": "You are a helpful assistant."
    }, {
      "role": "user",
      "content": "Hi"
    }, {
      "role": "assistant",
      "content": response1 # Hello! How can I assist you today?
    }, {
      "role": "user",
      "content": "Can I ask you a question"
    }],
)

# Of course! Feel free to ask me anything you'd like.
response2 = completion.choices[0].message.content
```

Having such manual unrolled code doesn't scale, but illustrates the process.  To make the code more robust we dynamically build up the messages as shown below:

```python
class ChatBot(object):
    def __init__(self, system_prompt: Optional[str]):
        self.messages = [] if system_prompt is None else [{
            "role": "system",
            "content": system_prompt,
        }]

    def process(self, message: str):
        self.messages.append({"role": "user", "content": message})
        completion = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",
            messages=self.messages,
            temperature=0)
        result = cast(Any, completion).choices[0].message.content
        self.messages.append({"role": "assistant", "content": result})
        return result
```

Here we create a `ChatBot` and initialize the conversation as `self.messages` with the sytem prompt (i.e. "You are a helpful assistant.").  Then each call to `process` is supplied a message from the user, which is appended to the conversation as a user message.  Next, the LLM is asked to complete the conversation, and its response is added to the conversation (`self.messages`) as an assistant message.  Every time the user has an additional message to say, the `process` method is called on the `ChatBot`.

## Conclusion

Below you will find the final code as of the time of writing this blog entry that has comments to describe the different parts of the code and shows how to use the `ChatBot` class and get input from the user.  For the most recent version of the code check its repo at [artificial-brilliance/simple_chatbot](https://github.com/artificial-brilliance/simple_chatbot).

I would like to give thanks to Simon Willison for their great [post](https://til.simonwillison.net/llms/python-react-pattern) on the ReAct pattern that served as inspiration for this post.

I hope you found this blog post helpful, and feel free to leave any questions or comments in the question section.

## The final code

```python
# This code is Apache 2 licensed:
# https://www.apache.org/licenses/LICENSE-2.0

import os
from typing import cast, Any, Optional

import openai

# Read the OpenAI API token form the OPENAI_API_KEY environment variable
openai.api_key = os.environ["OPENAI_API_KEY"]

class ChatBot(object):
    """A simple chatbot that is able to have a conversation with a human
    by recording the conversation between the chatbot and the human.
    """

    def __init__(self, system_prompt: Optional[str]):
        # If a system prompt is specified, then it should be the first
        # message in the context.  The "system" role is used to specify
        # system messages.  These messages are helpful in to instruct
        # the model on how it should behavior or give it background
        # information relevant to the conversation.
        self.messages = [] if system_prompt is None else [{
            "role": "system",
            "content": system_prompt,
        }]

    def process(self, message: str):
        """Processes a message from the user, returning the chatbot's response.

        Parameters
        -----------
        message: the message from the user

        Returns
        -------
        The chatbot's response to the user.
        """

        # Add the user's message to the context (i.e. the conversation)
        self.messages.append({"role": "user", "content": message})

        # Ask the model to complete the conversation.  That is, the
        # messages up to this point represent the conversation
        # where the last message in the list is the most recent
        # message (i.e. the message the user just made).
        completion = openai.ChatCompletion.create(
            # The model GPT-3.5-turbo is used because it works really
            # well and has a nice price point
            model="gpt-3.5-turbo",
            # The messages are essentially the conversation at this point.
            # (i.e. the context).  Each entry in the conversation is marked
            # as being from the user or the assistant via the "role" field.
            messages=self.messages,
            # The temperature is used to essentially tell the API how
            # creative the model can be.  A value of 1 means it can be
            # really creative.  That is, the same input to the model can
            # have very different completions.  A value of 0, on the other
            # hand, means the same input to the model with always return
            # roughly the same completion (but note that the completions
            # won't necessary be exactly the same).
            #
            # A value of zero is useful for testing code understanding the
            # model because is reduces one degree of a freedom (the creativity
            # of the model) which helps in understanding how the model
            # responds to different inputs.
            temperature=0)

        # The completion of the messages is what the model thinks should
        # be the next part of the conversation.  As such, make sure
        # to add the response to the list of messages (which records the
        # conversation) making sure to specify the "assistant" role, which
        # instructs the ChatCompletion API that the message came from the
        # assistant.
        #
        # Specifically of all of the choices that the model returns for
        # possible completions for the conversation, we always just use
        # the first choice.
        #
        # Note: The cast to Any is needed because the type of completion
        #       is "Unknown".
        result = cast(Any, completion).choices[0].message.content

        # Update the conversation to include the assistant's response making
        # sure to specify the response is from the assistant.
        self.messages.append({"role": "assistant", "content": result})

        # The completion of the conversation up to this point is the chatbot's
        # response.
        return result

def main():
    print("You are chatting with a helpful assistant.  To stop the conversation press Ctrl+C.")
    chatbot = ChatBot("You are a helpful assistant.")
    while True:
        message = input("[You] ")
        response = chatbot.process(message)
        print(f"[Assistant] {response}")

main()
```

