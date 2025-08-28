# Guia de Deploy: Página Estática C'Alma com PM2, NGINX e HTTPS/SSL (Linode VM)

Este guia explica como rodar a página estática do C'Alma em uma VM da Linode usando NGINX como servidor web, PM2 para servir arquivos estáticos (opcional), e como configurar HTTPS/SSL (ou rodar apenas em HTTP).

---

## 1. Pré-requisitos

- Uma VM Linux (Ubuntu recomendado) na Linode.
- Acesso root ou sudo.
- Domínio configurado (opcional, mas recomendado para HTTPS).
- Arquivos do site (pasta `plume-template.webflow.io`).

---

## 2. Instale o NGINX

```bash
sudo apt update
sudo apt install nginx -y
```

---

## 3. Faça upload dos arquivos do site

- Faça upload da pasta `plume-template.webflow.io` para o diretório desejado na VM, por exemplo: `/var/www/calma-lp`.

```bash
sudo mkdir -p /var/www/calma-lp
sudo cp -r plume-template.webflow.io/* /var/www/calma-lp/
sudo chown -R www-data:www-data /var/www/calma-lp
```

---

## 4. Configure o NGINX

### Exemplo de configuração para HTTP (sem SSL):

```nginx
server {
    listen 80;
    server_name seu-dominio.com; # ou _ para qualquer domínio/IP

    root /var/www/calma-lp;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

- Salve este bloco em `/etc/nginx/sites-available/calma-lp`
- Ative o site:

```bash
sudo ln -s /etc/nginx/sites-available/calma-lp /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

Acesse: `http://SEU_IP` ou `http://seu-dominio.com`

---

## 5. (Opcional) Servir com HTTPS/SSL

### a) Instale Certbot

```bash
sudo apt install certbot python3-certbot-nginx -y
```

### b) Gere o certificado SSL

```bash
sudo certbot --nginx -d seu-dominio.com -d www.seu-dominio.com
```

- Siga as instruções para ativar HTTPS.
- O NGINX será recarregado automaticamente.

Acesse: `https://seu-dominio.com`

---

## 6. (Opcional) Servir com PM2 (Node.js)

Se quiser servir arquivos estáticos com PM2 (útil para preview local ou Node):

### a) Instale Node.js e PM2

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
sudo npm install -g pm2 serve
```

### b) Rode o servidor estático

```bash
cd /var/www/calma-lp
pm2 start serve --name calma-lp -- -s -l 8080
```

- O site estará disponível em `http://SEU_IP:8080`

### c) PM2 como serviço

```bash
pm2 startup
pm2 save
```

---

## 7. Dicas

- Para rodar **apenas HTTP** (sem HTTPS), basta não instalar o Certbot e não rodar o passo 5.
- Para rodar **apenas HTTPS**, siga o passo 5.
- Para rodar **sem NGINX** (apenas Node/PM2), use o comando do passo 6.
- Para rodar **com NGINX como proxy** para PM2, aponte o `proxy_pass` do NGINX para `http://localhost:8080`.

---

## 8. Referências

- [Documentação NGINX](https://nginx.org/en/docs/)
- [Certbot (Let's Encrypt)](https://certbot.eff.org/)
- [PM2](https://pm2.keymetrics.io/)
- [Serve (Node.js)](https://www.npmjs.com/package/serve)

---

**Dúvidas?**  
Abra um issue ou entre em contato com o time técnico.
