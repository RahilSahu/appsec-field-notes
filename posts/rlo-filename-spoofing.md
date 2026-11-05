# The RLO filename trick: when a PDF is not a PDF

Download a file called `invoice.pdf`, double click it, and a program
runs. No exploit, no vulnerability in the reader, just a filename
that does not look the way it reads. The trick behind it is a single
invisible Unicode character, and it has been used in phishing and
malware campaigns for over a decade because it still works.

## The character: U+202E

Unicode includes control characters for languages that write
right-to-left, like Arabic and Hebrew. U+202E, RIGHT-TO-LEFT
OVERRIDE, tells the renderer: display everything after me reversed.
Operating systems apply it inside filenames too.

An attacker names a file like this, where `<RLO>` is that invisible
character:

```text
invoice<RLO>fdp.exe
```

Everything after the override renders backwards: `fdp.exe` becomes
`exe.pdf`. The screen shows:

```text
invoiceexe.pdf
```

The file manager shows a document. The true name ends in `.exe`.
Windows even helpfully hides the real extension in some views, and
the icon can be swapped for a PDF icon to finish the costume. MITRE
tracks this as ATT&CK T1036.002, Masquerading: Right-to-Left
Override.

## Where it shows up

- Email attachments that survive a gateway which checks extensions
  by reading the displayed name instead of the real one
- Download lures and fake invoice campaigns
- Upload filters: a file uploaded as one type and presented to a
  victim as another, which is why I test it during upload assessments

## Detect it in one command

My tool both generates these names for authorised tests and detects
them in suspicious files:

```bash
git clone https://github.com/RahilSahu/rlo-filename-spoofer
python3 rlo_spoof.py check "invoice<RLO>fdp.exe"

Raw filename:       invoice<RIGHT-TO-LEFT OVERRIDE (U+202E)>fdp.exe
Displayed filename: invoiceexe.pdf
True extension:     .exe
Displayed extension: .pdf
RISK: executable file disguised as a document or image
```

The check mode exits non-zero when any bidirectional control
character is present (there are eleven of them, not just U+202E), so
it can sit in a script that sweeps an uploads folder or a mail
quarantine and flags anything whose name lies about itself.

## Defences that hold up

- Mail gateways and upload handlers should reject, or strip, bidi
  control characters in filenames outright. They have no legitimate
  use in a filename
- Always decide file type from the true final extension and the
  file's actual bytes, never from a rendered name
- Train users on the pattern, not the character: a file whose icon,
  type and name disagree is a file to report, not to open
- Show extensions. The oldest advice in Windows hardening survives
  because this trick depends on hiding the truth in plain sight

## The general lesson

Filenames are user interface, and user interfaces can lie. Any
security control that reads a name the way a human reads it, instead
of the way the filesystem stores it, is one invisible character away
from being bypassed.
