Class: [[Bootcamp]]
Date: 2026-11-04
Topics: #security 
Objetivo: 
- Compreender ataques de força bruta em diferentes serviços (FTP, Web, SMB);
- Utilizar o **Kali Linux** e o **Medusa** para auditoria de segurança em ambiente controlado;
- Documentar processos técnicos de forma clara e estruturada;
- Reconhecer vulnerabilidades comuns e propor medidas de mitigação;
- Utilizar o GitHub como portfólio técnico para compartilhar documentação e evidências.

Isso é basicamente oque eu fiz durante todo o curso, acho que comuntar isso fica simples, *só preciso lembrar de incluir imagens e trechos de codigo*

# Medusa e Kali Linux e a vulnerabilidade dos sistemas

## 1.0 Introdução

Essa é uma documentação que busca demonstrar como ataques de força bruta funcionam, e as diferentes maneiras de realiza-los. Para isso usarei a VirtualBox da Oracle, com o Kali Linux em conjunto com Nmap e Medusa além de usar o Metasploitable 2, e o DVWA para simular um ataque a formulario. Com o objetivo de demonstrar como a utilização de senhas fracas pode comprometer gravemente a segurança de sistemas 
## 1.1 Preparação do ambiente

Para prosseguir com os testes primeiro precisamos preparar o ambiente e para isso usei o **Metasploitable 2** e **Kali Linux**, após intala-los configurei a rede do Kali, fazendo os seguintes passos Configurações -> Network -> Adapter 2, apos isso eu clico em "Enable Network Adapter" e seleciono "Host-only adapter" em "Attached to". 
![[Configuração Rede.png]]
Assim eu permito que o Kali consiga se comunicar com o Metasploitable que sera o sistema a ser atacado, dessa maneira a configuração das VMs está finalizada e só precisamos abri-las 
## 2.0 Primeiros passos

Assim que as duas maquinas estiverem abertas, e importante que na maquina associada ao Metasploitable 2, apos efetuarmos o login e senha sendo ambos msfadmin, dentro do terminal efetuamos o seguinte comando 
``` 
ip a
```
Com esse sendo o resultado que esperamos estando localizado em, porque o etho0 diz respeito justamente a conexão entre o Kali e o Metasploitable 2
![[Configuração IP.png]]
Dessas informações só precisaremos lembrar, que será o endereço de IP atacado
```
inet 192.168.56.101
```
## 2.1 Utilizando o Nmap

Apos inicializar o Kali, no terminal efetuamos o seguinte comando 
```
sudo nmap -sV -p 21,22,80,445,139 192.168.56.101
```
O Nmap funciona para verificar no IP, quais portas estão abertas, os hosts ativos e os serviços disponiveis. Os parametros que passei funcionam da sequinte forma
- -sV: identifica os serviçoes e as versões das portas
- -p: especifica as portas que serão verificadas
- 21,22,80,445,139: lista de portas que serão analisadas
- 192.168.56.101: IP da maquina alvo
Com o resultado esperado sendo o seguinte![[Nmap.png]]Assim é possivel perceber que a porta ftp está aberta, oque é possivel confirmar usando o seguinte comando
![[FTP.png]]
Oque prova que a porta estava realmente aberta
## 3.0 Realizando testes de força bruta em FTP

Antes de efetuar o brute force, é necessario configurar quais usuarios e quais senhas a Medusa ira testar e para fazer isso usamos os seguintes comandos 
```
echo -e "usr\nmsfadmin\nadmin\nroot" > users.txt
```
```
echo -e "123456\nmsfadmin\npassword\nqwerty" > pass.txt 
```
os parametros passados funcionam da seguinte forma
- echo: exibe um texto no terminal 
- -e: permite a interpretação de sequencias especiais como \n
- "usr\nmsfadmin\nadmin\nroot" "123456\nmsfadmin\npassword\nqwerty": conteudo que será escrito no arquivo
- >:  redireciona o conteudo para o arquivo que sera criado
- users.txt e pass.txt: o nome dos arquivos que seram criados 
Apos essa configuração é possivel utilizar o medusa com o seguinte comando
```
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -t 6
```
No qual os parametros significam 
- -h: define o host alvo
- -U: o arquivo que possui a lista de usuarios
- -P: o arquivo que possui a lista de senhas
- -M: seleciona o modulo, nesse caso o ftp
- -t 6: define o numero de threads simultaneas por host, nesse caso 6
Assim quando a Medusa consegue achar um binario usuario e senha que funcionam, ele retorna dessa seguinte forma
![[Account found.png|520]]
E podemos verificar que isso realmente funciona, mostrando que as credenciais são veridicas e validadas pelo serviço
![[FTP Sucess.png|381]]

## 3.1 Realizando testes de autentificação em aplicações Web

