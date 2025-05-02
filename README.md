# Learning Docker

## Guia Completo para Uso do Docker e Docker Compose

## Índice

- [Learning Docker](#learning-docker)
  - [Guia Completo para Uso do Docker e Docker Compose](#guia-completo-para-uso-do-docker-e-docker-compose)
  - [Índice](#índice)
  - [Introdução](#introdução)
  - [Conceitos Básicos](#conceitos-básicos)
    - [Container](#container)
    - [Imagem (Image)](#imagem-image)
    - [Volume](#volume)
    - [Docker Compose](#docker-compose)
  - [Principais Comandos e Flags](#principais-comandos-e-flags)
    - [Docker Run e Flags](#docker-run-e-flags)
    - [Outros Comandos Interessantes](#outros-comandos-interessantes)
  - [Trabalhando com Docker Compose](#trabalhando-com-docker-compose)
    - [Comandos do Docker Compose](#comandos-do-docker-compose)
  - [Visualizando Logs](#visualizando-logs)
    - [Docker (Container Único)](#docker-container-único)
    - [Docker Compose logs](#docker-compose-logs)
  - [Exemplos Práticos](#exemplos-práticos)
    - [1. Rodando um Container em Modo Detached com Port Mapping](#1-rodando-um-container-em-modo-detached-com-port-mapping)
    - [2. Executando um Container em Modo Interativo](#2-executando-um-container-em-modo-interativo)
    - [3. Definindo Variáveis de Ambiente no Container](#3-definindo-variáveis-de-ambiente-no-container)
    - [4. Construindo e Rodando uma Aplicação Multi-Container com Docker Compose](#4-construindo-e-rodando-uma-aplicação-multi-container-com-docker-compose)
  - [Como instalar](#como-instalar)
    - [Postgres](#postgres)
    - [PgAdmin](#pgadmin)
  - [Entendendo o Dockerfile](#entendendo-o-dockerfile)
    - [Exemplo Atualizado de Dockerfile](#exemplo-atualizado-de-dockerfile)
    - [Explicação dos Elementos](#explicação-dos-elementos)
  - [Explorando o docker-compose.yml](#explorando-o-docker-composeyml)
    - [Exemplo Atualizado de docker-compose.yml](#exemplo-atualizado-de-docker-composeyml)
    - [Explicação dos Elementos - Compose](#explicação-dos-elementos---compose)
  - [Considerações Finais](#considerações-finais)

---

## Introdução

O **Docker** é uma ferramenta que utiliza a tecnologia de container para empacotar, distribuir e executar aplicações de forma isolada. Com ele, conseguimos criar ambientes consistentes, replicáveis e portáveis através de imagens que contêm todo o software e dependências necessárias em um container.

Este guia foi desenvolvido para ajudar o Samuel a entender e utilizar o Docker, seja para desenvolvimento, testes ou ambientes de produção.

---

## Conceitos Básicos

### Container

- **Container** é uma instância de uma imagem em execução.  
- Ele isola a aplicação e suas dependências em um ambiente leve e portátil.  
- Comandos comuns: `docker run`, `docker container ls`, `docker container ls -a`, `docker container stop`, `docker container rm`.

### Imagem (Image)

- **Imagem** é o "template" que contém o sistema de arquivos e as dependências necessárias para executar uma aplicação.  
- Você pode construir suas próprias imagens com um **Dockerfile** ou baixá-las do Docker Hub.  
- Comandos comuns: `docker build`, `docker pull`, `docker images`, `docker image ls`, `docker image rm`, `docker tag`, `docker push`.

### Volume

- **Volume** é um recurso usado para persistir dados gerados e utilizados pelos containers.  
- Diferentemente do sistema de arquivos dos containers (efêmero), os volumes permitem que os dados sobrevivam a reinicializações de containers.  
- Comandos comuns: `docker volume create`, `docker volume ls`, `docker volume inspect`, `docker volume rm`.

### Docker Compose

- **Docker Compose** é uma ferramenta que permite definir e orquestrar múltiplos containers com um único arquivo de configuração (geralmente chamado de `docker compose.yml`).  
- Você pode definir serviços, volumes, networks e dependências em um único arquivo para facilitar o gerenciamento do ambiente.

---

## Principais Comandos e Flags

### Docker Run e Flags

O comando `docker run` é utilizado para iniciar um container. A seguir, explicamos as flags mais comuns:

- **`-d`**:  
  Executa o container em **modo detached** (em background).  
  Exemplo:

  ```bash
  docker run -d my_image
  ```

- **`-p`**:  
  Faz o mapeamento de portas entre o host e o container.  
  Sintaxe: `-p [porta_host:porta_container]`.  
  Exemplo:  

  ```bash
  docker run -p 8080:80 my_image
  ```

- **`-t`**:  
  Aloca um terminal pseudo-TTY para o container, facilitando a interação, especialmente se usado junto com `-i`.  
  Exemplo:

  ```bash
  docker run -t my_image
  ```

- **`-i`**:  
  Mantém STDIN (entrada padrão) aberto mesmo se não estiver conectado a um terminal.  
  Normalmente utilizado com `-t` para sessões interativas (`-it`).  
  Exemplo:

  ```bash
  docker run -it my_image bash
  ```

- **`-e`**:  
  Define variáveis de ambiente para o container.  
  Sintaxe: `-e VARIAVEL=valor`.  
  Exemplo:

  ```bash
  docker run -e ENV=production my_image
  ```

Outras flags úteis:

- **`--name`**:  
  Especifica um nome personalizado para o container.  
  Exemplo:

  ```bash
  docker run --name meu_container my_image
  ```

- **`--rm`**:  
  Remove automaticamente o container quando ele parar.  
  Exemplo:

  ```bash
  docker run --rm my_image
  ```

### Outros Comandos Interessantes

- **Listar containers em execução:**

  ```bash
  docker ps
  ```

- **Listar todos os containers (incluindo os parados):**

  ```bash
  docker ps -a
  ```

- **Inspecionar um container:**

  ```bash
  docker inspect nome_ou_id_do_container
  ```

- **Remover uma imagem:**

  ```bash
  docker rmi nome_da_imagem
  ```

- **Construir uma imagem a partir de um Dockerfile:**

  ```bash
  docker build -t meu_app:latest .
  ```

---

## Trabalhando com Docker Compose

O **Docker Compose** permite orquestrar múltiplos containers que compõem sua aplicação. Um exemplo básico de um arquivo `docker compose.yml`:

```yaml
services:
  db:
    container_name: db
    image: postgres:16.8
    environment:
      POSTGRES_DB: docker_db
      POSTGRES_USER: docker_user
      POSTGRES_PASSWORD: docker_password
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - network_private

  web: &web
    container_name: web
    build: .
    command: >
      sh -c "
        python manage.py migrate
        gunicorn --bind 0.0.0.0:8000 mysite.wsgi
      "
    environment:
      - DJANGO_SETTINGS_MODULE=mysite.settings
      - DB_NAME=docker_db
      - DB_USER=docker_user
      - DB_PASSWORD=docker_password
      - DB_HOST=db
      - DB_PORT=5432
    volumes:
      - ./web:/web
    depends_on:
      - db
    networks:
      - network_private
      - network_public

  web-2:
    container_name: web-2
    <<: *web
  nginx:
    container_name: nginx
    image: nginx:1.27
    volumes:
      - ./nginx:/etc/nginx/conf.d
    ports:
      - "8080:80"
    depends_on:
      - web
      - web-2
    restart: always
    networks:
      - network_public

volumes:
  db_data:

networks:
  network_private:
    driver: bridge
  network_public:
    driver: bridge
```

### Comandos do Docker Compose

- **Subir os serviços e construir as imagens:**

  ```bash
  docker compose up --build
  ```

  Use a flag `-d` para execução em segundo plano:

  ```bash
  docker compose up --build -d
  ```

- **Parar os serviços:**

  ```bash
  docker compose down
  ```

- **Listar os serviços em execução:**

  ```bash
  docker compose ps
  ```

- **Verificar os logs dos serviços:**

  ```bash
  docker compose logs
  ```

  Para seguir os logs em tempo real:

  ```bash
  docker compose logs -f
  ```

---

## Visualizando Logs

### Docker (Container Único)

Para visualizar os logs de um container específico, utilize:

- **Ver logs:**

  ```bash
  docker logs nome_ou_id_do_container
  ```

- **Seguir (tail) os logs em tempo real:**

  ```bash
  docker logs -f nome_ou_id_do_container
  ```

### Docker Compose logs

Para ver os logs dos serviços que compõem sua aplicação orquestrada com Docker Compose:

- **Exibir logs de todos os serviços:**

  ```bash
  docker compose logs
  ```

- **Seguir os logs em tempo real:**

  ```bash
  docker compose logs -f
  ```

- **Exibir logs de um serviço específico, por exemplo, o serviço *web*:**

  ```bash
  docker compose logs -f web
  ```

---

## Exemplos Práticos

### 1. Rodando um Container em Modo Detached com Port Mapping

```bash
docker run -d -p 8080:80 --name meu_nginx nginx:latest
```

- **`-d`**: Executa em segundo plano.
- **`-p 8080:80`**: Mapeia a porta 80 do container para a porta 8080 do host.

### 2. Executando um Container em Modo Interativo

```bash
docker run -it --rm ubuntu bash
```

- **`-it`**: Modo interativo com pseudo-TTY.
- **`--rm`**: Remove o container após o término da sessão.

### 3. Definindo Variáveis de Ambiente no Container

```bash
docker run -e ENV=production -e DEBUG=false my_app_image
```

- **`-e`**: Define uma ou mais variáveis de ambiente.

### 4. Construindo e Rodando uma Aplicação Multi-Container com Docker Compose

```bash
docker compose up --build -d
```

- Constrói e inicia todos os containers definidos no arquivo `docker compose.yml` em background.

## Como instalar

### Postgres

```shell
    docker run \
    --name postgres-container \
    -e POSTGRES_USER=user \
    -e POSTGRES_PASSWORD=postgres \
    -e POSTGRES_DB=database \
    -d \
    -p 5434:5432 \
    -v postgres-data:/var/lib/postgresql/data \
    postgres:latest
```

### PgAdmin

```shell
    docker run -d \
    --name pg-admin-4 \
    -e "PGADMIN_DEFAULT_EMAIL=email@email.com" \
    -e "PGADMIN_DEFAULT_PASSWORD=123456" \
    -p 7878:80 \
    dpage/pgadmin4
```

---

## Entendendo o Dockerfile

O **Dockerfile** é um arquivo de texto que contém uma lista de instruções para construir uma imagem Docker. A seguir, veja o exemplo atualizado e a explicação de cada elemento utilizado:

### Exemplo Atualizado de Dockerfile

```dockerfile
FROM python:3.14-rc
COPY ./web /web/
WORKDIR /web
EXPOSE 8000
RUN pip install -r requirements.txt
```

### Explicação dos Elementos

- **`FROM python:3.14-rc`**  
  Especifica a imagem base a ser utilizada. Aqui, usamos o Python na versão 3.14-rc (release candidate), representando uma versão recente e em teste.

- **`COPY ./web /web/`**  
  Copia o conteúdo da pasta local `./web` para dentro do container no diretório `/web/`.

- **`WORKDIR /web`**  
  Define o diretório de trabalho dentro do container. Todos os comandos subsequentes serão executados a partir desse diretório.

- **`EXPOSE 8000`**  
  Indica que o container utilizará a porta 8000 para comunicação. Essa instrução serve como documentação e ajuda ferramentas de integração, mas não realiza o mapeamento da porta automaticamente.

- **`RUN pip install -r requirements.txt`**  
  Executa o comando para instalar as dependências Python listadas no arquivo `requirements.txt`. Essa etapa criará uma nova camada na imagem Docker.

---

## Explorando o docker-compose.yml

O arquivo **docker-compose.yml** permite que você defina e orquestre múltiplos containers que compõem sua aplicação. A seguir, confira o exemplo atualizado e a explicação detalhada de cada parte:

### Exemplo Atualizado de docker-compose.yml

```yaml
version: "3.8"

services:
  db:
    container_name: db
    image: postgres:16.8
    environment:
      POSTGRES_DB: docker_db
      POSTGRES_USER: docker_user
      POSTGRES_PASSWORD: docker_password
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - network_private

  web: &web
    container_name: web
    build: .
    command: >
      sh -c "
        python manage.py migrate
        gunicorn --bind 0.0.0.0:8000 mysite.wsgi
      "
    environment:
      - DJANGO_SETTINGS_MODULE=mysite.settings
      - DB_NAME=docker_db
      - DB_USER=docker_user
      - DB_PASSWORD=docker_password
      - DB_HOST=db
      - DB_PORT=5432
    volumes:
      - ./web:/web
    depends_on:
      - db
    networks:
      - network_private
      - network_public

  web-2:
    container_name: web-2
    <<: *web

  nginx:
    container_name: nginx
    image: nginx:1.27
    volumes:
      - ./nginx:/etc/nginx/conf.d
    ports:
      - "8080:80"
    depends_on:
      - web
      - web-2
    restart: always
    networks:
      - network_public

volumes:
  db_data:

networks:
  network_private:
    driver: bridge
  network_public:
    driver: bridge
```

### Explicação dos Elementos - Compose

- **`version: "3.8"`**  
  Define a versão do esquema do Docker Compose. A versão 3.8 é compatível com os recursos mais recentes.

- **`services`**  
  Agrupa os containers que compõem a aplicação:

  - **Serviço `db`:**  
    - **`container_name: db`** – Nome personalizado para o container.  
    - **`image: postgres:16.8`** – Imagem do Postgres na versão 16.8.  
    - **`environment`** – Define variáveis de ambiente para configuração inicial do banco.  
    - **`volumes`** – Mapeia um volume para a persistência dos dados do PostgreSQL.  
    - **`networks`** – Conecta este serviço à rede privada para comunicação interna.

  - **Serviço `web`:**  
    - **`container_name: web`** – Define um nome identificador para o container.  
    - **`build: .`** – Informa que a imagem deve ser construída usando o _Dockerfile_ presente no diretório atual.  
    - **`command`** – Sobrescreve o comando padrão para executar as migrações e iniciar o Gunicorn.  
    - **`environment`** – Configura variáveis de ambiente para o Django se conectar ao banco de dados e outras configurações.  
    - **`volumes`** – Monta o diretório local `./web` dentro do container, permitindo autorefresh de código.  
    - **`depends_on`** – Garante que o serviço `db` esteja rodando antes de iniciar o `web`.  
    - **`networks`** – Conecta o container às redes privada e pública para comunicação interna e externa.

  - **Serviço `web-2`:**  
    - Usa a âncora `&web` para duplicar as configurações do serviço `web`, facilitando a escalabilidade (balanceamento de carga).

  - **Serviço `nginx`:**  
    - **`container_name: nginx`** – Nome personalizado para o container Nginx.  
    - **`image: nginx:1.27`** – Imagem oficial do Nginx na versão 1.27.  
    - **`volumes`** – Monta a configuração do Nginx a partir do diretório local.  
    - **`ports`** – Mapeia a porta 80 interna para a porta 8080 externa do host.  
    - **`depends_on`** – Garante que os serviços `web` e `web-2` estejam ativos antes de iniciar o Nginx.  
    - **`restart: always`** – Configura o container para reiniciar automaticamente em caso de falhas.  
    - **`networks`** – Conecta o Nginx à rede pública para exposição externa.

- **`volumes`**  
  Define o volume `db_data` usado para persistir os dados do PostgreSQL.

- **`networks`**  
  Cria duas redes:
  - **`network_private`** para a comunicação interna entre os containers.
  - **`network_public`** para a exposição externa (por exemplo, para o Nginx).

---

## Considerações Finais

Foi apresentado um guia abrangente para:

- **Utilização do Docker e Docker Compose** para gerenciar containers, imagens e volumes.
- Compreender as **principais flags** (como `-d`, `-p`, `-t`, `-e`) que influenciam no comportamento dos containers.
- **Acessar e monitorar logs** dos containers, tanto individualmente quanto via Docker Compose.

Dominar esses conceitos e comandos irá facilitar o gerenciamento das suas aplicações em ambientes Docker, proporcionando um fluxo de trabalho mais robusto e escalável.

Se surgirem dúvidas ou se você precisar de mais exemplos, consulte a [documentação oficial do Docker](https://docs.docker.com/) ou este guia novamente. Zé da moeda ou Hurk agiota! 🤔🤔🤔
