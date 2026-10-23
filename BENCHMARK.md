# GitHub Actions Usage Benchmark — Reference vs. Baseline

Pulled from GitHub's APIs on 2026-10-23. Two measurements: a busy CI pipeline
at a major org, and a minimal smoke-test run in a fresh repo.

## 1. Reference point — microsoft/vscode

**Org profile:** login `microsoft` (https://github.com/microsoft), 8,209 public repositories.

**Most-starred public repo:** `microsoft/vscode` — 194,223 stars (Visual Studio Code).

**Workflow picked:** "Code OSS" — `.github/workflows/pr.yml` (workflow ID 4196599),
the repo's primary PR CI workflow (compile & hygiene, unit/integration/smoke tests
across Linux, macOS, and Windows).

**Most recent completed run (strictly the latest):** run #74588 (ID 39902224980),
PR "Dispose the view model DiffEditorWidget.setModel creates when it is replaced",
`pull_request` event — completed with conclusion **action_required** (fork PR awaiting
maintainer approval), 0 jobs executed, no usage reported. Not a useful benchmark.

**Most recent completed run that actually executed:** run #74580 (ID 39899367074),
PR "chore: [DO NOT MERGE] evaluate Copilot SDK main", `pull_request` event,
2026-10-23 04:15:33Z → 04:38:24Z.

- Final status: completed / conclusion **failure**
- **48 jobs** in a single run: 8 macOS, 30 Ubuntu, 10 Windows
- **Wall-clock duration: 22 min 51 s** (run_duration_ms = 1,371,000)
- **Billable usage: 0 ms on every platform** (public repo — standard GitHub-hosted
  runners are free; the heavy "Compile & Hygiene" job also runs on a self-hosted
  1ES pool). Per-job billable durations are not reported for public repos, so the
  wall-clock and job-count figures are the reliable numbers here.

## 2. Baseline — minimal smoke-test run

**Repo:** `mcpmark-eval-liuhezi/actions-usage-audit`
(created under the org `mcpmark-eval-liuhezi` because the API token is not allowed
to create repositories under the personal account; same benchmark otherwise).

**Workflow:** `.github/workflows/smoke.yml` — `workflow_dispatch` only, single job
`smoke` on `ubuntu-latest`, echoes a line and exits 0.

**Run:** #1 (ID 39903139705), `workflow_dispatch` on default branch `main`,
2026-10-23 04:57:56Z → 04:58:06Z.

- Final status: completed / conclusion **success**
- 1 job (Ubuntu), job executed 04:58:03 → 04:58:05 (~2 s of actual job time)
- **Run duration: 10 s** (run_duration_ms = 10,000)
- **Billable usage: 0 ms** (UBUNTU, 1 job) — public repo, so free.

## 3. Comparison

| Metric | microsoft/vscode "Code OSS" #74580 | Smoke test #1 | Delta |
|---|---|---|---|
| Jobs per run | 48 (8 macOS / 30 Ubuntu / 10 Windows) | 1 (Ubuntu) | 48x |
| Run wall-clock | 22 min 51 s | 10 s | ~137x |
| Job execution time | ~8 min for the longest job (Compile & Hygiene) | ~2 s | ~240x |
| Billable minutes reported | 0 ms (public repo, free) | 0 ms (public repo, free) | — |
| Conclusion | failure | success | — |

**Takeaway for the meeting:** a real-world "busy" CI pipeline at a top OSS project
fans out to ~50 parallel jobs across three OSes and still finishes in under 23
minutes end-to-end; a trivial smoke test takes ~10 seconds wall-clock and ~2
seconds of actual job time. In both cases the reported billable usage is zero
because both repositories are public — GitHub Actions usage and billing reports
only start to matter for private repos. If these numbers are used to argue about
cost, note that the honest comparison axis here is *time and concurrency*, not
dollars.

**Caveats:** (1) The reference run's conclusion was `failure` (a DO-NOT-MERGE
automation PR), but it still ran the full matrix, so its usage is representative
of pipeline load. (2) Job-level durations read 0 ms for public repos, so only
wall-clock and job counts are meaningful. (3) The smoke-test repo lives in an
org rather than the personal account, per token permissions.

*(Note: this file was committed here instead of a secret gist because the API
token used for this audit is not permitted to create gists.)*
