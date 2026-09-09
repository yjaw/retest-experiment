# retest-experiment

Throwaway repo to prototype the `/retest` slash command for kuberay
([ray-project/kuberay#5037](https://github.com/ray-project/kuberay/issues/5037))
before touching the real fork.

## What's here

| File | Purpose |
| --- | --- |
| `.github/workflows/flaky.yaml` | Deliberately flaky GitHub Actions pipeline (each job fails ~50%). |
| `.buildkite/pipeline.yml` | Deliberately flaky Buildkite pipeline (each step fails ~50%). |
| `.github/workflows/retest.yaml` | The thing under test: reacts to `/retest` and `/retest-all` PR comments. |
| `app.txt` | Bump this in a PR to trigger CI. |

## `/retest` behaviour

Comment on a PR:

- `/retest` — re-run only failed runs: GitHub Actions *failed jobs* of the latest
  run per workflow, plus a Buildkite **retry of failed jobs** for the latest build.
- `/retest-all` — re-run everything: all GitHub Actions jobs, plus a **fresh**
  Buildkite build.

The workflow reacts to the comment (🚀 accepted, 👍 done, 😕 not permitted) and
posts a summary comment.

Permission: the commenter must be the PR author or have write/maintain/admin
permission (`ALLOWED` env in `retest.yaml`). Final policy is TBD with kuberay maintainers.

## Setup — see SETUP.md

1. Create the GitHub repo and push `main`.
2. Create a Buildkite pipeline pointed at the repo, enable "Build pull requests".
3. Create a Buildkite API token; add it as the `BUILDKITE_API_TOKEN` **secret**.
4. Add `BUILDKITE_ORG` and `BUILDKITE_PIPELINE` repo **variables**.
5. Open a PR that edits `app.txt`, wait for CI to (probably) flake, comment `/retest`.
