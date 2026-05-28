# Copilot Hub — SharePoint Site Template

A communication-site template modeled after the [Power Platform adoption hub template](https://learn.microsoft.com/en-us/power-platform/guidance/adoption/sharepoint-site-template), adapted for:

- **Microsoft 365 Copilot Chat**
- **Microsoft 365 Copilot**
- **Microsoft 365 Agents** (declarative agents / Agent Builder)
- **Microsoft Copilot Studio agents**

Training is built into the existing **Guided Learning** page and is backed by a `Learning Paths` list, pre-seeded with **9 verified Microsoft Learn courses**.

---

## What's in the box

| File | Purpose |
|---|---|
| `CopilotHub.PnP.xml`        | PnP provisioning template — pages, lists, site columns, nav |
| `Deploy-CopilotHub.ps1`     | Creates/connects the site, applies the template, registers as a hub |
| `Seed-LearningPaths.ps1`    | Adds the 9 Microsoft Learn courses to the `Learning Paths` list |
| `README.md`                 | This guide |

---

## Site structure

**Site type:** Communication site (`SITEPAGEPUBLISHING#0`)
**Default URL:** `/sites/copilothub`

### Pages (16)

| Page | Section |
|---|---|
| `Accelerate-work-with-Microsoft-365-Copilot.aspx` | Home |
| `Copilot-Chat.aspx` | Products |
| `Microsoft-365-Copilot.aspx` | Products |
| `Microsoft-365-Agents.aspx` | Products |
| `Copilot-Studio-Agents.aspx` | Products |
| `Responsible-AI-and-Acceptable-Use.aspx` | Governance |
| `Data-Security-and-Sensitivity-Labels.aspx` | Governance |
| `DLP-for-Copilot-and-Agents.aspx` | Governance |
| `Agent-Lifecycle-and-Approval.aspx` | Governance |
| `Environments.aspx` | Governance |
| **`Guided-Learning.aspx`** | **Learn — hosts the training tracks + Learning Paths list view** |
| `Consultation-and-Office-Hours.aspx` | Learn |
| `Internal-Communities.aspx` | Learn |
| `Copilot-at-Company.aspx` | About |
| `Requesting-a-Copilot-License.aspx` | About |
| `Support.aspx` | About |

### Lists (5)

| List | Purpose |
|---|---|
| **Learning Paths** | Microsoft Learn courses (Product, Audience, Requirement, Duration, Level, URL) |
| Agent Catalog       | Registered agents + approval state |
| Approved Connectors | Connectors approved under DLP |
| Prompt Library      | Vetted, shareable prompts |
| Events              | Office hours and training sessions |

### Training tracks (Guided Learning page)

| Track | Audience | Courses |
|---|---|---|
| 1 | End users | Copilot Chat (Basic), Get started with M365 Copilot |
| 2 | Makers    | Copilot Studio learning path, Declarative agents docs, Agent Builder docs |
| 3 | Admins    | Prepare your org, Security & compliance, Implement M365 Copilot, MS-4017 |
| 4 | Hub       | Microsoft Copilot learning hub |

All URLs are verified against `learn.microsoft.com`.

---

## Prerequisites

### 1. PowerShell modules

```powershell
Install-Module PnP.PowerShell -Scope CurrentUser
```

Requires **PnP.PowerShell 2.x** or later on **PowerShell 7.2+**.

### 2. Permissions on the SPO tenant

You must run the deployment as either:

- A **SharePoint Administrator** (or Global Admin), **OR**
- An **Entra app** with `Sites.FullControl.All` (application permission, admin-consented)

Certificate-based app auth is recommended for repeatable deployments and CI/CD.

### 3. Entra app registration (one-time)

```powershell
# In an interactive PowerShell session, signed in as a Global Admin:
Register-PnPEntraIDAppForInteractiveLogin `
    -ApplicationName "PnP-CopilotHub-Deploy" `
    -Tenant          "contoso.onmicrosoft.com" `
    -SharePointApplicationPermissions "Sites.FullControl.All" `
    -CertificatePath ".\PnP-Cert.pfx"
```

This:

1. Registers the app in Entra ID
2. Generates a self-signed certificate (`PnP-Cert.pfx` + `.cer`)
3. Uploads the public key to the app
4. Grants `Sites.FullControl.All` and prompts for admin consent

**Save the `ClientId` printed at the end** — you'll pass it to the deployment script.

> Alternative: if you can't grant `Sites.FullControl.All`, use **interactive** auth instead. Replace the `-Client/-Tenant/-Cert*` parameters in both scripts with `-Interactive`, and run as a SharePoint Admin.

### 4. Files in one folder

Put all three files in the same folder:

```
.\CopilotHub\
    CopilotHub.PnP.xml
    Deploy-CopilotHub.ps1
    Seed-LearningPaths.ps1
    PnP-Cert.pfx
```

---

## Deploy

```powershell
# Open PowerShell 7 in the CopilotHub folder
$certPwd = Read-Host -AsSecureString "Certificate password"

.\Deploy-CopilotHub.ps1 `
    -TenantUrl   "https://contoso.sharepoint.com" `
    -SiteUrl     "https://contoso.sharepoint.com/sites/copilothub" `
    -SiteTitle   "Copilot Hub" `
    -CompanyName "Contoso" `
    -Owner       "admin@contoso.onmicrosoft.com" `
    -ClientId    "<your-client-id>" `
    -Tenant      "contoso.onmicrosoft.com" `
    -CertificatePath     ".\PnP-Cert.pfx" `
    -CertificatePassword $certPwd `
    -RegisterAsHub
```

### What it does, in order

1. **Verifies PnP.PowerShell** is installed
2. **Connects to `-admin.sharepoint.com`** to check whether the site exists
3. **Creates** the communication site if it doesn't exist (skips if it does)
4. **Connects to the site** and **applies `CopilotHub.PnP.xml`** — this provisions all 16 pages, 5 lists, 9 site columns, and the global navigation
5. **Runs `Seed-LearningPaths.ps1`** to insert the 9 Microsoft Learn courses (skips rows that already exist — idempotent)
6. **Registers the site as a SharePoint hub** if `-RegisterAsHub` is set

Re-running the script is safe — `Invoke-PnPSiteTemplate` is idempotent and the seed script skips existing rows.

---

## Customize before you ship

Open `CopilotHub.PnP.xml` and edit:

| What | Where |
|---|---|
| Default `CompanyName` value | `<pnp:Parameters>` near the bottom |
| Page body copy              | `<pnp:CanvasControlProperty Key="Text" ... >` inside each `<pnp:ClientSidePage>` |
| Navigation nodes            | `<pnp:StructuralNavigation>` block |
| Lists / columns             | `<pnp:Lists>` and `<pnp:SiteFields>` blocks |

Open `Seed-LearningPaths.ps1` and edit the `$courses` array to add internal training (LinkedIn Learning, Viva Learning, in-house Teams Live Events, etc.).

---

## Update later

To push template changes to a deployed site, just re-run:

```powershell
.\Deploy-CopilotHub.ps1 -SiteUrl ... -CompanyName ... -ClientId ... <etc.>
```

`Invoke-PnPSiteTemplate` will reconcile pages, lists, fields, and nav. Existing list data is preserved.

To add or change Learning Paths rows, edit `$courses` and re-run:

```powershell
.\Seed-LearningPaths.ps1 -SiteUrl ... -ClientId ... <etc.>
```

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| `PnP.PowerShell is not installed` | `Install-Module PnP.PowerShell -Scope CurrentUser` |
| `AccessDenied` during `New-PnPSite` | App needs `Sites.FullControl.All` (admin-consented), OR run interactively as a SharePoint Admin |
| `Get-PnPTenantSite` fails with 403 | You're not connected to the `-admin` endpoint — the script handles this, but if connecting manually use `https://<tenant>-admin.sharepoint.com` |
| `Invoke-PnPSiteTemplate` warns about a missing list | Re-run the script — list provisioning sometimes lags behind page provisioning on the first apply |
| `List 'Learning Paths' not found` | Apply the template first, then run the seed script |
| Hub registration fails | You need `SharePoint Administrator` or `Global Administrator` for `Register-PnPHubSite` |
| Pages show `{parameter:CompanyName}` literally | You missed the `-CompanyName` parameter on `Deploy-CopilotHub.ps1` |

---

## References

- [PnP PowerShell docs](https://pnp.github.io/powershell/)
- [PnP provisioning schema](https://github.com/pnp/PnP-Provisioning-Schema)
- [Power Platform site template (model for this one)](https://learn.microsoft.com/en-us/power-platform/guidance/adoption/sharepoint-site-template)
- [Microsoft Copilot learning hub](https://learn.microsoft.com/en-us/copilot/)
