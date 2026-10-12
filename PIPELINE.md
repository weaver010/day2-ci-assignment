# Pipeline Design

## 1. Events and why

| Event | Why |
|-------|-----|
| `push` to `main` (ignoring `**.md` and `docs/**`) | Verify every code change that lands on main; docs-only changes cannot break code, so skipping saves minutes. |
| `pull_request` targeting `main` | Validate changes *before* merge and give reviewers a green/red signal. |
| `workflow_dispatch` (inputs: `python-version`, `create-package`) | Manual re-runs and testing on a chosen Python version without a commit. |
| `schedule` (Mondays 07:00 UTC) | Catch silent breakage from dependency or runner-image updates when no one is pushing. |

## 2. Jobs and job graph

```
            ┌──────────┐
            │   test   │
            └────┬─────┘
                 │
            ┌────▼─────┐
            │ package  │   (skipped if manual run with create-package = false)
            └─┬──┬───┬─┘
              │  │   │
       ┌──────▼┐ │  ┌▼────────────┐
       │verify │ │  │ pr-comment  │  (pull requests only)
       └───────┘ │  └─────────────┘
                 │
   ┌─────────────▼──────────────┐
   │ report (needs test+package,│
   │ runs always)               │
   └────────────────────────────┘
```

- `test` runs first, alone.
- `package` waits for `test`.
- `verify` and `pr-comment` wait for `package` and run in parallel with each other.
- `report` waits for `test` and `package` but uses `if: always()`, so it runs after them in every case (it does not wait for `verify`/`pr-comment`).

## 3. Artifacts

- `test-report`: JUnit XML (`reports/junit.xml`), kept 5 days, uploaded even on failure.
- `package`: `dist/temperature-<version>.tar.gz`: **the release artifact** of this pipeline (version `0.1.<run_number>`).

## 4. CI, Continuous Delivery or Continuous Deployment?

It is **Continuous Integration** (with a packaging step): it builds, tests and verifies every change, but nothing is released or deployed to any environment.

## 5. What happens to `package` when tests fail?

It is **skipped** (because `needs: test` requires success): we must never build or publish an artifact from code that failed its tests, and skipping saves runner time while `report` still surfaces the failure.
