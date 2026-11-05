# HTTP security headers, explained honestly

Security headers are the cheapest findings in any web assessment.
They take minutes to add, tools check them automatically, and yet
most sites are missing at least one that matters. This post separates
the headers that genuinely reduce risk from the ones people add for
scanner scores.

## The ones that matter

**Content-Security-Policy (CSP)** is the only header on this list that
can stop an attack class on its own. A strict CSP limits where scripts
can load from, which turns most cross-site scripting bugs from "full
session theft" into "annoying broken page for the attacker too". The
honest catch: a CSP full of `'unsafe-inline'` is mostly decoration.
The value is in `default-src 'self'` with an explicit `script-src`,
and `frame-ancestors` for clickjacking control.

**Strict-Transport-Security (HSTS)** tells browsers to only ever use
HTTPS for your domain, for the `max-age` you set. Without it, the
first visit and every expired-cache visit can be downgraded. Set
`max-age` to at least a year (31536000). An HSTS header served over
plain HTTP is ignored by design, so it only counts on the HTTPS site.

**X-Content-Type-Options: nosniff** stops browsers from guessing a
file's type against what the server declared. This one matters most
on sites that host user uploads: without nosniff, an uploaded text
file can be sniffed and executed as script in your origin.

## The ones that help, quietly

**Referrer-Policy** controls how much of your URLs leak to third
parties through the Referer header. URLs carry tokens, password reset
links and search terms more often than anyone admits.
`strict-origin-when-cross-origin` is a sane default.

**Permissions-Policy** switches off browser features your site does
not use: camera, microphone, geolocation, payment. Every feature you
disable is a feature an injected script cannot abuse either.

**Clickjacking protection** used to mean `X-Frame-Options`. Today,
CSP `frame-ancestors 'none'` (or an allowlist) is the modern form.
You need one of them; having both is belt and braces, not a problem.

## The ones that are mostly hygiene

A `Server` header shouting `Apache/2.4.49` does not create the
vulnerability, but it hands attackers a version to search exploits
for. Same for `X-Powered-By`. Remove or genericise them, and spend
the saved effort on patching, which is what actually protects you.

## Check your own site

I wrote a zero-dependency checker that grades all of the above and
explains each miss in one line:

```bash
git clone https://github.com/RahilSahu/http-security-headers-checker
python3 headers_check.py https://your-site.example
```

## How I write this finding in a report

Severity: usually Low or Informational, raised when a specific
weakness combines with something real, for example no CSP on a page
with a known reflected XSS, or user uploads served without nosniff.
Evidence: the raw response headers, and the missing header named one
by one. Remediation: the exact header values to add, not a link to a
generic hardening guide. A finding the developer can fix in five
minutes is a finding that actually gets fixed.
