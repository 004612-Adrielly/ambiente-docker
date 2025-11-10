# Ambiente Docker para projetos Laravel

Este setup executa qualquer projeto Laravel montado dentro de um container **PHP 8.2 + Apache**.  
O container é construído pelo `Dockerfile` localizado em `.docker/php/Dockerfile`, que instala extensões comuns de Laravel, drivers do SQL Server (`sqlsrv`/`pdo_sqlsrv`) e Composer.

## Estrutura relevante

```
.
├── .docker
│   ├── apache
│   │   └── vhost.conf
│   └── php
│       └── Dockerfile
├── docker-compose.yml
├── seu-projeto-laravel
└── (outras pastas)
```

- A pasta do seu projeto Laravel é montada dentro do container em `/var/www/html` (por padrão usamos `./seu-projeto-laravel:/var/www/html` no `docker-compose.yml`).
- `vhost.conf` substitui a configuração padrão do Apache para apontar o DocumentRoot correto (Laravel `public/`).
- Certifique-se de que a pasta com os arquivos de configuração se chama `.docker`. Se o projeto vier com uma pasta `docker`, renomeie para `.docker` ou ajuste os caminhos em `docker-compose.yml`.

---

## Requisitos

- Docker Desktop (ou engine compatível) instalado e em execução.
- Git (opcional, se o projeto já estiver baixado).
- Acesso a internet para baixar imagens e dependências.

---

## Versões utilizadas

- Docker Compose v2+
- Imagem base: `php:8.2.4-apache` (Debian 12/bookworm)
- Apache 2.4.56
- PHP 8.2.4 com extensões: `gd`, `intl`, `zip`, `bcmath`, `mysqli`, `pdo_mysql`, `pdo_sqlsrv`, `sqlsrv`
- Microsoft ODBC Driver 18 (`msodbcsql18`) e `mssql-tools18`
- Composer 2.x (copiado da imagem oficial `composer:2`)
- Laravel conforme o `composer.json` do projeto que você montar em `/var/www/html`

---

## Subindo o ambiente

1. Abra um terminal na raiz do repositório (substitua pelo caminho em sua máquina):

   ```powershell
   cd /caminho/para/ancora-proweb
   ```

2. Construa e suba o container:

   ```powershell
   docker compose up -d --build
   ```

   - O serviço expõe o Apache em `http://localhost:8090`.
   - O primeiro build pode demorar porque instala extensões PHP e ferramentas do SQL Server.

3. (Opcional) Acompanhe os logs:

   ```powershell
   docker compose logs -f app
   ```

---

## Dependências PHP

Como o volume mapeia seu projeto Laravel, rode o Composer diretamente dentro do container para garantir a mesma versão de PHP/extensões:

```powershell
docker compose run --rm app composer install
```

Outros comandos úteis:

```powershell
docker compose exec app php artisan migrate
docker compose exec app php artisan config:cache
```

---

## Testando o acesso

- A aplicação estará disponível em `http://localhost:8090`.
- Se ainda não carregar:
  - Confirme se o container está rodando (`docker compose ps`).
  - Verifique logs de aplicação (`storage/logs/laravel.log`) via `docker compose exec app tail -f storage/logs/laravel.log`.
  - Confira se as dependências foram instaladas (`vendor/`).

---

## Encerrando

```powershell
docker compose down        # Para o container
docker compose down -v     # Para e remove volumes nomeados (cuidado com dados)
```

---

## Observações

- O atributo `version` no `docker-compose.yml` foi removido porque está obsoleto nas versões atuais do Compose.
- Os avisos do Apache sobre `ServerName` podem ser ignorados ou ajustados adicionando `ServerName localhost` em `.docker/apache/vhost.conf`.
- Ajuste permissões no container se necessário:

  ```powershell
  docker compose exec app bash -lc "chmod -R 775 storage bootstrap/cache"
  ```
