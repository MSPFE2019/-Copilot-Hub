# Copilot Hub — SharePoint Site Template

> A drop-in SharePoint communication-site template for organizations rolling out **Microsoft 365 Copilot**, **Copilot Chat**, **Microsoft 365 Agents**, and **Microsoft Copilot Studio agents** — modeled after the [Power Platform adoption hub template](https://learn.microsoft.com/en-us/power-platform/guidance/adoption/sharepoint-site-template) and adapted for the Copilot product family.

[![PnP PowerShell](https://img.shields.io/badge/PnP.PowerShell-2.x-0078D4?logo=powershell&logoColor=white)](https://pnp.github.io/powershell/)
[![SharePoint Online](https://img.shields.io/badge/SharePoint-Online-038387?logo=microsoftsharepoint&logoColor=white)](https://learn.microsoft.com/en-us/sharepoint/)
[![Microsoft 365](https://img.shields.io/badge/Microsoft%20365-Copilot-7719AA?logo=microsoft&logoColor=white)](https://learn.microsoft.com/en-us/copilot/)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

**Two design principles:**

1. **Least privilege by default** — no tenant-wide `Sites.FullControl.All` grant, no Entra app registration, no certificates, no stored secrets. You sign in interactively as a **SharePoint Administrator** for the duration of the deploy, and nothing is granted permanently.
2. **Config-driven** — every tenant-specific value (company name, site URL, owner) lives in a single `Config.psd1` file. You never edit the XML, and you never pass long parameter lists.

---

## Table of contents

- [What you get](#what-you-get)
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

```
CopilotHub/
├── Config.psd1                 # ← edit this once
├── CopilotHub.PnP.xml          # PnP provisioning template (pages, lists, nav)
├── Deploy-CopilotHub.ps1       # Reads Config.psd1, creates site, applies template
├── Seed-LearningPaths.ps1      # Reads Config.psd1, seeds 9 Microsoft Learn courses
└── README.github.md
```

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
│   └── Copilot Studio Agents
├── Governance
│   ├── Responsible AI & Acceptable Use
│   ├── Data Security & Sensitivity Labels
│   ├── DLP for Copilot & Agents
│   ├── Agent Lifecycle & Approval
│   └── Environments
├── Learn
│   ├── Guided Learning        ← training tracks live here
│   ├── Consultation & Office Hours
│   └── Internal Communities
├── Copilot at {CompanyName}
├── Get a License
└── Support
```

### Lists (5)

| List | Purpose |
| --- | --- |
| **Learning Paths** | Microsoft Learn courses (Product / Audience / Requirement / Duration / Level / URL) |
| Agent Catalog | Registered declarative, Agent Builder, and Copilot Studio agents |
| Approved Connectors | Connectors approved under DLP for use by agents |
| Prompt Library | Vetted, shareable prompts |
| Events | Office hours, training sessions, community events |

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

### Track 4 — Always-on hub

- [Microsoft Copilot learning hub](https://learn.microsoft.com/en-us/copilot/)
- [Microsoft 365 Copilot adoption resources](https://learn.microsoft.com/en-us/microsoft-365/copilot/microsoft-365-copilot-enablement-resources)

---

## Prerequisites

### PowerShell

```powershell
Install-Module PnP.PowerShell -Scope CurrentUser
```

Requires **PnP.PowerShell 2.x** on **PowerShell 7.2+**.

### People

| Role | What they do | When |
| --- | --- | --- |
| **SharePoint Administrator** (or Global Admin) | Runs the deployment interactively | Each deploy run |
| **Site owner** | The UPN listed in `Config.psd1` as `Owner` | At site creation |

You do **not** need to register an Entra app, generate a certificate, or grant `Sites.FullControl.All` to use this template. Authorization comes from your SharePoint Admin role in the browser sign-in, and lasts only for the session.

---

## Configure

Open `Config.psd1` and fill in your tenant values:

```powershell
@{
    TenantUrl     = "https://contoso.sharepoint.com"
    SiteUrl       = "https://contoso.sharepoint.com/sites/copilothub"
    SiteTitle     = "Copilot Hub"
    CompanyName   = "Contoso"
    Owner         = "admin@contoso.onmicrosoft.com"
    RegisterAsHub = $true
}
```

Both `Deploy-CopilotHub.ps1` and `Seed-LearningPaths.ps1` read from this file. There are no other tenant-specific values anywhere in the repo, and the XML template is never edited.

---

## Deploy

```powershell
# 1. Edit Config.psd1
# 2. Run:
.\Deploy-CopilotHub.ps1
```

A browser window opens. **Sign in as a SharePoint Administrator** (or Global Admin). Your role authorizes the deployment for the duration of the session — nothing is granted permanently, and no app registration is created.

The script is fully idempotent. Re-run it any time to push template changes; existing list data is preserved.

---

## How it works

`Deploy-CopilotHub.ps1` runs these phases in order:

| # | Phase | What it does |
| --- | --- | --- |
| 1 | Module check | Verifies PnP.PowerShell is installed |
| 2 | Connect to `-admin` endpoint | Interactive browser sign-in as SharePoint Admin |
| 3 | Ensure site exists | Creates the communication site if missing |
| 4 | Apply `CopilotHub.PnP.xml` | Substitutes `{parameter:CompanyName}` from `Config.psd1`; provisions columns, lists, pages, navigation |
| 5 | Seed `Learning Paths` list | Adds 9 Microsoft Learn courses (idempotent on Title) |
| 6 | Register as hub site | Only if `RegisterAsHub = $true` |

Every phase is idempotent. Safe to re-run.

---

## Customizing the template

### Branding & copy

Edit `CopilotHub.PnP.xml`:

| What | Where |
| --- | --- |
| Page body copy | `<pnp:CanvasControlProperty Key="Text" ... >` inside each `<pnp:ClientSidePage>` |
| Navigation nodes | `<pnp:StructuralNavigation>` block |
| Lists / columns | `<pnp:Lists>` and `<pnp:SiteFields>` blocks |

> You do NOT need to edit `<pnp:Parameters>` for `CompanyName` — it comes from `Config.psd1` at deploy time.

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

`Invoke-PnPSiteTemplate` reconciles pages, lists, fields, and navigation. Existing list data is preserved.

---

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| `PnP.PowerShell is not installed` | `Install-Module PnP.PowerShell -Scope CurrentUser` |
| `Config.psd1 is missing required value: X` | Open `Config.psd1` and fill in the missing value |
| Browser prompt loops on sign-in | Make sure your account holds the **SharePoint Administrator** role |
| `Get-PnPTenantSite` returns 403 | You're not signed in as a SharePoint Admin — re-run and sign in with the correct account |
| `List 'Learning Paths' not found` | Run `Deploy-CopilotHub.ps1` before the seed script |
| Pages show `{parameter:CompanyName}` literally | `CompanyName` is blank in `Config.psd1` |
| `Register-PnPHubSite` fails | Confirm `RegisterAsHub = $true` and that your account is a SharePoint Admin |

---

## Roadmap

- [ ] Power Automate flow: auto-assign required training when a Copilot license is requested
- [ ] Power BI report on training completion rates
- [ ] Sample declarative agent definition + Copilot Studio solution under `/samples`
- [ ] Localized content packs (es-ES, fr-FR, de-DE)

---

## References

- [PnP PowerShell docs](https://pnp.github.io/powershell/)
- [PnP provisioning schema](https://github.com/pnp/PnP-Provisioning-Schema)
- [Power Platform site template (origin model)](https://learn.microsoft.com/en-us/power-platform/guidance/adoption/sharepoint-site-template)
- [Microsoft Copilot learning hub](https://learn.microsoft.com/en-us/copilot/)

---

## License

[MIT](LICENSE) — provided as-is. Microsoft, M365 Copilot, Copilot Studio, and SharePoint are trademarks of the Microsoft group of companies.
