# NOTES
First of all, thank you to the author of `jackMort/ChatGPT.nvim` for creating a seamless framework
to interact with OGPT in neovim!

**THIS IS A FORK of the original OGPT.nvim that supports Ollama (<https://ollama.ai/>), which
allows you to run complete local LLMs.**

Ollama is still in its infancy, so there are numerous pull requests open to expand its capabilities.
One of which is to conform to the OGPT API - <https://github.com/jmorganca/ollama/pull/991>.
Because of this, this repo is a hack together solution for me to test out the local LLMs that I have running.

**THIS PLUGIN MAY NOT LAST VERY LONG, depending on the state of Ollama.**

## Plan for this plugin:
+ [x] original functionality of OGPT.nvim with Ollama
+ [x] Custom settings per session
+ [ ] Support model selection on the fly, query from Ollama
+ [ ] Support model creation on the fly
+ [ ] help docs for all parameters

## General usage

TO USE with local Ollama, should be the same as the original OGPT.nvim plugin.
change your API HOST and Key can be empty string.

```sh
OPENAI_API_HOST=http://localhost:11434 OPENAI_API_KEY=" " nvim
```

----
# OGPT.nvim

![GitHub Workflow Status](http://img.shields.io/github/actions/workflow/status/jackMort/OGPT.nvim/default.yml?branch=main&style=for-the-badge)
![Lua](https://img.shields.io/badge/Made%20with%20Lua-blueviolet.svg?style=for-the-badge&logo=lua)

`OGPT` is a Neovim plugin that allows you to effortlessly utilize the OpenAI OGPT API,
empowering you to generate natural language responses from OpenAI's OGPT directly within the editor in response to your inquiries.

![preview image](https://github.com/jackMort/OGPT.nvim/blob/media/preview-2.png?raw=true)

## Features
- **Interactive Q&A**: Engage in interactive question-and-answer sessions with the powerful gpt model (OGPT) using an intuitive interface.
- **Persona-based Conversations**: Explore various perspectives and have conversations with different personas by selecting prompts from Awesome OGPT Prompts.
- **Code Editing Assistance**: Enhance your coding experience with an interactive editing window powered by the gpt model, offering instructions tailored for coding tasks.
- **Code Completion**: Enjoy the convenience of code completion similar to GitHub Copilot, leveraging the capabilities of the gpt model to suggest code snippets and completions based on context and programming patterns.
- **Customizable Actions**: Execute a range of actions utilizing the gpt model, such as grammar correction, translation, keyword generation, docstring creation, test addition, code optimization, summarization, bug fixing, code explanation, Roxygen editing, and code readability analysis. Additionally, you can define your own custom actions using a JSON file.

For a comprehensive understanding of the extension's functionality, you can watch a plugin showcase [video](https://www.youtube.com/watch?v=7k0KZsheLP4)

## Installation

- Make sure you have `curl` installed.
- Get an API key from OpenAI, which you can [obtain here](https://beta.openai.com/account/api-keys).

The OpenAI API key can be provided in one of the following two ways:

1. In the configuration option `api_key_cmd`, provide the path and arguments to
   an executable that returns the API key via stdout.
1. Setting it via an environment variable called `$OPENAI_API_KEY`.

Custom OpenAI API host with the configuration option `api_host_cmd` or
environment variable called `$OPENAI_API_HOST`. It's useful if you can't access
OpenAI directly

For Azure deployments, you also need to set environment variables
`$OPENAI_API_TYPE` to `azure`, `$OPENAI_API_BASE` to your own resource URL,
e.g. `https://{your-resource-name}.openai.azure.com`, and `$OPENAI_API_AZURE_ENGINE`
to your deployment ID. Optionally, if you need a different API version,
set `$OPENAI_API_AZURE_VERSION` as well. Note that edit models have been deprecated
so they might not work.

```lua
-- Packer
use({
  "huynle/ogpt.nvim",
    config = function()
      require("ogpt").setup()
    end,
    requires = {
      "MunifTanjim/nui.nvim",
      "nvim-lua/plenary.nvim",
      "nvim-telescope/telescope.nvim"
    }
})

-- Lazy
{
  "huynle/ogpt.nvim",
    event = "VeryLazy",
    config = function()
      require("ogpt").setup()
    end,
    dependencies = {
      "MunifTanjim/nui.nvim",
      "nvim-lua/plenary.nvim",
      "nvim-telescope/telescope.nvim"
    }
}
```

## Configuration

`OGPT.nvim` comes with the following defaults, you can override them by passing config as setup param

https://github.com/huynle/ogpt.nvim/blob/9508429d514509b96d8ca9ec3f16933cfb30ac8a/lua/ogpt/config.lua#L10-L157

### Secrets Management

Providing the OpenAI API key via an environment variable is dangerous, as it
leaves the API key easily readable by any process that can access the
environment variables of other processes. In addition, it encourages the user
to store the credential in clear-text in a configuration file.

As an alternative to providing the API key via the `OPENAI_API_KEY` environment
variable, the user is encouraged to use the `api_key_cmd` configuration option.
The `api_key_cmd` configuration option takes a string, which is executed at
startup, and whose output is used as the API key.

The following configuration would use 1Passwords CLI, `op`, to fetch the API key
from the `credential` field of the `OpenAI` entry.

```lua
require("ogpt").setup({
    api_key_cmd = "op read op://private/OpenAI/credential --no-newline"
})
```

The following configuration would use GPG to decrypt a local file containing the
API key

```lua
local home = vim.fn.expand("$HOME")
require("ogpt").setup({
    api_key_cmd = "gpg --decrypt " .. home .. "/secret.txt.gpg"
})
```

Note that the `api_key_cmd` arguments are split by whitespace. If you need whitespace inside an argument (for example to reference a path with spaces), you can wrap it in a separate script.

## Usage

Plugin exposes following commands:

#### `OGPT`
`OGPT` command which opens interactive window using the `mistral:7b`
model.
(also known as `OGPT`)

#### `OGPTActAs`
`OGPTActAs` command which opens a prompt selection from [Awesome OGPT Prompts](https://github.com/f/awesome-ogpt-prompts) to be used with the `mistral:7b` model.

![preview image](https://github.com/jackMort/OGPT.nvim/blob/media/preview-3.png?raw=true)

#### `OGPTEditWithInstructions`
`OGPTEditWithInstructions` command which opens interactive window to edit selected text or whole window using the `codellama:13b` model (GPT 3.5 fine-tuned for coding).

You can map it using the Lua API, e.g. using `which-key.nvim`:
```lua
local ogpt = require("ogpt")
wk.register({
    p = {
        name = "OGPT",
        e = {
            function()
                ogpt.edit_with_instructions()
            end,
            "Edit with instructions",
        },
    },
}, {
    prefix = "<leader>",
    mode = "v",
})
```

- [demo video](https://www.youtube.com/watch?v=dWe01EV0q3Q).

![preview image](https://github.com/jackMort/OGPT.nvim/blob/media/preview.png?raw=true)

#### `OGPTRun`

`OGPTRun [action]` command which runs specific actions -- see [`actions.json`](./lua/ogpt/flows/actions/actions.json) file for a detailed list. Available actions are:
  1. `grammar_correction`
  2. `translate`
  3. `keywords`
  4. `docstring`
  5. `add_tests`
  6. `optimize_code`
  7. `summarize`
  8. `fix_bugs`
  9. `explain_code`
  10. `roxygen_edit`
  11. `code_readability_analysis` -- see [demo](https://youtu.be/zlU3YGGv2zY)

All the above actions are using `mistral:7b` model.

It is possible to define custom actions with a JSON file. See [`actions.json`](./lua/ogpt/flows/actions/actions.json) for an example. The path of custom actions can be set in the config (see `actions_paths` field in the config example above).

An example of custom action may look like this: (`#` marks comments)
```python
{
  "action_name": {
    "type": "chat", # or "completion" or "edit"
    "opts": {
      "template": "A template using possible variable: {{filetype}} (neovim filetype), {{input}} (the selected text) an {{argument}} (provided on the command line)",
      "strategy": "replace", # or "display" or "append" or "edit"
      "params": { # parameters according to the official OpenAI API
        "model": "mistral:7b", # or any other model supported by `"type"` in the OpenAI API, use the playground for reference
        "stop": [
          "```" # a string used to stop the model
        ]
      }
    },
    "args": {
      "argument": {
          "type": "strig",
          "optional": "true",
          "default": "some value"
      }
    }
  }
}
```
The `edit` strategy consists in showing the output side by side with the input and
available for further editing requests.
For now, `edit` strategy is implemented for `chat` type only.

The `display` strategy shows the output in a float window.

`append` and `replace` modify the text directly in the buffer.

### Interactive popup
When using `OGPT` and `OGPTEditWithInstructions`, the following
keybindings are available:
- `<C-Enter>` [Both] to submit.
- `<C-y>` [Both] to copy/yank last answer.
- `<C-o>` [Both] Toggle settings window.
- `<Tab>` [Both] Cycle over windows.
- `<C-f>` [Chat] Cycle over modes (center, stick to right).
- `<C-c>` [Both] to close chat window.
- `<C-u>` [Chat] scroll up chat window.
- `<C-d>` [Chat] scroll down chat window.
- `<C-k>` [Chat] to copy/yank code from last answer.
- `<C-n>` [Chat] Start new session.
- `<C-d>` [Chat] draft message (create message without submitting it to server)
- `<C-r>` [Chat] switch role (switch between user and assistant role to define a workflow)
- `<C-s>` [Both] Toggle system message window.
- `<C-i>` [Edit Window] use response as input.
- `<C-d>` [Edit Window] view the diff between left and right panes and use diff-mode
  commands

When the setting window is opened (with `<C-o>`), settings can be modified by
pressing `Enter` on the related config. Settings are saved across sections

### Whichkey plugin mappings
Add these to your [whichkey](https://github.com/folke/which-key.nvim) plugin mappings for convenient binds

```lua
c = {
  name = "OGPT",
    c = { "<cmd>OGPT<CR>", "OGPT" },
    e = { "<cmd>OGPTEditWithInstruction<CR>", "Edit with instruction", mode = { "n", "v" } },
    g = { "<cmd>OGPTRun grammar_correction<CR>", "Grammar Correction", mode = { "n", "v" } },
    t = { "<cmd>OGPTRun translate<CR>", "Translate", mode = { "n", "v" } },
    k = { "<cmd>OGPTRun keywords<CR>", "Keywords", mode = { "n", "v" } },
    d = { "<cmd>OGPTRun docstring<CR>", "Docstring", mode = { "n", "v" } },
    a = { "<cmd>OGPTRun add_tests<CR>", "Add Tests", mode = { "n", "v" } },
    o = { "<cmd>OGPTRun optimize_code<CR>", "Optimize Code", mode = { "n", "v" } },
    s = { "<cmd>OGPTRun summarize<CR>", "Summarize", mode = { "n", "v" } },
    f = { "<cmd>OGPTRun fix_bugs<CR>", "Fix Bugs", mode = { "n", "v" } },
    x = { "<cmd>OGPTRun explain_code<CR>", "Explain Code", mode = { "n", "v" } },
    r = { "<cmd>OGPTRun roxygen_edit<CR>", "Roxygen Edit", mode = { "n", "v" } },
    l = { "<cmd>OGPTRun code_readability_analysis<CR>", "Code Readability Analysis", mode = { "n", "v" } },
  },
```

[!["Buy Me A Coffee"](https://www.buymeacoffee.com/assets/img/custom_images/orange_img.png)](https://www.buymeacoffee.com/jackMort)
