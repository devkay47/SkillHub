---
name: email-deliverability
description: A Claude Code skill that builds and checks SPF, DKIM, and DMARC records using live DNS lookups. Use when the user asks to set up, fix, or check email authentication for a domain, build SPF or DMARC records, choose a DMARC policy, or work out why mail goes to spam or bounces.
---

# Email Deliverability

Build and check SPF, DKIM, and DMARC records for a domain, using live DNS lookups. The skill covers the records themselves, the lookup budget, alignment, and the safe rollout from monitoring to enforcement.

## When to Use

- The user asks to set up email authentication for a domain, or to fix mail that goes to spam, bounces, or fails DMARC.
- The user asks to build an SPF record, a DMARC record, or both, for a domain that sends through one or more providers.
- The user pastes an SPF or DMARC record and asks whether it is correct, or why a provider reports a failure.
- The user asks to move a DMARC policy from none to quarantine or reject.
- The user asks which providers a domain sends through and what each one costs against the SPF lookup limit.

## Hard Rules

These rules are not suggestions. Every record the skill produces must satisfy all of them.

1. **Always verify with live DNS lookups, never trust pasted records.** A record the user pastes may be old, edited, or from a different domain. Before judging a record, fetch the current TXT records for the domain, for `_dmarc.<domain>`, and for the DKIM selector, with a live DNS lookup. If a lookup fails, say so and stop; do not guess from the pasted text.
2. **SPF has a 10-lookup limit.** Each `include`, `a`, `mx`, `ptr`, and `exists` mechanism costs one lookup, and so does a `redirect`. Count every one, including the lookups inside included records. A record that causes more than 10 lookups fails outright at receivers, so a record over the limit is broken, not nearly right.
3. **DMARC alignment is part of the record, not an afterthought.** SPF alignment and DKIM alignment decide whether a passing check counts toward DMARC. A record that passes SPF but fails alignment still fails DMARC.
4. **Roll out DMARC none, then quarantine, then reject. Never jump straight to reject.** Run each stage long enough to read the aggregate reports and fix what fails, then move on. Jumping to reject on day one bounces legitimate mail.
5. **Never use `+all`.** It authorizes the whole internet to send as the domain, so the record is worthless.
6. **Never emit the retired DMARC tags `pct`, `rf`, or `ri`.** They were removed in RFC 9989. A record that still carries them is out of date.

## Provider Includes and Their Lookup Costs

These costs were measured against live DNS. They are estimates, not promises; a provider can change its own record, and the checker in the premium pack is the authority after a record is published. The cost is the number of lookups the include itself causes, counting its own nested includes.

| Provider | Include | Lookups |
| --- | --- | --- |
| Google Workspace | `include:_spf.google.com` | 1 |
| Microsoft 365 | `include:spf.protection.outlook.com` | 1 |
| SendGrid | `include:sendgrid.net` | 2 |
| Mailchimp | `include:servers.mcsv.net` | 1 |
| Mailgun | `include:mailgun.org` | 5 |
| Amazon SES | `include:amazonses.com` | 1 |
| Postmark | `include:spf.mtasv.net` | 1 |
| Zoho Mail | `include:zohomail.com` | 2 |
| HubSpot | `include:_spf.hubspotemail.net` | 1 |
| Shopify | `include:shops.shopify.com` | 1 |
| Brevo | `include:spf.brevo.com` | 1 |

Notes:

- Mailgun is the expensive one at 5 lookups. A domain that also uses two or three other providers can run out of budget fast with Mailgun in the list.
- Shopify and Brevo usually need a sending subdomain for DMARC alignment, because their default return-path uses their own domain. If the domain uses either, check the subdomain setup with the provider before publishing.
- The full table, with what each include covers and the alignment note for each provider, is in `references/provider-includes.md`.

## How to Build an SPF Record

Start from the domain's real sending setup, not from a template.

1. List every service that sends mail as the domain: the mailbox provider, the marketing tool, the transactional sender, the shop, the helpdesk. One missing sender is a common cause of mail failing SPF.
2. For each sender, take its include from the table above, or from the provider's own documentation when it is not in the table.
3. Add the mechanisms for the domain's own mail servers when they send directly: `mx` when the domain's MX hosts send mail, `a` when the domain's own web or mail host sends mail.
4. Count the lookups. Every `include`, `a`, `mx`, `ptr`, and `exists` costs one, and a `redirect` costs one. Then add the nested cost of each include from the table. The total must stay at or under 10. At 8 or 9, warn the user that one more provider may push it over.
5. End the record with `~all` (softfail) for a new record, and `-all` (hard fail) only once the record has run clean for a while and the sender list is certain. Never end with `+all`, and never leave the record with no `all` mechanism at all.
6. If the domain already has an SPF record, merge into it. Never publish a second SPF record; two records is itself a permanent error at receivers. Keep the existing mechanisms that are still in use, add the new includes, recount, and publish the single merged record.
7. Publish the record as a TXT record at the domain root, then verify it with a live lookup. DNS changes take time to spread, so re-check after the TTL has passed.

