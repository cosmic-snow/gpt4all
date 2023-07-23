# GPT4All Python Generation API
The `GPT4All` python package provides bindings to our C/C++ model backend libraries.
The source code and local build instructions can be found [here](https://github.com/nomic-ai/gpt4all/tree/main/gpt4all-bindings/python).

## Quickstart

```bash
pip install gpt4all
```

=== "GPT4All Example"
    ``` py
    from gpt4all import GPT4All
    model = GPT4All("orca-mini-3b.ggmlv3.q4_0.bin")
    output = model.generate("The capital of France is ", max_tokens=3)
    print(output)
    ```
=== "Output"
    ```
    1. Paris
    ```

This will:

- Instantiate `GPT4All`,  which is the primary public API to your large language model (LLM).
- Automatically download the given model to `~/.cache/gpt4all/` if not already present.
- Through `model.generate(...)` the model starts working on a response. There are various ways to
  steer that process. Here, `max_tokens` sets an upper limit, i.e. a hard cut-off point to the output.


### Chatting with GPT4All
Local LLMs can be optimized for chat conversations by reusing previous computational history.

Use the GPT4All `chat_session` context manager to hold chat conversations with the model.

=== "GPT4All Example"
    ``` py
    model = GPT4All(model_name='orca-mini-3b.ggmlv3.q4_0.bin')
    with model.chat_session():
        response = model.generate(prompt='hello', top_k=1)
        response = model.generate(prompt='write me a short poem', top_k=1)
        response = model.generate(prompt='thank you', top_k=1)
        print(model.current_chat_session)
    ```
=== "Output"
    ``` json
    [
       {
          'role': 'user',
          'content': 'hello'
       },
       {
          'role': 'assistant',
          'content': 'What is your name?'
       },
       {
          'role': 'user',
          'content': 'write me a short poem'
       },
       {
          'role': 'assistant',
          'content': "I would love to help you with that! Here's a short poem I came up with:\nBeneath the autumn leaves,\nThe wind whispers through the trees.\nA gentle breeze, so at ease,\nAs if it were born to play.\nAnd as the sun sets in the sky,\nThe world around us grows still."
       },
       {
          'role': 'user',
          'content': 'thank you'
       },
       {
          'role': 'assistant',
          'content': "You're welcome! I hope this poem was helpful or inspiring for you. Let me know if there is anything else I can assist you with."
       }
    ]
    ```
When using GPT4All models in the `chat_session` context:

-  Consecutive chat exchanges are taken into account and not discarded until the session ends; as long as the model has capacity.
- Internal K/V caches are preserved from previous conversation history, speeding up inference.
- The model is given a system and prompt template which make it chatty. Depending on `allow_download=True` (default),
  it will obtain the latest version of [models.json] from the repository, which contains specifically tailored templates
  for models. However, if it is not allowed to download, it falls back to default templates instead.

[models.json]: https://github.com/nomic-ai/gpt4all/blob/main/gpt4all-chat/metadata/models.json


### Streaming Generations
To interact with GPT4All responses as the model generates, use the `streaming=True` flag during generation.

=== "GPT4All Streaming Example"
    ``` py
    from gpt4all import GPT4All
    model = GPT4All("orca-mini-3b.ggmlv3.q4_0.bin")
    tokens = []
    for token in model.generate("The capital of France is", max_tokens=20, streaming=True):
        tokens.append(token)
    print(tokens)
    ```
=== "Output"
    ```
    [' Paris', ' is', ' a', ' city', ' that', ' has', ' been', ' a', ' major', ' cultural', ' and', ' economic', ' center', ' for', ' over', ' ', '2', ',', '0', '0']
    ```

### The Generate Method API
::: gpt4all.gpt4all.GPT4All.generate


## [Insert Good Title Here]
### Influencing Generation 
- [most importantly: temp, top-P, top-K]
  - [temp 0 means deterministic]
- [other parameters can have an influence, too]

The three most important parameters influencing generation are Temperature (`temp`), Top-P and Top-K. In a nutshell,
during the process of selecting the next token, not just one or a few are considered, but every single one is given a
probability. Then through the three parameters, the field of candidates is influenced:

- Top-P and Top-K both narrow the field:
    - **Top-K** simply limits the candidates to a fixed number, after sorting by probability. [TODO: how to disable?] 
    - **Top-P** selects tokens by total probability. E.g. a value of 0.8 means "include the best tokens, whose accumulated
      probabilities reach or just surpass 80%". So if there are a few very good candidates, only a few are selected.
      Setting Top-P to 1, which is 100%, effectively disables it.

- **Temperature** makes the process either more or less random. A Temperature above 1 increasingly "levels the playing
  field", whereas a temperature between 0 and 1 increases the likelihood of the best token candidates even more. A
  Temperature of 0 means that the single best token is selected. So that makes the output deterministic. A
  Temperature of 1 effectively disables the parameter.


### [Some Examples] [How Do I ...?]
- [specify full model path of existing model; optionally point to chat model folder?]
``` py
from pathlib import Path

model = GPT4All(model_name='name', model_path=(Path.home() / 'path' / 'to' / 'models'), allow_download=False)
response = model.generate(...)
print(response)
```

- [custom templates in session] [example Vicuna variants] [TODO: show default templates here?]
``` py
model = GPT4All(...)
# many models use triple hash '###' for keywrords, Vicunas are simpler
system_template = ...
prompt_template = ...
with model.chat_session(system_template, prompt_template):
    response1 = ...
    print(response1)
    response2 = ...
    print(response2)
```

- [templates in simple generate() calls (no session)]
``` py
model = GPT4All(...)  # TODO not sure which one yet
system_template = 'something {0} something; maybe without the placeholder'
prompt_template = 'something {0} something'
response = model.generate(
    system_template.format('instructions go here') + prompt_template.format('hm, maybe no system template?'),
)
```

- [prompt introspection with logging]
A feature which isn't immediately visible is the ability to display the final prompt that gets sent to the model.
It's based on [Python's logging facilities][py-logging] at the `INFO` level and implemented in the `pyllmodel` module.
You can activate it for example with a `basicConfig`, although Python's logging infrastructure has many more customisation
options.

