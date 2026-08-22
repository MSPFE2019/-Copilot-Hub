# Copilot Hub — SharePoint Site Template

> A drop-in SharePoint communication-site template for organizations rolling out **Microsoft 365 Copilot**, **Copilot Chat**, **Microsoft 365 Agents**, **Microsoft Copilot Studio**, and **GitHub Copilot** — modeled after the [Power Platform adoption hub template](https://learn.microsoft.com/en-us/power-platform/guidance/adoption/sharepoint-site-template) and adapted for the Copilot product family.

[![PnP PowerShell](https://img.shields.io/badge/PnP.PowerShell-1.12.x-0078D4?logo=powershell&logoColor=white)](https://pnp.github.io/powershell/)
[![SharePoint Online](https://img.shields.io/badge/SharePoint-Online-038387?logo=microsoftsharepoint&logoColor=white)](https://learn.microsoft.com/en-us/sharepoint/)
[![Microsoft 365](https://img.shields.io/badge/Microsoft%20365-Copilot-7719AA?logo=microsoft&logoColor=white)](https://learn.microsoft.com/en-us/copilot/)
> **Distribution note:** The supported release artifact is `release/CopilotHub-v4.zip`. The root `CopilotHub-v3.zip` is a separate, inconsistently named local artifact whose embedded `VERSION.txt` says `4.0.0`; do not treat its filename as a v3 release or substitute it for the v4 release. Extract the supported release archive and run the scripts from the extracted package directory. The README alone is not a deployable copy of the template.

**Two design principles:**

1. **Interactive authentication** — deployment uses a tenant-owned, single-tenant public-client Entra app and delegated permissions. No certificates or stored secrets are required.
2. **Config-driven** — every tenant-specific value (company name, site URL, owner, app ID) lives in a single `Config.psd1` file. You do not edit the provisioning XML for routine deployment.

> **Security warning:** This workflow requires the delegated SharePoint `AllSites.FullControl` permission. Although it is not an application permission and is limited by the signed-in user's access, it is still a high-privilege permission that can affect all SharePoint sites accessible to that administrator during the session. Use a dedicated administrative identity, review admin-consent policy, run the deployment from a managed workstation, and remove or disable the app registration when it is no longer needed.

> **Why an Entra app at all?** As of September 2024, Microsoft retired the shared "PnP Management Shell" multi-tenant app. PnP PowerShell now requires each tenant to register its own app — even for interactive sign-in. The registration is one-time and uses delegated permissions only.

> **Release 4.0.0 hardening:** The provisioning package contains no notification webhook. Designer access, declarative workflow creation, and member sharing are disabled by default; enable them only after tenant governance review. Review all sample pages and events before publishing.

---

## Package contents and integrity

After extracting the supported `release/CopilotHub-v4.zip`, the package should contain the files described below. If a required file is missing, do not run the deployment; download the package again and verify its checksum.

```text
CopilotHub/
├── Config.psd1
├── Register-PnPApp.ps1
├── Deploy-CopilotHub.ps1
├── Seed-LearningPaths.ps1
├── template.pnp
├── assets/
└── ReadMe.md
```

Before extracting or running administrative scripts, calculate the archive hash and compare it with the SHA-256 value published with the corresponding release or deployment announcement:

```powershell
Get-FileHash .\CopilotHub-v4.zip -Algorithm SHA256
```

For the supported `release/CopilotHub-v4.zip` artifact (`v4.0.0`), the verified SHA-256 value is:

```text
A86EDA7FBC51A3D7A54F1EAFDC299552636922474520197DB5B309EA918A35DE
```

Only use a release or commit from the supported repository history. Do not substitute an archive copied from an untrusted location.

The root `CopilotHub-v3.zip` currently hashes to `6765CE62A8B264148DBA8F7A5267A7280804E4BE1C4B61A20017ED066358AD8D` while carrying `VERSION.txt` value `4.0.0`. Because that artifact is mislabeled/inconsistent, do not use it as the v4 release.

---

## Table of contents

- [What you get](#what-you-get)
- [Package contents and integrity](#package-contents-and-integrity)
- [Site structure](#site-structure)
- [Training tracks](#training-tracks)
- [Prerequisites](#prerequisites)
- [Configure](#configure)
- [Deploy](#deploy)
- [How it works](#how-it-works)
- [Customizing the template](#customizing-the-template)
- [Updating an existing deployment](#updating-an-existing-deployment)
- [Troubleshooting](#troubleshooting)
- [Roadmap](#roadmap)
- [References](#references)
- [License](#license)

---

## What you get

The content is original plain-language guidance. Product facts link to the official [Microsoft 365 Copilot](https://www.microsoft.com/en-us/microsoft-365/copilot) and [GitHub Copilot](https://github.com/features/copilot) pages rather than reproducing marketing copy. The template uses the supplied Microsoft 365 Copilot mark and a brand-safe palette (`#7719AA` Microsoft 365 Copilot purple, `#24292F` GitHub neutral); no third-party assets are fetched during deployment.

```
CopilotHub/
├── Config.psd1                 # ← edit this once
├── Register-PnPApp.ps1         # One-time: registers an Entra app for PnP sign-in
├── Deploy-CopilotHub.ps1       # Hybrid deploy: site + columns + lists (native) + pages/nav/Events (template)
├── Seed-LearningPaths.ps1      # Reads Config.psd1, seeds 10 learning resources
├── template.pnp               # PnP provisioning template — 26 rich pages, Events list, nav, 23 stock photos
├── assets/                     # Branded PNGs uploaded to /SiteAssets/CopilotHub/ at deploy time
│   ├── site-logo.png           #   96×96 — official M365 Copilot ribbon glyph
│   ├── welcome-banner.png      #   1345×436 — hero on the home page
│   ├── icon-copilot-chat.png   #   600×600 — product card
│   ├── icon-m365-copilot.png   #   600×600 — product card
│   ├── icon-m365-agents.png    #   600×600 — product card
│   └── icon-copilot-studio.png #   600×600 — product card
└── ReadMe.md
```

Review the sample personas, stories, metrics, and event content before publishing the site. They are example content and must not be presented as real employee or organizational information without approval.

---

## Site structure

**Type:** Communication site (`SITEPAGEPUBLISHING#0`)
**Default URL:** `/sites/copilothub`

### Navigation

```
Home
├── Products
│   ├── Copilot Chat
│   ├── Microsoft 365 Copilot
│   ├── Microsoft 365 Agents
│   ├── Copilot Studio Agents
│   └── GitHub Copilot
├── Governance
│   ├── Responsible AI & Acceptable Use
│   ├── Data Security & Sensitivity Labels
│   ├── DLP for Copilot & Agents
│   ├── Agent Lifecycle & Approval
│   └── Environments
├── Learn
│   ├── Getting Started
│   ├── Guided Learning        ← training tracks live here
│   ├── Help & Learning
│   ├── Office Hours
│   ├── Consultation
│   └── Internal Communities
├── Community
│   ├── Champion of the Week   (+ Adele Vance, James Williams seeded)
│   ├── Success Stories        (+ Finance Copilot Studio Build seeded)
│   ├── Hackathons             (+ Research & Innovation hackathon seeded)
│   └── News & Announcements
├── Copilot at {CompanyName}
├── Get a License
└── Support
```

### Content templates

Hidden under `/SitePages/Templates/`. Use as starting points when posting community content:

- `Templates/News` - News story template
- `Templates/Story-template` - Success story template
- `Templates/Champion` - Champion of the Week template
- `Templates/Hackathon` - Hackathon event template

### Lists (4 native + 1 from template)

| List | Purpose | Provisioned by |
| --- | --- | --- |
| **Learning Paths** | Microsoft Learn courses (Product / Audience / Requirement / Duration / Level / URL) | Native cmdlets |
| Agent Catalog | Registered declarative, Agent Builder, and Copilot Studio agents | Native cmdlets |
| Approved Connectors | Connectors approved under DLP for use by agents | Native cmdlets |
| Prompt Library | Vetted, shareable prompts | Native cmdlets |
| Events | Office hours, training sessions, community events | `template.pnp` template |

### Site columns (9)

`CopilotProduct`, `CopilotAudience`, `CopilotRequirement`, `CopilotDurationMinutes`, `CopilotLevel`, `CopilotLearnUrl`, `CopilotOwner`, `CopilotConnector`, `CopilotApprovalState` — all grouped under **Copilot Hub** in the site column gallery.

---

## Training tracks

The **Guided Learning** page hosts four role-based tracks. Every URL is a verified Microsoft Learn resource.

### Track 1 — End users

| Course | Length | Level |
| --- | --- | --- |
| [Transform ideas into action with Copilot Chat (Basic)](https://learn.microsoft.com/en-us/training/paths/explore-microsoft-365-copilot-business-chat/) | 1h 23m · 3 modules | Beginner |
| [Get started with Microsoft 365 Copilot](https://learn.microsoft.com/en-us/training/paths/get-started-with-microsoft-365-copilot/) | 1h 31m · 3 modules | Beginner |

### Track 2 — Makers

| Course | Length | Level |
| --- | --- | --- |
| [Create agents in Microsoft Copilot Studio](https://learn.microsoft.com/en-us/training/paths/create-extend-custom-copilots-microsoft-copilot-studio/) | 4h 5m · 5 modules | Intermediate |
| [Declarative agents for Microsoft 365 Copilot (overview)](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/overview-declarative-agent) | Docs | Intermediate |
| [Agent Builder in Microsoft 365 Copilot](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/agent-builder) | Docs | Beginner |

### Track 3 — Admins / IT Pros

| Course | Length | Level |
| --- | --- | --- |
| [Prepare your organization for Microsoft 365 Copilot](https://learn.microsoft.com/en-us/training/paths/prepare-your-organization-microsoft-365-copilot/) | 1h 20m · 2 modules | Intermediate |
| [Prepare security and compliance to support Microsoft 365 Copilot](https://learn.microsoft.com/en-us/training/paths/prepare-security-compliance-support-microsoft-365-copilot/) | 6h 27m · 7 modules | Intermediate |
| [Implement Microsoft 365 Copilot](https://learn.microsoft.com/en-us/training/modules/implement-microsoft-365-copilot/) | 51 min · 10 units | Intermediate |
| [MS-4017: Manage and extend Microsoft 365 Copilot](https://learn.microsoft.com/en-us/training/courses/ms-4017) | 1 day (ILT) | Intermediate |

### Track 4 — GitHub Copilot

| Resource | Length | Level |
| --- | --- | --- |
| [GitHub Copilot documentation](https://docs.github.com/en/copilot) | Docs | Beginner |

### Track 5 — Always-on hub

- [Microsoft Copilot learning hub](https://learn.microsoft.com/en-us/copilot/)
- [Microsoft 365 Copilot adoption resources](https://learn.microsoft.com/en-us/microsoft-365/copilot/microsoft-365-copilot-enablement-resources)
- [Microsoft 365 Copilot product overview](https://www.microsoft.com/en-us/microsoft-365/copilot)
- [GitHub Copilot product overview](https://github.com/features/copilot)
- [GitHub Copilot documentation](https://docs.github.com/en/copilot)

---

## Prerequisites

### PowerShell

```powershell
Install-Module PnP.PowerShell -Scope CurrentUser
```

Requires **PnP.PowerShell 1.12.x** on **PowerShell 7.2+**. This is pinned for the bundled PnP.Framework 1.10.2 template.

### People

| Role | What they do | When |
| --- | --- | --- |
| **Global Administrator** | Runs `Register-PnPApp.ps1` to register the Entra app | One time, per tenant |
| **SharePoint Administrator** (or Global Admin) | Runs the deployment interactively | Each deploy run |
| **Site owner** | The UPN listed in `Config.psd1` as `Owner` | At site creation |

You do **not** generate certificates or store secrets. The Entra app uses delegated permissions only, but `AllSites.FullControl` is high privilege: authorization at deploy time comes from your SharePoint Admin role and can affect every SharePoint site that account can access during the session.

---

## Configure

Open `Config.psd1` and fill in your tenant values (leave `ClientId` empty for now — you'll fill it in after the one-time app registration):

```powershell
@{
    TenantUrl     = "https://contoso.sharepoint.com"
    SiteUrl       = "https://contoso.sharepoint.com/sites/copilothub"
    SiteTitle     = "Copilot Hub"
    CompanyName   = "Contoso"
    Owner         = "admin@contoso.onmicrosoft.com"
    Tenant        = "contoso.onmicrosoft.com"
    ClientId      = ""                          # filled in by step 1 below
    RegisterAsHub = $true
}
```

Both `Deploy-CopilotHub.ps1` and `Seed-LearningPaths.ps1` read from this file. There are no other tenant-specific values anywhere in the repo, and the XML template is never edited.

---

## Deploy

### Step 0 — Extract and preflight the package

Run the following from the directory containing `CopilotHub-v3.zip`:

```powershell
Expand-Archive .\CopilotHub-v3.zip -DestinationPath .\CopilotHub -Force
Set-Location .\CopilotHub
Get-ChildItem .\Config.psd1, .\Register-PnPApp.ps1, .\Deploy-CopilotHub.ps1, .\Seed-LearningPaths.ps1, .\template.pnp
```

The final command must list every required file. Stop and obtain a complete, trusted package if any file is missing.

### Step 1 — One-time: register the Entra app

You have two options. Pick whichever works on your machine.

**Option A — Script (PowerShell 7.2+ with PnP.PowerShell 2.x):**

```powershell
.\Register-PnPApp.ps1 -Tenant contoso.onmicrosoft.com
```

A browser opens. Sign in as a Global Administrator. The script calls `Register-PnPEntraIDAppForInteractiveLogin` and prints the `ClientId`.

> If you see `The term 'Register-PnPEntraIDAppForInteractiveLogin' is not recognized`, you're on Windows PowerShell 5.1 / PnP.PowerShell 1.x. The script will detect this and print Option B for you.

**Option B — Entra portal (any PowerShell version):**

1. Open [https://entra.microsoft.com](https://entra.microsoft.com) and sign in as **Global Administrator**.
2. **Identity → Applications → App registrations → New registration**:
   - Name: `PnP-CopilotHub-Deploy`
   - Account types: *Accounts in this organizational directory only (single tenant)*
   - Redirect URI: **Public client/native (mobile & desktop)** → `http://localhost`
   - Click **Register**.
3. On the **Overview** page, copy the **Application (client) ID**.
4. **API permissions → Add a permission**:
   - **SharePoint → Delegated → `AllSites.FullControl`**
   - **Microsoft Graph → Delegated → `User.Read`**
   - Remove the default `User.Read` if it was auto-added twice.
   - Click **Grant admin consent for *(tenant)***.
5. **Authentication → Advanced settings → "Allow public client flows"** → **Yes** → Save.

> These are the permissions required by this workflow. They are **delegated**, meaning the app inherits the signed-in user's scope. There is no `Sites.FullControl.All` *application* permission, but delegated `AllSites.FullControl` remains high privilege and requires careful admin-consent review.
>
> **Why not Graph `Sites.FullControl.All`?** PnP PowerShell uses SharePoint REST endpoints (not Microsoft Graph) for site creation, template apply, and hub registration in this workflow. The SharePoint `AllSites.FullControl` delegated permission is sufficient on its own. Graph `User.Read` is included only so Entra can resolve the signed-in user's identity during the interactive sign-in flow.

**Then for either option:**

Paste the `ClientId` into `Config.psd1` → `ClientId`. Wait ~1 minute for Entra propagation.

> If you later see `AADSTS700016: Application with identifier ... was not found` when running Deploy, the `ClientId` in `Config.psd1` is missing, wrong, or from a different tenant.

### Step 2 — Deploy

```powershell
.\Deploy-CopilotHub.ps1
```

A browser window opens. **Sign in as a SharePoint Administrator** (or Global Admin). Your role authorizes the deployment for the duration of the session — nothing is granted permanently.

The script is fully idempotent. Re-run it to push template changes; existing list data and site owners are preserved. It never permanently removes a deleted or errored site automatically. After explicit administrator approval, use `.\Deploy-CopilotHub.ps1 -PurgeConflictingSite`.

---

## How it works

`Deploy-CopilotHub.ps1` runs a **hybrid deployment** — native PnP cmdlets for structured data (giving per-step error messages), and `Invoke-PnPSiteTemplate` for rich modern pages (giving News carousels, Events web parts, QuickLinks, and full navigation that can't be replicated cleanly via cmdlets alone):

| # | Phase | How | What it does |
| --- | --- | --- | --- |
| 1 | Module check | cmdlet | Verifies PnP.PowerShell is installed |
| 2a | Connect to `-admin` endpoint | cmdlet | Interactive browser sign-in as SharePoint Admin |
| 2b | Register as hub site | cmdlet | Only if `RegisterAsHub = $true` (runs in admin context so no extra login) |
| 3 | Ensure site exists | cmdlet | Creates the communication site if missing |
| 4 | Connect to site | cmdlet | Reconnects to the new site URL |
| 5 | Apply PnP template | `Invoke-PnPSiteTemplate` | Provisions 26 rich pages (including GitHub Copilot guidance), Events, QuickLinks, and top navigation — from `template.pnp` |
| 6 | Site columns | cmdlet | Creates 9 Copilot Hub site columns via `Add-PnPField` |
| 7 | Lists | cmdlet | Creates 4 lists (Learning Paths, Agent Catalog, Approved Connectors, Prompt Library) via `New-PnPList` and wires field refs — Events list is handled by phase 5 |
| 8 | Views | cmdlet | Creates `By Audience`, `By Product`, `Required Only` views on Learning Paths |
| 9 | Branded assets | cmdlet | Uploads `assets/*.png` to `/SiteAssets/CopilotHub/` and sets the site logo |
| 10 | Seed `Learning Paths` | script | Adds 10 official learning resources (idempotent on Title) |

Every phase is idempotent. Safe to re-run — existing columns, lists, pages, and list items are skipped. Template data rows use overwrite behavior, so review sample Events before rerunning in production.

> **Why the hybrid approach?** `Invoke-PnPSiteTemplate` gives rich web parts (News carousel, Events, QuickLinks, People) that are impossible to reproduce cleanly via cmdlets alone — and the `.pnp` template carries 23 stock photos and the full navigation tree. Native cmdlets handle Copilot-specific lists and columns because they give a real error per step (template engine only reports "Template is not valid" with no detail when any element fails).

---

## Customizing the template

### Branding & copy

**Pages and navigation** live inside `template.pnp`. To modify them you'll need to:
1. Unzip `template.pnp` (it's a standard zip archive — rename to `.zip` if needed).
2. Edit the main provisioning XML (named by GUID — check `ProvisioningTemplate/files-map.xml` to find `template.xml`).
3. Re-zip with the same internal structure and rename back to `.pnp`.

For most branding changes, only three things need editing:

| What | Where |
| --- | --- |
| Company name substitution | `Config.psd1` → `CompanyName` (injected by Deploy into site metadata and description) |
| Site logo | Replace `assets/site-logo.png` and re-run Deploy |
| Hero banner image | Replace `assets/welcome-banner.png` and re-run Deploy |

### Swap in official Microsoft brand assets

The `assets/` folder ships with self-generated PNG icons in Copilot colors. They are intentionally **self-contained** — no external CDN dependencies — so the site renders correctly on tenants with restrictive egress policy (e.g., GCC).

To replace them, drop new PNGs of the same name into `assets/` and re-run `Deploy-CopilotHub.ps1`:

| File | Size | Used for |
| --- | --- | --- |
| `site-logo.png` | 96×96 | Site logo |
| `welcome-banner.png` | 1345×436 | Hero banner on the home page |
| `icon-copilot-chat.png` | 600×600 | Product card |
| `icon-m365-copilot.png` | 600×600 | Product card |
| `icon-m365-agents.png` | 600×600 | Product card |
| `icon-copilot-studio.png` | 600×600 | Product card |

Deploy uploads everything in `assets/` to `/SiteAssets/CopilotHub/` and the `.pnp` template references them by site-relative path — so they work on every tenant URL without edits. The 23 stock photo backgrounds (hero and section images) come bundled inside `template.pnp` itself and are uploaded to `/SiteAssets/Images/` automatically.

### Add internal training rows

Append to `$courses` in `Seed-LearningPaths.ps1`:

```powershell
@{
    Title       = "Contoso Copilot Onboarding (instructor-led)"
    Product     = "Microsoft 365 Copilot"
    Audience    = "End User"
    Requirement = "Required"
    Duration    = 60
    Level       = "Beginner"
    Url         = "https://contoso.sharepoint.com/sites/copilothub/SitePages/Onboarding.aspx"
}
```

Re-run — it skips existing titles.

---

## Updating an existing deployment

```powershell
# Push template changes (pages, columns, lists, nav)
.\Deploy-CopilotHub.ps1

# Refresh just the Learning Paths list
.\Seed-LearningPaths.ps1
```

`Deploy-CopilotHub.ps1` is idempotent — each phase skips resources that already exist and only creates what's missing. Existing list data is always preserved.

---

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| `PnP.PowerShell is not installed` | `Install-Module PnP.PowerShell -Scope CurrentUser` |
| `Config.psd1 is missing required value: X` | Open `Config.psd1` and fill in the missing value |
| `template.pnp not found` | Extract the complete package and ensure `template.pnp` is in the same folder as `Deploy-CopilotHub.ps1` |
| `AADSTS700016: Application with identifier ... was not found` | You haven't done the one-time app registration yet, or `ClientId` in `Config.psd1` is wrong / from a different tenant |
| `Register-PnPEntraIDAppForInteractiveLogin is not recognized` | You're on Windows PowerShell 5.1 / PnP.PowerShell 1.x. Use **Option B** (Entra portal) under Step 1, or upgrade to PowerShell 7.2+ and `Install-Module PnP.PowerShell -Force` |
| Sign-in works but `Connect-PnPOnline` returns `AccessDenied` | Wait ~1 minute after registration for Entra propagation, then retry. If still failing, confirm admin consent was granted for `AllSites.FullControl` and `User.Read`, and verify that the signed-in account is a SharePoint Administrator |
| `AADSTS65001: The user or administrator has not consented to use the application` | Open the app in Entra portal → API permissions → click **Grant admin consent for *(tenant)*** |
| Browser prompt loops on sign-in | Make sure your account holds the **SharePoint Administrator** role |
| `Get-PnPTenantSite` returns 403 | You're not signed in as a SharePoint Admin — re-run and sign in with the correct account |
| `New-PnPSite : {"RawSiteProvisionState":1,"SiteId":"","SiteStatus":3,"SiteUrl":""}` | A previous create attempt left state behind, OR the `Owner` UPN isn't valid in your tenant. The current `Deploy-CopilotHub.ps1` auto-detects deleted/errored sites and verifies the owner UPN before retrying — re-pull the latest script. If you still hit it, inspect the SharePoint admin center → **Active sites** and **Deleted sites**. Do not delete or permanently remove a site/data as a troubleshooting shortcut; obtain an approved recovery/cleanup plan first. |
| Site creation: "Owner UPN ... could not be resolved" | `Owner` in `Config.psd1` is a placeholder (`admin@contoso.onmicrosoft.com`) or a user that doesn't exist in your tenant. Set it to a real licensed UPN (often your own) |
| `List 'Learning Paths' not found` | Run `Deploy-CopilotHub.ps1` before the seed script |
| Pages show `{parameter:CompanyName}` literally | `CompanyName` is blank in `Config.psd1` |
| Welcome page shows broken image icons | Asset upload phase (phase 9) failed. Confirm `assets/*.png` exists next to the deploy script, then re-run. `/SiteAssets/CopilotHub/` should contain the PNGs |
| Pages or sections are missing after deploy | `Invoke-PnPSiteTemplate` (phase 5) failed silently. Re-run with `-Verbose` to see which web part caused the error. If a stock photo is missing, the template will not apply — ensure `template.pnp` is not corrupt |
| Edits to page content aren't showing | Pages and nav are owned by the `.pnp` template. Edit `template.xml` inside `template.pnp` (it's a zip), re-zip, re-run Deploy |
| `Register-PnPHubSite` fails | Confirm `RegisterAsHub = $true` and that your account is a SharePoint Admin |

---

## Roadmap

- [ ] Power Automate flow: auto-assign required training when a Copilot license is requested
- [ ] Power BI report on training completion rates
- [ ] Sample declarative agent definition + Copilot Studio solution (planned; not included in the current package)
- [ ] Localized content packs (es-ES, fr-FR, de-DE)

---

## References

- [PnP PowerShell docs](https://pnp.github.io/powershell/)
- [PnP provisioning schema](https://github.com/pnp/PnP-Provisioning-Schema)
- [Power Platform site template (origin model)](https://learn.microsoft.com/en-us/power-platform/guidance/adoption/sharepoint-site-template)
- [Microsoft Copilot learning hub](https://learn.microsoft.com/en-us/copilot/)

---

## License

No `LICENSE` file is present at this repository root. Until a repository license is added, treat repository contents as all rights reserved and obtain permission before redistributing or modifying them. The extracted release package may contain its own license file; that does not automatically license the repository, local artifacts, or separately copied assets. Microsoft, Microsoft 365 Copilot, Copilot Studio, and SharePoint are trademarks of the Microsoft group of companies.
