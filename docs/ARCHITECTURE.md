# Architecture

WatchTower runs as two processes: a React single-page app and a FastAPI server, talking over HTTP. The backend keeps incidents, users and uploaded evidence in local SQLite databases and files.

## Frontend

- Vite, React 18, TypeScript, Tailwind CSS and shadcn/ui components.
- React Router for pages, with route guards by role (reporter or admin).
- i18next for English and Hindi strings.

| Page | What it shows |
| --- | --- |
| Landing | What the portal is for, and how to report |
| Report incident | The submission form: URL, message or file, notes, location, voice input |
| Submitted report | A confirmation for the reporter |
| Dashboard, analysis, incident details | The admin views (sample data in the demo build) |
| Trends | Charts over time (sample data in the demo build) |
| Zero trust | Device, session and anomaly panels |

## Incident pipeline

A submission goes through these steps on the server:

1. **Authenticate** the reporter.
2. **Scan an attached file**, if there is one: file type, known byte patterns and an optional VirusTotal lookup. The file is kept in an evidence store.
3. **Check URLs** with a DNS lookup, a trusted-domain list and suspicious-pattern rules.
4. **Analyse the content** with Google Gemini, using a prompt extended with Army-specific context. The model returns a risk score, severity, threat type, indicators and recommendations. If Gemini is not configured or fails, keyword rules produce the same fields.
5. **Merge the file-scan results** into the risk score and indicators.
6. **Map the location** from free text to an Army command region.
7. **Look for fake profiles.** For messages and social posts, check for fake-officer and honeytrap patterns, which can raise the severity.
8. **Match repeats** against earlier incidents with similar content and record how often the threat has been seen.
9. **Store** the incident and write a JSON copy of the report.
10. **Escalate** automatically when the risk score, repeat count and military relevance cross set thresholds, and add timeline events.
11. **Respond by role.** Reporters get a confirmation, and admins get the full analysis.

## Storage

| Store | Holds |
| --- | --- |
| Incident database | Incidents, timelines and per-region statistics |
| User database | Accounts and sessions |
| Zero-trust database | Devices, sessions and anomalies |
| Evidence store | Uploaded files and their metadata |
| Report files | One JSON report per incident |

## External services

- Google Gemini (`gemini-2.5-flash`), optional. The rule-based fallback covers the case where it is not set up.
- VirusTotal, optional and off by default.

## Not finished

Some modules were written but not wired into the request path before the finale: an AES encryption helper, a hash-chained audit log, an NLP module and a zero-trust middleware layer. A second, Supabase-based backend exists in the codebase but is not used.
