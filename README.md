# Spain Software Jobs — n8n + Ollama

A local, review-only job discovery workflow: Greenhouse and Lever feeds → Spain/software-role filters → Ollama assessment → 75% threshold → sponsorship priority → Google Sheets duplicate check → review queue and Gmail digest. It never submits applications.

## Included configuration

- Daily `0 8 * * *`, explicitly using `Europe/Madrid` (daylight saving adjusts automatically).
- Nine Greenhouse boards: Auctane, Aircall, Cabify, Ebury, Celonis, Sofico, Scopely, New Relic, Socialpoint.
- Lever: Rover (global API), Prima (EU API).
- Local Ollama `qwen3:4b`, up to **six vacancies per run**, preserving the current limit.
- Scores 75–100 only; `explicit_support` first, then `possible_unknown`; restricted or unclassified cases excluded.
- Existing IDs read from Sheets; existing rows skipped; within-run duplicates removed.

This is a sanitized export of an existing 19-node workflow, not an independently verified fresh installation. Credential bindings, personal candidate details, email, spreadsheet references, execution data and instance metadata have been removed. The exported copy is inactive. Two publication adjustments set the Madrid timezone explicitly and build the digest from `Filter new jobs` rather than the earlier pre-duplicate shortlist. The local running workflow has not been changed.

## Setup

1. Keep your existing Docker n8n installation and persistent volume. Do not replace or recreate its volume. For a fresh installation, choose a tested n8n version and use the optional Compose file below.
2. Install Ollama and run `ollama pull qwen3:4b` on the host. The workflow calls `http://host.docker.internal:11434/api/chat`, intended for Docker Desktop with Ollama on the host. Verify connectivity from the container; do not expose Ollama publicly. Other host/container layouts require an adjusted URL.
3. Import `spain-software-jobs.sanitized.json` as a **new inactive workflow**.
4. In `OpenAI evaluate match` (the historical name; it calls Ollama), replace `REPLACE_WITH_YOUR_LOCAL_CANDIDATE_PROFILE` with your own truthful skills and authorization requirements. Keep the personalized copy private. Adjust the model and timeout if needed.
5. Create a Google spreadsheet tab called `Review Queue` with these headers:

   `id,date,company,title,location,score,visa_status,summary,cv_focus,cover_letter_draft,evidence,url,source,status`

6. Configure Google OAuth inside n8n: enable Google Sheets and Gmail APIs in your Google Cloud project, use the redirect URL displayed by n8n, and authorize the intended account. Store client secrets and tokens only in n8n credentials. Select a Sheets credential in both Sheets nodes and a Gmail credential in the email node. OAuth apps in Testing may require reauthorization.
7. Replace `REPLACE_WITH_SPREADSHEET_ID` in both Sheets nodes and select the intended tab. `Check existing applications` must read all rows, execute once, always output data on an empty sheet, and stop on errors. Preserve the append/update mapping keyed by `id`.
8. Replace `REPLACE_WITH_YOUR_EMAIL` in the Gmail node. Review all node settings after import.
9. Confirm workflow timezone `Europe/Madrid`, schedule `0 8 * * *`, and the six-item limit. Test manually before publishing/activating it in n8n.

`.env.example` configures only the optional Docker deployment. Workflow email, spreadsheet and candidate placeholders are edited in n8n; copying `.env.example` does not configure those nodes.

### Optional fresh Docker instance

Copy `.env.example` to `.env`, choose a tested image tag, and generate a strong unique encryption key locally. Run `docker compose up -d`, then visit `http://localhost:5678`. The named volume persists n8n data. Keep the encryption key stable and private. Do not use this Compose project to replace your existing installation without a backup and migration plan.

## Validate before activation

- Confirm every source branch completes, and a qualifying assessment retains the correct vacancy ID and URL.
- Save qualifying jobs, then repeat a run: existing IDs must not be appended or have manually edited statuses overwritten.
- Confirm email and Sheets use the same score and only newly saved jobs appear in the digest.
- Test an empty sheet, zero qualifying jobs and a failing Sheets read. A lookup error must stop writes.
- Keep executions sequential. Sheets append/update is not transactional; overlapping runs can race or overwrite statuses.
- Keep the host awake, Docker running, and Ollama available at 08:00 Madrid. Browser and ChatGPT need not remain open.

## Known limits

The six-item cap can repeatedly evaluate early feed items and miss later vacancies. AI scores and sponsorship classifications require human verification; unknown sponsorship is not a promise. Public boards and APIs can change. The inherited Greenhouse company-name fallback can display `Unknown Company`. No email is sent when there are no new qualifying jobs. The current Ollama prompt does not generate cover-letter drafts, so that field may be blank.

Keep `.env`, credentials, resumes, personalized exports, databases, backups, execution histories and screenshots out of Git. `.gitignore` reduces accidental additions but cannot sanitize arbitrary files. Re-sanitize every subsequent export before publishing it.
