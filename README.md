# Deploy de aplicação com Nginx e Docker

- Domínio: dominiodonovosite.com.br
- A documentação já está atualizada com o domínio correto, não é necessário alterar nada.
- O domínio deve estar apontando para o servidor, caso contrário, o comando não vai funcionar.
- A documentação já está considerando que a pipeline do github actions já está configurada (pasta .git).

## 1. Configuração do servidor (debian) - Caso o servidor já esteja configurado, ir direto para o passo 2

```bash

apt-get update && apt-get upgrade -y

```

OBS: O servidor só deve ser atualizado se não houver aplicações em produção, pois o servidor pode reiniciar e causar indisponibilidade.

## 2. instalar o nginx e certbot (Caso o servidor já esteja configurado, ir direto para o passo 3)

```bash

sudo apt install certbot python3-certbot-nginx
sudo apt install nginx

```

## 3. instalar o docker no Debian (Caso o servidor já esteja configurado com Docker, ir direto para o passo 4)

```bash

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo docker run hello-world # Teste de instalação do docker

```

## 4. Criar o usuário que vai rodar o docker e buildar a aplicação

```bash

adduser meuusuario # Colocar a senha com o mesmo nome do usuário
usermod -aG sudo meuusuario # Adiciona o usuário ao grupo sudo
usermod -aG docker meuusuario # Adiciona o usuário ao grupo docker
sudo su - meuusuario ## entrar no usuario

```

## 5. Instalar o runner do github actions

- Ir nas opções do repositório, runners, linux e seguir os passos para instalar o runner.
- Após a instalação do runner, rodar o comando `sudo ./svc.sh install` para instalar o serviço do runner.
- Após a instalação do serviço, rodar o comando `sudo ./svc.sh start` para iniciar o serviço do runner.
- Após a instalação do serviço, rodar o comando `sudo ./svc.sh status` para verificar se o serviço está rodando.

## 6. Criar o arquivo de configuração do nginx

```bash
sudo nano /etc/nginx/sites-available/dominiodonovosite.com.br

```

- Adicionar o seguinte conteúdo no arquivo:

```nginx

server {
    listen 80;
    listen [::]:80;

    server_name  dominiodonovosite.com.br;

    location / {
        proxy_pass http://localhost:3002; # Porta que o docker está rodando
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

- Salvar e sair do arquivo (CTRL + X, Y, ENTER)
- Criar um link simbólico para o arquivo de configuração do nginx:

```bash
sudo ln -s /etc/nginx/sites-available/dominiodonovosite.com.br /etc/nginx/sites-enabled/
```

- Testar a configuração do nginx:

```bash
sudo nginx -t

```

## 7. Criar o certificado SSL

- Rodar o comando para criar o certificado SSL:
- OBS: O domínio deve estar apontando para o servidor, caso contrário, o comando não vai funcionar.
- OBS: No caso de subdomínios, não é necessário gerar o certificado para o www, pois o nginx já redireciona para o domínio principal.

```bash

sudo certbot --nginx -d  dominiodonovosite.com.br

```

- Reiniciar o nginx:

```bash
sudo systemctl restart nginx
```
