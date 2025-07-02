# Test PR Action

[![github stars](https://badgen.net/github/stars/phucbm/test-pr-action?icon=github)](https://github.com/phucbm/test-pr-action/)
[![github license](https://badgen.net/github/license/phucbm/test-pr-action?icon=github)](https://github.com/phucbm/test-pr-action/blob/main/LICENSE)
[![Made in Vietnam](https://raw.githubusercontent.com/webuild-community/badge/master/svg/made.svg)](https://webuild.community)

A GitHub Action that automatically tests pull requests and optionally auto-merges Dependabot PRs when tests pass. Perfect for maintaining high code quality with minimal manual intervention.

## Features
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   TRIGGER   │───▶│  DETECTION  │───▶│    TEST     │───▶│   ACTION    │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
       │                   │                   │                   │
       ▼                   ▼                   ▼                   ▼
   PR created         Auto-detect pkg      Run tests if         Comment results
   Comment command    manager & scripts    scripts exist        Auto-merge if
   PR updated         Support npm/pnpm/    Skip if missing      Dependabot + pass
                      yarn detection       Handle failures      Manual commands
```

## Quick Start

1. **Create Workflow File**
   Create `.github/workflows/test-pr.yml`:

```yaml
name: Test PRs

on:
  pull_request:
    types: [opened, synchronize]
  issue_comment:
    types: [created]

permissions:
  contents: write      # To auto-merge PRs
  pull-requests: write # To comment on PRs
  issues: write        # To comment on PRs

jobs:
  test:
    if: github.event_name == 'pull_request' || contains(github.event.comment.body, '/test')
    runs-on: ubuntu-latest
    steps:
      - name: Test PR
        uses: phucbm/test-pr-action@v1
        with:
          dependabot-auto-merge: 'true'  # Optional: auto-merge Dependabot PRs
```

2. **That's it!** 🎉
   - All new PRs are automatically tested
   - Results are commented directly on the PR
   - Use `/test` commands for manual testing
   - Dependabot PRs can be auto-merged when tests pass

## Commands

Comment these commands on any PR to trigger actions:

| Command | Description |
|---------|-------------|
| `/test` | Run tests and comment results |
| `/retest` | Re-run tests and comment results |
| `/test-merge` | Run tests and auto-merge if Dependabot PR passes |

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `dependabot-auto-merge` | Auto-merge Dependabot PRs if tests pass | ❌ No | `false` |
| `github-token` | GitHub token for commenting and merging | ❌ No | `${{ github.token }}` |
| `node-version` | Node.js version to use | ❌ No | `18` |
| `package-manager` | Package manager to use (npm, pnpm, yarn, auto) | ❌ No | `auto` |

## Basic Usage

**Minimal setup** (just test and comment):
```yaml
- name: Test PR
  uses: phucbm/test-pr-action@v1
```

**With auto-merge** for Dependabot PRs:
```yaml
- name: Test PR
  uses: phucbm/test-pr-action@v1
  with:
    dependabot-auto-merge: 'true'
```

## Advanced Usage

All available options:

```yaml
- name: Test PR
  uses: phucbm/test-pr-action@v1
  with:
    dependabot-auto-merge: 'true'           # Auto-merge Dependabot PRs if tests pass
    github-token: ${{ secrets.GITHUB_TOKEN }} # Custom GitHub token
    node-version: '20'                      # Node.js version
    package-manager: 'pnpm'                 # Force specific package manager
```

## How It Works

```
🎯 TRIGGER PHASE
   └── Automatically on PR create/update
   └── Manually via comment commands
   └── Smart filtering for relevant events

🔍 DETECTION PHASE  
   └── Auto-detect package manager (npm/pnpm/yarn)
   └── Check for test scripts in package.json
   └── Identify Dependabot PRs for auto-merge

🧪 TEST PHASE
   └── Install dependencies with detected package manager
   └── Run test scripts if they exist
   └── Handle test failures gracefully
   └── Skip tests if no script found

💬 ACTION PHASE
   └── Comment test results on PR
   └── Include helpful command instructions
   └── Auto-merge Dependabot PRs if configured
   └── Provide clear success/failure feedback
```

## Comment Examples

**Tests passed:**
```
✅ Tests passed! All tests completed successfully.

---
Test PR Action commands and options
• /test will run tests and comment the results
• /retest will re-run tests and comment the results  
• /test-merge will run tests and auto-merge this Dependabot PR if tests pass

💡 Tip: Auto-merge is currently enabled for Dependabot PRs.
```

**Tests failed:**
```
❌ Tests failed! Please check the workflow logs for details.

---
Test PR Action commands and options
• /test will run tests and comment the results
• /retest will re-run tests and comment the results

💡 Tip: Fix the failing tests and use /retest to verify the fixes.
```

**No tests:**
```
ℹ️ No test script found in package.json. Skipping tests.

---
Test PR Action commands and options
• /test will check for tests and comment the results

💡 Tip: Add a test script to your package.json to enable automatic testing.
```

## Package Manager Support

The action automatically detects your package manager:

| Package Manager | Detection Method | Install Command | Test Command |
|----------------|------------------|-----------------|--------------|
| **PNPM** | `pnpm-lock.yaml` exists | `pnpm install --no-frozen-lockfile` | `pnpm test` |
| **Yarn** | `yarn.lock` exists | `yarn install --no-immutable` | `yarn test` |
| **NPM** | Default fallback | `npm ci` | `npm test` |

You can also force a specific package manager using the `package-manager` input.

## Workflow Examples

**Basic testing for all PRs:**
```yaml
name: Test PRs
on:
  pull_request:
    types: [opened, synchronize]
  issue_comment:
    types: [created]

jobs:
  test:
    if: github.event_name == 'pull_request' || contains(github.event.comment.body, '/test')
    runs-on: ubuntu-latest
    steps:
      - uses: phucbm/test-pr-action@v1
```

**Auto-merge Dependabot PRs:**
```yaml
name: Test and Auto-merge
on:
  pull_request:
    types: [opened, synchronize]
  issue_comment:
    types: [created]

permissions:
  contents: write
  pull-requests: write
  issues: write

jobs:
  test:
    if: github.event_name == 'pull_request' || contains(github.event.comment.body, '/test')
    runs-on: ubuntu-latest
    steps:
      - uses: phucbm/test-pr-action@v1
        with:
          dependabot-auto-merge: 'true'
```

**Using with different Node.js versions:**
```yaml
strategy:
  matrix:
    node-version: [16, 18, 20]

steps:
  - uses: phucbm/test-pr-action@v1
    with:
      node-version: ${{ matrix.node-version }}
```

## Requirements

- Repository must have a `package.json` file for Node.js projects
- Test scripts should be defined in `package.json` (optional)
- Workflow must have appropriate permissions for commenting and merging

## Troubleshooting

**Action doesn't trigger on comments**
- Ensure your workflow includes `issue_comment` trigger
- Check that the comment contains `/test`, `/retest`, or `/test-merge`

**Auto-merge not working**
- Verify `dependabot-auto-merge: 'true'` is set
- Check that workflow has `contents: write` permission
- Ensure the PR is created by `dependabot[bot]`

**Tests not running**
- Verify you have a `test` script in `package.json`
- Check the workflow logs for package manager detection
- Try using `/test` command to manually trigger

**Permission denied**
- Ensure workflow has required permissions:
  ```yaml
  permissions:
    contents: write
    pull-requests: write
    issues: write
  ```

## Best Practices

1. **Use with Dependabot** for automatic dependency management
2. **Enable auto-merge** for trusted dependency updates
3. **Add comprehensive tests** to your package.json
4. **Monitor workflow logs** for any issues
5. **Use comment commands** for manual testing when needed

## License

MIT License - feel free to use in your projects!

## Contributing

Issues and pull requests welcome! This action is designed to be simple, reliable, and helpful for all JavaScript/TypeScript projects.
