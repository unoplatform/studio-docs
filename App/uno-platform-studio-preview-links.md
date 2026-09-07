---
uid: Uno.PlatformStudio.PreviewLinks
---

# Preview links from your CI

A pull request changes your app's UI. Reviewing it means reading a diff and imagining the result, or checking
out the branch and building it — a several-minute detour most reviewers skip, and one that designers and
product reviewers often cannot take at all.

A preview link removes that step. Your repository builds the app on its own CI runner, publishes the result to
storage you own, and posts one link on the pull request. Clicking it opens Uno Platform Studio with that
branch's app already running: no clone, no build, no local toolchain.

## What you need, and what you do not

You need:

- A place to put files that can be read over HTTPS — an Azure Blob container, an S3-compatible bucket, or any
  static host you control.
- One workflow in your repository.

You do **not** need an account with us, a service key, or any credential of ours. The build runs on your
runner; the files live on your storage; the only credentials involved are your own, for your own storage,
managed the way you already manage your deployment credentials.

Nothing about your app is uploaded to us.

> [!IMPORTANT]
> **The `uno-studio` CLI is not published yet.** `dotnet tool install --global uno.studio` cannot succeed
> today — neither it nor the build engine it fetches has been pushed to NuGet. Everything below is complete
> and exercised on our own CI against locally built packages, so the workflow, the option names and the
> failure behavior are accurate rather than aspirational; what is missing is the publication step alone.
> Until it lands, this article is a description of how the feature will be used, not something you can run.

## How it works

1. Your workflow builds the app in the format Studio can open, using the `uno-studio` CLI.
2. Your workflow uploads the result to your storage with that storage's own tooling.
3. Your workflow posts one comment on the pull request, rewritten in place as new commits arrive.
4. A reviewer clicks the link. Studio fetches the files, checks that they are byte-for-byte what the link was
   created for, shows where the app claims to come from, and asks before running anything.

If any of that does not work out — the files were removed, the storage is unreachable, the preview was built
for a different version of Studio — the link still opens the project and builds it. **A preview link never
becomes a dead link.**

## The workflow

