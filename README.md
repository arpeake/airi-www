# airi-www

Coming-soon landing page for [airi-usa.com](https://airi-usa.com). Static HTML deployed to Azure
Static Web Apps (Free tier, $0/mo). Seed for the future AIRI marketing and signup site.

## What this is

- Bare apex `airi-usa.com` -- AIRI logo, "Coming soon" headline, Login button (top-right) linking to
  the live prod app at `https://app.airi-usa.com`.
- No build step. Plain static files in `src/` deployed as-is via GitHub Actions + Azure SWA.
- Style: no em-dashes or en-dashes (ASCII `-` or ` -- `), US English everywhere.

## One-time Azure setup (operator, do this before the first deploy works)

### 1. Create the Azure resource

In the Azure portal (or `az` CLI under your personal login -- the CI identity `id-airi-cicd` cannot
create resources in a new resource group):

| Field | Value |
|---|---|
| Subscription | `AIRI` (`7e47d097-3621-4136-98e3-faf139c05737`) |
| Resource group | `rg-airi-web-eus2` (create new) |
| Name | `stapp-airi-www` |
| Region | East US 2 |
| Plan | Free |
| Deployment source | Other (you will link GitHub manually) |

After creation, the portal shows the **default hostname** -- looks like
`<random-name>.azurestaticapps.net`. Copy it; you need it for DNS and for the custom domain binding.

### 2. Get the deployment token

In the portal: `stapp-airi-www` -> Overview -> Manage deployment token. Copy the token.

### 3. Add the token to this repo

GitHub repo `arpeake/airi-www` -> Settings -> Secrets and variables -> Actions -> New repository
secret:

| Name | Value |
|---|---|
| `AZURE_STATIC_WEB_APPS_API_TOKEN` | (the token from step 2) |

Push anything to `main` to trigger the first deploy and verify the default hostname serves the page.

## DNS records (apply after verifying the default hostname works)

Cutover sequence matters -- do it in this order to avoid any gap:

1. Stand up SWA and verify `<random>.azurestaticapps.net` serves the page.
2. Bind `www.airi-usa.com` in the portal + apply the `www` records below. Wait for the managed cert
   to issue and verify `https://www.airi-usa.com` loads.
3. Confirm `https://app.airi-usa.com` login is live (the Login button's target).
4. Then apply the apex records to cut `airi-usa.com` off the legacy host (`108.22.241.42`).

### Records to add at your registrar

Replace `<swa-hostname>` with the actual default hostname from step 1 above.
Replace `<www-token>` and `<apex-token>` with the validation strings shown in the SWA portal when
you add each custom domain (Azure -> Static Web App -> Custom domains -> + Add).

| Host | Type | Value | TTL | Notes |
|---|---|---|---|---|
| `www` | CNAME | `<swa-hostname>.azurestaticapps.net` | 3600 | content host |
| `_dnsauth.www` | TXT | `<www-token>` | 3600 | SWA domain validation (exact host shown in portal) |
| `@` | ALIAS / ANAME | `<swa-hostname>.azurestaticapps.net` | 3600 | apex -- only if registrar supports ALIAS/ANAME/CNAME-flattening |
| `@` | TXT | `<apex-token>` | 3600 | SWA apex validation |

**If your registrar does not support ALIAS/ANAME on the apex** (many do not -- DNS forbids a bare
CNAME at `@`): use the registrar's URL-forward/redirect feature to send `airi-usa.com` -> `https://www.airi-usa.com` (HTTP 301). Content lives on `www`; the apex redirects there. Both paths
end up on the same SWA page with a valid TLS cert.

To check which applies, run: `nslookup -type=ns airi-usa.com` and look up whether your registrar's
DNS manager has an ALIAS, ANAME, or CNAME-flattening option at the root (`@`).

Do NOT touch the existing records: `app`, `app-dev`, `eus`, `eus-dev` -- those are the live platform.

## Ongoing deploys

Push to `main` -- the `azure-static-web-apps.yml` workflow builds (no-op, skip_app_build=true) and
deploys to `stapp-airi-www`. PRs automatically get free preview environments on a staging URL.

## Future: NinjaOne-style marketing site

When ready to expand: add pages alongside `index.html` in `src/`, add a free SWA managed Functions
API for lead capture/signup forms under `api/`, and reuse the brand tokens in `styles.css`. No
re-platform needed -- SWA Free already supports it.
