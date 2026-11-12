NatureForce é um exercício de DFIR disponibilizado no site do [Malware Traffic Analysis](https://www.malware-traffic-analysis.net/). O exercício conta com algumas informações que vou apresentar da mesma forma que se encontram no site:

- 107.175.82[.]242:9000 - Windows executable (EXE) file sent from IP address over unusual TCP port
- 195.64.128[.]106:443 - CNCmachineRMS RAT C2 traffic
- LAN segment range: 10.10.1[.]0/24 (10.10.1[.]0 through 10.10.1[.]255)
- Domain: natureforce[.]com
- AD environment name: NATUREFORCE
- Active Directory (AD) domain controller: 10.10.1[.]10 - WIN-LEKBU2OY51N
- LAN segment gateway: 10.10.1[.]1
- LAN segment broadcast address: 10.10.1[.]255

Essas são as perguntas que o site espera que você responda durante a análise:

- What is the IP address of the infected Windows client?
- What is the MAC address of the infected Windows client?
- What is the host name of the infected Windows client?
- What is the user account name from the infected Windows client?
- What is the SHA-256 hash of the Windows EXE file sent from 107.175.82[.]242:9000 to the Windows client?

[Ínicio da análise]
Logo no início do sample, temos algumas informações importantes.
![[{E69AB915-3ED4-4FDE-B64A-1800E1C3B24B}.png]]
Um hostname e um endereço IP fornecido pelo servidor DHCP.
![[{DAF1837C-C3A8-4C75-ACA5-70B9D6A6A599}.png]]
Antes de começar a análise, gosto de filtrar algumas informações importantes, como as estatísticas de conversas e os protocolos que apresentam maior utilização. Vejo isso no menu de estatísticas do Wireshark.
Para que essas informações sirvam como resposta, precisamos seguir com a análise e sintetizar alguns fatos. Ao observar uma porção das conversas e dos protocolos, fica evidente que o IP 10.10.1.128 possui um volume enorme de tráfego, sem decoys ou outros hosts na rede. Dependendo de onde o tráfego foi capturado ou de como esse PCAP foi processado, isso costuma ser normal.  Deduzo que 10.10.1.128 seja o IP do host suspeito, juntamente com seu endereço MAC: 00:22:fb:ec:7e.

![[{E0209E8D-0F82-4BF0-A564-E4E32E3C04EC}.png]]
![[{2920DD90-44B5-42DC-A614-661E90FFEEC0}.png]]

Como 10.10.1.128 é o principal suspeito, voltando naqueles primeiros pacotes do DHCP, podemos concluir as 3 primeiras informações com a seguinte captura:
![[{FE2D28E1-D4D3-435F-9170-367116221CBA}.png]]

Existem alguns protocolos de interesse nessa captura.
![[{65A37B85-4714-4975-8D9E-E2701BB7567E}.png]]

Cada um deles pode fornecer informações consideravelmente importantes para obtermos uma visão melhor da infraestrutura e mapearmos os processos que acontecem nessa rede.
 Como o exercício pede para encontrar a conta de usuário usuária infectada, vou atrás de protocolos de interesse, que são aqueles que fornecem dados ou exigem algum mecanismo de autenticação. Até mesmo os protocolos de diretório, como o LDAP, poderiam ser interessantes.

Ao mapear os protocolos SMB, Kerberos e LDAP em busca de strings que possam conter nomes de usuários, não encontrei nada relevante no SMB e no LDAP. Portanto, foquei no Kerberos.
Analisando mais a fundo os pacotes de autenticação do Kerberos, verificamos alguns pacotes com a seguinte referência no campo `cname-string`. A partir disso, deduzimos que bdiaz seja a resposta para essa pergunta.
![[{7B787D49-EE47-4667-8B88-71CEA8AF3A8C}.png]]

A última pergunta é bem direta, conseguimos filtrar o fluxo de download HTTP (supondo que HTTP seja utilizado nessa comunicação). Filtramos primeiro pelo ip 107.175.82[.]242 e porta 9000.
![[{A7F4246C-664C-4243-A108-06558E2747E3}.png]]
Pela porta 9000, é enviado um executável. Utilizei a máquina virtual para calcular o sha256 desse possível IoC.

![[{FF67AB4E-F8FF-466A-B129-E2BF1F21EC26}.png]]

- **IP:** 10.10.1.128
- **MAC:** 00:22:FB:EC:7E
- **Hostname:** DESKTOP-4VGYQX7.natureforce.com
- **Usuário:** bdiaz (Kerberos Auth)
- **Arquivo:** runner_ilove.exe
- **SHA-256:** b789adce70c028a6d17be45ec34667aa95586350ee0c4057e75b138b1ad35061

