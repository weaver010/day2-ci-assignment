# Runbook: setup and Part 9 scenarios

Replace `YOUR-USERNAME` in README.md (badge) first. Commands use the GitHub CLI (`gh`).

## One-time setup
```bash
pip install -r requirements-dev.txt && pytest -v      # must pass locally
git init -b main && git add . && git commit -m "Initial commit"
gh repo create day2-ci-assignment --public --source=. --push
gh variable set REPORT_TITLE --body "Temperature CI Report"
gh secret set NOTIFY_TOKEN --body "fake-token-123456"
```
(If you use the web UI: Settings > Secrets and variables > Actions.)

## Scenario 1: green run on main
The initial push triggers CI. Check: all jobs ✅, summary visible, artifacts `test-report` and `package`.

## Scenario 2: docs-only change
```bash
echo "" >> README.md && git commit -am "docs: tweak readme" && git push
```
No new run should appear.

## Scenario 3: failing PR
```bash
git switch -c break-test
sed -i 's/== 212/== 213/' test_temperature.py   # breaks the first test
git commit -am "Break test" && git push -u origin break-test
gh pr create --fill
```
Expected: test ❌, 🚨 step runs, test-report uploaded, package/verify/pr-comment skipped, report runs.

## Scenario 4: fixed PR
```bash
sed -i 's/== 213/== 212/' test_temperature.py
git commit -am "Fix test" && git push
gh pr merge --merge
```
Expected: all green and bot comment "✅ CI passed. Package version 0.1.N is ready."

## Scenario 5: manual run
```bash
gh workflow run ci.yml -f python-version=3.12 -f create-package=false
```
Expected: test on 3.12, package/verify ⏭️, summary version `n/a`.

Collect links with `gh run list` / `gh run view --web`.
