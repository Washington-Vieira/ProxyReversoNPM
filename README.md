# Proxy Reverso — Nginx Proxy Manager (NPM)

Setup de proxy reverso com [Nginx Proxy Manager](https://nginxproxymanager.com/guide/) via Docker Compose, para rodar numa VPS recebendo e direcionando tráfego HTTP/HTTPS.

## Pré-requisitos na VPS

- Docker + Docker Compose instalados
- Portas **80**, **81** e **443** liberadas no firewall da VPS / provedor
- (Para SSL) Um domínio apontando (registro **A**) para o IP da VPS

Instalação rápida do Docker (Ubuntu/Debian):

```bash
curl -fsSL https://get.docker.com | sh
```

## Subir o serviço

Dentro da pasta onde está o `docker-compose.yml`:

```bash
docker compose up -d
```

Verificar se subiu:

```bash
docker compose ps
docker compose logs -f npm
```

As pastas `data/` e `letsencrypt/` são criadas automaticamente e guardam o banco (SQLite), os hosts configurados e os certificados. **Faça backup delas.**

## Primeiro acesso ao painel

1. Abra no navegador: `http://IP_DA_VPS:81`
2. Login padrão inicial:
   - **E-mail:** `admin@example.com`
   - **Senha:** `changeme`
3. O sistema vai **obrigar a trocar** o e-mail e a senha no primeiro login. Faça isso.

## Criar um Proxy Host (direcionar tráfego)

No painel: **Hosts → Proxy Hosts → Add Proxy Host**

- **Domain Names:** `seu-dominio.com` (e `www.seu-dominio.com` se quiser)
- **Scheme:** `http` (ou `https` se o serviço de destino já usa TLS)
- **Forward Hostname / IP:** IP ou nome do serviço de destino
  - Outro container na mesma rede Docker → use o **nome do container**
  - Serviço rodando na própria VPS → use o IP interno (ex.: `172.17.0.1`) ou o IP da LAN
- **Forward Port:** porta do serviço (ex.: `3000`, `8080`)
- Ative **Block Common Exploits** e, se for app web, **Websockets Support**

### SSL automático (Let's Encrypt)

Na aba **SSL** do host:

- **SSL Certificate:** `Request a new SSL Certificate`
- Marque **Force SSL** e **HTTP/2 Support**
- Marque **I Agree to the Let's Encrypt Terms**
- Salve — o certificado é emitido e renovado automaticamente

> O DNS do domínio precisa já estar apontando para a VPS, senão a emissão falha.

## Comandos úteis

```bash
docker compose up -d        # subir / aplicar mudanças
docker compose down         # parar e remover o container (dados ficam nos volumes)
docker compose pull         # baixar nova versão da imagem
docker compose up -d        # recriar com a versão nova
docker compose logs -f npm  # ver logs em tempo real
```

## Atualizar

```bash
docker compose pull && docker compose up -d
```

## Dicas de rede entre containers

Se os apps que você quer expor também rodam em Docker, o mais limpo é colocá-los na **mesma rede** do NPM. Exemplo de rede externa compartilhada:

```bash
docker network create proxy
```

E referenciar essa rede nos dois `docker-compose.yml` (o do NPM e o dos apps) como `external: true`. Aí no Proxy Host você usa o nome do container como *Forward Hostname*.
# ProxyReversoNPM
