# Planka com Docker Compose

Este repositório sobe o **Planka** (Kanban) com **PostgreSQL** e **Redis** usando **Docker Compose**.

## Pré-requisitos

- Docker Desktop instalado e rodando
- Docker Compose (já vem com Docker Desktop)
- Porta `1337` livre na máquina

## Estrutura

- `docker-compose.yml`: define os serviços `planka`, `postgres` e `redis`
- `.env`: configurações (URL, chave secreta e credenciais do banco)

## Configuração (arquivo `.env`)

1. Crie (ou edite) o arquivo `.env` na raiz do projeto.
2. Use este modelo como base:

```env
# --- Configurações do Planka ---
BASE_URL=https://seu-dominio-ou-ip-aqui
SECRET_KEY=gere_uma_chave_longa_e_aleatoria_aqui
TRUST_PROXY=1

# --- Banco de Dados ---
POSTGRES_USER=planka_admin
POSTGRES_PASSWORD=uma_senha_forte_aqui
POSTGRES_DB=planka_db
```

### Como gerar um `SECRET_KEY` forte

Opção 1 (PowerShell):

```powershell
$bytes = New-Object byte[] 32
[Security.Cryptography.RandomNumberGenerator]::Create().GetBytes($bytes)
([Convert]::ToHexString($bytes)).ToLower()
```

Opção 2 (OpenSSL, se instalado):

```bash
openssl rand -hex 32
```

## Subir os serviços (passo a passo)

No diretório do projeto:

1. Suba os containers:

```bash
docker compose up -d
```

2. Verifique se os serviços estão de pé:

```bash
docker compose ps
```

3. Acompanhe logs do Planka (se precisar):

```bash
docker compose logs -f planka
```

4. Acesse o Planka:

- URL local: `http://localhost:1337`
- Em produção: use o valor configurado em `BASE_URL`

## Parar / reiniciar

Parar:

```bash
docker compose down
```

Parar mantendo dados (volumes) intactos:

```bash
docker compose down
```

Parar e remover volumes (apaga dados do banco e uploads):

```bash
docker compose down -v
```

Reiniciar:

```bash
docker compose restart
```

## Atualizar a versão das imagens

1. Baixe imagens mais recentes:

```bash
docker compose pull
```

2. Recrie os containers:

```bash
docker compose up -d
```

## Backup e restore do banco (PostgreSQL)

### Backup (gera um arquivo `.sql`)

```bash
docker compose exec -T postgres pg_dump -U "$POSTGRES_USER" -d "$POSTGRES_DB" > backup_planka.sql
```

Se o comando acima não expandir as variáveis no seu shell, use diretamente os valores do `.env`:

```bash
docker compose exec -T postgres pg_dump -U planka_admin -d planka_db > backup_planka.sql
```

### Restore (importa um `.sql`)

```bash
docker compose exec -T postgres psql -U "$POSTGRES_USER" -d "$POSTGRES_DB" < backup_planka.sql
```

Ou (com valores fixos):

```bash
docker compose exec -T postgres psql -U planka_admin -d planka_db < backup_planka.sql
```

## Troubleshooting

### Porta 1337 já está em uso

- Troque o mapeamento de portas no `docker-compose.yml`:
  - de: `1337:1337`
  - para: `1338:1337`

Depois:

```bash
docker compose up -d
```

### Verificar saúde dos serviços

```bash
docker inspect --format='{{json .State.Health}}' planka_app
docker inspect --format='{{json .State.Health}}' planka_db
docker inspect --format='{{json .State.Health}}' planka_redis
```

### Resetar ambiente (cuidado: apaga dados)

```bash
docker compose down -v
docker compose up -d
```

## Segurança

- Não versionar o `.env` com senhas/chaves reais em repositórios públicos.
- Use uma `SECRET_KEY` forte e única por ambiente.
- Se estiver atrás de proxy reverso (Nginx/Traefik/Cloudflare), mantenha `BASE_URL` com HTTPS e garanta headers corretos.
