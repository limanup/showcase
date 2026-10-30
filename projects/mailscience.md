# MailScience

**An email deliverability checker, built in a day to test a product hypothesis.**

`Python` `TypeScript` `DNS`

---

## What it does

Given a domain, it checks the things that decide whether mail lands in an inbox:

- MX records — is there a mail server, and who runs it
- SPF — is the sender policy present, and how strict
- DMARC — is there a policy, and does it enforce
- DKIM — is a signing key published
- Reverse DNS — does the sending IP resolve back
- WHOIS — registration and ownership context
- Blacklists — 19 major DNS blocklists, with rate limiting so the check does not become the
  abuse

It prints a readable report with per-check timing and saves it to a file.

## Why it exists

The question was whether a small deliverability audit could be a product. The fastest way to
find out was to build the audit. It is implemented twice — Python and TypeScript — partly to
compare the DNS libraries and partly because the two languages were the two candidate
stacks for whatever came next.

## Notes

- The blacklist checks dominate runtime, so they run with rate limiting and report their own
  timing rather than being hidden behind a single total.
- The TypeScript version exists to check that the result was not an artifact of one
  library's DNS behaviour.
- It is a prototype, not a service: no persistence, no scheduling, no UI.

Repo: private for now. It holds domain health reports for real client domains, so it goes
public only after those are purged from its history.
