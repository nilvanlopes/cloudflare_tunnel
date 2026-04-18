# Cloudflare Tunnel with Docker

Este diretório contém os arquivos necessários para rodar um Cloudflare Tunnel via Docker, sem instalar `cloudflared` na máquina.

## Pré-requisitos

- Docker
- Conta na Cloudflare
- Domínio gerenciado pela Cloudflare
- Host com suporte aos `sysctl` `net.core.rmem_max` e `net.core.wmem_max`

## Ajustes de kernel usados na stack

O serviço `cloudflared` no [docker-compose.yml](/mnt/d/docker/cloudflare_tunnel/docker-compose.yml) define:

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
docker run --rm -i \
  -v "$(pwd)/.cloudflared:/home/nonroot/.cloudflared" \
  cloudflare/cloudflared:latest \
  tunnel create traefik_swarm_routing
```

Esse comando:

- cria o tunnel `traefik_swarm_routing`
- retorna o UUID do tunnel
- gera o arquivo `./.cloudflared/<TUNNEL_UUID>.json`

## Tunnel criado neste diretório

O tunnel criado foi:

- Nome: `traefik_swarm_routing`
- UUID: `3647717f-6246-4d24-b5b8-1319be027c80`
- Domínio: `nilvanlopes.com`

O arquivo de credenciais gerado foi:

- `./.cloudflared/3647717f-6246-4d24-b5b8-1319be027c80.json`

## Configuração

O arquivo [config.yml](/mnt/d/docker/cloudflare_tunnel/config.yml) foi configurado para:

- usar o tunnel `3647717f-6246-4d24-b5b8-1319be027c80`
- usar o arquivo de credenciais em `/etc/cloudflared/3647717f-6246-4d24-b5b8-1319be027c80.json`
- encaminhar `*.nilvanlopes.com` para `https://traefik.nilvanlopes.com:443`

Exemplo atual:

```yaml
tunnel: 3647717f-6246-4d24-b5b8-1319be027c80
credentials-file: /etc/cloudflared/3647717f-6246-4d24-b5b8-1319be027c80.json

ingress:
  - hostname: "*.nilvanlopes.com"
    service: https://traefik.nilvanlopes.com:443
  - service: http_status:404
```

## Observação importante

Neste fluxo, a Cloudflare não gera um arquivo chamado `credentials.json`. O arquivo real de credenciais é nomeado com o UUID do tunnel:

- `<TUNNEL_UUID>.json`

Se quiser usar o nome `credentials.json`, isso exigiria ajustar o `docker-compose.yml` para montar ou renomear esse arquivo. Com a configuração atual, o recomendado é usar o nome original gerado pelo `cloudflared`.
