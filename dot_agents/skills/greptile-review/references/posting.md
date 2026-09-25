# GitHub Review Delivery

Read this for PR history and authorized posting. Commands below use explicit OWNER/REPO/NUMBER
placeholders; replace them with the resolved target, not the current checkout's repository.
For GitHub Enterprise, use the target hostname with `gh api --hostname HOST` and
`gh pr view --repo HOST/OWNER/REPO`.

## Pin identity and revision

```bash
gh auth status
gh api user --jq .login
gh pr view NUMBER --repo OWNER/REPO --json number,url,state,isDraft,author,baseRefName,baseRefOid,headRefName,headRefOid,title,body
```

Record base/head SHAs and review scope. Revalidate them before posting. A changed base can alter
the diff even when the head is unchanged. Fetch source from the resolved base repository,
including `refs/pull/NUMBER/head` for fork PRs, and verify it matches the recorded head.

## Read complete history

```bash
gh api --paginate repos/OWNER/REPO/pulls/NUMBER/reviews
gh api --paginate repos/OWNER/REPO/pulls/NUMBER/comments
gh api --paginate repos/OWNER/REPO/issues/NUMBER/comments
```

Retain IDs, author, body, state, timestamps, commit IDs and reply relationships. REST pagination
returns one JSON array per page; consume each page or use `--slurp` when a single JSON value is
needed. Do not assume the first page contains the latest review or all replies.

Fetch thread resolution with a paginated GraphQL query. The REST comments above supply complete
reply bodies; the first comment here links each thread to that history.

```bash
gh api graphql --paginate -f query='
  query($owner: String!, $repo: String!, $pr: Int!, $endCursor: String) {
    repository(owner: $owner, name: $repo) {
      pullRequest(number: $pr) {
        reviewThreads(first: 100, after: $endCursor) {
          nodes {
            id isResolved isOutdated path line
            comments(first: 1) { nodes { databaseId url } }
          }
          pageInfo { hasNextPage endCursor }
        }
      }
    }
  }' -f owner=OWNER -f repo=REPO -F pr=NUMBER
```

If history is unavailable, disclose it. Do not infer a clean ledger or approve on that basis.
An existing pending review also needs to be inspected before creating another review; do not
submit or discard unrelated draft work.

## Prepare and validate

Build the complete body and inline comments before submitting. Store valid JSON in a temporary
file using a JSON serializer; never interpolate review prose into shell commands. `--input`
reads the JSON file directly. This single-line example is valid JSON; replace all values:

```json
{"commit_id":"HEAD_SHA","event":"COMMENT","body":"Review body","comments":[{"path":"src/example.ts","line":42,"side":"RIGHT","body":"Finding body"}]}
```

Validate anchors against `gh pr diff NUMBER --repo OWNER/REPO`, not the wider local context diff:

- `path` is the repository-relative path in GitHub's diff.
- `line` is a file line number, not a diff position. `RIGHT` is the head version; `LEFT` is the
  base version. Use the appropriate side for context lines and deletions.
- A multi-line comment includes `start_line` and `start_side` along with `line` and `side`.
  Keep the range short and on a valid diff hunk; prefer a single line when sufficient.
- If the causal location cannot be anchored, put the finding in the body with explicit
  `file:line` evidence. Do not attach it to an unrelated nearby line to satisfy the API.

Use the event policy in SKILL.md. Own-PR approvals and change requests may be prohibited;
prepare a COMMENT report instead and explain the limitation. Do not submit a formal verdict on
a closed/merged PR without an explicit request for that action; return the report locally.

Immediately before POST, re-fetch PR base/head and recent history. If source changed, reassess
changed paths, ledger and anchors. If this happens repeatedly, return the completed report with
its reviewed SHA instead of chasing an indefinitely moving target. If an equivalent review is
already posted for this SHA, link it instead of posting a duplicate; an explicit request to
publish a materially revised review can justify another submission.

## Submit and verify

```bash
gh api --method POST repos/OWNER/REPO/pulls/NUMBER/reviews --input /path/to/payload.json
```

Always set `event` explicitly; omitting it creates a pending review. Check the returned review
ID, URL, commit ID and state before saying it was posted. Link that review and give a finding
tally. A successful APPROVE does not prove all branch protection requirements are satisfied.

On failure:

- **Authentication/permission failure:** return the report locally and state it was not posted.
- **422:** inspect the response. It can mean invalid anchors, an invalid event, or an existing
  pending review. Correct the actual cause; do not blindly move every finding or resend.
- **Timeout or ambiguous response:** read recent reviews/comments first to determine whether
  submission succeeded. Never retry an uncertain mutation blindly. If the result cannot be
  established, stop and report the uncertain delivery state with the prepared report.

Do not delete comments, resolve threads, dismiss other reviews, or merge the PR as part of
posting. Those actions require their own user request. For re-reviews, summarize still-open
threads in the ledger and create new inline comments only for new distinct findings.

API references: [Review endpoints](https://docs.github.com/en/rest/pulls/reviews),
[review comment anchors](https://docs.github.com/en/rest/pulls/comments), and
[CLI pagination](https://cli.github.com/manual/gh_api).
