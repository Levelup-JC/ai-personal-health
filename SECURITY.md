# Security

## Reporting

Please report suspected vulnerabilities or secret exposure to `<SECURITY_CONTACT>` (Owner-defined).

## Policy

- No tokens, Bearer strings, tunnel URLs, or real health exports in git.
- Rotate credentials immediately if exposure is suspected.
- Enable GitHub Secret Scanning and Push Protection on `Levelup-JC/ai-personal-health`.
- Prefer `gitleaks` (or equivalent) in CI before merge.
