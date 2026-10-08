# Lab 02 — Roteamento entre Duas Redes

## Objetivo

Este laboratório tem como objetivo demonstrar, na prática, a comunicação entre dispositivos pertencentes a duas redes IPv4 diferentes utilizando um roteador.

O laboratório foi desenvolvido no Cisco Packet Tracer e permite compreender como um computador identifica que o destino está fora de sua rede local, utiliza o gateway padrão e envia o tráfego para um roteador, que realiza o encaminhamento do pacote até a rede de destino.

Durante o laboratório foram aplicados conceitos fundamentais de redes, como:

- IPv4
- Máscara de rede
- Gateway padrão
- Interfaces de rede
- Endereço MAC
- ARP
- Ethernet
- Switch
- Roteador
- Tabela de roteamento
- ICMP
- Ping
- Cisco IOS

---

## Topologia

A infraestrutura utilizada possui duas redes locais conectadas por um roteador.

```text
Rede A — 192.168.10.0/24

PC0
192.168.10.10
Gateway: 192.168.10.1
        |
        |
     Switch0
        |
        |
     Router0
G0/0: 192.168.10.1/24
G0/1: 192.168.20.1/24
        |
        |
     Switch1
        |
        |
PC1
192.168.20.10
Gateway: 192.168.20.1

Rede B — 192.168.20.0/24
```

O roteador possui uma interface em cada rede e é responsável por realizar a comunicação entre elas.

---

## Endereçamento IP

| Dispositivo | Interface | Endereço IP | Máscara | Gateway |
|---|---|---:|---:|---:|
| PC0 | FastEthernet0 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| Router0 | GigabitEthernet0/0 | 192.168.10.1 | 255.255.255.0 | — |
| Router0 | GigabitEthernet0/1 | 192.168.20.1 | 255.255.255.0 | — |
| PC1 | FastEthernet0 | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 |

Foram utilizadas duas redes:

```text
192.168.10.0/24
192.168.20.0/24
```

Como os computadores pertencem a redes diferentes, eles não conseguem se comunicar diretamente apenas através de switches.

É necessário um dispositivo responsável pelo roteamento entre as redes.

---

## Máscara de Rede

As duas redes utilizam:

```text
255.255.255.0
```

que corresponde a:

```text
/24
```

Uma rede `/24` possui:

```text
32 bits IPv4
- 24 bits de rede
= 8 bits de host
```

Portanto:

```text
2^8 = 256 endereços totais
```

Em uma rede IPv4 tradicional:

```text
256 endereços
- 1 endereço de rede
- 1 endereço de broadcast
= 254 hosts utilizáveis
```

Por exemplo, na rede:

```text
192.168.10.0/24
```

temos:

```text
Rede:
192.168.10.0

Hosts:
192.168.10.1 até 192.168.10.254

Broadcast:
192.168.10.255
```

---

## Gateway Padrão

O gateway padrão é o endereço utilizado por um host quando ele precisa alcançar uma rede diferente da sua.

No PC0:

```text
IP:
192.168.10.10/24

Gateway:
192.168.10.1
```

O endereço:

```text
192.168.10.1
```

corresponde à interface `GigabitEthernet0/0` do roteador.

No PC1:

```text
IP:
192.168.20.10/24

Gateway:
192.168.20.1
```

O endereço:

```text
192.168.20.1
```

corresponde à interface `GigabitEthernet0/1` do roteador.

Portanto:

```text
PC0
↓
Gateway 192.168.10.1
↓
Router0
↓
Gateway/interface 192.168.20.1
↓
PC1
```

O gateway utilizado por cada computador precisa estar acessível dentro da própria rede local.

---

## Interfaces do Roteador

O roteador participa das duas redes através de duas interfaces diferentes.

```text
GigabitEthernet0/0
192.168.10.1/24
```

Essa interface pertence à rede:

```text
192.168.10.0/24
```

Já:

```text
GigabitEthernet0/1
192.168.20.1/24
```

