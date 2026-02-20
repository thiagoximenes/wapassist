# 01 — Infraestrutura: VPS + Evolution API (WhatsApp)

> **Objetivo:** Configurar a VPS com Ubuntu 22.04, Docker, Nginx, SSL e a Evolution API para envio de mensagens WhatsApp.  
> **Tempo estimado:** 2–3 horas  
> **Pré-requisito:** Domínio `wapassist.com.br` com acesso ao painel DNS.

---

## Visão Geral da Infraestrutura

```
Internet
    │
    ▼
[DNS: api.wapassist.com.br → IP da VPS]
    │
    ▼
[VPS Ubuntu 22.04]
    │
    ├── Nginx (porta 80/443) — proxy reverso + SSL (Certbot)
    │       │
    │       └── Evolution API (Docker, porta 8080)
    │               │
    │               └── WhatsApp conectado via QR Code
    │
    └── Firewall UFW (libera 22, 80, 443)
```

A VPS hospeda **somente** a Evolution API. O backend e o frontend ficam em serviços gratuitos (Render + Vercel).

---

## Etapa 1.1 — Contratar a VPS

### Opções recomendadas

| Provedor | Plano | Custo | Diferencial |
|---|---|---|---|
| **Hostinger** ⭐ | KVM 1 (1 vCPU / 2 GB RAM) | ~R$ 20/mês | Suporte PT-BR, datacenter SP |
| **Hetzner** | CX11 (1 vCPU / 2 GB RAM) | ~R$ 18/mês | Melhor custo-benefício, datacenter EU |

**Requisitos obrigatórios ao criar:**
- Sistema operacional: **Ubuntu 22.04 LTS**
- RAM mínima: **2 GB**

### Checklist

- [ ] VPS contratada com Ubuntu 22.04 LTS e mínimo 2 GB RAM
- [ ] IP público da VPS anotado em local seguro
- [ ] Acesso SSH funcionando: `ssh root@IP_DA_VPS`

---

## Etapa 1.2 — Apontar DNS

No painel do domínio `wapassist.com.br`, criar o registro:

| Tipo | Nome | Valor | TTL |
|---|---|---|---|
| A | `api` | `IP_DA_SUA_VPS` | 300 |

