# Guia Completo: Instalação do GLPI em Servidor Linux (Ubuntu/Debian) com Apache ou Nginx

Este documento fornece um passo a passo detalhado para instalar e configurar o sistema de gerenciamento de chamados e inventário GLPI em um servidor Linux Ubuntu 22.04/24.04 LTS (ou Debian equivalente).

## Índice Navegável

* [Pré-requisitos](#pre-requisitos)
* [Passo 1: Atualização do Sistema](#passo-1-atualizacao-do-sistema)
* [Passo 2: Instalação do Banco de Dados (MariaDB)](#passo-2-instalacao-do-banco-de-dados-mariadb)
* [Passo 3: Instalação do PHP e Extensões Necessárias](#passo-3-instalacao-do-php-e-extensoes-necessarias)
* **[SEÇÃO A: Configuração com Servidor Web Apache](#secao-a-configuracao-com-servidor-web-apache)**
    * [Passo 4A: Instalação do Apache](#passo-4a-instalacao-do-apache)
    * [Passo 5A: Download e Preparação do GLPI](#passo-5a-download-e-preparacao-do-glpi)
    * [Passo 6A: Configuração do Virtual Host do Apache](#passo-6a-configuracao-do-virtual-host-do-apache)
    * [Passo 7A: Configuração de Permissões (Apache)](#passo-7a-configuracao-de-permissoes-apache)
* **[SEÇÃO B: Configuração com Servidor Web Nginx](#secao-b-configuracao-com-servidor-web-nginx)**
    * [Passo 4B: Instalação do Nginx](#passo-4b-instalacao-do-nginx)
    * [Passo 5B: Download e Preparação do GLPI](#passo-5b-download-e-preparacao-do-glpi)
    * [Passo 6B: Configuração do Server Block do Nginx](#passo-6b-configuracao-do-server-block-do-nginx)
    * [Passo 7B: Configuração de Permissões (Nginx)](#passo-7b-configuracao-de-permissoes-nginx)
* [Passo 8: Instalação do GLPI via Interface Web](#passo-8-instalacao-do-glpi-via-interface-web)
* [Passo 9: Tarefas Críticas Pós-Instalação](#passo-9-tarefas-criticas-pos-instalacao)
* [Conclusão](#conclusao)

---

### Pré-requisitos
* Um servidor com uma instalação limpa do **Ubuntu 22.04/24.04 LTS** ou **Debian 11/12**.
* Acesso ao servidor via SSH com um usuário com privilégios `sudo`.
* Um endereço de IP público e, opcionalmente, um nome de domínio (FQDN) apontando para o IP do servidor.

### Passo 1: Atualização do Sistema
Sempre comece garantindo que todos os pacotes do sistema estão atualizados.

```bash
sudo apt update && sudo apt upgrade -y
```

### Passo 2: Instalação do Banco de Dados (MariaDB)
O GLPI precisa de um banco de dados para armazenar suas informações. MariaDB é um substituto de código aberto para o MySQL e é totalmente compatível.

1.  **Instale o MariaDB:**
    ```bash
    sudo apt install mariadb-server -y
    ```

2.  **Execute o script de segurança:**
    Este script ajudará a remover configurações inseguras e proteger seu banco de dados.
    ```bash
    sudo mysql_secure_installation
    ```
    > Responda às perguntas conforme a recomendação:
    > * `Enter current password for root (enter for none):` Pressione **Enter**.
    > * `Switch to unix_socket authentication [Y/n]` Pressione **Y**.
    > * `Change the root password? [Y/n]` Pressione **Y** e defina uma senha forte para o root do banco de dados.
    > * `Remove anonymous users? [Y/n]` Pressione **Y**.
    > * `Disallow root login remotely? [Y/n]` Pressione **Y**.
    > * `Remove test database and access to it? [Y/n]` Pressione **Y**.
    > * `Reload privilege tables now? [Y/n]` Pressione **Y**.

3.  **Crie o banco de dados e o usuário para o GLPI:**
    Faça login no MariaDB com o usuário root que você acabou de configurar.
    ```bash
    sudo mysql -u root -p
    ```
    Execute os seguintes comandos SQL para criar o banco de dados e o usuário. Substitua `sua_senha_forte` por uma senha segura de sua escolha.

    ```sql
    CREATE DATABASE glpidb CHARACTER SET UTF8MB4 COLLATE utf8mb4_unicode_ci;
    CREATE USER 'glpiuser'@'localhost' IDENTIFIED BY 'sua_senha_forte';
    GRANT ALL PRIVILEGES ON glpidb.* TO 'glpiuser'@'localhost';
    FLUSH PRIVILEGES;
    EXIT;
    ```

### Passo 3: Instalação do PHP e Extensões Necessárias
O GLPI é baseado em PHP. A versão 10.x do GLPI requer PHP 7.4 ou superior. Vamos instalar uma versão recente e todas as extensões necessárias.

```bash
sudo apt install php php-cli php-fpm php-mysql php-gd php-xml php-curl php-mbstring php-intl php-zip php-bz2 php-imap php-apcu php-cas php-ldap -y
```
> **Nota:** `php-fpm` é estritamente necessário para o Nginx, mas é bom tê-lo instalado de qualquer forma.

**Opcional, mas recomendado:** Ajuste o `php.ini` para otimizar o desempenho do GLPI. O arquivo pode estar em `/etc/php/8.x/apache2/php.ini` ou `/etc/php/8.x/fpm/php.ini` dependendo do servidor web que você usará.
```bash
# Para Apache (exemplo com PHP 8.3)
sudo nano /etc/php/8.3/apache2/php.ini

# Para Nginx (exemplo com PHP 8.3)
sudo nano /etc/php/8.3/fpm/php.ini
```
Procure e altere os seguintes valores (use `Ctrl+W` para buscar no nano):
```ini
memory_limit = 256M
upload_max_filesize = 64M
date.timezone = America/Sao_Paulo
```

---

**Agora, escolha UMA das seções abaixo (A ou B) de acordo com o servidor web de sua preferência.**

---

### SEÇÃO A: Configuração com Servidor Web Apache

#### Passo 4A: Instalação do Apache
```bash
sudo apt install apache2 -y
```

#### Passo 5A: Download e Preparação do GLPI
1.  **Baixe a última versão estável do GLPI:** Verifique no [site oficial do GLPI](https://glpi-project.org/downloads/) o número da última versão.
    ```bash
    # Exemplo com a versão 10.0.15 (substitua pela mais recente)
    wget [https://github.com/glpi-project/glpi/releases/download/10.0.15/glpi-10.0.15.tgz](https://github.com/glpi-project/glpi/releases/download/10.0.15/glpi-10.0.15.tgz)
    ```

2.  **Extraia o arquivo e mova para o diretório web:**
    ```bash
    tar -xvf glpi-10.0.15.tgz
    sudo mv glpi /var/www/html/
    ```

#### Passo 6A: Configuração do Virtual Host do Apache
Criar um arquivo de configuração dedicado para o GLPI é a melhor prática.

1.  **Crie o arquivo de configuração:**
    ```bash
    sudo nano /etc/apache2/sites-available/glpi.conf
    ```

2.  **Cole o seguinte conteúdo.** Substitua `glpi.seudominio.com` pelo seu domínio ou pelo IP do servidor.
    ```apache
    <VirtualHost *:80>
        ServerAdmin webmaster@localhost
        ServerName glpi.seudominio.com
        DocumentRoot /var/www/html/glpi

        <Directory /var/www/html/glpi>
            AllowOverride All
            Require all granted
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/glpi_error.log
        CustomLog ${APACHE_LOG_DIR}/glpi_access.log combined
    </VirtualHost>
    ```

3.  **Ative o novo site, o módulo de reescrita e reinicie o Apache:**
    ```bash
    sudo a2ensite glpi.conf
    sudo a2enmod rewrite
    sudo systemctl restart apache2
    ```

#### Passo 7A: Configuração de Permissões (Apache)
O Apache precisa ter permissão para escrever nos diretórios do GLPI.

```bash
sudo chown -R www-data:www-data /var/www/html/glpi/
sudo chmod -R 755 /var/www/html/glpi/
```
**Pule a Seção B e vá direto para o [Passo 8](#passo-8-instalacao-do-glpi-via-interface-web).**

---

### SEÇÃO B: Configuração com Servidor Web Nginx

#### Passo 4B: Instalação do Nginx
```bash
sudo apt install nginx -y
```

#### Passo 5B: Download e Preparação do GLPI
1.  **Baixe a última versão estável do GLPI:**
    ```bash
    # Exemplo com a versão 10.0.15 (substitua pela mais recente)
    wget [https://github.com/glpi-project/glpi/releases/download/10.0.15/glpi-10.0.15.tgz](https://github.com/glpi-project/glpi/releases/download/10.0.15/glpi-10.0.15.tgz)
    ```
2.  **Extraia e mova para o diretório web:**
    ```bash
    tar -xvf glpi-10.0.15.tgz
    sudo mv glpi /var/www/html/
    ```

#### Passo 6B: Configuração do Server Block do Nginx
1.  **Crie o arquivo de configuração do Nginx:**
    ```bash
    sudo nano /etc/nginx/sites-available/glpi
    ```

2.  **Cole o seguinte conteúdo.** Substitua `glpi.seudominio.com` pelo seu domínio/IP e verifique a versão do `php-fpm.sock`.
    ```nginx
    server {
        listen 80;
        server_name glpi.seudominio.com;
        root /var/www/html/glpi;

        index index.php;

        location / {
            try_files $uri $uri/ /index.php$is_args$args;
        }

        location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            # Exemplo com PHP 8.3. Verifique sua versão!
            fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
        }

        # Nega o acesso a diretórios sensíveis
        location ~ ^/(config|files|locales|install|scripts)/ {
            deny all;
        }

        # Nega acesso a arquivos .log e .md
        location ~* \.(log|md)$ {
            deny all;
        }
    }
    ```

3.  **Ative a configuração criando um link simbólico:**
    ```bash
    sudo ln -s /etc/nginx/sites-available/glpi /etc/nginx/sites-enabled/
    ```

4.  **Teste a configuração e reinicie o Nginx:**
    ```bash
    sudo nginx -t
    # Se a sintaxe estiver OK, reinicie
    sudo systemctl restart nginx
    ```

#### Passo 7B: Configuração de Permissões (Nginx)
O Nginx (e o processo PHP-FPM) precisa de permissão de escrita.
```bash
sudo chown -R www-data:www-data /var/www/html/glpi/
sudo chmod -R 755 /var/www/html/glpi/
```

---

### Passo 8: Instalação do GLPI via Interface Web
Agora que o ambiente do servidor está pronto, a instalação final é feita pelo navegador.

1.  Abra seu navegador e acesse o endereço que você configurou (ex: `http://glpi.seudominio.com` ou `http://seu_ip_do_servidor`).
2.  **Seleção de Idioma:** Escolha "Português do Brasil" e clique em OK.
3.  **Licença:** Leia e aceite os termos da licença.
4.  **Instalação:** Clique em **Instalar**.
5.  **Verificação de Compatibilidade:** O GLPI verificará se todas as extensões PHP estão instaladas e se as permissões de diretório estão corretas. Se você seguiu todos os passos, tudo deve estar verde. Clique em **Continuar**.
6.  **Configuração do Banco de Dados:** Preencha os detalhes do banco de dados que criamos no Passo 2.
    * **Servidor SQL:** `localhost`
    * **Usuário SQL:** `glpiuser`
    * **Senha SQL:** `sua_senha_forte`
    * Clique em **Continuar**.
7.  **Seleção do Banco de Dados:** Selecione `glpidb` na lista e clique em **Continuar**. A instalação irá criar todas as tabelas.
8.  **Finalização:** A instalação está quase completa. A tela mostrará os usuários e senhas padrão. **Anote-os!**
    * `glpi` (super-admin)
    * `tech` (técnico)
    * `normal` (usuário normal)
    * `post-only` (apenas para abrir chamado)
9.  Clique em **Usar o GLPI**.

### Passo 9: Tarefas Críticas Pós-Instalação
Por segurança e funcionalidade, estas etapas são obrigatórias.

1.  **Remova o arquivo de instalação:**
    ```bash
    sudo rm /var/www/html/glpi/install/install.php
    ```

2.  **Altere as senhas padrão:** Faça login com o usuário `glpi` e altere imediatamente a senha de todos os usuários padrão em **Administração > Usuários**.

3.  **Configure o Cron Job do GLPI:** Isso é essencial para tarefas automáticas como recebimento de e-mails, notificações, etc.
    Abra o editor de cron para o usuário do servidor web.
    ```bash
    sudo crontab -e -u www-data
    ```
    Adicione a seguinte linha no final do arquivo:
    ```crontab
    * * * * * /usr/bin/php /var/www/html/glpi/front/cron.php &>/dev/null
    ```
    Isso executará as tarefas automáticas a cada minuto.

### Conclusão
Se você seguiu todos os passos, agora você tem uma instalação funcional e segura do GLPI. O próximo passo recomendado é configurar um certificado SSL/TLS (usando Let's Encrypt, por exemplo) para proteger a comunicação com seu servidor. Explore o menu de **Administração > Configuração Geral** para começar a personalizar o sistema para as necessidades da sua organização.