pertence à rede:

```text
192.168.20.0/24
```

Uma interface de rede representa um ponto de conexão do dispositivo com uma rede.

Nesse laboratório, o roteador possui uma interface em cada rede e consegue encaminhar pacotes entre elas.

---

## Configuração do PC0

O PC0 foi configurado com:

```text
IP Address:
192.168.10.10

Subnet Mask:
255.255.255.0

Default Gateway:
192.168.10.1
```

Com essa configuração, o PC0 pertence à rede:

```text
192.168.10.0/24
```

---

## Configuração do PC1

O PC1 foi configurado com:

```text
IP Address:
192.168.20.10

Subnet Mask:
255.255.255.0

Default Gateway:
192.168.20.1
```

Com essa configuração, o PC1 pertence à rede:

```text
192.168.20.0/24
```

---

## Configuração do Roteador

A configuração do roteador foi realizada através da CLI do Cisco IOS.

### Acesso ao modo privilegiado

```bash
enable
```

O prompt muda de:

```text
Router>
```

para:

```text
Router#
```

O símbolo `#` indica o modo EXEC privilegiado.

---

### Acesso ao modo de configuração global

```bash
configure terminal
```

O prompt passa a ser:

```text
Router(config)#
```

---

## Configuração da interface G0/0

A interface conectada à rede `192.168.10.0/24` foi configurada com:

```bash
interface gigabitEthernet 0/0
ip address 192.168.10.1 255.255.255.0
no shutdown
```

### Explicação dos comandos

```bash
interface gigabitEthernet 0/0
```

Seleciona a interface que será configurada.

```bash
ip address 192.168.10.1 255.255.255.0
```

Define o endereço IPv4 e a máscara da interface.

```bash
no shutdown
```

Remove o estado de desligamento administrativo da interface.

---

## Configuração da interface G0/1

A segunda interface foi configurada para a rede `192.168.20.0/24`.

```bash
interface gigabitEthernet 0/1
ip address 192.168.20.1 255.255.255.0
no shutdown
```

Após essa configuração, o roteador passou a possuir conectividade direta com as duas redes.

---

## Verificação das Interfaces

Para verificar rapidamente o estado das interfaces foi utilizado:

```bash
show ip interface brief
```

Esse comando apresenta informações como:

```text
Interface
Endereço IP
Status
Protocol
```

O estado esperado para as interfaces utilizadas é:

```text
up
up
```

Por exemplo:

```text
GigabitEthernet0/0    192.168.10.1    up    up
GigabitEthernet0/1    192.168.20.1    up    up
```

Se uma interface aparecer como:

```text
administratively down
```

significa que ela está desativada administrativamente.

Nesse caso, deve ser utilizado:

```bash
no shutdown
```

---

## Switches

Os switches Cisco 2960 utilizados neste laboratório não precisaram de configuração adicional.

Eles funcionam como switches de camada 2 e são responsáveis por encaminhar quadros Ethernet dentro de cada rede local.

Neste laboratório:

```text
Switch0
```

conecta o PC0 à interface `G0/0` do roteador.

Já:

```text
Switch1
```

conecta a interface `G0/1` do roteador ao PC1.

Os switches aprendem automaticamente os endereços MAC observando os quadros que passam por suas portas.

---

## Endereço MAC

O endereço MAC é utilizado para identificar uma interface durante a comunicação dentro de uma rede local Ethernet.

Enquanto o endereço IP é utilizado para o endereçamento lógico e roteamento entre redes, o MAC é utilizado para a entrega do quadro naquele trecho da rede.

De forma simplificada:

```text
IP
→ identifica origem e destino lógico

MAC
→ identifica origem e destino no trecho Ethernet atual
```

---

## ARP

ARP significa:

```text
Address Resolution Protocol
```

Sua função no IPv4 é descobrir qual endereço MAC corresponde a determinado endereço IP dentro da rede local.