> ⚠️ Aguarde 10 minutos para propagar. Verifique em [dnschecker.org](https://dnschecker.org) antes de prosseguir para o SSL.

### Checklist

- [ ] Registro DNS tipo A criado: `api.wapassist.com.br → IP_DA_VPS`
- [ ] Propagação confirmada no dnschecker.org

---

## Etapa 1.3 — Configurar o Servidor

Conecte via SSH e execute os comandos **em ordem**:

### 1. Atualizar sistema e instalar base

```bash
apt update && apt upgrade -y
apt install -y curl git ufw nano
```

### 2. Configurar Firewall

```bash
ufw allow OpenSSH
ufw allow 80/tcp
ufw allow 443/tcp
ufw enable
```

### 3. Instalar Docker

```bash
curl -fsSL https://get.docker.com | sh
usermod -aG docker $USER && newgrp docker
```

### 4. Instalar Nginx + Certbot

```bash
apt install -y nginx certbot python3-certbot-nginx
systemctl enable nginx && systemctl start nginx
```

### Checklist

- [ ] Sistema atualizado sem erros
- [ ] Firewall UFW ativo (portas 22, 80, 443 liberadas)
- [ ] Docker instalado: `docker --version` retorna versão
- [ ] Nginx instalado e rodando: `systemctl status nginx` mostra `active`

---

## Etapa 1.4 — Subir a Evolution API com Docker

```bash
mkdir -p ~/evolution && cd ~/evolution
nano docker-compose.yml
```

Cole o conteúdo abaixo no arquivo:

```yaml
version: '3.8'
services:
  evolution-api:
    image: atendai/evolution-api:latest
    container_name: evolution_api
    restart: always
    ports:
      - "8080:8080"
    environment:
      AUTHENTICATION_TYPE: apikey
      AUTHENTICATION_API_KEY: "GERE_COM_openssl_rand_-hex_32"
      AUTHENTICATION_EXPOSE_IN_FETCH_INSTANCES: "true"
      DEL_INSTANCE: "false"
      DATABASE_ENABLED: "false"
    volumes:
      - evolution_instances:/evolution/instances
      - evolution_store:/evolution/store
volumes:
  evolution_instances:
  evolution_store:
```

> ⚠️ **Gere a API Key antes de salvar:** `openssl rand -hex 32`  
> Salve essa chave — ela vai para `EVOLUTION_APIKEY` no `.env` do backend.

```bash
docker compose up -d
docker logs evolution_api --tail 20
```

✅ O log deve exibir: `Server is listening on port 8080`

### Checklist

- [ ] Arquivo `docker-compose.yml` criado com API Key gerada
- [ ] Container `evolution_api` rodando: `docker ps` mostra o container
- [ ] Log confirma: `Server is listening on port 8080`

---

## Etapa 1.5 — Configurar Nginx e SSL

```bash
nano /etc/nginx/sites-available/evolution
```

Cole o conteúdo:

```nginx
server {
    listen 80;
    server_name api.wapassist.com.br;
    location / {
        proxy_pass http://localhost:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 120s;
    }
}
```

```bash
ln -s /etc/nginx/sites-available/evolution /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx
certbot --nginx -d api.wapassist.com.br --email SEU@EMAIL.COM --agree-tos --non-interactive
```

### Checklist

- [ ] Nginx configurado como proxy reverso para porta 8080
- [ ] `nginx -t` retorna `syntax is ok`
- [ ] SSL ativo: `https://api.wapassist.com.br` abre sem alertas de segurança
- [ ] Certbot configurado para renovação automática

---

## Etapa 1.6 — Conectar WhatsApp via QR Code

> ⚠️ **Use um número dedicado para o wapassist — NÃO o seu número pessoal.**

### Criar a instância

```bash
curl -X POST https://api.wapassist.com.br/instance/create \
  -H "Content-Type: application/json" \
  -H "apikey: SUA_CHAVE" \
  -d '{"instanceName": "wapassist", "qrcode": true}'
```

### Obter o QR Code

```bash
curl -X GET "https://api.wapassist.com.br/instance/connect/wapassist" \
  -H "apikey: SUA_CHAVE"
```

A resposta contém um campo `base64` com a imagem do QR Code.

1. Acesse [base64.guru/converter/decode/image](https://base64.guru/converter/decode/image)
2. Cole o valor `base64`
3. Escaneie com o WhatsApp do número dedicado

### Verificar conexão

```bash
curl -X GET "https://api.wapassist.com.br/instance/fetchInstances" \
  -H "apikey: SUA_CHAVE"
```

O campo `state` deve retornar `open`.

### Enviar mensagem de teste

```bash
curl -X POST "https://api.wapassist.com.br/message/sendText/wapassist" \
  -H "Content-Type: application/json" \
  -H "apikey: SUA_CHAVE" \
  -d '{"number": "5521999998888", "text": "Teste wapassist funcionando!"}'
```

### Checklist

- [ ] Instância `wapassist` criada na Evolution API
- [ ] QR Code escaneado com número dedicado
- [ ] Status da instância: `open`
- [ ] Mensagem de teste recebida no WhatsApp

---

## Manutenção da Instância WhatsApp

| Problema | Causa | Solução |
|---|---|---|
| Instância desconecta | Celular sem bateria ou internet | Manter celular carregado e conectado |
| Instância desconecta | WhatsApp Web aberto simultaneamente | Não usar o mesmo número no WhatsApp Web |
| Container para | Reinicialização da VPS | `restart: always` no docker-compose garante reinício automático |

### Reescanear QR Code (quando necessário)

```bash
curl -X DELETE "https://api.wapassist.com.br/instance/logout/wapassist" \
  -H "apikey: SUA_CHAVE"
# Depois repita o processo da Etapa 1.6
```