[py-logging]: https://docs.python.org/3/howto/logging.html

``` py
import logging
logging.basicConfig(level=logging.INFO)

[TODO: complete example which gives logging output, maybe with templates?]
```

- [example: no internet access - maybe with local models.json?]
- [TODO: ending generation on a keyword]


### FAQ
#### There's a problem with the download
If `allow_download=True` (default) a model is automatically downloaded into `.cache/gpt4all` in the user's home folder if not already present.

It can happen that the download isn't completing successfully or there's an error in the file afterwards. When in doubt, verify the MD5
checksum by comparing it with the one in [models.json].

As an alternative to the simple downloader built into the bindings, you might want to download from the <https://gpt4all.io/> website instead.
Scroll down to 'Model Explorer' and pick your preferred model.


#### I need the chat GUI and bindings to behave the same
The chat GUI and the bindings are based on the same backend, so it's possible to make them behave the same way. Follow these steps:

- First of all, make sure all the parameters in the chat GUI settings and the ones passed to `generate()` are the same.

- To make comparing the output easier, set Temperature in both to 0 for now. That will make the output deterministic.

- Next you'll have to compare the templates and make them the same. What you have to do depends on how you're using the bindings:
    - With simple `generate()` calls, the input has to be surrounded by both system and prompt templates.
    - When using a chat session, it depends on whether the bindings are allowed to download [models.json]. If yes, and in the chat GUI the default
      templates are used, it'll be handled automatically and nothing else has to be done. If no, use the `chat_session()` template parameters to
      customize them.

- Once you're done, don't forget to reset Temperature to your previous value in both chat GUI and your Python code.


## API Documentation
::: gpt4all.gpt4all.GPT4All