Por exemplo, o PC0 sabe que seu gateway é:

```text
192.168.10.1
```

Porém, para enviar um quadro Ethernet ao roteador, precisa conhecer o endereço MAC da interface correspondente.

O processo é semelhante a:

```text
PC0:
"Quem possui 192.168.10.1?"

        ↓

ARP Request

        ↓

Router0:
"Esse IP é meu.
Este é meu endereço MAC."

        ↓

ARP Reply
```

Depois dessa descoberta, a associação pode ser armazenada temporariamente na tabela ARP.

Essa tabela pode ser consultada utilizando:

```bash
arp -a
```

---

## Comunicação entre as Redes

Quando o PC0 deseja se comunicar com:

```text
192.168.20.10
```

ele precisa primeiro descobrir se esse endereço pertence à sua própria rede.

PC0 possui:

```text
192.168.10.10/24
```

Sua rede é:

```text
192.168.10.0/24
```

O destino é:

```text
192.168.20.10
```

que pertence à:

```text
192.168.20.0/24
```

Portanto:

```text
192.168.10.0/24
≠
192.168.20.0/24
```

O PC0 conclui que o destino está fora de sua rede local.

Nesse momento, utiliza seu gateway padrão:

```text
192.168.10.1
```

---

## Fluxo Completo da Comunicação

O processo de comunicação pode ser representado da seguinte forma:

```text
PC0 quer acessar 192.168.20.10
              |
              v
PC0 verifica sua própria rede
192.168.10.0/24
              |
              v
Percebe que 192.168.20.10
está em outra rede
              |
              v
Consulta sua rota padrão
              |
              v
Utiliza o gateway
192.168.10.1
              |
              v
Usa ARP para descobrir
o MAC do gateway
              |
              v
Monta um quadro Ethernet
destinado ao roteador
              |
              v
Switch0 recebe o quadro
              |
              v
Encaminha para Router0
              |
              v
Router0 recebe o quadro
              |
              v
Analisa o pacote IP
              |
              v
Consulta sua tabela de roteamento
              |
              v
Encontra a rede
192.168.20.0/24
              |
              v
Seleciona a interface G0/1
              |
              v
Descobre o MAC do PC1
caso necessário utilizando ARP
              |
              v
Cria um novo quadro Ethernet
              |
              v
Switch1 recebe
              |
              v
PC1 recebe o pacote
```

---

## Pacote IP e Quadro Ethernet

Um dos conceitos mais importantes observados neste laboratório foi a diferença entre endereçamento IP e endereçamento MAC.

No primeiro trecho:

```text
PC0 → Router0
```

o quadro possui aproximadamente:

```text
MAC origem:
MAC do PC0

MAC destino:
MAC da interface G0/0 do Router0
```

Dentro desse quadro existe um pacote IP:

```text
IP origem:
192.168.10.10

IP destino:
192.168.20.10
```

Depois que o roteador recebe o quadro, ele remove o encapsulamento Ethernet e analisa o pacote IP.

Para encaminhar para a segunda rede, cria um novo quadro.

No segundo trecho:

```text
Router0 → PC1
```

temos:

```text
MAC origem:
MAC da interface G0/1 do Router0

MAC destino:
MAC do PC1
```

Porém, o pacote continua representando:

```text
IP origem:
192.168.10.10

IP destino:
192.168.20.10
```

Portanto:

```text
MAC
→ muda conforme o trecho local

IP
→ representa origem e destino lógico da comunicação
```

---

## Tabela de Roteamento

A tabela de roteamento permite que o roteador determine por onde um pacote deve ser encaminhado.

Ela pode ser consultada utilizando:

```bash
show ip route
```

Como o roteador possui interfaces diretamente conectadas às duas redes, ele aprende automaticamente rotas para:

```text
192.168.10.0/24
```

e:

```text
192.168.20.0/24
```

De forma simplificada:

```text
192.168.10.0/24
→ GigabitEthernet0/0

192.168.20.0/24
→ GigabitEthernet0/1
```

