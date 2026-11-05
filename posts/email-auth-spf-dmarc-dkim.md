# SPF, DMARC and DKIM: why spoofing your domain is probably trivial

Try this in the next phishing-awareness session you run: ask who in
the room could send an email that arrives as `ceo@yourcompany.com`.
If your domain lacks enforced email authentication, the honest
answer is "anyone with a laptop", and the three DNS records in this
post are the entire reason why.

## SPF: who may send for you

A Sender Policy Framework record is a DNS TXT record that lists the
servers allowed to send mail for your domain:

```text
v=spf1 include:_spf.google.com -all
```

The ending is the whole story. `-all` means "reject everything not
listed". `~all` means softfail, which most receivers treat as a
suggestion. `?all` documents your senders and stops nobody. And
`+all`, which I still find in the wild, formally authorises the
entire internet to send as you.

SPF alone has a structural weakness: it checks the envelope sender,
which the reader never sees, not the `From:` header they do see.
That gap is what DMARC exists to close.

## DMARC: what to do when checks fail

A DMARC record lives at `_dmarc.yourdomain.com`:

```text
v=DMARC1; p=reject; rua=mailto:dmarc@yourdomain.com
```

It requires that the visible `From:` domain aligns with SPF or DKIM,
and its `p=` policy tells receivers what to do with mail that fails:
`none`, `quarantine`, or `reject`.

Here is the finding I write most often: **p=none is monitor mode**.
It generates reports and stops nothing. Organisations publish it as
step one of a DMARC project, then the project ends, and the domain
spends years believing it is protected. If your policy is not
`quarantine` or `reject`, spoofed mail from your domain is being
delivered, and your own reports (`rua`) contain the evidence.

## DKIM: the signature

DKIM signs outgoing mail with a private key and publishes the public
key in DNS at `selector._domainkey.yourdomain.com`. A valid signature
proves the mail really came from a system holding your key and was
not altered on the way. It is the only one of the three that survives
forwarding intact, which is why DMARC accepts alignment from either
SPF or DKIM.

## Check any domain in one command

```bash
git clone https://github.com/RahilSahu/dns-email-auth-checker
python3 dns_auth_check.py yourcompany.com
```

It pulls the MX, SPF, DMARC, DKIM (common selectors) and CAA records
over DNS-over-HTTPS and translates each one into plain language,
including the two traps above: softfail SPF and monitor-only DMARC.

## The rollout that actually works

1. Publish SPF correctly for every legitimate sender, including your
   CRM, helpdesk and marketing tools, or you will break real mail
2. Publish DMARC at p=none with reporting, and read the reports for
   a few weeks to find the senders you forgot
3. Move to p=quarantine, watch the reports again
4. Move to p=reject. This step is the entire point

Skipping steps 2 and 3 breaks business mail. Stopping at step 2
protects nothing. Most domains I check are stopped at step 2.
