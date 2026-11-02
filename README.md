# Scam Shield releases

This repository delivers signed Scam Shield releases to managed PCs. It holds no source history, no tests and no
secrets. Development happens in a separate private repository.

**Nothing here runs unless it is signed.** The Scam Shield script on each PC contains our public key and refuses
any file that doesn't verify against it. If this repository were altered by someone else, the PCs would ignore
the change and keep running their last verified release.

## Contents

| Path | What it is |
|------|------------|
| `channels/pilot.json` + `.sig` | Which release pilot PCs run. Signed. |
| `channels/stable.json` + `.sig` | Which release all other PCs run. Signed. |
| `keys/release-public.xml` | Our public signing key, the same one built into the PC script |
| Releases tab | `scamshield-<version>.zip` for each release (GitHub Release files, not committed) |

A channel file looks like this:

```json
{
  "schemaVersion": 1,
  "channel": "stable",
  "sequence": 7,
  "version": "2.0.3",
  "zipUrl": "https://github.com/<owner>/<repo>/releases/download/v2.0.3/scamshield-2.0.3.zip",
  "zipSha256": "<64 hex characters>",
  "minBootstrap": 1
}
```

`sequence` only ever goes up. A PC refuses a channel file with a lower sequence than one it has already seen, so
an old signed file can't be replayed. A rollback is published as a new file (a higher sequence) naming the older
version.

## How a PC checks a release

1. The channel file's signature (`.sig`, RSA SHA-256, base64) is checked against the public key.
2. The downloaded zip's SHA-256 must equal `zipSha256`.
3. The zip's `manifest.json` signature (`manifest.json.sig`) is checked against the same key.
4. Every file's SHA-256 must match the manifest. Any extra, missing or changed file means the release is refused.

## Checking a release by hand

In Windows PowerShell 5.1, from a clone of this repository:

```powershell
$rsa = New-Object System.Security.Cryptography.RSACryptoServiceProvider
$rsa.FromXmlString((Get-Content .\keys\release-public.xml -Raw))
$data = [IO.File]::ReadAllBytes('.\channels\stable.json')
$sig  = [Convert]::FromBase64String((Get-Content .\channels\stable.json.sig -Raw).Trim())
$rsa.VerifyData($data, 'SHA256', $sig)   # True = signed by us
```

## Publishing

The owner publishes releases from the private repository. Nothing in this repository is edited by hand.