No Cisco IOS, redes diretamente conectadas podem aparecer identificadas com:

```text
C
```

de:

```text
Connected
```

Quando chega um pacote destinado a:

```text
192.168.20.10
```

o roteador identifica que esse endereço pertence à rede:

```text
192.168.20.0/24
```

e encaminha o pacote pela interface correspondente.

---

## Rota Diretamente Conectada

Uma rede é considerada diretamente conectada quando o roteador possui uma interface configurada dentro daquela rede.

Por exemplo:

```text
G0/0:
192.168.10.1/24
```

faz com que o roteador saiba que está diretamente conectado à:

```text
192.168.10.0/24
```

Da mesma forma:

```text
G0/1:
192.168.20.1/24
```

faz com que conheça diretamente:

```text
192.168.20.0/24
```

Por esse motivo, não foi necessário configurar rotas estáticas neste laboratório.

---

## Testes de Conectividade

Antes de testar a comunicação completa entre PC0 e PC1, é uma boa prática validar a rede por etapas.

### Teste do PC0 até o gateway

```bash
ping 192.168.10.1
```

Esse teste valida:

```text
PC0
↓
Switch0
↓
Interface G0/0 do roteador
```

---

### Teste do PC1 até o gateway

```bash
ping 192.168.20.1
```

Esse teste valida:

```text
PC1
↓
Switch1
↓
Interface G0/1 do roteador
```

---

### Teste entre as duas redes

No PC0:

```bash
ping 192.168.20.10
```

O caminho percorrido é:

```text
PC0
↓
Switch0
↓
Router0
↓
Switch1
↓
PC1
```

O comando `ping` utiliza o protocolo ICMP para verificar conectividade IP.

---

## ICMP

ICMP significa:

```text
Internet Control Message Protocol
```

O `ping` utiliza mensagens ICMP para testar se outro host está acessível.

De forma simplificada:

```text
PC0
 |
 | ICMP Echo Request
 v
PC1

PC0
 ^
 | ICMP Echo Reply
 |
PC1
```

O recebimento do `Echo Reply` confirma que existe comunicação entre os dois hosts.

---

## Simulation Mode

O modo Simulation do Cisco Packet Tracer permite acompanhar o fluxo dos protocolos durante a comunicação.

Durante um ping entre as duas redes é possível observar o tráfego passando por:

```text
PC0
↓
Switch0
↓
Router0
↓
Switch1
↓
PC1
```

Também é possível visualizar protocolos como:

```text
ARP
ICMP
STP
```

O ARP participa da descoberta de endereços MAC.

O ICMP é utilizado pelo ping.

O STP é executado pelos switches para prevenir loops em redes Ethernet com caminhos redundantes.

Para estudar um fluxo específico, os filtros do Simulation Mode podem ser utilizados para reduzir a quantidade de protocolos exibidos.

---

## Comparação com Comunicação na Mesma Rede

Em uma comunicação dentro da mesma rede:

```text
PC A
192.168.10.10
        |
      Switch
        |
PC B
192.168.10.20
```

o PC A percebe que o destino pertence à sua própria rede.

Nesse caso:

```text
PC A
↓
ARP procura MAC do PC B
↓
Switch
↓
PC B
```

O gateway não é necessário.

Já entre redes diferentes:

```text
PC0
192.168.10.10

        ↓

Router0

        ↓

PC1
192.168.20.10
```

o PC0 percebe que o destino está fora da rede local.

Então:

```text
PC0
↓
Gateway
↓
Roteador
↓
Rede de destino
↓
PC1
```

Essa é uma das principais diferenças entre comunicação local e comunicação roteada.

---

## Principais Comandos Utilizados

### Entrar no modo privilegiado

```bash
enable
```

### Entrar no modo de configuração

```bash
configure terminal
```

### Selecionar uma interface

```bash
interface gigabitEthernet 0/0
```

