# cmd_gpt_utils

Command GPT Utilities, a cross-platform command-line tool for seamless interaction with GPT APIs directly from your terminal.

## Features

- **Flexible Configuration**: Use `.conf` and `.yaml` files to manage API keys, models, and default behaviors.
- **Model Selection**: Easily switch between different models using command-line flags.
- **Multiple Prompt Inputs**: Provide prompts via command-line arguments, files, or standard input.
- **Context Chaining**: Maintain conversation context across multiple calls.
- **Chain-of-Thought (CoT)**: Automatically filter out reasoning or "think" blocks from the final output.
- **Streaming (SSE)**: Get real-time responses from the API.
- **Shell Auto-completion**: Integrated support for `argcomplete` for easy use.

## Installation

### PyPI

```bash
python -m pip install --upgrade cmd_gpt_utils
# Recommend `pipx` for global installation 
cmd-gpt Hello World
```

### Manually

1.  **Clone the repository and install the package:**

    ```bash
    git clone https://github.com/CreeperLKF/cmd_gpt_utils.git
    cd cmd_gpt_utils
    pip install .
    ```

2.  **Activate command-line completion:** (Optional)

    Add the following to your `.bashrc` or `.zshrc` file for permanent auto-completion.

    ```bash
    eval "$(register-python-argcomplete cmd-gpt)"
    ```

    You may need to restart your shell for the changes to take effect.

## Configuration

On the first run, `cmd-gpt` will automatically create a configuration directory at `~/.config/cmd_gpt/` with default template files. You can override this path by setting the `CMDGPT_CONFIG_PATH` environment variable.

- `default.conf`: For general settings.
- `models.yaml`: For model definitions.

**The configuration file is not stable yet. We consdier separating API Provider and Models in the future update.**

### Overriding Configurations

You can override settings by using the `-f` flag. The tool loads configurations in this order:
1.  `default.conf` (or `default.local`)
2.  Each file specified with `-f path` in the order they appear.

This means settings in later files will override those in earlier ones. The `-f` flag can be used multiple times.

When a relative path is given to `-f`, the tool searches in this order:
1.  `./path`
2.  `./path.local`
3.  `$CMDGPT_CONFIG_PATH/path`
4.  `$CMDGPT_CONFIG_PATH/path.local`

## Usage

**The documentation is currently under construction. Please refer to `cmd-gpt --help` for more information.**

### Basic Query

#### Positional Arguments

```bash
$ cmd-gpt "What is love"
```

You can also enter interactive mode by running `cmd-gpt` without any prompt arguments. By default, you press `Ctrl+D` to send. Use the `-n` flag to send the prompt on the first `Enter` press instead.

```bash
$ cmd-gpt -n
What is love?
<Assistant's response will appear here>
```

You can use `-p path` to read prompts from a file. Multiple files are concatenated with `PROMPT_CONCAT_NL` logic.

### Piping from stdin

```bash
$ echo "Summarize this text." | cmd-gpt
```

### Prompt Concatenation

You can add text before or after your main prompt using the `-B` (before) and `-A` (after) flags. These can be used multiple times.

```bash
# Joins positional arguments with a space by default
$ cmd-gpt I think Python "is a great language" -B "At any time, C++ is the best." -B "In my opinion," -A "What is the best language?" -A "(Only answer C++/Python according to the context)"
# Final Prompt: "At any time, C++ is the best.\nIn my opinion,\nI think Python is a great language\nWhat is the best language?\n(Only answer C++/Python according to the context)\n"
```

You can also set default `PROMPT_BEFORE` and `PROMPT_AFTER` values in your `.conf` file. The final prompt is assembled in this order: `Config Before -> CLI Before -> Main Prompt -> CLI After -> Config After`.

By default, parts are joined with a newline (controlled by `PROMPT_CONCAT_NL=true|false`). The positional arguments that form the Main Prompt are joined by a space (controlled by `PROMPT_CONCAT_SP=true|false`).

#### Using `--` to separate arguments

You can use `--` to tell `cmd-gpt` to treat everything that follows as part of the prompt, even if it looks like a flag.

```bash
$ cmd-gpt -m gpt4 -- -f is not a file, it is part of the prompt.
```

### Using a different model

```bash
$ cmd-gpt -m deepseek-v3 "What's your favourite operator in Arknights?"
```

If `-m` is not used, the model with the minimal id will be chosen. Otherwise, it will first try match id, then name of models.

Both partial matching and full matching are supported. If name in `-m` partially matches, it will select a model with minimal id.

### Using Context Chaining

Start a conversation. The output will be only the assistant's response.

```bash
cmd-gpt "My name is Bob. Remember it."
```

Continue the conversation by piping the history back in and getting only the new response. The `-c o` flag prints the full history and `-c i` flag accepts a history json from stdin (which means stdin cannot receive prompts in this mode).

```bash
# First turn, save full history to chat.json
cmd-gpt "My name is Bob" -c o > chat.json

# Second turn, provide history via stdin, provide new prompt, get full history back
cat chat.json | cmd-gpt "What is my name?" -c io > chat_updated.json

# You can also get just the final response
cat chat_updated.json | cmd-gpt "Thank you." -c i
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
pytest --cov=cmd_gpt_utils
```
