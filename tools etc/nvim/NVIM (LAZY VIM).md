I use this configuration: https://github.com/LazyVim/LazyVim

And add LSP-server for Phython in init.lua:

```lua
local nvim_lsp = require("lspconfig")

nvim_lsp.pyright.setup({})
```
