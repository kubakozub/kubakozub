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

I write tooling for the unglamorous half of security research: keeping evidence safe and
staying inside scope. All of it is offline by design — no telemetry, no cloud, nothing
leaves the machine.

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
