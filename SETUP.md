# Setup steps

Run these yourself — they need your GitHub / Buildkite accounts.

## 1. Create the GitHub repo and push

```bash
cd ~/Developer/retest-experiment
git add -A
git commit -m "Scaffold retest experiment"
gh repo create retest-experiment --private --source=. --remote=origin --push
```

`issue_comment` workflows only run from the repository's **default branch**,
so the `retest.yaml` file must already be on `main` before PR comments can
trigger it. This does not mean the branch must be merged to `main` before CI
runs; it only means the trigger itself must live on the default branch while the
PR branch continues to test normally and can be merged after it passes.

## 2. Create the Buildkite pipeline

1. Buildkite -> **New Pipeline**.
2. Connect it to the `retest-experiment` GitHub repo (install/authorize the
   Buildkite GitHub App on the repo if prompted).
3. Leave the default step as `buildkite-agent pipeline upload` — it will pick up
   `.buildkite/pipeline.yml`.
4. Pipeline **Settings -> GitHub**:
   - Enable **Build pull requests**.
   - (Optional) Enable **Build pull request forks** if you'll test from a fork.
   - Enable **Update commit statuses** so the result shows on the PR.
5. Note the **org slug** and **pipeline slug** from the pipeline URL:
   `https://buildkite.com/<ORG_SLUG>/<PIPELINE_SLUG>`.

## 3. Buildkite API token

1. https://buildkite.com/user/api-access-tokens -> **New API Access Token**.
2. Scope it to your org, with **Read Builds** and **Write Builds**.
3. Copy the token.

## 4. Wire secrets/vars into the repo

```bash
gh secret set BUILDKITE_API_TOKEN --repo <you>/retest-experiment   # paste token
gh variable set BUILDKITE_ORG      --repo <you>/retest-experiment --body "<ORG_SLUG>"
gh variable set BUILDKITE_PIPELINE --repo <you>/retest-experiment --body "<PIPELINE_SLUG>"
```

## 5. Exercise it

```bash
git checkout -b test-pr
printf 'v2\n' >> app.txt
git commit -am "bump app.txt"
git push -u origin test-pr
gh pr create --fill
```

- Wait for **Flaky CI** (GHA) and the Buildkite build to finish. Usually at least
  one job/step flakes red.
- Comment `/retest` on the PR.
  - Expect: 🚀 reaction, then a summary comment listing which GHA workflows had
    failed jobs re-run and the Buildkite failed-job retry link, then a 👍 reaction.
  - Only the failed GHA jobs and a Buildkite failed-job retry should start.
- Comment `/retest-all` to confirm every job + a fresh Buildkite build start.
- From a second account with no write access, comment `/retest` and confirm the
  😕 reaction + failed check.

## Teardown

Delete the GitHub repo and the Buildkite pipeline when done.
