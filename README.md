### Jakub Kozub

SOC L2 analyst moving into application security. Detection and incident response by day;
web and API security research on the side, with a focus on **access control, authentication
and business logic** — the classes that scanners do not find — in software a lot of people
quietly rely on: WordPress, Elastic, GitLab, Matomo.

I read source, run every proof of concept in a local lab before filing, and report only what
I measured.

- Portfolio — https://jakubkozub.com
- HackerOne — https://hackerone.com/jakubk
- Certifications — eCTHP · CDSA · BTL1 · postgraduate offensive security (University of Warsaw)
- Approved Wordfence researcher · 4× CTF podium (2× 1st)

---

### A toolkit that follows the work

Everything below is one workflow — **scope → find → test → redact → report** — built for the
parts of research that decide how fast you reach a real finding, and for keeping evidence safe
on the way out. All of it is Python, standard library only, offline by design: no telemetry,
no cloud, nothing leaves the machine.

#### Scope — confirm you're allowed to touch it

**[scope-sentinel](https://github.com/kubakozub/scope-sentinel)** · Python, stdlib only
Answers one question before any request goes out: is this target actually in scope? Parses a
program's scope — wildcard domains, CIDR ranges, URL-prefix rules — and returns
IN / OUT / NOT-LISTED, where out-of-scope always beats in-scope. Its exit codes make it a gate
you can put in front of a script, so a fat-fingered host never gets tested.

**[ReADQuest](https://github.com/kubakozub/ReADQuest)** · Chrome MV3
Tags and blocks traffic by bug bounty program scope, enforced locally through
declarativeNetRequest. Scope rules stay on disk; the extension makes no requests of its own.

#### Find — narrow tens of thousands of lines to the few that matter

**[wp-authz-audit](https://github.com/kubakozub/wp-authz-audit)** · Python, stdlib only
Finds the WordPress plugin code an attacker can reach and checks whether anything guards
it. A plugin is tens of thousands of lines; the code an unauthenticated request reaches is
a couple of dozen callbacks, so it maps those first and ranks them by the privilege an
attacker actually needs. Two rules do most of the work, and both are places where the usual
tools give the wrong answer: a nonce is CSRF defence, not authorization, and
`current_user_can('edit_posts')` is not an ownership check when the object id comes from
the request. Its `diff` mode compares the entry-point graph between two releases, so a
version bump touching hundreds of lines reports only what became reachable or lost a guard.

**[wp-hook-guard](https://github.com/kubakozub/wp-hook-guard)** · Python, stdlib only
Maps every server-side entry point a WordPress plugin exposes — `wp_ajax_`/`wp_ajax_nopriv_`,
`admin_post_`, `register_rest_route`, and `init` handlers that read the request — and flags
the ones with no `current_user_can`, nonce or `permission_callback` behind them. A hand-written
PHP lexer keeps hooks in comments and strings out of the results, and a `diff` mode compares
two releases so a version bump only reports what newly became reachable or lost a guard.

#### Test — prove the authorization gap

**[authz-differ](https://github.com/kubakozub/authz-differ)** · Python, stdlib only
Takes the request you suspect and replays it under two or more identities — the owner, another
user, nobody — then diffs the responses. When the wrong identity gets the same 200 and the same
body, that is the IDOR; when unauthenticated gets data at all, that is missing authentication.
One request per identity, never a scanner; every cookie and token is redacted to a
non-reversible fingerprint in the output, which drops straight into a report.

#### Redact — never leak a session or someone's data

**[har-sanitizer](https://github.com/kubakozub/har-sanitizer)** · Python, stdlib only
Cleans a HAR, a raw request, or a curl command before it becomes an attachment — cookies,
bearer/JWT tokens, API keys, and PII (emails, Luhn-checked card numbers) become clear
placeholders while the request stays reproducible. `--strict` re-scans the output and fails
closed on any high-entropy token the structured pass did not reach.

**[redact-request](https://github.com/kubakozub/redact-request)** · Python, stdlib only
Prints the *shape* of a captured HTTP request and never its contents, so a request can be
discussed without handing over the session. Method, host, path with identifiers replaced,
parameter names without values, header names with sensitive ones marked. Its test suite is
written as leak tests: they assert on what must be **absent** from the output.

**[evidence-redaction-gate](https://github.com/kubakozub/evidence-redaction-gate)** · Python, stdlib only
A fail-closed gate that refuses to let secrets and PII leave the machine inside evidence.
Built after I measured that two detectors in my own workflow had silently drifted apart —
9/31 and 13/31 on realistic secret shapes. One shared module, an adversarial corpus, and a
test asserting both gates import it: 31/31 with zero false alarms. Handles the shapes a
naive grep misses — pretty-printed HAR entries split across lines, bodies wrapped in
`postData.text`, tokens in the URL fragment, `curl -b`.

#### Report — one clean, reproducible write-up

**[finding-forge](https://github.com/kubakozub/finding-forge)** · Python, stdlib only
Turns a confirmed finding into a report a triager can act on: a per-bug-class scaffold with the
sections they expect, a CVSS 3.1 scorer that matches the FIRST calculator to the decimal, and a
lint that fails the draft when a section is empty or a secret slipped into the write-up.

---

### How I work

Authorized testing only, inside an agreed scope. Responsible disclosure, vendor first.
Findings are worth reporting when they are reproducible and the impact is stated honestly —
including where it stops.
