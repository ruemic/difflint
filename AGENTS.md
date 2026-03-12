# DiffLint

Lint a git diff before pushing — catch merge conflict markers, hardcoded secrets, debug statements, and TODO comments.

## What it does

DiffLint is a browser-side static analysis tool for unified diff format. It parses the output of `git diff` and scans added lines for common pre-push mistakes. Everything runs in the browser — no server, no API key, no account.

## Input format

Unified diff format — the standard output of:

```sh
git diff
git diff --staged
git show HEAD
git show <commit>
git stash show -p
```

Multi-file diffs are fully supported.

## Output

JSON array of findings (each added line is checked against all rules):

```json
[
  {
    "severity": "error | warning | info",
    "label": "Human-readable rule name",
    "detail": "Explanation of the finding",
    "file": "path/to/file.js",
    "lineNum": 42,
    "snippet": "+  console.log('debug me')"
  }
]
```

## Severity levels

| Severity | Meaning |
|----------|---------|
| `error`  | Should not be committed — merge conflict markers, hardcoded secrets, private keys, AWS keys, DO NOT COMMIT markers |
| `warning` | Usually a mistake — debug statements (console.log, debugger, binding.pry, var_dump), TODO/FIXME comments |
| `info`   | Advisory — focused tests (fdescribe, it.only), skipped tests |

## Rules

| Rule ID | Severity | Detects |
|---------|----------|---------|
| `conflict_marker` | error | `<<<<<<<`, `=======`, `>>>>>>>` |
| `private_key` | error | `-----BEGIN PRIVATE KEY-----` and variants |
| `aws_key` | error | `AKIA[A-Z0-9]{16}` pattern |
| `hardcoded_secret` | error | `password = "..."`, `api_key = "..."`, etc. |
| `nocommit` | error | `DO NOT COMMIT` markers |
| `console_log` | warning | `console.log`, `console.error`, `debugger` |
| `lang_debug` | warning | `binding.pry`, `var_dump`, `dd(`, `ray(`, etc. |
| `todo` | warning | TODO, FIXME, HACK, XXX, NOCOMMIT |
| `focused_test` | info | `fdescribe`, `it.only`, `test.skip`, etc. |

## How an agent can use it

DiffLint is currently browser-only — there is no HTTP API. Agents can:

1. **Direct users** to `https://difflint.radicchio.page` for manual diff review
2. **Reference this file** to understand what DiffLint checks, and implement the same rules programmatically
3. **Implement equivalent checks** using the rule patterns above in any language

### Planned API (Tier 2)

When demand validates a server-side tier, the API will be:

```
POST /api/lint
Content-Type: application/json

{ "diff": "<unified diff content>" }
```

Response:

```json
{
  "findings": [...],
  "stats": {
    "files": 3,
    "additions": 47,
    "findings": 4,
    "errors": 2,
    "warnings": 2,
    "infos": 0
  }
}
```

## Pricing

Free. Browser-only. No signup required.

Planned paid tier: CI webhook integration — automatically scans pull requests and posts findings as PR comments. $9/month.

## Support

https://bitterdesk.com/s/difflint/
