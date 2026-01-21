# Cloudflare Tunnel with Docker

This project sets up a Cloudflare Tunnel using Docker to expose a local service to the internet.

## Prerequisites

- Docker
- Docker Compose
- A Cloudflare account
- A domain managed by Cloudflare

## Setup

1.  **Create a Cloudflare Tunnel:** Follow the [Cloudflare documentation](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-run/create-tunnel) to create a tunnel and get your `TUNNEL_ID`.

2.  **Create `credentials.json`:** Download the credentials file for your tunnel and save it as `credentials.json` in this directory.

3.  **Create `cert.pem`:** If your service uses a self-signed certificate, you will need to provide the certificate to Cloudflare. Download your origin certificate from the Cloudflare dashboard and save it as `cert.pem`.

4.  **Configure `config.yml`:** Create a `config.yml` file based on the `config.example.yml`:

    ```yaml
    # config.yml
    tunnel: <TUNNEL_ID>
    credentials-file: /etc/cloudflared/credentials.json
    ingress:
      # Wildcard for all subdomains
      - hostname: "*.example.com"
        service: https://traefik.example.com:443


      # Mandatory catch-all rule (always last)
      - service: http_status:404
    ```

    Replace `<TUNNEL_ID>` with your tunnel ID and `*.example.com` and `https://traefik.example.com:443` with your domain and service.

5.  **Start the tunnel:**

    ```bash
    docker-compose up -d
    ```

## `docker-compose.yml`

The `docker-compose.yml` file defines the `cloudflared` service. It mounts the `credentials.json`, `cert.pem` and `config.yml` files into the container.

**Note:** The `traefik-public` network is an external network. You may need to create it or change it to your own network.

## `config.yml`

The `config.yml` file is the configuration for the Cloudflare Tunnel. See the [Cloudflare documentation](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/configuration/configuration-file) for more information.
