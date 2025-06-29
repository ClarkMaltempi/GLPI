# Guia Completo: Instalação do GLPI em Servidor Linux (Ubuntu/Debian) com Apache ou Nginx

Este documento fornece um passo a passo detalhado para instalar e configurar o sistema de gerenciamento de chamados e inventário GLPI em um servidor Linux Ubuntu 22.04/24.04 LTS (ou Debian equivalente).

## Índice Navegável

* [Pré-requisitos](#pré-requisitos)
* [Passo 1: Atualização do Sistema](#passo-1-atualizacao-do-sistema)
* [Passo 2: Instalação do Banco de Dados (MariaDB)](#passo-2-instalação-do-banco-de-dados-(mariadb))
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

* 1.Instale o MariaDB:
