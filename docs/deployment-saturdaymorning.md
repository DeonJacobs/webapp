# Deployment: saturdaymorning (Azure Tenant a7a18fe7)

Provisioned: 2026-06-05

## Target Environment

| Property | Value |
|---|---|
| Azure Tenant ID | `a7a18fe7-908e-4061-92d3-6a699da041fd` |
| Subscription | Pay-As-You-Go (`68cfffbe-c761-4488-ab02-7a1b76786ae4`) |
| Resource Group | `fivekapp-rg` |
| Region | `australiaeast` |

## Provisioned Resources

| Resource | Name | Status |
|---|---|---|
| Resource Group | `fivekapp-rg` | ✅ Done |
| PostgreSQL Flexible Server | `saturdaymorning-db` (Standard_B1ms, PG 16) | ✅ Done |
| Database | `iteration1` | ✅ Done |
| DB Firewall Rule | `AllowAzureServices` (0.0.0.0–0.0.0.0) | ✅ Done |
| Storage Account | `saturdaymorningstorage` | ✅ Done |
| Blob Container | `uploads` (private) | ✅ Done |
| App Service Plan | `saturdaymorning-plan` (B1 Linux) | ✅ Done |
| Web App | `saturdaymorning` — https://saturdaymorning.azurewebsites.net | ✅ Done |
| Front Door CDN | `saturdaymorning-cdn` — https://saturdaymorning-assets-dya4gth9dnc0begj.z03.azurefd.net | ✅ Done |
| App Service env vars | See table below | ✅ Done |

## App Service Environment Variables (as configured)

| Variable | Value |
|---|---|
| `DATABASE_URL` | `postgresql://fivekadmin:<password>@saturdaymorning-db.postgres.database.azure.com:5432/iteration1?sslmode=require` |
| `AZURE_STORAGE_CONNECTION_STRING` | Connection string for `saturdaymorningstorage` (retrieve from portal — contains account key) |
| `NEXT_PUBLIC_API_URL` | `https://saturdaymorning.azurewebsites.net` |
| `NEXT_PUBLIC_CDN_URL` | `https://saturdaymorning-assets-dya4gth9dnc0begj.z03.azurefd.net` |
| `AUTH_TRUST_HOST` | `true` |
| `NODE_ENV` | `production` |
| `WEBSITE_NODE_DEFAULT_VERSION` | `~20` |
| `AUTH_SECRET` | ⚠️ **Not yet set** — must be added |
| `AUTH_GOOGLE_ID` | ⚠️ **Not yet set** — requires Google OAuth app |
| `AUTH_GOOGLE_SECRET` | ⚠️ **Not yet set** — requires Google OAuth app |
| `AUTH_FACEBOOK_ID` | ⚠️ **Not yet set** — requires Facebook OAuth app |
| `AUTH_FACEBOOK_SECRET` | ⚠️ **Not yet set** — requires Facebook OAuth app |

## Remaining Steps

### 1. Generate and set AUTH_SECRET

```powershell
# Generate the secret (requires openssl or use PowerShell alternative)
openssl rand -base64 32

# Set it on the App Service
az account set --subscription "68cfffbe-c761-4488-ab02-7a1b76786ae4"
az webapp config appsettings set `
  --resource-group fivekapp-rg `
  --name saturdaymorning `
  --settings "AUTH_SECRET=<paste-generated-value>"
```

### 2. Run Prisma database migrations

```powershell
cd C:\BMAD\fivekapp\webapp
$env:DATABASE_URL = "postgresql://fivekadmin:<password>@saturdaymorning-db.postgres.database.azure.com:5432/iteration1?sslmode=require"
pnpm exec prisma migrate deploy
```

### 3. Create Google and Facebook OAuth apps

**Google:**
1. Go to [Google Cloud Console](https://console.cloud.google.com) → **APIs & Services → Credentials**
2. Create an **OAuth 2.0 Client ID** (Web application)
3. Add authorised redirect URI: `https://saturdaymorning.azurewebsites.net/api/auth/callback/google`
4. Note the **Client ID** and **Client Secret**

**Facebook:**
1. Go to [Meta for Developers](https://developers.facebook.com) → **My Apps → Create App**
2. Add the **Facebook Login** product
3. Add Valid OAuth Redirect URI: `https://saturdaymorning.azurewebsites.net/api/auth/callback/facebook`
4. Note the **App ID** and **App Secret**

### 4. Add GitHub Secrets

Go to the GitHub repo → **Settings → Secrets and variables → Actions** and add:

| Secret | Value |
|---|---|
| `AZURE_WEBAPP_NAME` | `saturdaymorning` |
| `AZURE_WEBAPP_PUBLISH_PROFILE` | Download from portal: App Service → **Get publish profile** |
| `DATABASE_URL` | `postgresql://fivekadmin:<password>@saturdaymorning-db.postgres.database.azure.com:5432/iteration1?sslmode=require` |
| `NEXT_PUBLIC_CDN_URL` | `https://saturdaymorning-assets-dya4gth9dnc0begj.z03.azurefd.net` |
| `AUTH_GOOGLE_ID` | From step 3 (Google Client ID) |
| `AUTH_GOOGLE_SECRET` | From step 3 (Google Client Secret) |
| `AUTH_FACEBOOK_ID` | From step 3 (Facebook App ID) |
| `AUTH_FACEBOOK_SECRET` | From step 3 (Facebook App Secret) |

### 5. Deploy

Push to `main` — the GitHub Actions `deploy.yml` workflow will build and deploy automatically.

## Re-provisioning

To re-run the full provisioning script against this tenant:

```powershell
cd C:\BMAD\fivekapp\webapp
.\scripts\provision-azure.ps1 `
  -TenantId       "a7a18fe7-908e-4061-92d3-6a699da041fd" `
  -SubscriptionId "68cfffbe-c761-4488-ab02-7a1b76786ae4" `
  -AppName        "saturdaymorning"
```

> **Note:** The `Microsoft.DBforPostgreSQL` provider must be registered in the subscription before running.
> The provisioning script now handles this automatically (step 0/9).
