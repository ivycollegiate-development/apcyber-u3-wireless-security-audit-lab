# apcyber-u3-wireless-security-audit-lab — Wireless Security Audit Lab

AP Cybersecurity Unit 3, Lesson 5. A simulated wireless environment with multiple networks to audit.

**You do not write the audit engine.** The detection logic is provided in `wireless_audit.py`. You run it over a synthetic dataset of wireless networks, then you do the hard part: decide which findings are critical, which are noise, and write remediation for the top 3.

## The Scenario

You are a security analyst contracted to audit the wireless infrastructure of **Taiwan Semiconductor Manufacturing Company (TSMC) Fab 18** — a 3nm process facility in Tainan. The audit covers 27 wireless networks across the campus: corporate Wi-Fi, guest networks, IoT sensor networks, building management systems, and isolated OT networks.

**27 wireless networks, 13 SSIDs, 6 encryption types, 4 rogue APs planted.** The data is `wireless_networks.csv`. The script is `wireless_audit.py`.

## Your Job

1. **Fork** this repo to your account.
2. Open it in VS Code — a Codespace from the repo page, or on the class server at `vscode.ivycollegiate.org` (log in with your school account). `git pull` first, always.
3. Review the **audit checklist** (`audit_checklist.md`) — it defines what you're looking for.
4. Run the script in the **terminal panel**, not a notebook, so you watch the results land on disk:

   ```bash
   python3 wireless_audit.py
   ```

   The environment has Python 3.11 with **pandas** and **scapy** preinstalled. Do not edit the script.
5. The audit runs, prints findings, and writes **`audit_results.csv`** — your graded deliverable.
6. **Read your own output.** This is the actual assignment.
7. Commit `audit_results.csv`. Then open it and write your **analyst notes** in the `analyst_notes` column for each finding — classify severity (Critical/High/Medium/Low), justify, and propose remediation for the top 3.
8. Commit again with your annotated results.

## The Data Schema

```csv
bssid,ssid,encryption,channel,signal_dbm,vendor_oui,is_hidden,client_count,notes
00:11:22:33:44:55,TSMC-Corp,WPA3-Enterprise,36,-45,00:11:22,False,12,802.1X-RADIUS
```

| Field | Description |
|-------|-------------|
| `bssid` | MAC address of the AP |
| `ssid` | Network name (empty if hidden) |
| `encryption` | `Open`, `WEP`, `WPA-Personal`, `WPA-Enterprise`, `WPA2-Personal`, `WPA2-Enterprise`, `WPA3-Personal`, `WPA3-Enterprise` |
| `channel` | Wi-Fi channel (1-165) |
| `signal_dbm` | Signal strength in dBm (negative, closer to 0 = stronger) |
| `vendor_oui` | First 3 octets of BSSID — manufacturer ID |
| `is_hidden` | `True`/`False` — hidden SSID |
| `client_count` | Number of associated clients observed |
| `notes` | Free-form context (e.g., "OT network", "Guest Wi-Fi", "Building mgmt") |

## The Audit Checks (What the Script Detects)

| Check ID | Detection | Why It Matters |
|----------|-----------|----------------|
| **WEP-001** | Any network using WEP encryption | WEP is broken; trivial to crack in minutes |
| **WPA1-001** | Any network using WPA (TKIP) — not WPA2/WPA3 | TKIP has known vulnerabilities; deprecated |
| **OPEN-001** | Open networks (no encryption) with clients | Anyone can join and sniff traffic |
| **HIDDEN-001** | Hidden SSIDs with clients | Security through obscurity; still discoverable |
| **ROGUE-001** | Same SSID where one AP's vendor OUI differs from that SSID's other APs — Evil Twin candidate | Classic evil twin: attacker mimics legitimate AP |
| **ROGUE-002** | SSID mimicking known corporate SSID with typo (e.g., "TSMC-C0rp") | Typosquatting for credential harvest |
| **CHANNEL-001** | Co-channel interference: >3 APs on same channel in same area | Performance + security: easier to jam/confuse |
| **WEAK-001** | WPA2-Personal with short/pre-shared key (inferred from context) | PSK vulnerable to offline dictionary attack |
| **OT-001** | OT/ICS network on same VLAN as IT traffic (flagged in notes) | IT/OT convergence failure — lateral movement path |
| **VENDOR-001** | Unknown/Unofficial vendor OUI on corporate network | Unauthorized hardware — potential rogue AP |

## The Real Skill Is Triage

**Every check will fire on something.** Your job is not to report every hit. Your job is to decide:

- Which findings are **actionable threats** vs. **accepted risk**?
- The guest Wi-Fi being open? **Accepted risk** — it's designed for visitors.
- The building management system on WPA-Personal? **High** — OT network with weak crypto.
- An evil twin mimicking "TSMC-Corp"? **Critical** — active credential harvesting.

**Context is everything.** The `notes` column and `vendor_oui` are your primary context clues. Use them.

## What You Hand In

1. **`audit_results.csv`** — committed, with your `analyst_notes` column filled in for every row.
2. **A one-paragraph executive summary** (in the commit message or a separate `EXECUTIVE_SUMMARY.md`):
   - How many total findings?
   - How many Critical/High/Medium/Low?
   - **The one finding you would fix first if you had budget for only one** — and why.
3. **A specific false positive from your output**, named by BSSID and SSID, with one sentence on why it is not a finding.

## The Gate (Auto-Grading)

The GitHub Action will verify:

- `audit_results.csv` exists and is committed.
- It contains all 10 check columns + `analyst_notes`.
- Every finding row has a severity classification in `analyst_notes`.
- At least 3 rows have remediation proposals.
- You named a specific false positive.

---

## Files

| File | What It Is |
|------|------------|
| `wireless_networks.csv` | The synthetic dataset. Do not edit it. |
| `wireless_audit.py` | The audit engine. Read it. Do not edit it. |
| `audit_checklist.md` | The checklist of what to look for. Read before running. |
| `audit_results.csv` | **Your output.** Created when you run the script. You edit this. |
| `EXECUTIVE_SUMMARY.md` | Optional — your executive summary if not in commit message. |