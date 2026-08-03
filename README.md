### Jakub Kozub

SOC L2 analyst moving into application security. Detection and incident response by day;
web and API security research on the side, with a focus on **access control, authentication
and business logic** — the classes that scanners do not find.

I read source, run every proof of concept in a local lab before filing, and report only what
I measured.

- Portfolio — https://cybersecurity-portfolio.jakubkozub.workers.dev
- HackerOne — https://hackerone.com/jakubk
- Certifications — eCTHP · CDSA · BTL1

---

### Tools I build for my own engagements

I write tooling for the parts of security research that decide how fast you reach a real
finding: narrowing where to look, and keeping evidence safe on the way out. All of it is
offline by design — no telemetry, no cloud, nothing leaves the machine.

**[wp-authz-audit](https://github.com/kubakozub/wp-authz-audit)** · Python, stdlib only
Finds the WordPress plugin code an attacker can reach and checks whether anything guards
it. A plugin is tens of thousands of lines; the code an unauthenticated request reaches is
a couple of dozen callbacks, so it maps those first and ranks them by the privilege an
attacker actually needs. Two rules do most of the work, and both are places where the usual
tools give the wrong answer: a nonce is CSRF defence, not authorization, and
`current_user_can('edit_posts')` is not an ownership check when the object id comes from
the request. Its `diff` mode compares the entry-point graph between two releases, so a
version bump touching hundreds of lines reports only what became reachable or lost a guard.

**[evidence-redaction-gate](https://github.com/kubakozub/evidence-redaction-gate)** ·
Python, stdlib only
A fail-closed gate that refuses to let secrets and PII leave the machine inside evidence.
Built after I measured that two detectors in my own workflow had silently drifted apart —
9/31 and 13/31 on realistic secret shapes. One shared module, an adversarial corpus, and a
test asserting both gates import it: 31/31 with zero false alarms. Handles the shapes a
naive grep misses — pretty-printed HAR entries split across lines, bodies wrapped in
`postData.text`, tokens in the URL fragment, `curl -b`.

**[redact-request](https://github.com/kubakozub/redact-request)** · Python, stdlib only
Prints the *shape* of a captured HTTP request and never its contents, so a request can be
discussed without handing over the session. Method, host, path with identifiers replaced,
parameter names without values, header names with sensitive ones marked. Its test suite is
written as leak tests: they assert on what must be **absent** from the output.

**[ReADQuest](https://github.com/kubakozub/ReADQuest)** · Chrome MV3
Tags and blocks traffic by bug bounty program scope, enforced locally through
declarativeNetRequest. Scope rules stay on disk; the extension makes no requests of its own.

---

### How I work

Authorized testing only, inside an agreed scope. Responsible disclosure, vendor first.
Findings are worth reporting when they are reproducible and the impact is stated honestly —
including where it stops.
