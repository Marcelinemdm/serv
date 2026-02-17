# Servidor pessoal

O servidor utiliza um fluxo de **tunel reverso**. A conexão é originada de dentro da rede para a Cloudflare, eliminando a necessidade de abertura de portas

---

## Componentes do Sistema

### 1. cloudflared

* **Função:** Cria um tunel seguro de saída, contorna o firewall da VIVO e gerencia o certificado SSL
* **Configuração:** `/etc/cloudflared/config.yml`
* **Vantagem:** O IP dinâmico não é problema; o tunel se reconecta automaticamente.

### 2. Server

* **Função:** Gerenciador central da biblioteca, extensões e progresso.
* **Porta Local:** `4567` (O tunel redireciona o tráfego da URL externa para esta porta).

### 3. Flaresolverr

* **Função:** Servidor proxy para burlar proteções antibot nas fontes.
* **Porta Local:** `8191`

---

## Manutenção

### Verificação de Status

Confirma se todos os "serviços" estão ativos:

```bash
sudo systemctl status cloudflared ****-server flaresolverr

```

### Verificação de Rede

Confirme se as portas internas estão em modo `LISTEN`:

```bash
ss -tulpn | grep -E '4567|8191'

```

### Logs em Tempo Real

Erros de conexão no tunel:

```bash
sudo journalctl -u cloudflared -f

```

---

## Problemas

### 404 / Timeout / Conflito de Porta

O **Caddy deve permanecer desativado**. O Cloudflare Tunnel já faz o proxy reverso. Se houver conflito nas portas 80/443:

```bash
sudo systemctl stop caddy
sudo systemctl disable caddy
sudo systemctl restart ****-server flaresolverr cloudflared

```

### Problemas com Capas ou Downloads

Se as capas não carregarem, reinicie o proxy de bypass:

```bash
sudo systemctl restart flaresolverr

```

---

## Atualizações

Manter os bin para evitar incompatibilida com extensões:

```bash
# Atualizar sistema e AUR
yay -Syu

# Reinstalar/Atualizar apenas o binário do tunel
yay -S cloudflared-bin

```

---

Para iniciar o servidor manualmente via CLI:

`cloudflared tunnel run su******`

## Diagrama de Arquitetura

```mermaid
graph LR
    subgraph Internet_Publica [Internet Pública]
        U[Usuário / App] ---|Acesso HTTPS| CF{Cloudflare Edge}
    end

    subgraph Rede_Local [Servidor Local / Host]
        CT[Cloudflare Tunnel] <--> CF
        CT <--> SU[Server :4567]
        SU <--> FS[FlareSolverr :8191]
    end

    subgraph Fontes_Externas [Web]
        FS ---|Bypass Anti-Bot| SC[fontes scraping]
    end

    style CF fill:#f60,stroke:#fff,color:#fff
    style CT fill:#059,stroke:#fff,color:#fff
    style SU fill:#333,stroke:#fff,color:#fff
    style FS fill:#636,stroke:#fff,color:#fff
