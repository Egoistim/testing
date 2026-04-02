---
name: Issue Review Agent
description: Reviews newly opened issues for clarity and actionability.
run-name: Issue Review #${{ github.event.issue.number }}
on:
  issues:
    types: [opened, reopened]
  roles: [read, write, maintainer, admin]
  skip-bots: [github-actions, copilot-swe-agent]
permissions:
  contents: read
  issues: read
  pull-requests: read
  actions: read
engine:
  id: codex
  env:
    OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
    OPENAI_BASE_URL: ${{ vars.OPENAI_BASE_URL }}
strict: true
timeout-minutes: 8
network: defaults
concurrency:
  group: gh-aw-issue-review-${{ github.event.issue.number }}
  cancel-in-progress: true
checkout:
  fetch-depth: 1
tools:
  github:
    toolsets: [default, issues]
  bash:
    - pwd
    - ls
    - cat
    - rg
safe-outputs:
  add-comment:
    max: 1
    footer: false
---
# Issue Review Agent

You are reviewing the triggering issue in `${{ github.repository }}`.

Your job is to leave one concise, high-signal issue review comment that helps
maintainers decide whether the issue is clear and actionable.

## Review Process

1. Read the issue title, body, and existing comments.
2. Inspect the repository files that are needed to understand the request.
3. Decide whether the issue is:
   - `Clear`
   - `Needs Detail`
   - `Likely Duplicate`
   - `Out of Scope`
4. Post exactly one issue comment.

## Comment Format

- Start with one verdict line using one of the four labels above.
- Then provide 2-4 short bullets covering:
  - your interpretation of the request
  - the biggest missing detail or risk, if any
  - the next concrete step
- If the issue is already actionable, say so directly and avoid unnecessary questions.
- Keep the tone concise and professional.

## Constraints

- Do not modify files.
- Do not create issues or pull requests.
- Do not speculate. If evidence is weak, say so.
