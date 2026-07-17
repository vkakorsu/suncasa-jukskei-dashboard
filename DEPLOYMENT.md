# Deployment Guide

This guide explains how to build the Johannesburg Nature-based Solutions Impact Dashboard and deploy it within a Microsoft compatible environment. The site is a static build, so it can be hosted on City of Johannesburg infrastructure or Azure with no server runtime.

- **Recommended for the City of Johannesburg:** Azure Static Web Apps (Microsoft compatible)
- **Portable or on premises:** Docker image behind nginx

The production build output is always the `dist/` folder, created by `npm run build`.

## 0. Prerequisites

- Node.js 20 or newer and npm 10 or newer
- The project installed and building locally:

```bash
npm install
npm run build      # outputs static files to dist/
npm run preview    # optional, preview the production build locally
```

If `npm run build` succeeds and `npm run preview` shows the site, you are ready to deploy.

---

## Option A. Azure Static Web Apps (recommended for the City)

This is the Microsoft compatible path and the natural fit for City of Johannesburg infrastructure. A `staticwebapp.config.json` is already included for routing, headers, and the fallback page.

### A1. Using the Azure Portal and GitHub
1. Push this `prototype/` folder to a GitHub repository.
2. In the Azure Portal, create a resource, then choose Static Web App.
3. Sign in and select the repository and branch.
4. For build details set:
   - App location: `/` (or `/prototype` if the repo root contains more than this folder)
   - Api location: leave blank
   - Output location: `dist`
5. Create. Azure adds a GitHub Actions workflow and builds the site. When it finishes you get a URL such as `https://your-app.azurestaticapps.net`.

### A2. Using the Azure CLI (Static Web Apps CLI)
```bash
npm install -g @azure/static-web-apps-cli
npm run build
swa deploy ./dist --env production
```
Follow the prompts to sign in and select or create the Static Web App. The CLI prints the live URL when done.

### Custom domain (optional)
In the Static Web App resource, open Custom domains and add the City domain, then follow the DNS instructions. TLS is provisioned automatically.

---

## Option B. Docker (portable or on premises)

Use this to run the dashboard on Azure App Service, Azure Container Apps, or City data centre infrastructure with no code changes. A `Dockerfile` and `nginx.conf` are included.

```bash
# Build the image
docker build -t suncasa-jukskei-dashboard .

# Run it locally on http://localhost:8080
docker run --rm -p 8080:80 suncasa-jukskei-dashboard
```

To publish, push the image to a registry the City uses, for example Azure Container Registry:

```bash
az acr login --name <registryName>
docker tag suncasa-jukskei-dashboard <registryName>.azurecr.io/suncasa-jukskei-dashboard:latest
docker push <registryName>.azurecr.io/suncasa-jukskei-dashboard:latest
```

Then create an Azure Container App or Web App for Containers that pulls this image. The container serves the site on port 80.

---

## Updating the data before a deploy

If you change figures in `data/`, regenerate the typed JSON and rebuild before deploying:

```bash
npm run ingest
npm run build
```

## Troubleshooting

- **Blank map area on first load.** The map is a client only component and appears a moment after the page loads. This is expected and keeps the base page light.
- **404 on a refreshed sub page.** Make sure the output or publish directory is set to `dist`. The included Azure and nginx configs already handle fallback routing.
- **Build fails on the host.** Confirm the host uses Node.js 20 or newer and the build command `npm run build`.