### Configurar endereço IP

```bash
ip address 192.168.10.1 255.255.255.0
```

### Ativar uma interface

```bash
no shutdown
```

### Verificar interfaces

```bash
show ip interface brief
```

### Consultar a tabela de roteamento

```bash
show ip route
```

### Testar conectividade

```bash
ping <endereço-ip>
```

### Consultar tabela ARP

```bash
arp -a
```

---

## Boas Práticas Aplicadas

Durante o laboratório foram utilizadas algumas práticas importantes:

- Planejar o endereçamento IP antes da configuração.
- Utilizar redes diferentes para cada lado do roteador.
- Configurar o gateway de cada host dentro da própria rede.
- Validar primeiro a comunicação local com o gateway.
- Testar a comunicação completa somente depois das verificações locais.
- Utilizar comandos de diagnóstico antes de alterar configurações.
- Verificar o estado das interfaces utilizando `show ip interface brief`.
- Consultar a tabela de roteamento para entender as decisões do roteador.
- Utilizar o Simulation Mode para analisar o funcionamento dos protocolos.

---

## Troubleshooting

Caso a comunicação entre PC0 e PC1 não funcione, alguns pontos devem ser verificados.

### Endereço IP dos hosts

Confirmar se:

```text
PC0:
192.168.10.10/24

PC1:
192.168.20.10/24
```

---

### Gateway

Confirmar:

```text
PC0:
192.168.10.1

PC1:
192.168.20.1
```

---

### Interfaces do roteador

Utilizar:

```bash
show ip interface brief
```

As interfaces devem estar:

```text
up / up
```

---

### Endereços do roteador

Confirmar:

```text
G0/0:
192.168.10.1/24

G0/1:
192.168.20.1/24
```

---

### Testar por etapas

Primeiro:

```bash
ping 192.168.10.1
```

Depois:

```bash
ping 192.168.20.1
```

E somente depois:

```bash
ping 192.168.20.10
```

Essa abordagem ajuda a identificar em qual trecho da infraestrutura está o problema.

---

## O que Aprendi

Com este laboratório foi possível compreender como ocorre a comunicação entre dispositivos localizados em redes IPv4 diferentes.

O computador utiliza o endereço IP e a máscara de rede para determinar se o destino está dentro ou fora da rede local.

Quando o destino está fora da rede, o tráfego é encaminhado ao gateway padrão.

O gateway corresponde, neste cenário, ao endereço IP da interface do roteador presente na mesma rede local do host.

O ARP permite descobrir o endereço MAC necessário para realizar a entrega do quadro Ethernet dentro da rede local.

O switch utiliza endereços MAC para encaminhar quadros dentro da LAN.

O roteador utiliza endereços IP e sua tabela de roteamento para determinar por qual interface um pacote deverá seguir.

Também foi possível observar que os endereços MAC podem mudar a cada trecho Ethernet, enquanto os endereços IP continuam representando os hosts de origem e destino.

O fluxo principal estudado foi:

```text
Host identifica outra rede
↓
utiliza gateway
↓
ARP descobre MAC do gateway
↓
switch encaminha o quadro
↓
roteador recebe o pacote
↓
consulta tabela de roteamento
↓
seleciona interface de saída
↓
novo quadro Ethernet
↓
switch da rede de destino
↓
host de destino
```

---

## Tecnologias e Conceitos Utilizados

- Cisco Packet Tracer
- Cisco IOS
- IPv4
- CIDR
- Ethernet
- Endereço MAC
- ARP
- ICMP
- Switching
- Routing
- Gateway padrão
- Interfaces de rede
- Tabela de roteamento

---

## Próximos Passos

A partir deste laboratório, os próximos estudos podem incluir:

- Rotas estáticas
- Múltiplos roteadores
- Next Hop
- Rota padrão
- DHCP
- DNS
- VLANs
- Trunk
- Inter-VLAN Routing
- NAT
- ACLs
- Roteamento dinâmico
