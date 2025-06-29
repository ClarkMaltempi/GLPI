# GLPI
Infraestrutura Implementação de Sistema para gerenciamento de chamados.

Guia Completo: Instalação do GLPI em Servidor Linux (Ubuntu/Debian) com Apache ou Nginx
Este documento fornece um passo a passo detalhado para instalar e configurar o sistema de gerenciamento de chamados e inventário GLPI em um servidor Linux Ubuntu 22.04/24.04 LTS (ou Debian equivalente).

Índice Navegável
Pré-requisitos

Passo 1: Atualização do Sistema

Passo 2: Instalação do Banco de Dados (MariaDB)

Passo 3: Instalação do PHP e Extensões Necessárias

SEÇÃO A: Configuração com Servidor Web Apache

Passo 4A: Instalação do Apache

Passo 5A: Download e Preparação do GLPI

Passo 6A: Configuração do Virtual Host do Apache

Passo 7A: Configuração de Permissões (Apache)

SEÇÃO B: Configuração com Servidor Web Nginx

Passo 4B: Instalação do Nginx

Passo 5B: Download e Preparação do GLPI

Passo 6B: Configuração do Server Block do Nginx

Passo 7B: Configuração de Permissões (Nginx)

Passo 8: Instalação do GLPI via Interface Web

Passo 9: Tarefas Críticas Pós-Instalação

Conclusão

Pré-requisitos
Um servidor com uma instalação limpa do Ubuntu 22.04/24.04 LTS ou Debian 11/12.

Acesso ao servidor via SSH com um usuário com privilégios sudo.

Um endereço de IP público e, opcionalmente, um nome de domínio (FQDN) apontando para o IP do servidor.

Passo 1: Atualização do Sistema
Sempre comece garantindo que todos os pacotes do sistema estão atualizados.

Bash

sudo apt update && sudo apt upgrade -y
Passo 2: Instalação do Banco de Dados (MariaDB)
O GLPI precisa de um banco de dados para armazenar suas informações. MariaDB é um substituto de código aberto para o MySQL e é totalmente compatível.

Instale o MariaDB:

Bash

sudo apt install mariadb-server -y
Execute o script de segurança:
Este script ajudará a remover configurações inseguras e proteger seu banco de dados.

Bash

sudo mysql_secure_installation
Responda às perguntas conforme a recomendação:

Enter current password for root (enter for none): Pressione Enter.

Switch to unix_socket authentication [Y/n] Pressione Y.

Change the root password? [Y/n] Pressione Y e defina uma senha forte para o root do banco de dados.

Remove anonymous users? [Y/n] Pressione Y.

Disallow root login remotely? [Y/n] Pressione Y.

Remove test database and access to it? [Y/n] Pressione Y.

Reload privilege tables now? [Y/n] Pressione Y.

Crie o banco de dados e o usuário para o GLPI:
Faça login no MariaDB com o usuário root que você acabou de configurar.

Bash

sudo mysql -u root -p
Execute os seguintes comandos SQL para criar o banco de dados e o usuário. Substitua sua_senha_forte por uma senha segura de sua escolha.

SQL

CREATE DATABASE glpidb CHARACTER SET UTF8MB4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'glpiuser'@'localhost' IDENTIFIED BY 'sua_senha_forte';
GRANT ALL PRIVILEGES ON glpidb.* TO 'glpiuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
Passo 3: Instalação do PHP e Extensões Necessárias
O GLPI é baseado em PHP. A versão 10.x do GLPI requer PHP 7.4 ou superior. Vamos instalar uma versão recente e todas as extensões necessárias.

Bash

sudo apt install php php-cli php-fpm php-mysql php-gd php-xml php-curl php-mbstring php-intl php-zip php-bz2 php-imap php-apcu php-cas php-ldap -y
Nota: php-fpm é estritamente necessário para o Nginx, mas é bom tê-lo instalado de qualquer forma.

Opcional, mas recomendado: Ajuste o php.ini para otimizar o desempenho do GLPI. O arquivo pode estar em /etc/php/8.x/apache2/php.ini ou /etc/php/8.x/fpm/php.ini dependendo do servidor web que você usará.

Bash

# Para Apache (exemplo com PHP 8.3)
sudo nano /etc/php/8.3/apache2/php.ini

# Para Nginx (exemplo com PHP 8.3)
sudo nano /etc/php/8.3/fpm/php.ini
Procure e altere os seguintes valores (use Ctrl+W para buscar no nano):

Ini, TOML

memory_limit = 256M
upload_max_filesize = 64M
date.timezone = America/Sao_Paulo
Agora, escolha UMA das seções abaixo (A ou B) de acordo com o servidor web de sua preferência.

SEÇÃO A: Configuração com Servidor Web Apache
Passo 4A: Instalação do Apache
Bash

sudo apt install apache2 -y
Passo 5A: Download e Preparação do GLPI
Baixe a última versão estável do GLPI: Verifique no site oficial do GLPI o número da última versão.

Bash

