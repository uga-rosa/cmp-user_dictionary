# cmp-dictionary

Dictionary completion source for [nvim-cmp](https://github.com/hrsh7th/nvim-cmp)  

# Usage

Example setting

```lua
require("cmp").setup({
  -- other settings
  sources = {
    -- other sources
    {
      name = "dictionary",
      keyword_length = 2,
    },
  }
})

require("cmp_dictionary").setup({
    dic = {
        ["*"] = { "/usr/share/dict/words" },
        ["lua"] = "path/to/lua.dic",
        ["javascript,typescript"] = { "path/to/js.dic", "path/to/js2.dic" },
        filename = {
            ["xmake.lua"] = { "path/to/xmake.dic", "path/to/lua.dic" },
        },
        filepath = {
            ["%.tmux.*%.conf"] = "path/to/tmux.dic"
        },
    },
    -- The following are default values, so you don't need to write them if you don't want to change them
    exact = 2,
    first_case_insensitive = false,
    document = false,
    document_command = "wn %s -over",
    async = false, 
    capacity = 5,
    debug = false,
})
```

# Option

#### dic

table, default: { [*] = {}, filename = nil, filepath = nil }

All but three special keys are file types, and the values are the corresponding dictionary arrays.
You can also use comma-separated file types for the key.
If one dictionary, you can use a string instead of an array.

The special key `filename` takes a table as its value, which has keys of file names and values of corresponding dictionary array.
The keys are used in exact match with the result of `expand("%:t")`.

The special key `filepath` is a table in a format similar to filename.
The difference is that the keys are lua patterns and are used to match `expand("%:p")`.

The special key `*` is a global setting.

The priority is `filename` > `filepath` > `filetype` > `*`

#### exact

integer, default: 2

It decides how many characters at the beginning are used as the exact match.
If -1, only candidates with an exact prefix match will be returns.  

#### first_case_insensitive

boolean, default: false

If true, it will ignore the case of the first character.
For example, if you have "Example" and "excuse" in your dictionary, typing "Ex" will bring up "Example" and "Excuse" as candidates,
while typing "ex" will bring up "example" and "excuse".

#### document

boolean, default: false

If true, activate document using external command. See [document_command](#document_command)

#### document_command

string, default: 'wn %s -over'

This command is used above document feature.
The `%s` will contain the candidate word.
The default `wn` command is [wordnet](https://wordnet.princeton.edu/)

#### async

boolean default: false

If true, perform the initialization in a separate thread.
If you are using a very large dictionary and the body operation is blocked, try this.

You need module mpack, so you need to install lua51-mpack or build neovim of 0.6 or higher.

#### capacity

integer, default: 5

Determines the maximum number of dictionaries to be cached.
This will prevent duplicate reads when you switch dictionaries with the settings described above.

#### debug

boolean, default: false

If true, debug messages are output.

# Where to find dictionaries

You can download dic from [aspell.net](https://ftp.gnu.org/gnu/aspell/dict/0index.html) or installing by package manager, xbps extract to

```bash
$ ls /usr/share/dict/
american-english  british-english  words
```

After installing aspell and dictionary you want, run following command to get dic for this plugin (plain text).

```bash
aspell -d <lang> dump master | aspell -l <lang> expand > my.dict
```

# How to create your own dictionary

The dictionary is recognized as a list delimited by `%s`. `%s` is a space, `\t`, `\n`, `\r`, or `\f`.
For example, if you use the following file as a dictionary, the source to be added is `{"hello", "world", "!"}`.

```txt
hello
world !
```

# Lazy load

By default, reading dictionaries are fired by `BufEnter`.
So if this plugin loading is set to `InsertEnter` or something, the dictionary will not load and no candidates will appear.
The workaround is to fire this update yourself when the plugin is loaded (after setup).

For example, if you use packer.nvim, you can use

```lua
use({
    "hrsh7th/nvim-cmp",
    event = "InsertEnter",
    -- other setting
})
use({
    "uga-rosa/cmp-dictionary",
    after = "nvim-cmp",
    config = function()
        require("cmp_dictionary").setup({
            dic = {
                ["*"] = "/usr/share/dict/words",
            },
            first_case_insensitive = true,
            document = true,
        })
        require("cmp_dictionary").update() -- THIS
    end
})
```