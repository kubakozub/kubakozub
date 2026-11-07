### Jakub Kozub

SOC L2 analyst at Jagiellonian University, Krakow. I find the access-control bugs that scanners
routinely miss, then I write the tested detections that catch them.

Authorization and business-logic research on open-source software a lot of people quietly rely
on: WordPress, Elastic, GitLab, Matomo. Every proof of concept runs in a local lab before
anything is filed.

#### Two Critical findings in WordPress core and the Gutenberg plugin

One audit of freshly added media-handling code during the 7.1 beta. Both fixed before 7.1
shipped, both paid, neither a duplicate.

| Report | Severity | What it was |
|---|---|---|
| [#3931777](https://hackerone.com/reports/3931777) | **CVSS 9.9**, CWE-22 | The lowest role allowed to upload files could delete `wp-config.php` through the media finalize route. A WordPress without `wp-config.php` hands its installer to anyone. |
| [#3931771](https://hackerone.com/reports/3931771) | **CVSS 9.0**, CWE-79 | The same controllable filename ran as script in any administrator's session on merely opening the media library. |

Fixed in Gutenberg 23.8 and core r63200. The write-up, with the exploit replayed:
[jakubkozub.com/research/wordpress-media-finalize](https://jakubkozub.com/research/wordpress-media-finalize/)

---

### Find the bug, then ship the detection that catches it

Most researchers do the first half of this. Most blue teams do the second. Both halves of the
same finding are public here.

**[detections](https://github.com/kubakozub/detections)** carries the Sigma rules for what that
9.9 leaves in the logs, with the log lines that must fire them and the ones that must not. Every
request in the chain is authenticated, authorised and successful, so there is no request
signature to match: the rules watch a sequence, an outcome and a state change instead. Each rule
is rendered for Splunk, Elastic ES|QL and Kusto, and every rendering was run against the same
fixtures on the real engines (Elasticsearch 8.19.4, Splunk 9.4.3 and the Kusto emulator).

---

### Hire me

Contract work, taken alongside my job and independently of my employer. Three things:

1. **Access-control review** of one application, plugin, internal tool or REST API. You get the
   entry points a request can reach, ranked by who can reach them, and a report per confirmed
   finding.
2. **Vulnerability research** on one open-source component you depend on, audited the way the 9.9
   was found: recent changes first, one attacker-controlled value followed across subsystems.
   Coordinated disclosure to the maintainer where the code is not yours.
3. **Detection engineering**: a finding, an incident or a threat model turned into rules that run
   on your own log sources, each with the log sources it needs and tests against traffic you
   already have.

Scope, deliverables and how an engagement runs: [jakubkozub.com/#services](https://jakubkozub.com/#services)
Email: contact@jakubkozub.com

---

### Tooling from my own engagements

Python, standard library only, offline by design, MIT. No telemetry, nothing leaves the machine.

**Find the reachable code**

- **[wp-authz-audit](https://github.com/kubakozub/wp-authz-audit)**: maps the WordPress plugin
  code an attacker can reach and the guard on each path, catching a nonce standing in for
  authorization and a capability check standing in for an ownership check. Its `diff` mode
  compares two releases, so a version bump touching hundreds of lines reports only what became
  reachable or lost a guard.
- **[wp-hook-guard](https://github.com/kubakozub/wp-hook-guard)**: maps every server-side entry
  point a plugin exposes (`wp_ajax_`, `wp_ajax_nopriv_`, `admin_post_`, `register_rest_route`,
  `init` handlers that read the request) and flags those with no capability check, nonce or
  permission callback behind them. A hand-written PHP lexer keeps hooks in comments and strings
  out of the results.

**Prove the gap**

- **[authz-differ](https://github.com/kubakozub/authz-differ)**: replays one request under
  several identities, the owner, another user, nobody, then diffs the responses. Same 200 and
  same body for the wrong identity is the IDOR. One request per identity, never a scanner, every
  credential redacted to a non-reversible fingerprint.

**Get the evidence out safely**

- **[har-sanitizer](https://github.com/kubakozub/har-sanitizer)**: redacts cookies, bearer and
  JWT tokens, API keys and PII from a HAR, a raw request or a curl command while keeping the
  request reproducible. `--strict` re-scans its own output and fails closed on any high-entropy
  token the structured pass did not reach.
- **[evidence-redaction-gate](https://github.com/kubakozub/evidence-redaction-gate)**: a
  fail-closed gate that refuses to let secrets or PII leave inside evidence. Built after I
  measured that two detectors in my own workflow had silently drifted apart, 9 of 31 and 13 of 31
  on realistic secret shapes. One shared module and an adversarial corpus took it to 31 of 31
  with no false alarms.
- **[redact-request](https://github.com/kubakozub/redact-request)**: prints the shape of a
  captured request and never its contents, so a request can be discussed without handing over
  the session. Its tests are written as leak tests: they assert on what must be absent.

**Scope it and write it up**

- **[scope-sentinel](https://github.com/kubakozub/scope-sentinel)**: answers whether a target is
  actually in scope before any request goes out, where out of scope always beats in scope. Its
  exit codes make it a gate in front of a script.
- **[finding-forge](https://github.com/kubakozub/finding-forge)**: turns a confirmed finding into
  a report a triager can act on, with a CVSS 3.1 scorer that matches the FIRST calculator to the
  decimal and a lint that fails the draft on an empty section or a leaked secret.

---

### Credentials

eWPTX, eCTHP, CDSA, BTL2. Approved Wordfence researcher. 4x CTF podium (2x first), Vision One
European CTF finalist. MSc Information Technology (AHE Lodz). Postgraduate cybersecurity and MA
English Philology (University of Warsaw).

- Portfolio: [jakubkozub.com](https://jakubkozub.com)
- HackerOne: [hackerone.com/jakubk](https://hackerone.com/jakubk)

---

### How I work

Authorized testing only, inside an agreed scope. Vendor first, coordinated disclosure. Nothing is
filed until it is rebuilt in a lab, and I report only what I measured, including where the impact
stops.
