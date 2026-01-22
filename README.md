# tree-sitter-faber

Tree-sitter grammar and Zed extension for [Faber](https://github.com/ianzepp/faber), a Latin programming language.

## Installation (Zed)

1. Clone this repo as a sibling to your faber project:

   ```bash
   cd ~/github/ianzepp
   git clone https://github.com/ianzepp/tree-sitter-faber
   ```

2. In Zed: Command Palette → `zed: install dev extension` → select the `tree-sitter-faber` directory

3. Open any `.fab` file — syntax highlighting should work

## Updating the Grammar

After editing `grammar.js`:

```bash
cd ~/github/ianzepp/tree-sitter-faber
tree-sitter generate
git add -A && git commit -m "Update grammar" && git push
```

Then reinstall the dev extension in Zed. For local development, you do NOT need to update the `rev` SHA in `extension.toml` — that's only required for publishing to the Zed extension registry.

## Repository Structure

```
tree-sitter-faber/
├── extension.toml          # Zed extension manifest (MUST be at root)
├── grammar.js              # Tree-sitter grammar definition
├── package.json            # Tree-sitter package config
├── tree-sitter.json        # Tree-sitter metadata
├── queries/
│   └── highlights.scm      # Highlighting queries (for tree-sitter CLI)
├── languages/
│   └── faber/
│       ├── config.toml     # Zed language config
│       └── highlights.scm  # Highlighting queries (for Zed - YES, DUPLICATED)
├── src/                    # Generated parser (from tree-sitter generate)
└── grammars/               # Created by Zed at runtime (gitignored)
```

## Zed Extension Pain Points (Documentation for Future Us)

The Zed extension system has several frustrating design decisions:

### 1. Grammar must be in a git repo that Zed clones

Even for dev extensions, Zed insists on cloning the grammar from the `repository` URL specified in `extension.toml`. It creates a `grammars/<name>/` directory and clones into it. This means:

- You cannot have the grammar files in that location locally — Zed will complain "directory already exists, but is not a git clone"
- The grammar source files live at the repo root, but Zed clones them into `grammars/faber/`
- You must `.gitignore` the `grammars/` directory to avoid committing Zed's clone

### 2. highlights.scm must be in TWO places

- `queries/highlights.scm` — standard tree-sitter location, used by `tree-sitter highlight`
- `languages/faber/highlights.scm` — where Zed actually looks for it

If you only have it in `queries/`, Zed will parse correctly but show no highlighting.

### 3. extension.toml must be at repo root

The manifest cannot be in a subdirectory. "No extension manifest found" otherwise.

### 4. The rev SHA chicken-and-egg problem

For published extensions, `extension.toml` requires:

```toml
[grammars.faber]
repository = "https://github.com/ianzepp/tree-sitter-faber"
rev = "abc123..."  # Must be a valid commit SHA
```

This creates a chicken-and-egg problem: you need to commit to get a SHA, but the SHA goes in the file you're committing. The workflow becomes:

1. Make changes, commit with placeholder/old SHA
2. Get new SHA
3. Update extension.toml with new SHA
4. Commit again
5. Push

For dev extensions, this is unnecessary — Zed uses the local clone regardless of the SHA. Only matters for registry publishing.

### 5. No direct GitHub installation

Unlike VS Code (install from VSIX or URL), Zed requires either:

- Local dev extension install, or
- Publishing to the centralized Zed extension registry via PR to [zed-industries/extensions](https://github.com/zed-industries/extensions)

### 6. Registry publishing requires forking their repo

To publish, you must:

1. Fork zed-industries/extensions
2. Add your extension as a git submodule
3. Submit a PR
4. Wait for merge

Every update requires a new PR with updated SHA.

## Testing the Grammar

```bash
# Parse a file
echo 'functio main() { scribe "hello" }' | tree-sitter parse /dev/stdin

# Highlight (requires tree-sitter config for .fab files)
tree-sitter highlight /path/to/file.fab
```

## License

MIT