Example record, Google Workspace plus SendGrid:

```
v=spf1 include:_spf.google.com include:sendgrid.net ~all
```

That costs 3 lookups (1 for Google, 2 for SendGrid) and leaves 7 in budget.

## How to Build a DMARC Record

A DMARC record is a TXT record at `_dmarc.<domain>`, for example `_dmarc.example.com`.

```
v=DMARC1; p=none; rua=mailto:dmarc@example.com;
```

Tags the skill uses:

- `p`: the policy for mail that fails DMARC. `none` (monitor only), `quarantine` (send it to spam), or `reject` (refuse it). Start at `none`.
- `rua`: where aggregate reports go, as `mailto:` addresses. Always set it; without it, there is no way to see what fails. Separate several addresses with commas.
- `ruf`: where forensic (per-message failure) reports go. Optional, and most large providers no longer send them, so do not depend on it.
- `sp`: the policy for subdomains. Set it when subdomains send mail under a different policy from the main domain.
- `adkim` and `aspf`: alignment mode, `r` (relaxed, the default) or `s` (strict). Relaxed alignment allows a subdomain match; strict requires an exact domain match. Only set strict when the sending setup genuinely uses the exact domain everywhere.
- `pct`, `rf`, `ri`: never emit these. They are retired.

DMARC only works when either SPF or DKIM passes and aligns. A domain with a DMARC record but no SPF and no DKIM record gets nothing from it.

## DMARC Rollout: None, Then Quarantine, Then Reject

1. Publish with `p=none` and a `rua` address. Watch the aggregate reports for at least two weeks. Every legitimate sender must show up passing, and anything failing must be identified: a forgotten sender, a forwarder, a provider that needs its own subdomain.
2. Fix every legitimate failure first. Add the missing sender to SPF, fix the DKIM selector, or set up the provider's sending subdomain. Do not move on while real mail still fails.
3. Move to `p=quarantine`. Watch the reports for another two weeks. Check that nothing legitimate lands in spam, and that spoofed mail is being caught.
4. Move to `p=reject`. Keep the `rua` address in place and keep reading the reports. A policy of reject with no monitoring is how a later provider change silently starts bouncing mail.
5. If a new sender is added later, drop back to `p=none` while it is tested, then walk the stages again. Enforcement is a state to maintain, not a switch to set once.

The premium pack has a full rollout checklist (`dmarc-rollout-checklist.md`) with the monitoring steps and what to check at each stage.

## Bulk Sender Rules (Gmail and Yahoo, 2024)

Gmail and Yahoo apply extra rules to bulk senders, meaning a domain that sends 5,000 or more messages a day to Gmail addresses (Yahoo applies the same shape of rules to its own mail). The rules are:

- **Authenticate everything.** SPF, DKIM, and DMARC must all be set up for the sending domain. DMARC at `p=none` satisfies the setup requirement, but move to enforcement once the reports are clean; monitoring forever is not the intent.
- **One-click unsubscribe.** Bulk mail must carry a `List-Unsubscribe` header with a one-click unsubscribe option (RFC 8058), and unsubscribe requests must be honored within two days. A plain "reply to unsubscribe" or a link hidden in the footer does not satisfy this.
- **Keep the spam complaint rate under 0.3%.** Gmail measures this in Google Postmaster Tools. Stay under 0.3%, and aim under 0.1%. A domain that drifts above 0.3% starts seeing bulk mail throttled or sent to spam, even when its records are correct.

**Honest limit: this skill and the checker script cannot verify the last two rules from DNS.** A DNS lookup cannot see message headers, so it cannot confirm the one-click unsubscribe header is present on real mail. It also cannot see complaint rates; only Google Postmaster Tools shows those, and only for mail sent to Gmail. Check the unsubscribe header from a real sent message (open the raw headers and look for `List-Unsubscribe` with a one-click `List-Unsubscribe-Post` entry), and check the complaint rate in Postmaster Tools. The premium pack has a Google Postmaster Tools guide (`google-postmaster-guide.md`) with the setup steps and what to watch.

## Alignment

DMARC passes when either SPF or DKIM passes **and aligns** with the From domain.

- **SPF alignment** compares the domain in the return-path (the bounce address) with the From domain. Relaxed alignment allows a subdomain match, for example return-path `bounce.mail.example.com` aligns with From `example.com`. Strict alignment needs an exact match. When a provider sends with its own return-path domain, SPF does not align at all, and only DKIM can carry DMARC for that mail. That is why providers like Shopify and Brevo need a sending subdomain: it makes the return-path a subdomain of the customer's own domain.
- **DKIM alignment** compares the `d=` value in the signature with the From domain. Relaxed allows a subdomain, strict needs an exact match. DKIM alignment can only be verified from a signed message, not from DNS; a DNS lookup can confirm the key record exists and is well formed, but it cannot prove the signature will align. The checker in the premium pack says this plainly when it runs.
- **Both failing** means DMARC fails, even if one of SPF or DKIM passed without aligning. Check alignment before concluding a record is fine.

