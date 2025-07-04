# Deploy Postiz on Coolify

This repository provides a battle-tested `docker-compose.yml` file specifically configured for deploying the [Postiz](https://postiz.com/) application on a [Coolify v4](https://coolify.io/) instance.

This configuration solves common proxy and environment variable issues by using Coolify's native features correctly.

## Prerequisites

*   A running Coolify v4 instance.
*   A domain name (e.g., `postiz.your-domain.com`) with its DNS A record pointing to your Coolify server's IP address.
*   Your GitHub account connected to your Coolify instance.

## Deployment Steps

1.  **Fork this Repository:** Fork this repository to your own GitHub account so that Coolify can access it.

2.  **Create a New Resource in Coolify:**
    *   Log in to your Coolify dashboard and navigate to the project you want to deploy to.
    *   Click **"Add Resource"** and select **"Docker Compose"**.

3.  **Configure the Resource:**
    *   Choose the **"From a Git repository"** option.
    *   Select your forked repository and the correct branch (e.g., `main`).
    *   Coolify will load the `docker-compose.yml` from the repository.

4.  **Set Your Domain:**
    *   Navigate to the **"General"** tab for the new resource.
    *   In the **FQDN(s)** field, enter your full public domain, for example: `https://postiz.criticalspects.com`.

5.  **Deploy:**
    *   Review the **Secrets** tab. Coolify will automatically detect and generate secure passwords for `JWTSECRET`, `postgres`, and `redis`. You do not need to add these manually.
    *   Click **"Save"** and then **"Deploy"**.

Coolify will now build and deploy your Postiz instance. Once complete, it will be available at the domain you configured.

## How It Works: Key Configuration Details

This `docker-compose.yml` is not a standard file; it's tailored for Coolify's infrastructure. Here’s why it works:

### 1. Coolify Environment Variables

The configuration uses special variables that Coolify automatically replaces during deployment:
*   `${SERVICE_FQDN_postiz}`: Coolify injects the domain you set in the **FQDN(s)** field.
*   `${SERVICE_PASSWORD_postgres}`: Coolify generates and injects a secure password for the PostgreSQL database.
*   `${SERVICE_PASSWORD_redis}`: Coolify generates and injects a secure password for Redis.

**Note:** The variable names (`postiz`, `postgres`, `redis`) must be **lowercase** and match the service names defined in the `docker-compose.yml` file.

### 2. Traefik Proxy Labels

The `labels` section in the `postiz` service provides explicit instructions to Coolify's internal reverse proxy (Traefik).

```yaml
    labels:
      - "traefik.enable=true"
      # Explicitly define a router for API traffic on the /api path
      - "traefik.http.routers.postiz-api.rule=Host(`${SERVICE_FQDN_postiz}`) && PathPrefix(`/api`)"
      - "traefik.http.routers.postiz-api.service=postiz-svc"
      # Define a router for all other traffic
      - "traefik.http.routers.postiz-app.rule=Host(`${SERVICE_FQDN_postiz}`)"
      - "traefik.http.routers.postiz-app.service=postiz-svc"
      # Define the service that both routers will use
      - "traefik.http.services.postiz-svc.loadbalancer.server.port=5000"
```

This configuration solves the `404 Not Found` error on API routes (like `/api/auth/register`) by ensuring that the proxy does not strip the `/api` path from the URL before forwarding it to the Postiz application.

---