The example below is complete and uses Azure Blob Storage with federated identity, so no long-lived secret
exists anywhere. The S3 variant swaps two steps; see [Uploading](#uploading).

```yaml
name: Studio preview

on:
  pull_request:
    types: [opened, synchronize, reopened, closed]

permissions:
  contents: read
  id-token: write        # federated cloud login
  pull-requests: write   # the preview comment

env:
  PREVIEW_CONTAINER: previews
  PREVIEW_ACCOUNT: mycompanypreviews
  PREVIEW_PREFIX: pr/${{ github.event.number }}/${{ github.run_id }}

jobs:
  preview:
    if: github.event.action != 'closed' && github.event.pull_request.head.repo.full_name == github.repository
    runs-on: ubuntu-latest
    steps:
      # Build the HEAD the reviewer is looking at, not the merge commit CI would check out by default:
      # the two can render differently, and the link, the package and the source fallback should all
      # name one commit.
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.sha }}

      - uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '10.0.x'

      - name: Install the CLI
        run: dotnet tool install --global uno.studio

      # Exit 10 means "this app did not compile" — a result to report, not a broken task. So the exit
      # code is captured rather than allowed to fail the job, and the comment below reports it.
      - name: Build the preview
        id: build
        run: |
          set +e
          uno-studio build \
            --source . \
            --out "$RUNNER_TEMP/preview" \
            --project src/MyApp/MyApp.csproj
          echo "exit=$?" >> "$GITHUB_OUTPUT"
        env:
          GITHUB_PR_HEAD_SHA: ${{ github.event.pull_request.head.sha }}
          GITHUB_PR_NUMBER: ${{ github.event.number }}

      - uses: azure/login@v2
        with:
          client-id: ${{ vars.AZURE_CLIENT_ID }}
          tenant-id: ${{ vars.AZURE_TENANT_ID }}
          subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID }}

      - name: Upload the preview
        run: |
          az storage blob upload-batch \
            --auth-mode login \
            --account-name "$PREVIEW_ACCOUNT" \
            --destination "$PREVIEW_CONTAINER" \
            --destination-path "$PREVIEW_PREFIX" \
            --source "$RUNNER_TEMP/preview"

      # --emit puts the link in this step's outputs and in the job summary. `--output:json --no-logo`
      # would give you the same values on standard output if you would rather pipe them.
      - name: Compose the link
        id: link
        run: |
          uno-studio package link \
            --descriptor "$RUNNER_TEMP/preview/package-descriptor.json" \
            --published-base "https://$PREVIEW_ACCOUNT.blob.core.windows.net/$PREVIEW_CONTAINER/$PREVIEW_PREFIX" \
            --emit
        env:
          GITHUB_PR_HEAD_SHA: ${{ github.event.pull_request.head.sha }}
          GITHUB_PR_NUMBER: ${{ github.event.number }}

      - name: Comment on the pull request
        env:
          GH_TOKEN: ${{ github.token }}
          LINK: ${{ steps.link.outputs.link }}
          RUN_URL: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
        run: |
          {
            echo "### Studio preview"
            echo
            if [ "${{ steps.build.outputs.exit }}" = "0" ] && [ -n "$LINK" ]; then
              echo "[Open this pull request's app in Studio]($LINK)"
              echo
              echo "Built from \`${{ github.event.pull_request.head.sha }}\`."
            elif [ "${{ steps.build.outputs.exit }}" = "10" ]; then
              echo "This app did not compile, so there is nothing to preview yet."
              echo
              echo "[Build log]($RUN_URL)"
            else
              echo "The preview task could not run. [Details]($RUN_URL)"
            fi
          } > "$RUNNER_TEMP/comment.md"

          gh pr comment "${{ github.event.number }}" \
            --body-file "$RUNNER_TEMP/comment.md" \
            --edit-last --create-if-none

      # Report the app's own failure after the comment, so the pull request gets its feedback either way.
      - name: Fail if the app did not compile
        if: steps.build.outputs.exit != '0'
        run: exit 1

  cleanup:
    if: github.event.action == 'closed'
    runs-on: ubuntu-latest
    steps:
      - uses: azure/login@v2
        with:
          client-id: ${{ vars.AZURE_CLIENT_ID }}
          tenant-id: ${{ vars.AZURE_TENANT_ID }}
          subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID }}

      - name: Remove this pull request's previews
        run: |
          az storage blob delete-batch \
            --auth-mode login \
            --account-name mycompanypreviews \
            --source previews \
            --pattern "pr/${{ github.event.number }}/*"
```

A repository with a single app can drop `--project`. A repository with several **must** name one: guessing
would publish a preview of the wrong app, which a reviewer has no way to notice.

### Posting the comment

The example uses `gh pr comment --edit-last --create-if-none`, which is GitHub's own CLI, preinstalled on
hosted runners, and gives you a comment that is rewritten in place instead of a new one per commit. Nothing
third-party is involved and the only permission needed is `pull-requests: write`.

One limitation worth knowing: `--edit-last` edits the last comment **by the same account**. With the default
token that account is the Actions bot, so if another workflow in your repository also comments as the bot on
the same pull request, the two can overwrite each other. If that is your situation, either give this workflow
its own app token or find the comment yourself with `gh api` and a marker string in the body.

The comment is composed in the workflow, not by the CLI. The CLI's job is to give you the link — what you say
around it, in which language, and whether you say it at all, is yours.

### Where the link shows up

`package link` prints its result on standard output, so a script can read it directly:

```bash
LINK=$(uno-studio package link \
  --descriptor "$RUNNER_TEMP/preview/package-descriptor.json" \
  --published-base "$BASE" \
  --output:json --no-logo | jq -r .pinnedLink)
```

`--out <file>` writes the same document to a file when you want one; omit it and nothing touches the disk.

`--emit` surfaces the link inside the run itself:

- **The step's outputs** — `link`, `source-ref`, `descriptor-digest` and `status` — so a later step can use
  them. Read them the usual way, with an `id:` on the step and `steps.<id>.outputs.link`.
- **The job summary** — a clickable link on the run's own page, for anyone who would rather not open a
  comment thread.

Both are optional and independent of the comment. A repository that would rather not comment on pull requests
can drop the comment steps entirely and still get the link from the summary.

Outside a CI run `--emit` does nothing and says so, so the same command works on your own machine.

### Publishing somewhere else

`build` writes a `links.json` **before** anything is uploaded, holding the three values a link is made of:

```json
{
  "project": "myorg/myrepo@abc1234:src/MyApp",
  "descriptorDigest": "sha256-...",
  "descriptorFileName": "package-descriptor.json"
}
```

So any storage works, whatever its tooling: upload the output directory, then call `package link
--published-base <the address you uploaded to>` and it composes the link from the descriptor's actual bytes.

Do the substitution with `package link` rather than assembling the URL yourself. The link's `package=` value
is percent-encoded, and a hand-assembled one is a link that looks right and silently opens from source.

### Pull requests from forks

The `if:` on the `preview` job above skips forks deliberately, and you should keep it.

A fork's workflow run receives none of your repository's secrets and holds a read-only token, so it can
neither upload nor post a comment. Letting those steps run and fail would put a red job on every ordinary
outside contribution. Building a fork's code in a privileged context to work around that is a well-known way
to leak your credentials, and is not something to do.

Fork pull requests still get reviewed the way they are today: check out the branch and run it.

## What your storage has to do

The whole contract is small, because Studio fetches your files over plain HTTPS.

| Requirement | Why | Required |
| --- | --- | --- |
| Serve the published files over HTTPS to your intended readers | Studio fetches them directly | Yes |
| A cross-origin rule allowing `GET` and `OPTIONS` from Studio's origin | The browser enforces it | Yes |
| Refuse overwrites of existing files | Catches an accidental republication when it happens | Recommended |
| A retention rule on the preview prefix | Cleans up pull requests whose cleanup step never ran | Recommended |

Range requests, listing, custom headers, and signed URLs are all unnecessary.

### The cross-origin rule

Without this, every preview opens from source and the reviewer sees a notice saying so. It is a one-time
setting on the account or bucket.

For Azure Blob Storage:

```bash
az storage cors add \
  --account-name mycompanypreviews \
  --services b \
  --methods GET OPTIONS \
  --origins https://studio.platform.uno \
  --allowed-headers '*' \
  --exposed-headers '*' \
  --max-age 3600
```

For an S3-compatible bucket, apply this configuration:

```json
{
  "CORSRules": [
    {
      "AllowedMethods": ["GET", "HEAD"],
      "AllowedOrigins": ["https://studio.platform.uno"],
      "AllowedHeaders": ["*"],
      "MaxAgeSeconds": 3600
    }
  ]
}
```

> [!NOTE]
> If Studio is served from an additional origin in future, this rule needs that origin added before previews
> open from there. It is the one place where a change on our side asks for a change on yours.

### Uploading

The upload contains no code of ours: it is your storage's own tooling, authenticated the way your deployments
already authenticate. Prefer your CI provider's federated identity so no long-lived secret exists.

For Azure Blob Storage, use `az storage blob upload-batch` as shown in the workflow above.

For an S3-compatible bucket, replace the login and upload steps with:

```yaml
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ vars.AWS_ROLE_ARN }}
          aws-region: ${{ vars.AWS_REGION }}

      - name: Upload the preview
        run: |
          aws s3 cp "$RUNNER_TEMP/preview" \
            "s3://my-previews/$PREVIEW_PREFIX/" --recursive
```

Anything else works too. The contract is "make these files readable at HTTPS addresses" — `curl -T`,
`rclone`, a static host's deploy step, or a CDN push all qualify.

### Refusing overwrites

Each run publishes under a fresh directory named after the run, so a link's content is never replaced in
normal operation. Making that a property rather than a convention costs one setting, and it turns an
accidental republication into an error at publish time instead of a surprise at open time.

Both major storage families support it natively:

- **S3-compatible**: send `If-None-Match: *` with each upload, which fails with `412` if the object exists.
  You can make it a bucket-wide rule with the `s3:if-none-match` policy condition key, so a workflow cannot
  forget it.
- **Azure Blob Storage**: a service SAS scoped to one blob with only the **Create** permission refuses every
  write after the first, by permission alone.

> [!NOTE]
> Under federated login (`--auth-mode login`), Azure's blob data actions do not separate "create" from
> "overwrite", so the guarantee rests on a per-request conditional header instead. Verify that your upload
> command carries one before relying on it.

### Retention

Previews accumulate: one directory per run, per pull request. The cleanup job in the workflow above removes a
pull request's previews when it closes, and a lifecycle rule is the safety net for the closures it misses —
a cancelled run, a deleted branch, a workflow edited mid-flight.

Choose a window that outlasts a normal review. Thirty days is a reasonable starting point. A link whose files
have been removed still opens the project; expiry costs speed, not access.

## Who can see a preview

**A preview is exactly as visible as the place you published it.** Studio adds no audience: it fetches what
the reviewer's browser can reach, and nothing more.

For a public repository, publishing to publicly readable addresses is the straightforward choice and matches
what the repository already discloses.

For a private repository, understand this before you start: **publishing to a publicly readable address
discloses your built application to anyone with the link.** Keeping a preview private is possible but not
free, because an anonymous browser request cannot carry an authorization header. The options are:

- **Network perimeter** — storage reachable only from your network. Reviewers off the network fall back to
  opening from source.
- **Credentialed cross-origin** — `Access-Control-Allow-Credentials` with a non-wildcard origin, and a session
  the reviewer already holds on the storage's own domain. Only works where that is already true.
- **Signed URLs** — a short-lived signature in the address. Simple, with one consequence worth knowing: the
  link stops working when the signature expires, and from then on it opens from source.

If none of those fits, previews may not be the right feature for that repository yet.

### What the built files say about your runner

The built application carries the path it was built in. That is not something we add — the platform's own
generators record where a project lived when they ran — but it means the directory your workflow built in is
readable by anyone who downloads the preview.

On a GitHub-hosted runner that path names nobody. On a **self-hosted** runner it names the account the runner
runs as. There is no option to redirect it today: point the build at a neutral working directory instead, by
running it from one.

```bash
cd /tmp/preview-build && uno-studio build --source "$GITHUB_WORKSPACE" --out "$RUNNER_TEMP/preview"
```

The build log and any compile errors are handled separately: those are rewritten to paths relative to your
checkout before they are published, so a reviewer reading a build failure sees `src/MyApp/MainPage.xaml`, not
a path from your machine.

## What a reviewer sees

Before anything runs, Studio shows a confirmation naming the repository, the pull request, the commit, and
**the host the files came from** — and it is explicit about which of those it actually verified:

- **Verified**: the files are byte-for-byte what the link was created for. Always checked.
- **Declared**: the repository, pull request, commit and run, as stated by the workflow that published the
  preview. Studio does not, on its own, prove these.

Showing the host is deliberate. If you know where your previews live, a preview served from somewhere else is
something you can notice.

The confirmation is asked **every time**. There is no way to suppress it, on purpose: running a preview means
running code from a pull request, which is the same decision as checking out that branch and running it
locally, and the confirmation is the moment you make that decision knowingly.

## Proving the preview came from your CI

Everything above rests on one honest limitation: Studio proves the **files** match the link, and takes the
repository, pull request and commit as **stated** by whatever published the preview. A link and a set of files
that agree with each other can still have been assembled by someone else.

Closing that gap is optional and needs one thing we cannot avoid providing: a service that can check your CI
run's identity and sign a statement about it. That is the only part of this feature that involves us at all,
it is off by default, and **it is not available yet**.

Until it is, there is nothing to pass: `uno-studio package certify` is not offered by this release, and
`build`'s certification options are hidden. That is deliberate rather than incidental. A preview that
declares a signed statement Studio cannot verify is **refused** — not downgraded to the declared wording —
because "the statement is missing" and "the statement was removed" look identical from the reviewer's side,
and the safe reading of that is tampering. So an option that could only produce a refused link is better not
offered.

When it becomes available it will be two steps in your workflow: one to declare the statement as the build
runs, one to obtain and publish it. Both, or neither.

## When a preview opens from source instead

This is normal, and the reviewer is always told which path was taken. Common reasons:

| The reviewer sees | What happened |
| --- | --- |
| "This preview is no longer available" | The files were removed by your cleanup or your retention rule, or the storage was unreachable. Also what a missing cross-origin rule looks like. |
| "This preview was built for a different version of Studio" | Studio's build material changed since the preview was built. Push a new commit to rebuild. |
| "This preview was not built for the platform you are on" | Only the web preview is published today. |
| "The preview build did not succeed" | Your app did not compile. The comment links the build log. |
| "What this link points at is not what the link was created for" | The published files changed after the link was posted. Worth looking into — see [Refusing overwrites](#refusing-overwrites). |

## Troubleshooting

**Every preview opens from source, and the reviewer sees "no longer available".**
Almost always the cross-origin rule. A browser cannot tell a missing rule from a missing file, so both look
the same. Check the rule against the origin in the link.

**The build step fails with an ambiguous-project error.**
Your repository holds more than one app. The error lists them; pass one to `--project`.

**The comment is not posted, but the build succeeded.**
Check the pull request came from a branch of your repository rather than a fork, and that the workflow
declares `pull-requests: write`.

**The link opens the wrong app.**
Check `--project`. The link names the folder holding the app you built, and a reviewer opening it gets that
folder.
