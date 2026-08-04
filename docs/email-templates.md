# Email Templates

This document clarifies how (and whether) email templates relate to `gh`, since the term is sometimes assumed to be a CLI feature.

## `gh` does not define or render email templates

`gh` has no concept of an email template, and no code path in this repository composes or sends email. Notification emails (issue comments, PR reviews, mentions, etc.) are generated and delivered entirely server-side by GitHub.com or GitHub Enterprise Server. Their content and formatting are controlled by GitHub's backend and are not configurable from, or reachable by, the CLI.

If you are looking to customize the appearance of notification emails, that configuration lives in your GitHub account or organization notification settings on the web, not in this codebase.

## Related template features

The repository does define a few constructs that use the word "template" but are unrelated to email:

- **Issue and pull request templates** - `IssueTemplate` and `PullRequestTemplate` in `api/queries_repo.go` model the repository's `.github/ISSUE_TEMPLATE` and `.github/PULL_REQUEST_TEMPLATE` content fetched from the GitHub API. These are used to pre-populate the body of a new issue or pull request when creating one through `gh issue create` or `gh pr create`.
- **Editor template** - `EditorTemplateData` in `pkg/surveyext/editor.go` supplies the scaffold text shown when `gh` opens an external editor (for example, to compose an issue or PR body interactively).

Both are content templates for text the user authors or submits to GitHub, not templates for outbound email. If a future request references "email templates" in this repository, verify whether it actually means one of these two constructs.
