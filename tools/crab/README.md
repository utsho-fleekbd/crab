# Crab Language Support for Visual Studio Code

Syntax highlighting and basic language configuration for the **Crab** programming language.

Crab is a small statically typed language with a Rust interpreter.  
Repository: https://github.com/utshomh/crab

## Features

- Syntax highlighting for:
  - Keywords (`var`, `print`, `if`, `else`, `while`, `for`, `to`, `in`)
  - Booleans (`true`, `false`)
  - Numbers
  - Single- and double-quoted strings (including multi-line)
  - Collection built-ins (`len`, `push`, `pop`, `keys`, `values`, `has`, `get`, `remove`, `collect`)
  - Type annotations (`Number`, `String`, `Boolean`, `List`, `Map`, `Range`)
  - Ranges, dictionary separators, and field access
  - Operators
  - Comments (`//`)
  - Identifiers
- Bracket matching and auto-closing pairs
- Line comment support

## Installation

1. Open a terminal in the extension folder and run:

```bash
npx @vscode/vsce package
```

This creates a file named something like crab-0.0.2.vsix

2. Install the extension:

```bash
code --install-extension crab-0.0.2.vsix
```

(You can also right-click the .vsix file in VS Code and choose “Install Extension VSIX”.)

3. Reload VS Code (or restart it).

The extension is now installed and will automatically activate for any .crab file.
