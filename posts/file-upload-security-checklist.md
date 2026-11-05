# File upload security: the testing checklist I actually use

File upload is where web assessments get interesting, because the
feature sits at the junction of everything: file parsing, storage,
permissions, content types, and other users downloading what you put
there. This is the checklist I work through, in order.

## 1. What the handler thinks the file is

- **Double extension**: `invoice.pdf.php`, `photo.php.123test`. Does
  the server, or the web server behind it, ever interpret the middle
  extension?
- **Case and trailing tricks**: `file.PHP`, `file.php.`, `file.php `
  with a trailing space, `file.php::$DATA` on Windows and IIS stacks.
- **Null byte in the name** (`file.php%00.jpg`): dead on modern
  runtimes, still worth one request to prove it.
- **Content-Type header lies**: tell the handler the file is
  `image/png` while the body is a script. Handlers that trust the
  header instead of the bytes are common.
- **Magic byte mismatch**: a real script with a GIF header
  (`GIF89a` followed by code) and a plain text file renamed to `.jpg`.
  Which check wins, extension, declared type, or actual bytes?

## 2. What the file can contain

- **SVG with a script tag**, and HTML uploaded as an "image": if the
  file is later served from the application origin, that is stored
  XSS with an upload form instead of a text box.
- **Zip slip**: an archive whose entries are named
  `../../outside.txt`. Extraction code that joins paths without
  checking is still found in production.
- **XXE in office and SVG files**: docx, xlsx and svg are XML inside.
  A parser with DTDs enabled is an XXE waiting for a file.
- **The anti-malware test**: upload the standard EICAR test file. If
  nothing notices a file that exists purely to be noticed, the upload
  pipeline has no AV scanning at all, and that is a finding by itself.

## 3. Where the file goes and how it comes back

- Is the stored name predictable? Can I guess or enumerate other
  users' files, an IDOR with a download endpoint?
- Is the file served from the application domain, a separate storage
  domain, or as an attachment with
  `Content-Disposition: attachment`? The first one is where upload
  bugs become account takeovers.
- Are uploads stored inside the web root with execute permissions?
  That single configuration decides whether a bypass is a nuisance
  or a shell.

## 4. Size, count and boredom attacks

- A file far above the stated limit, many files in one request, and
  a slow trickle upload. Quotas that exist only in the UI are not
  quotas.

## The test kit

I generate all of these cases from one script, with harmless marker
contents only (scripts that print a fixed test string, never shells),
plus a manifest describing what a secure handler should do with each
file:

- [upload-test-payloads-generator](https://github.com/RahilSahu/upload-test-payloads-generator)
- [pdf-active-content-test-generator](https://github.com/RahilSahu/pdf-active-content-test-generator) for PDF-specific cases: embedded files and script actions
- [rlo-filename-spoofer](https://github.com/RahilSahu/rlo-filename-spoofer) for the filename rendering tricks, including U+202E spoofing

## The fix, in one paragraph

Allowlist extensions and validate the actual bytes, rename stored
files to random names, store outside the web root, serve downloads
from a separate domain with `Content-Disposition: attachment` and
`X-Content-Type-Options: nosniff`, scan with AV, and enforce size and
count limits on the server. Every item on the testing list above maps
to one of those sentences.
