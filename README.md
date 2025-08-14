# PipeAgent: An llm agent over pipe in unix philosophy

## Table of Contents

- [pipe-agent](#pipe-agent)
  - [Table of Contents](#table-of-contents)
  - [Features](#features)
  - [Plans](#plans)
  - [Quick Startup](#quick-startup)
  - [Installation](#installation)
    - [From PyPI](#from-pypi)
    - [From Source Code](#from-source-code)
  - [Configuration](#configuration)
    - [Overriding Configurations](#overriding-configurations)
  - [Usage](#usage)
    - [Basic Query](#basic-query)
    - [Prompt Concatenation](#prompt-concatenation)
      - [Using `--` to separate arguments](#using----to-separate-arguments)
    - [Using a different model](#using-a-different-model)
    - [Using Context Chaining](#using-context-chaining)
  - [Running Tests](#running-tests)

## Features

- **Unix-like CLI**: Rich and useful command line interface.
- **Prompt Concatenation**: Concatenate multiple prompt input sources via command-line arguments, files, or standard input.
- **Context Chaining**: Maintain conversation context across multiple calls and connect with pipe.
- **Chain-of-Thought (CoT)**: Automatically filter out reasoning content from the final output.
- **Streaming (SSE)**: Get real-time responses from the API.

- **Flexible Configuration**: Support multiple configuration file input with overriding.
- **Shell Auto-completion**: Integrated support for `argcomplete` for easy use.
- **Agent over shell**: Build your own agent with `pipe-agent` and text processing utilities and tricks.

## Plans

- [ ] Refine documentation.
- [ ] Support Multimodal LLMs.
- [ ] **Interface Change** Refine concatenation logic.
- [ ] **Features** Support workflow-based agent through configuration files.

## Quick Startup

1. `pip install .`
2. `pipe-agent` and then configures the model in `~/.config/pipe-agent/models.yaml`
3. `pipe-agent hello world`

## Installation

### From PyPI [Currently not Available]

```bash
python -m pip install --upgrade pipe-agent
# Recommend `pipx` for global installation 
pipe-agent Hello World
```

### From Source Code

1.  **Clone the repository and install the package:**

    ```bash
    git clone https://github.com/CreeperLKF/pipe-agent.git
    cd pipe-agent
    pip install .
    ```

2.  **Activate command-line completion:** (Optional)

    Add the following to your `.bashrc` or `.zshrc` file for permanent auto-completion.

    ```bash
    eval "$(register-python-argcomplete pipe-agent)"
    ```

    You may need to restart your shell for the changes to take effect.

## Configuration

On the first run, `pipe-agent` will automatically create a configuration directory at `~/.config/pipe-agent/` with default template files. You can override this path by setting the `PIPE_AGENT_CONFIG_PATH` environment variable.

- `default.conf`: For general settings.
- `models.yaml`: For model definitions.

**The configuration file is not stable yet. We consdier separating API Provider and Models in the future update.**

### Overriding Configurations

You can override settings by using the `-f` flag. The tool loads configurations in this order:
1.  `default.conf` (or `default.local` if exists)
2.  Each file specified with `-f path` in the order they appear.

When a relative path is given to `-f`, the tool searches in this order:
1.  `./path`
2.  `./path.local`
3.  `$PIPE_AGENT_CONFIG_PATH/path`
4.  `$PIPE_AGENT_CONFIG_PATH/path.local`

The `-f` flag can be used multiple times. Settings in later files will override those in earlier ones if and only if certain entries exists.

For example, `pipe-agent -f f1 -f f2.local` will load `default.conf` (or `default.local` if exists), `f1.local`, `f2.local`. Assume `PROMPT_CONCAT_NL=true` in `f1.local` and `PROMPT_CONCAT_NL=false` in `f2.local`. The final configuration used will be `PROMPT_CONCAT_NL=false`.

## Usage

**This document is currently under construction. Please refer to `pipe-agent --help` for more information.**

### Basic Query

```bash
$ pipe-agent "What is love"
$ pipe-agent -p "prompt.txt"
$ pipe-agent "劲发江潮落，气收秋毫平！" -p "重岳1.txt" "千招百式在一息！" -p "重岳2.txt"
```

Basically, `pipe-agent` get prompts from `-p` (files) and positional arguments. **Multiple** file inputs and positional arguments inputs are supported (use `-p path` multiple times). If `PROMPT_CONCAT_NL` is true, a newline will be appended after prompt from `-p`. If `PROMPT_CONCAT_SP` is true, a space will be appended after  prompt from a positional arguments.

You can also enter interactive mode by running `pipe-agent` without any prompt arguments. By default, you press `Ctrl+D` to send. Use the `-n` flag to send the prompt on the first `Enter` press instead.

```bash
$ pipe-agent -n
What is love?
<Assistant's response will appear here>
```

Or use pipe bump input into stdin. 

```bash
$ echo "Summarize this text." | pipe-agent
```

If a prompt is provided via command-line arguments or a file, the content from stdin will be appended to the existing prompt.

```bash
$ echo "and this part from stdin" | pipe-agent "This is a prompt from arguments"
# Final prompt will be: "This is a prompt from arguments\nand this part from stdin"
```

### Prompt Concatenation

Besides specifying multiple inputs, you can add text before or after your main prompt using the `-B` (before) and `-A` (after) flags. These can be used multiple times.

```bash
# Joins positional arguments with a space by default
$ pipe-agent I think Python "is a great language" -B "At any time, C++ is the best." -B "In my opinion," -A "What is the best language?" -A "(Only answer C++/Python according to the context)"
# Final Prompt: "At any time, C++ is the best.\nIn my opinion,\nI think Python is a great language\nWhat is the best language?\n(Only answer C++/Python according to the context)\n"
```

You can also set default `PROMPT_BEFORE` and `PROMPT_AFTER` values in your `.conf` file. The final prompt is assembled in this order: `Config Before -> CLI Before -> Main Prompt -> CLI After -> Config After`.

By default, parts are joined with a newline (controlled by `PROMPT_CONCAT_NL=true|false`). The positional arguments that form the Main Prompt are joined by a space (controlled by `PROMPT_CONCAT_SP=true|false`).

#### Using `--` to separate arguments

You can use `--` to tell `pipe-agent` to treat everything that follows as part of the prompt, even if it looks like a flag. It is **recommended** to use `--` in automated jobs (agent or your shell scripts) for **security concerns**.

```bash
$ pipe-agent -m gpt4 -- -f is not a file, it is part of the prompt.
```

### Using a different model

```bash
$ pipe-agent -m deepseek-v3 "What's your favourite operator in Arknights?"
```

If `-m` is not used, the model with the minimal id will be chosen. Otherwise, it will first try match id, then name of models.

Both partial matching and full matching are supported. If name in `-m` partially matches, it will select a model with minimal id.

### Using Context Chaining

Start a conversation. The output will be only the assistant's response.

```bash
pipe-agent "My name is Bob. Remember it."
```

Continue the conversation by piping the history back in and getting only the new response. The `-c o` flag prints the full history and `-c i` flag accepts a history json from stdin (which means stdin cannot receive prompts in this mode).

```bash
# First turn, save full history to chat.json
pipe-agent "My name is Bob" -c o > chat.json

# Second turn, provide history via stdin, provide new prompt, get full history back
cat chat.json | pipe-agent "What is my name?" -c io > chat_updated.json

# You can also get just the final response
cat chat_updated.json | pipe-agent "Thank you." -c i
```

## Running Tests

To run the test suite, first install the test dependencies:

```bash
pip install .[test]
```

Then, you can run the tests using `pytest`:

```bash
pytest
```

You can also generate a code coverage report:

```bash
pytest --cov=pipe_agent
```
