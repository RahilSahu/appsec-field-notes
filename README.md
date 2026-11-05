# AppSec Field Notes

Short, practical security notes written from real assessment work.
No theory for its own sake: every post explains what the issue is,
how to test for it, and how to fix it, with commands you can run.

Written by Rahil Sahu, Penetration Tester and Application Security
Engineer. The companion tools live in my other repositories and are
linked from each post.

## Posts

| Post | What it covers |
| --- | --- |
| [HTTP security headers, explained honestly](posts/http-security-headers.md) | Which headers actually matter, which are theatre, and how to check a site in one command |
| [File upload security: the testing checklist I actually use](posts/file-upload-security-checklist.md) | Extension, magic bytes, polyglots, path traversal, and the checks most upload handlers miss |
| [JWT security: how to audit a token by hand](posts/jwt-security-audit.md) | alg none, weak HMAC secrets, expiry and claim mistakes, decoded step by step |
| [SPF, DMARC and DKIM: why spoofing your domain is probably trivial](posts/email-auth-spf-dmarc-dkim.md) | The three records that stop email spoofing, and why p=none means you have not started |
| [The RLO filename trick: when a PDF is not a PDF](posts/rlo-filename-spoofing.md) | Right-to-Left Override filename spoofing, how it renders, and how to detect it |

## Why these exist

Interviewers and clients both ask the same thing in different words:
can this person explain a security issue clearly, to someone who has
to fix it? These notes are my answer, written the way I write findings
in assessment reports.

## Contributing

Corrections are welcome, especially where a post is wrong or out of
date. Open an issue or a pull request with evidence: a spec link, a
tool output, or a test case. Claims without evidence will be asked for
evidence, the same standard I hold my own posts to.

## License

Posts are shared under CC BY 4.0: reuse them with credit. Code samples
in the posts are MIT.
