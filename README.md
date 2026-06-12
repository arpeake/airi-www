# airi-www

Coming-soon landing page for [airi-usa.com](https://airi-usa.com). Static HTML deployed to Azure
Static Web Apps (Free tier, $0/mo). Seed for the future AIRI marketing and signup site.

## What this is

- Bare apex `airi-usa.com` -- AIRI logo, "Coming soon" headline, Login button (top-right) linking to
  the live prod app at `https://app.airi-usa.com`.
- No build step. Plain static files in `src/` deployed as-is via GitHub Actions + Azure SWA.
- Style: no em-dashes or en-dashes (ASCII `-` or ` -- `), US English everywhere.

## Azure setup -- ALREADY DONE

The Azure resource, deployment token, and GitHub Actions secret are all provisioned. The page is
live at the default hostname:

**Default hostname (live now):** `https://kind-rock-0f1ab130f.7.azurestaticapps.net`

| Resource | Value |
|---|---|
| Resource group | `rg-airi-web-eus2` |
| SWA name | `stapp-airi-www` |
| Subscription | `AIRI` (`7e47d097-3621-4136-98e3-faf139c05737`) |
| GitHub secret | `AZURE_STATIC_WEB_APPS_API_TOKEN` -- set |

## DNS records -- operator action required

The only remaining step is adding DNS records at your registrar and confirming so the custom
domain binding can be finalized. Cutover order matters:

1. Add the `www` CNAME below. Wait for DNS to propagate (~minutes to an hour).
2. Confirm `https://app.airi-usa.com` login is reachable (the Login button target).
3. Then add the apex record to cut `airi-usa.com` off the legacy host (`108.22.241.42`).
4. Let the team know DNS is in place -- the domain binding (cert issuance) can then be finalized.

### Exact records to add

Do NOT touch existing records: `app`, `app-dev`, `eus`, `eus-dev` -- those are the live platform.

| Host | Type | Value | TTL | Notes |
|---|---|---|---|---|
| `www` | CNAME | `kind-rock-0f1ab130f.7.azurestaticapps.net` | 3600 | points www to the SWA |
| `@` | ALIAS / ANAME | `kind-rock-0f1ab130f.7.azurestaticapps.net` | 3600 | apex -- only if registrar supports ALIAS/ANAME/CNAME-flattening |

**If the registrar does not support ALIAS/ANAME at the apex** (DNS forbids a bare CNAME at `@`):
use the registrar's URL-forward/redirect to send `airi-usa.com` -> `https://www.airi-usa.com`
(301). Content lives on `www`; the apex redirects there. To check, look for an ALIAS, ANAME, or
"CNAME-flattening" option at the root (`@`) in your registrar's DNS manager.

After DNS is set, the `az staticwebapp hostname set` commands are run to bind the custom domains
and trigger free managed-cert issuance. TXT validation tokens (if the registrar needs them) will
be provided at that point from the portal.

## Ongoing deploys

Push to `main` -- the `azure-static-web-apps.yml` workflow builds (no-op, skip_app_build=true) and
deploys to `stapp-airi-www`. PRs automatically get free preview environments on a staging URL.

## Future: NinjaOne-style marketing site

When ready to expand: add pages alongside `index.html` in `src/`, add a free SWA managed Functions
API for lead capture/signup forms under `api/`, and reuse the brand tokens in `styles.css`. No
re-platform needed -- SWA Free already supports it.
