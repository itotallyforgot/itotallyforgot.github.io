# Acceptance checklist — security-harden-public-site

- [✓] Analytics loading is transport-explicit and integrity checked — GoatCounter uses HTTPS, SHA-384 SRI, and anonymous CORS.
- [✓] Pages receive restrictive browser defaults — CSP and strict-origin referrer metadata are included in both layouts.
- [✓] Shared security automation is immutable — the caller uses the central merge commit SHA.
- [✓] The recorded SRI digest matches the live 9,213-byte script.
- [⚠] Local Jekyll rendering — Docker Desktop was unavailable; the pinned Pages CI job is the verification authority.
- [⚠] Foreign-model adversarial review — skipped by explicit user direction.
