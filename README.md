# ClearVue Connector Agent

Official distribution point for the **ClearVue Connector Agent** — the on-premises Windows service that connects an ERP system (QAD MFG/PRO) to the ClearVue platform.

**[⬇ Download the latest release](../../releases/latest)**

> **There is no source code in this repository.** It exists to distribute signed, checksummed builds and the documentation needed to verify and install them. The agent is built from a private repository; every release records the exact commit it was built from.

---

## What the agent does

- Runs as a **Windows Service** on a machine that can reach your ERP database.
- Makes **one outbound HTTPS connection** to ClearVue. **No inbound firewall rule and no VPN are required.**
- Ships **self-contained** — the .NET runtime is bundled, so the host needs no .NET installed.

It is enrolled with a one-time token that your ClearVue tenant administrator generates from **Integration Hub → Agents**. Until it is enrolled it does nothing.

---

## Verify before you install

Please do this. It takes a minute, and it is the whole reason this repository is public.

Each release carries exactly two files:

| File | What it is |
|---|---|
| `clearvue-connector-agent-<version>-win-x64.zip` | The package |
| `clearvue-connector-agent-<version>-win-x64.zip.sha256` | Its checksum |

### 1. Check the download against its checksum

```powershell
$expected = (Get-Content .\clearvue-connector-agent-<version>-win-x64.zip.sha256).Split(' ')[0]
$actual   = (Get-FileHash .\clearvue-connector-agent-<version>-win-x64.zip -Algorithm SHA256).Hash.ToLower()
if ($expected -eq $actual) { "OK - checksum matches" } else { "STOP - DOES NOT MATCH" }
```

Or, where coreutils is available:

```bash
sha256sum -c clearvue-connector-agent-<version>-win-x64.zip.sha256
```

**If it does not match, stop and contact SolvePoint.** Do not unzip it.

### 2. Check the signature

Unzip, then:

```powershell
Get-AuthenticodeSignature .\agent\ClearVue.ConnectorAgent.exe | Format-List Status, SignerCertificate, TimeStamperCertificate
```

`Status` should be **`Valid`**, and the signer should be:

```
CN=Solve Point Solutions LLC, O=Solve Point Solutions LLC, L=Akron, S=Ohio, C=US
```

Issued by:

```
CN=Microsoft ID Verified CS AOC CA 03, O=Microsoft Corporation, C=US
```

---

## ⚠️ For allowlisting: use the publisher, never the thumbprint

If you run WDAC, AppLocker, or application-control policy in your endpoint protection, **write the rule against the publisher subject or the issuing CA above.**

**Do not pin the certificate thumbprint.** These are **short-lived certificates — each is valid for 72 hours** and a new one is issued for every signing operation. That is by design, and it is how the signing service works; it is not a sign that anything is wrong. A thumbprint rule will work for one release and silently block the next.

**You will see an expiry date in the past. That is expected and correct.** Every signature is **RFC-3161 timestamped**, which records that the file was signed while the certificate was valid. The signature therefore stays valid indefinitely, and `Get-AuthenticodeSignature` continues to report `Valid` long after the certificate itself has expired. If you are reviewing this against a policy that treats an expired signing certificate as a failure, this is the paragraph to point at.

---

## Install

From an **elevated** PowerShell prompt, in the unzipped folder:

```powershell
cd install
.\Install-ClearVueAgent.ps1 -SourcePath ..\agent -AgentId <GUID> -SaasBaseUrl https://<tenant>.clearvue.com
```

Your ClearVue tenant administrator provides the `AgentId` and the enrollment token. Re-running the installer against a newer package **upgrades in place**.

`README.txt` inside the package has the full walkthrough, including uninstall.

---

## What is in the package

```
agent/                     the service executable (self-contained, signed)
install/                   Install-ClearVueAgent.ps1, Uninstall-ClearVueAgent.ps1,
                           appsettings.template.json
README.txt                 install + upgrade + uninstall walkthrough
release-manifest.json      version, executable SHA-256, and signed true|false
THIRD-PARTY-NOTICES.txt    licenses for every open-source component included
```

**`release-manifest.json` states outright whether the build was signed.** If you ever receive a build where it says `"signed": false`, the release notes will say so prominently too — such a build is installable but unsigned, and should be allowlisted by hash rather than by publisher.

One caveat worth being precise about: the manifest travels **inside** the zip, so on its own it only proves the package is internally consistent. **Step 1 and step 2 above are what tie the package to us** — the checksum covers the bytes you downloaded, and the signature proves who produced the executable.

---

## Getting help

- **Verification or installation problems** — contact SolvePoint Solutions through your usual support channel.
- **A checksum or signature that does not match** — do not install, and tell us immediately.

Issues are not monitored on this repository.