## Worked Example

Domain `example.com` sends through Google Workspace for mailboxes and SendGrid for transactional mail. The owner wants reports at `dmarc@example.com`.

Step 1, SPF. Google Workspace costs 1 lookup, SendGrid costs 2. Total 3, well under the limit.

```
v=spf1 include:_spf.google.com include:sendgrid.net ~all
```

Step 2, DMARC at `_dmarc.example.com`, starting at `none` with reports.

```
v=DMARC1; p=none; rua=mailto:dmarc@example.com;
```

Step 3, verify with live lookups. Fetch the TXT records for `example.com` and for `_dmarc.example.com`, count the SPF lookups from the live records, and confirm the DMARC record is the one just published. After two clean weeks of reports, move `p=none` to `p=quarantine`, then after two more clean weeks to `p=reject`. The full records for this example are in `examples/example-com-records.txt`.

## The Free Web Tool

A free SPF and DMARC generator web tool already exists at https://websmartools.com/spf-dmarc-generator/. It builds both records in the browser, with a live lookup counter and a merge parser for an existing SPF record. This skill complements it: the tool builds the records, and the skill teaches Claude Code to build, check, and roll them out, including the live verification and the enforcement stages the tool does not cover. Do not duplicate the tool's copy here; point the user at it when they want to build the records by hand.

## The Checker Script (Premium)

The premium pack includes `check.py`, a script that fetches the live SPF, DKIM, and DMARC records for a domain and reports what is wrong. It checks the SPF lookup total (including nested includes), flags `+all`, over-limit records, and missing records, checks the DMARC policy and retired tags, validates the `rua` and `ruf` report addresses as `mailto:` URIs, and checks that a DKIM key record exists for the selector, that its key is 2048-bit or stronger, and that its `d=` tag matches the domain. When a record was published moments ago, pass `--retries <n>` with `--wait <seconds>` and it re-checks DNS up to that many times, waiting that long between tries, instead of false-failing on a record that has not spread yet. Run it before publishing a record and again after the DNS change has spread. It exits 1 when the domain has errors. See the premium README for the exact command.

## Troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| Receivers report "SPF permanent error" | More than 10 lookups, or more than one SPF record | Count the lookups including nested includes and remove a provider or move one to a subdomain; merge the records into one |
| Mail passes SPF but still fails DMARC | Alignment fails | Check the return-path domain for SPF alignment and the DKIM `d=` value for DKIM alignment; set up the provider's sending subdomain |
| DMARC reports show a sender failing | A legitimate sender is missing from SPF or DKIM | Add the sender's include, or fix its DKIM selector, then wait for reports to confirm |
| Everything fails after a provider change | The new provider was added without updating SPF or DKIM | Add the new include and selector, drop DMARC back to `p=none` while testing, and walk the rollout again |
| A lookup returns no records at all | The record was never published, or the name is wrong | Check the record exists at the right name (`_dmarc.` for DMARC, `<selector>._domainkey.` for DKIM) and that DNS has spread |
| The checker reports a missing DKIM record | The provider uses a different selector | Ask the provider for its selector name, or check its documentation, and pass it with `--selector` |
| The checker warns about a 1024-bit DKIM key | The provider generated a short key | Ask the provider to rotate to a 2048-bit key; 1024-bit keys are being phased out by receivers |
| A record was just published but the checker says it is missing | DNS has not spread to the resolver yet | Re-run with `--retries 3 --wait 30`; the checker re-reads DNS and says when the record appears |
| Gmail or Yahoo throttle bulk mail even though the records pass | A bulk sender rule is failing | Check the one-click unsubscribe header in a real sent message and the spam complaint rate in Google Postmaster Tools; DNS cannot verify either one |

## What This Skill Does Not Do

- It does not send mail, and it does not read anyone's mailbox. It only reads public DNS records.
- It does not verify DKIM alignment from DNS alone; that needs a signed message.
- It does not set up a provider account or publish DNS records. The user (or their DNS provider) publishes the records the skill writes.
- It does not claim a record is safe to enforce until the reports say so. The rollout stages exist because guessing wrong bounces real mail.
- It does not cover BIMI. BIMI needs a Verified Mark Certificate from a certificate authority, which costs far more than this skill; a logo SVG template alone does not enable BIMI, and a buyer who expects a working BIMI record from a template file would be misled. If a domain wants BIMI, it needs the certificate first, and the BIMI record comes after that, not before.
