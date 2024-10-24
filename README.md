# Jogo de Adivinhação com Flask e React

Este é um jogo de adivinhação, onde você tenta descobrir uma senha gerada aleatoriamente. Ele foi feito usando Flask no backend (parte do servidor) e React no frontend (parte visual). O jogo te dá dicas sobre quantas letras da sua tentativa estão corretas e se estão nas posições certas.

## O que o jogo faz

- Você pode criar um novo jogo com uma senha secreta.
- Tente adivinhar a senha e receba dicas sobre quais letras estão certas e se estão no lugar certo.
- As senhas são guardadas de forma segura usando base64.
- Se você errar, o jogo te dá dicas para tentar de novo.

## O que você precisa para rodar o jogo

- Docker 20.10.0 ou mais recente
- Docker Compose 3.9 ou mais recente

## Como rodar o jogo

1. Primeiro, clone o repositório (código do jogo) para o seu computador:

   ```bash
   git clone https://github.com/srjm23/Jogo-adivinha-senha-PUC.git
   cd Jogo-adivinha-senha-PUC
   ```

2. Depois, rode o jogo usando Docker Compose:

   ```bash
   docker compose up -d
   ```

3. Agora, abra o navegador e acesse o jogo em: [http://localhost:3000](http://localhost:3000) 🎮

## Como jogar

### 1. Criar um novo jogo

- Acesse: [http://localhost:3000/maker](http://localhost:3000/maker)
- Digite uma senha secreta.
- Envie e guarde o `game-id` que será gerado (você vai precisar dele depois).

### 2. Adivinhar a senha

- Acesse: [http://localhost:3000/breaker](http://localhost:3000/breaker)
- Insira o `game-id` que você salvou.
- Tente adivinhar a senha!

Agora é só seguir os passos e começar a jogar! 😄

## Como o jogo funciona com Docker

### Estrutura do Projeto

O jogo está dividido em três partes principais, chamadas de "serviços", que são gerenciadas pelo Docker Compose:

#### Serviços:

```yaml
services:
  db:
  backend:
  frontend:
```

- **db**: É um banco de dados PostgreSQL, que utiliza um volume persistente para armazenar os dados do jogo, garantindo que as informações não sejam perdidas em caso de reinício dos container.

```yml
db:
  image: postgres:14.12
  secrets:
    - db-password
  environment:
    - POSTGRES_PASSWORD=/run/secrets/db-password
  volumes:
    - db-data:/var/lib/postgresql/data
  networks:
    - netback
  expose:
    - 5432
  restart: always
```

- **backend**: É um servidor Flask, que cuida da lógica do jogo e se comunica com o banco de dados, esse serviço está escalado com dois containers. Além disso, só começa a funcionar depois que o banco de dados estiver rodando.

```yml
backend:
  build:
    context: backend
  environment:
    - FLASK_DB_PASSWORD=/run/secrets/db-password
    - FLASK_DB_HOST=db
  scale: 2
  expose:
    - 5000
  networks:
    - netback
    - netfront
  depends_on:
    - db
  restart: always
```

- **frontend**: É um servidor Nginx, que mostra a interface do jogo (React) e faz a conexão entre o frontend e o backend.

```yml
frontend:
  build:
    context: frontend
  ports:
    - 3000:80
  networks:
    - netfront
  depends_on:
    - backend
  restart: always
```

#### Volumes

O volume `db-data` é usado para garantir que os dados do banco de dados não sejam perdidos, mesmo que o container seja reiniciado.

```yml
volumes:
  db-data:
```

#### Redes

O projeto usa duas redes:

- **netback**: Conecta o banco de dados ao backend.
- **netfront**: Conecta o frontend ao backend.

```yml
networks:
  netback:
  netfront:
```

#### Balanceamento de Carga

O Nginx, além de servir o frontend, faz o balanceamento de carga entre dois containers do backend, garantindo melhor desempenho e disponibilidade

```nginx
upstream backend {
  server backend:5000;
}

server {
  listen 80;

  location / {
    root /usr/share/nginx/html;
    index index.html index.htm;
    try_files $uri $uri/ /index.html;
  }

  location /api {
    proxy_pass http://backend/;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
}
```

### Reinício automático
Está habilitado nos serviços o **restart: always** que garante que eles sejam reiniciados em caso de falhas.

#### Atualização

Para atualizar o backend, frontend ou banco de dados, basta modificar a versão da imagem Docker ou alterar o código fonte e executar o seguinte comando:

```bash
  docker-compose up --build
```
### Remoção da infra

Caso não queira mais jogar e com isso deseje remover a infraestrutura de container, basta executar o comando:

```bash
  docker-compose down
```