# Exemplo com a versão 10.0.15 (substitua pela mais recente)
wget https://github.com/glpi-project/glpi/releases/download/10.0.15/glpi-10.0.15.tgz
Extraia o arquivo e mova para o diretório web:

Bash

tar -xvf glpi-10.0.15.tgz
sudo mv glpi /var/www/html/
Passo 6A: Configuração do Virtual Host do Apache
Criar um arquivo de configuração dedicado para o GLPI é a melhor prática.

Crie o arquivo de configuração:

Bash

sudo nano /etc/apache2/sites-available/glpi.conf
Cole o seguinte conteúdo. Substitua glpi.seudominio.com pelo seu domínio ou pelo IP do servidor.

Apache

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
Ative o novo site, o módulo de reescrita e reinicie o Apache:

Bash

sudo a2ensite glpi.conf
sudo a2enmod rewrite
sudo systemctl restart apache2
Passo 7A: Configuração de Permissões (Apache)
O Apache precisa ter permissão para escrever nos diretórios do GLPI.

Bash

sudo chown -R www-data:www-data /var/www/html/glpi/
sudo chmod -R 755 /var/www/html/glpi/
Pule a Seção B e vá direto para o Passo 8.

SEÇÃO B: Configuração com Servidor Web Nginx
Passo 4B: Instalação do Nginx
Bash

sudo apt install nginx -y
Passo 5B: Download e Preparação do GLPI
Baixe a última versão estável do GLPI:

Bash

# Exemplo com a versão 10.0.15 (substitua pela mais recente)
wget https://github.com/glpi-project/glpi/releases/download/10.0.15/glpi-10.0.15.tgz
Extraia e mova para o diretório web:

Bash

tar -xvf glpi-10.0.15.tgz
sudo mv glpi /var/www/html/
Passo 6B: Configuração do Server Block do Nginx
Crie o arquivo de configuração do Nginx:

Bash

sudo nano /etc/nginx/sites-available/glpi
Cole o seguinte conteúdo. Substitua glpi.seudominio.com pelo seu domínio/IP e verifique a versão do php-fpm.sock.

Nginx

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
Ative a configuração criando um link simbólico:

Bash

sudo ln -s /etc/nginx/sites-available/glpi /etc/nginx/sites-enabled/
Teste a configuração e reinicie o Nginx:

Bash

sudo nginx -t
# Se a sintaxe estiver OK, reinicie
sudo systemctl restart nginx
Passo 7B: Configuração de Permissões (Nginx)
O Nginx (e o processo PHP-FPM) precisa de permissão de escrita.

Bash

sudo chown -R www-data:www-data /var/www/html/glpi/
sudo chmod -R 755 /var/www/html/glpi/
Passo 8: Instalação do GLPI via Interface Web
Agora que o ambiente do servidor está pronto, a instalação final é feita pelo navegador.

Abra seu navegador e acesse o endereço que você configurou (ex: http://glpi.seudominio.com ou http://seu_ip_do_servidor).

Seleção de Idioma: Escolha "Português do Brasil" e clique em OK.

Licença: Leia e aceite os termos da licença.

Instalação: Clique em Instalar.

Verificação de Compatibilidade: O GLPI verificará se todas as extensões PHP estão instaladas e se as permissões de diretório estão corretas. Se você seguiu todos os passos, tudo deve estar verde. Clique em Continuar.

Configuração do Banco de Dados: Preencha os detalhes do banco de dados que criamos no Passo 2.

Servidor SQL: localhost

Usuário SQL: glpiuser

Senha SQL: sua_senha_forte

Clique em Continuar.

Seleção do Banco de Dados: Selecione glpidb na lista e clique em Continuar. A instalação irá criar todas as tabelas.

Finalização: A instalação está quase completa. A tela mostrará os usuários e senhas padrão. Anote-os!

glpi (super-admin)

tech (técnico)

normal (usuário normal)

post-only (apenas para abrir chamado)

Clique em Usar o GLPI.

Passo 9: Tarefas Críticas Pós-Instalação
Por segurança e funcionalidade, estas etapas são obrigatórias.

Remova o arquivo de instalação:

Bash

sudo rm /var/www/html/glpi/install/install.php
Altere as senhas padrão: Faça login com o usuário glpi e altere imediatamente a senha de todos os usuários padrão em Administração > Usuários.

Configure o Cron Job do GLPI: Isso é essencial para tarefas automáticas como recebimento de e-mails, notificações, etc.
Abra o editor de cron para o usuário do servidor web.

Bash

sudo crontab -e -u www-data
Adicione a seguinte linha no final do arquivo:

Snippet de código

* * * * * /usr/bin/php /var/www/html/glpi/front/cron.php &>/dev/null
Isso executará as tarefas automáticas a cada minuto.

Conclusão
Se você seguiu todos os passos, agora você tem uma instalação funcional e segura do GLPI. O próximo passo recomendado é configurar um certificado SSL/TLS (usando Let's Encrypt, por exemplo) para proteger a comunicação com seu servidor. Explore o menu de Administração > Configuração Geral para começar a personalizar o sistema para as necessidades da sua organização.
