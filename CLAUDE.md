# Tanweer redaction pipeline
OCR scanned Arabic archive pages, detect personal data, redact, audit.

## Commands
- Tests: `make test` (runs pytest inside Docker; never on the host)
- Lint and types: `make lint` (ruff, mypy)
- Secret scan before commit: `make secrets` (gitleaks)

## Never read, print or send
- `.env`, keys, tokens, `*.pem`.
- Real client documents. Only `data/synthetic/` exists for us.
- Never send page images or text to any hosted API unless the task says
  so; Tanweer's policy forbids personal data leaving its control.
- Never log extracted text or ID numbers. Log document ID and page number only.

## Conventions that differ from defaults
- Text is Arabic (RTL). Keep Unicode as is. Never strip diacritics or
  convert Arabic-Indic digits (٠١٢) silently; normalise in one place:
  `src/text/normalize.py`.
- National IDs are 14-digit strings, never integers (leading zeros).
- ID validation follows the fictional rule in `docs/id-rule.md`.
  Never assume it applies to real IDs.
- Redaction must remove the text from the output file, not just draw a
  black box over it. Test by extracting text from the redacted PDF.
- Bounding boxes use pixel coordinates of the original scan: `[x0, y0, x1, y1]`.
- Presidio has no Arabic recognisers. Custom ones live in `src/pii/recognisers/`.

## Where things are (read when relevant, not up front)
- Client brief: `docs/brief.md`
- Decisions and buy/build verdict: `docs/adr/`
- Test set and labels: `data/synthetic/labels.csv`
- Audit log schema: `docs/audit-schema.md`

## Known traps
- Handwritten pages have low OCR confidence. Below 0.6, send the page to
  human review; never auto-approve.
- Stamps over text hide characters. Missing text means unknown, not clean.
- Surya model weights have a revenue-limited licence. Check
  `docs/licences.md` before adding any model.

## Roles and audit
- Only the `supervisor` role can undo a redaction.
- Every redact, review, release and original access writes one audit row
  (who, what, when). Audit rows are append-only.

## Before saying a task is done
- `make test` passes and new behaviour has a new test.
- Fixtures use `data/synthetic/` only. No real names, IDs or phone numbers.
- Redacted output re-checked: no PII text remains.

- ## Security
- Threat model: `docs/threat-model.md`. Update it when a new data flow is added.
- Before any PR: `make secrets` and `make lint` pass; no new network call.
- New dependency or model: check `docs/licences.md`; must run on-premise.
