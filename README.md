# askcode — find code by asking what it does

Semantic code search in your terminal. Instead of grepping for keywords you *think* exist, describe what the code does — `askcode` scans the repo, asks a language model which files actually answer your question, and prints them ranked with the reason each one matched.

```
$ askcode "where do we refresh expired sessions?"

1. internal/auth/sessions.go:88   rotates the session row on each refresh
2. internal/api/middleware.go:31  rejects expired cookies and re-issues
3. ...
```

## Why

Keyword grep fails when you don't know the vocabulary of the codebase — "session refresh" might live in a function called `rotateStaleTokens`. `askcode` reads the actual code and matches by *meaning*, not by string.

Built for coding agents and humans alike: `--json` gives machine-readable output, the single-file CLI has **zero dependencies** (Python 3.9+ stdlib only), and it works against any OpenAI-compatible endpoint.

## Install

No package needed — grab the one file:

```bash
curl -fsSL https://raw.githubusercontent.com/Micolaabdi/askcode-cli/main/askcode -o ~/bin/askcode
chmod +x ~/bin/askcode
```

## Configure

```bash
export ATMOROUTER_API_KEY=ar-...        # free key, 100M-token free tier
# optional:
export ASKCODE_BASE_URL=https://atmorouter.dev/v1   # any OpenAI-compatible API
export ASKCODE_MODEL=atmo/deepseek-v4.1-flash       # cheap+fast default
```

Works with any OpenAI-compatible provider — set `ASKCODE_BASE_URL`/`ASKCODE_MODEL`/`OPENAI_API_KEY` and point it wherever you like.

## Use

```bash
askcode "which file computes invoice totals"
askcode "where is rate limiting applied" --path ../other-repo
askcode "how does retry backoff work" --json
```

| Flag | Meaning |
|---|---|
| `--path DIR` | project root to scan (default `.`) |
| `--json` | machine-readable output |
| `--model` | model id (default `atmo/deepseek-v4.1-flash`) |
| `--base-url` | OpenAI-compatible endpoint |
| `--timeout` | API timeout seconds (default 90) |

## How it works

1. Walks the tree, skipping `node_modules`, `.git`, `vendor`, build dirs and files >120 KB.
2. Splits files into ~60-line chunks (cap ~8 000 lines/run — token budget stays sane on big repos).
3. Sends the catalog to the model with a strict JSON-only ranking prompt.
4. Parses (tolerantly — fences, stray prose) and prints paths, line numbers, match reasons and snippets.

## Powered by AtmoRouter

The default model (`atmo/deepseek-v4.1-flash`) runs on [**AtmoRouter**](https://atmorouter.dev) — a frontier-model API gateway with a free tier and models at up to 90% below official pricing. Grab a free API key at [atmorouter.dev](https://atmorouter.dev) and `askcode` works out of the box with zero setup.

> 100M tokens of free monthly-tier usage per account — enough to search a codebase hundreds of times.

## License

MIT
