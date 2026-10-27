# Assignment Submission Portal (CSC418 Lab Midterm)

Flask app + pytest tests + GitHub Actions CI that posts the test report to a local receiver, which saves `test-report.json`.

```
app/        Flask portal (validation, JSON storage, template)
tests/      3 required test cases
receiver/   local HTTP receiver -> test-report.json
scripts/    runs pytest, builds the JSON report, POSTs it
demo/       static demo deployed to GitHub Pages (bonus)
.github/workflows/ci.yml
```

## 1. Run the application
```bash
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
python -m app.main            # http://localhost:5000
```
Submissions are saved to `data/submissions.json`.

## 2. Run the tests
```bash
pytest -v
```

## 3. Start the receiver (keep it running during the demo)
```bash
python receiver/receiver.py --port 8000     # writes ./test-report.json
```
Try it locally without CI:
```bash
RECEIVER_URL=http://localhost:8000/report python scripts/run_tests_and_report.py
```

## 4. Connect GitHub Actions to your computer
A GitHub-hosted runner can't see your `localhost`, so expose the receiver with a tunnel (second terminal):
```bash
cloudflared tunnel --url http://localhost:8000      # or: ngrok http 8000
```
Copy the printed `https://....trycloudflare.com` URL, then in the repo go to
**Settings > Secrets and variables > Actions** and add secret `RECEIVER_URL` = `https://....trycloudflare.com/report`.
Quick-tunnel URLs change each time you restart the tunnel, so update the secret if you do.
Optional: set `RECEIVER_TOKEN` as a secret and as an environment variable when starting the receiver.

Alternative: register a self-hosted runner on your machine, change `runs-on` to `self-hosted`, and set `RECEIVER_URL=http://localhost:8000/report`.

## 5. Demo workflow
1. Push as-is: all 3 tests pass; `test-report.json` shows `"status": "passed"`.
2. Introduce a defect, e.g. in `app/validators.py` change `is_valid_url` to `return True`. Push: the `invalid_url` test fails, the job goes red, and `test-report.json` shows `"status": "failed"` with the failure message.
3. Revert the defect and push: green again, report updated.

## Bonus: GitHub Pages (conditional deploy)
Settings > Pages > Source: **GitHub Actions**. The `deploy` job runs only on `main` and only when `test` succeeds, so a failing run leaves the published demo untouched. URL: `https://<user>.github.io/<repo>/`.

## Bonus: email report
Add secrets `SMTP_SERVER`, `SMTP_USERNAME`, `SMTP_PASSWORD` (an app password), and the repository **variable** `REPORT_EMAIL_TO`. The `email` job runs after passing and failing runs, with the summary in the body and `test-report.json` attached. No credentials are stored in the code.
