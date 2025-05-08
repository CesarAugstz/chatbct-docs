# Guia de Instalação e Configuração: ChatBCT

Referência: [SSH Login](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys#logging-in-to-a-server-with-a-different-port)

## 1. Login no Servidor

### Conexão Inicial via SSH

Para se conectar a um servidor SSH utilizando uma porta específica, use o seguinte comando:

```bash
ssh -p port_num username@remote_host
```

---

## 2. Instalação do Docker

Referência: [Instalação do Docker](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#step-1-installing-docker)

### Passos de Instalação

1. **Atualizar repositórios**

   ```bash
   sudo apt update
   ```

2. **Instalar pacotes de pré-requisito**

   ```bash
   sudo apt install apt-transport-https ca-certificates curl software-properties-common
   ```

3. **Adicionar chave GPG do Docker**

   ```bash
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   ```

4. **Adicionar repositório Docker**

   ```bash
   sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
   ```

5. **Instalar Docker**

   ```bash
   sudo apt install docker-ce
   ```

6. **Verificar instalação**

   ```bash
   sudo systemctl status docker
   ```

---

## 3. Instalação do Docker Compose

1. **Baixar o binário mais recente**

   ```sh
   sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   ```

2. **Definir permissões corretas**

   ```sh
   sudo chmod +x /usr/local/bin/docker-compose
   ```

3. **Verificar instalação**

   ```sh
   docker-compose --version
   ```

---

## 4. Instalação e Configuração do Chatwoot com Docker Compose

Referência: [Docker Chatwoot](https://chatwoot.com/docs/self-hosted/deployment/docker/)

### Passos:

1. **Baixar os arquivos necessários**:

   ```bash
   wget -O .env https://raw.githubusercontent.com/chatwoot/chatwoot/develop/.env.example
   wget -O docker-compose.yaml https://raw.githubusercontent.com/chatwoot/chatwoot/develop/docker-compose.production.yaml
   ```

2. **Editar o arquivo `.env`**:

   ```bash
   vi .env
   ```

   Adicione ou edite a senha do PostgreSQL:

   ```bash
   POSTGRES_PASSWORD=sua_senha
   ```

3. **Iniciar o Chatwoot**:

   ```bash
   docker compose up -d
   ```

4. **Verificar funcionamento**:

   ```bash
   curl -I localhost:3000
   ```

---

## 5. Instalação e Configuração do Evolution API com Docker Compose

Referência: [Evoluiton-API](https://doc.evolution-api.com/v2/pt/install/docker)

### Criar o Arquivo `docker-compose.yml`

```yaml
version: "3.9"
services:
  evolution-api:
    container_name: evolution_api
    image: atendai/evolution-api:v2.1.1
    restart: always
    ports:
      - "8080:8080"
    env_file:
      - .env
    volumes:
      - evolution_instances:/evolution/instances

volumes:
  evolution_instances:
```

### Criar o Arquivo `.env`

```env
AUTHENTICATION_API_KEY=mude-me
```

### Inicializar o Serviço

```sh
docker compose up -d
```

Verificar contêiner ativo:

```sh
docker ps
```

---

## 6. Integração do Evolution API com Chatwoot

Referência: [Integração Chatwoot-EvolutionAPI](https://doc.evolution-api.com/v2/pt/integrations/chatwoot)

## Requisitos

- Usar a **versão 1.8.2 do Evolution** para evitar erro relacionado ao _database provider_.

---

## 1. Configurar o arquivo `.env` do Evolution

1. Acesse o diretório onde o Evolution foi instalado.
2. Edite o arquivo `.env` com o comando:

   ```bash
   vi .env
   ```

3. Pressione `i` para entrar no modo de edição (_insert_).
4. Adicione/edite as seguintes linhas:

   ```env
   AUTHENTICATION_API_KEY="sua api key"
   DATABASE_ENABLED=false
   ```

5. Salve e saia (`ESC`, depois `:wq` e pressione `ENTER`).

---

## 2. Criar uma network customizada com Docker

Execute o seguinte comando:

```bash
docker network create --driver=bridge --subnet=172.13.0.0/24 nome-da-sua-rede
```

Substitua `nome-da-sua-rede` pelo nome desejado para sua rede Docker.

---

## 3. Configurar `docker-compose.yaml` do Evolution

1. Edite o arquivo `docker-compose.yaml`:

   ```bash
   vi docker-compose.yaml
   ```

2. Adicione no final do arquivo:

   ```yaml
   networks:
     nome-da-sua-rede:
       external: true
   ```

3. No serviço do **evolution**, adicione dentro da definição do serviço:

   ```yaml
   networks:
     nome-da-sua-rede:
       ipv4_address: 172.13.0.x
   ```

   > Substitua `x` por um número único para evitar conflitos com outros serviços.

---

## 4. Configurar `docker-compose.yaml` do Chatwoot

1. Acesse o diretório do Chatwoot.
2. Edite o arquivo `docker-compose.yaml`:

   ```bash
   vi docker-compose.yaml
   ```

3. Adicione no final do arquivo (sem sobrescrever outras `networks`):

   ```yaml
   networks:
     nome-da-sua-rede:
       external: true
   ```

4. Para cada serviço do Chatwoot, adicione:

   ```yaml
   networks:
     nome-da-sua-rede:
       ipv4_address: 172.13.0.x
   ```

   > Novamente, use IPs diferentes para cada serviço para evitar conflitos.

---

## 5. Acessar e Configurar Chatwoot

1. Acesse o Chatwoot pelo IP que você definiu no serviço `rails`.
2. Crie uma conta.
3. Vá em **Configurações do Usuário** e, no final da página, copie o **Token da Conta**.

---

## 6. Configurar o Evolution

1. Acesse o Evolution via:

   ```
   http://localhost:8080/manager
   ```

2. Crie uma instância.
3. Conecte o WhatsApp.
4. Configure o Chatwoot:
   - IP do Chatwoot (o mesmo usado no serviço `rails`)
   - ID da conta: `1`
   - Token da conta (copiado anteriormente)
   - Habilite e salve

---

## 7. Finalizar Integração no Chatwoot

1. Crie um **agente**.
2. Crie uma **inbox** usando a **API** do Chatwoot.
   - Use o **mesmo nome** da instância criada no Evolution.
   - Use o **webhook** fornecido pelo Evolution (ao clicar em "Como configurar o Chatwoot").

---

## Resultado

Após conectar o WhatsApp no Evolution, todas as mensagens serão entregues diretamente na inbox configurada no Chatwoot.

---

# TypeBot - Instalação com Docker e MinIO

Referência [Guia Typebot](https://docs.typebot.io/self-hosting/installation)

## Passo 1: Criação da estrutura

Crie um diretório para o typebot e dentro dele configure o arquivo `docker-compose.yml`, podendo baixar os templates usando os comandos:

```bash
wget https://raw.githubusercontent.com/baptisteArno/typebot.io/latest/docker-compose.yml
wget https://raw.githubusercontent.com/baptisteArno/typebot.io/latest/.env.example -O .env
```

## Passo 2: Configuração do `docker-compose.yml`

O arquivo deverá ter essas configurações:

```yaml
  - '8082:3000'
    extra_hosts:
      - 'host.docker.internal:host-gateway'
    env_file: .env

  typebot-viewer:
    image: baptistearno/typebot-viewer:latest
    restart: always
    networks:
      docker-network2:
        ipv4_address: 172.13.0.32
    ports:
      - '8083:3000'
    env_file: .env

  minio:
    image: minio/minio
    restart: always
    command: server /data
    ports:
      - '9000:9000'
    networks:
      docker-network2:
        ipv4_address: 172.13.0.33
    environment:
      MINIO_ROOT_USER: minio
      MINIO_ROOT_PASSWORD: minio123
    volumes:
      - s3-data:/data

  createbuckets:
    image: minio/mc
    restart: always
    depends_on:
      - minio
    entrypoint: >
      /bin/sh -c "
      sleep 10;
      /usr/bin/mc config host add minio http://172.13.0.33:9000 minio minio123;
      /usr/bin/mc mb minio/typebot;
      /usr/bin/mc anonymous set public minio/typebot/public;
      exit 0;
      "

networks:
  docker-network2:
    external: true
```

> Lembrando de **não repetir um IP já utilizado anteriormente** nas suas outras configurações. Este MinIO é um serviço levantado que será **necessário para a configuração do Typebot**.

## Passo 3: Configuração do `.env`

Configure também o `.env`, que ficará como:

```env
# Segurança
ENCRYPTION_SECRET=gere uma encryption key com esse comando "openssl rand -base64 24 | tr -d '\n' ; echo"

# Banco de dados
DATABASE_URL=postgresql://postgres:typebot@typebot-db:5432/typebot

# Autenticação
DEBUG=true
ADMIN_EMAIL=seuemail@email.com

SMTP_USERNAME=chatbct.ufmt@gmail.com
SMTP_PASSWORD='wkgg lqov bqpv ngjy'
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
NEXT_PUBLIC_SMTP_FROM='chatbct.ufmt@gmail.com'
SMTP_SECURE=false

# S3 (MinIO local)
S3_ENDPOINT=http://172.13.0.33:9000
S3_ACCESS_KEY=teste
S3_SECRET_KEY=teste12345
S3_BUCKET=typebot
S3_REGION=us-east-1
S3_SSL=true

# FRONTEND: navegador usa localhost
NEXT_PUBLIC_S3_ENDPOINT=http://localhost:9000

# Viewer URL local
NEXT_PUBLIC_VIEWER_URL=http://172.13.0.32:3000
NEXTAUTH_URL=http://localhost:8110
NEXTAUTH_URL_INTERNAL=http://172.13.0.32:3000
```

> Lembrando que localhost sera trocar pelo domínio que você deseja usar.

## Passo 4: Levantar os containers

Use o comando:

```bash
docker-compose up -d
```

para subir os containers.

---

## Passo 5: Acessar o MinIO

O MinIO nesse exemplo é acessado via:

```
http://172.13.0.33:9000
```

Caso não acesse por esse ip rodando o comando:

```bash
docker-compose logs
```

será possível visualizar o endereço que o MinIO está, pois o comando retornará algo como:

```
WebUI: http://172.13.0.34:35041
```

Acesse com as credenciais **teste** e **teste12345** que você definiu em:

```env
S3_ACCESS_KEY=teste
S3_SECRET_KEY=teste12345
```

Crie um bucket chamado `typebot`.

---

E pronto, o Typebot está instalado.