Nesse exercicio o site Dvwa sera usado, e para acessa-lo dentro do FireFox, o endereço web sera
```
http://192.168.56.101/dvwa/login.php
```
![[DVWA.png]]Uma vez que acessado, preencho as infomações de Username e Password com quaisquer dados para que eu possa ver o que aparece quando vamos para area Inspecionar na pagina web, para isso precisamos ir para as opções de desenvolver clicando em Botão direito do mouse -> Inspect, apos isso clicamos em Network, apos isso é mostrado 
![[Inspecionar.png]]
Assim clico na primeira opção POST, e posteriormente em Request sendo possivel obter as seguintes informações
![[POST.png]]
Desse modo é possivel perceber que além dos campos que conseguimos ver que são Username e Password, o formulario ainda pede o Login, e usaremos isso a nosso favor.
Apos isso podemos usar os mesmos arquivos de senhas e usuarios que já foram criados durando a tentativa de invasão ftp, assim usamos o seguinte comando
```
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M http \
	-m PAGE:'/dvwa/login.php' \
	-m FORM:"username='USER'&password='PASS'&Login=Login" \
	-m 'FAIL=Login Failed' -t 6
```
No qual os seus parametros funcionam para
- -h: define o host alvo
- -U: o arquivo que possui a lista de usuarios
- -P: o arquivo que possui a lista de senhas
- -M: seleciona o modulo, nesse caso o http
- -m PAGE:'/dvwa/login.php': informa o caminho da pagina de login
- -m FORM:"username='USER'&password='PASS'&Login=Login": define os campos do formulario http
- -m 'FAIL=Login Failed': define a mensagem que é exibida quando uma tentativa falha
- -t 6: define o numero de threads simultaneas por host, nesse caso 6
Depois de usarmos esse comando o resultado positivo sera assim
![[SMB Sucess.png]]
Testanto para ver se as credenciais encontradas são realmente verdedeiras dentro do Dvwa, encontramos esse resultado
![[Resultado DVWA.png]]
comprovando que realmente as credenciais encontradas estavam corretas e são validadas pelo serviço
## 3.2 Realizando testes de Password Spraying em SMB

Nesse ataque em especifico não sera um brute force comum mão sim um password spraying que consiste em testar poucas senhas comuns em muitos usuarios. Desse modo diferentemente do brute force tradicional, que concentra o seu ataque em uma unica conta o password spraying tenta evitar mecanismos de bloqueio de acesso, que em geral são acionados após diversas tentativas de autentificação
Porém antes de continuarmos precisamos entender que o serviço SMB serve para compartilhar arquivos e recursos em redes internas, feito esses primeiros adendos é possivel iniciar o ataque
Portanto a primeira etapa para isso usei o enum4 para enumerar os usuarios, utilizando o seguinte comando
```
enum4linux -a 192.168.56.101 | tee enum4_output.txt
```
Onde os parametros funcionam para da seguinte forma
- -a: utiliza todas as formas de enumeração
- tee: grava o resultado no arquivo escolhido no caso enum4_output.txt
Apos isso para abrir o resultado, o seguinte comando é usado 
```
less enum4_output.txt
```
Sendo possivel encontrar o nome dos usuarios da seguinte forma
![[Password Spraying Result.png]]
Uma vez que possuimos os nomes do usuarios é possivel criar uma word list focada nesses usuarios, utilizando o comando 
```
echo -e "user\nmsfadmin\nservice\nroot" > smb_users.txt
```
E para a word list de senhas
```
echo -e "123456\npassword\nwelcome123\nmsfadmin" > senhas_spray.txt
```
Apos a criação dessas duas listas podemos usar a medusa para efeutar o ataque
```
medusa -h 192.168.56.101 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 6
```
Assim o resultado positivo sera da seguinte maneira
![[SMB Found.png]]
Outra coisa que é possivel notar e como o password spraying foi conduzido, uma vez que é possivel ver que uma mesma senha foi testada em varios usuario, como mostra a imagem
![[Password Spraying Result.png]]
Desse modo para verificarmos essa informação usamos o seguinte comando 
```
smbclient -L //192.168.56.101 -U msfadmin
```
E usamos a mesma senha encontrada pelo medusa, com o resultado sendo esse
![[SMB Result.png]]
Comprovando portanto que as credenciais são aceitas e validadas pelo serviço
## 4.0 Medidas de Segurança

O cerne para que sistemas e portas não sejam invadidos consistem em ações bem simples, como por exemplo utilizar senha mais complexas oque acaba por fazer com que ataques de brute force demorem muito mais para serem realizados uma vez que as possibilidades de senhas aumentam exponencialmente, outro cuidado necessario é sempre estar antenado quando a vazamentos de dados porque ataques reais utilizam listas de usuarios e senhas com varias palavras muitas vezes utilizando vazamentos para abastecer essas listas e as tornarem cada vez mais certeiras. Outro metodo que deve ser usado em combinação com senhas mais complesão é utilizar o MFA.
Tendo como foco aplicações Web uma maneira de aumentar a sua segurança é por meio de bloqueio temporario quando houverem muitas requisições simultaneas de modo a parecer suspeito e usar o CAPTCHA.
Pensando em serviços como FTP, uma medida de segurança e sempre desabilita-lo quando ele for desnecessario, com isso garantindo que a porta ligada a ele não fique aberta e acabe se tornando potencialmente um vetor de ataque.
Já se tratando de serviços SMB, o ideial e utilizar um firewall para restringir o seu acesso e aplicar politicas de autentificação mais rigidas, para inibir combinações de usuario e senha muito faceis de se descobrir

## 5.0 Referencias

As referencias das ferrementas utilizadas são as seguintes
- [Medusa – GitHub](https://github.com/jmk-foofus/medusa?utm_source=chatgpt.com)
- [Kali Linux](https://www.kali.org/docs/?utm_source=chatgpt.com)
- [Nmap Documentation](https://nmap.org/book/man.html?utm_source=chatgpt.com)
- [DVWA – GitHub](https://github.com/digininja/DVWA?utm_source=chatgpt.com)
- [Metasploitable 2 – Rapid7](https://docs.rapid7.com/metasploit/metasploitable-2/?utm_source=chatgpt.com)