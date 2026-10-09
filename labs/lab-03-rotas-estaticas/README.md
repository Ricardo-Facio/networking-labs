# Lab 03 — Rotas Estáticas entre Dois Roteadores

## Objetivo

Este laboratório tem como objetivo demonstrar, na prática, como dois roteadores podem encaminhar pacotes entre redes que não estão diretamente conectadas utilizando rotas estáticas.

O laboratório foi desenvolvido no Cisco Packet Tracer e aprofunda conceitos de roteamento IPv4, incluindo:

- Rotas diretamente conectadas
- Rotas estáticas
- Next Hop
- Gateway padrão
- Interfaces de rede
- Tabela de roteamento
- IPv4
- Subnetting
- ARP
- ICMP
- Ping
- Tracert
- Cisco IOS

---

## Topologia

A infraestrutura possui duas redes locais conectadas através de dois roteadores.

```text
Rede A
192.168.10.0/24

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
G0/1: 10.0.0.1/30
        |
        |
        | Rede entre roteadores
        | 10.0.0.0/30
        |
        |
Router1
G0/0: 10.0.0.2/30
G0/1: 192.168.20.1/24
        |
        |
     Switch1
        |
        |
PC1
192.168.20.10
Gateway: 192.168.20.1

Rede B
192.168.20.0/24
```

---

## Redes Utilizadas

Foram utilizadas três redes IPv4:

```text
192.168.10.0/24
10.0.0.0/30
192.168.20.0/24
```

A rede:

```text
192.168.10.0/24
```

é utilizada pela LAN do PC0.

A rede:

```text
192.168.20.0/24
```

é utilizada pela LAN do PC1.

A rede:

```text
10.0.0.0/30
```

é utilizada exclusivamente para a comunicação entre os dois roteadores.

---

## Endereçamento IP

| Dispositivo | Interface | Endereço IP | Máscara | Gateway |
|---|---|---|---|---|
| PC0 | FastEthernet0 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| Router0 | G0/0 | 192.168.10.1 | 255.255.255.0 | — |
| Router0 | G0/1 | 10.0.0.1 | 255.255.255.252 | — |
| Router1 | G0/0 | 10.0.0.2 | 255.255.255.252 | — |
| Router1 | G0/1 | 192.168.20.1 | 255.255.255.0 | — |
| PC1 | FastEthernet0 | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 |

---

## Uso da Rede /30 entre os Roteadores

A rede:

```text
10.0.0.0/30
```

foi utilizada para conectar diretamente Router0 e Router1.

Uma rede `/30` possui 4 endereços totais:

```text
10.0.0.0 → endereço da rede

10.0.0.1 → Router0

10.0.0.2 → Router1

10.0.0.3 → broadcast
```

Como apenas dois dispositivos precisam utilizar endereços IP nesse enlace, a máscara `/30` é adequada para esse cenário.

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

O gateway corresponde à interface do Router0 presente na mesma rede local.

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

O gateway corresponde à interface do Router1 presente na mesma rede local.

---

## Configuração do Router0

A configuração foi realizada pela CLI do Cisco IOS.

### Acesso ao modo privilegiado

```bash
enable
```

### Acesso ao modo de configuração

```bash
configure terminal
```

### Interface conectada à LAN do PC0

```bash
interface gigabitEthernet 0/0
ip address 192.168.10.1 255.255.255.0
no shutdown
```

### Interface conectada ao Router1

```bash
interface gigabitEthernet 0/1
ip address 10.0.0.1 255.255.255.252
no shutdown
```

---

## Configuração do Router1

### Interface conectada ao Router0

```bash
interface gigabitEthernet 0/0
ip address 10.0.0.2 255.255.255.252
no shutdown
```

### Interface conectada à LAN do PC1

```bash
interface gigabitEthernet 0/1
ip address 192.168.20.1 255.255.255.0
no shutdown
```

---

## Verificação das Interfaces

Para verificar rapidamente o estado das interfaces foi utilizado:

```bash
show ip interface brief
```

O estado esperado das interfaces utilizadas é:

```text
Status: up
Protocol: up
```

---

## O Problema sem Rotas Estáticas

Após configurar todas as interfaces, Router0 conhece automaticamente apenas as redes às quais está diretamente conectado.

Router0 conhece:

```text
192.168.10.0/24

10.0.0.0/30
```

Porém, não conhece:

```text
192.168.20.0/24
```

Essa rede está atrás do Router1.

Da mesma forma, Router1 conhece:

```text
10.0.0.0/30

192.168.20.0/24
```

mas não conhece:

```text
192.168.10.0/24
```

Por isso, inicialmente, os roteadores não sabem como encaminhar pacotes entre as duas LANs.

---

## O que é uma Rota Estática

Uma rota estática é uma rota configurada manualmente pelo administrador.

Ela informa ao roteador:

```text
Para chegar a determinada rede
→ envie o pacote para determinado próximo salto
```

A sintaxe utilizada no Cisco IOS é:

```bash
ip route REDE_DESTINO MASCARA PROXIMO_SALTO
```

---

## Configuração da Rota Estática no Router0

No Router0 foi configurada uma rota para a rede:

```text
192.168.20.0/24
```

utilizando Router1 como próximo salto.

```bash
ip route 192.168.20.0 255.255.255.0 10.0.0.2
```

Esse comando significa:

```text
Rede de destino:
192.168.20.0/24

Próximo salto:
10.0.0.2
```

Ou seja:

```text
Para chegar à rede 192.168.20.0/24
→ envie o pacote para Router1
```

---

## Configuração da Rota Estática no Router1

No Router1 foi configurada a rota de retorno:

```bash
ip route 192.168.10.0 255.255.255.0 10.0.0.1
```

Isso significa:

```text
Rede de destino:
192.168.10.0/24

Próximo salto:
10.0.0.1
```

Agora Router1 sabe como retornar pacotes para a rede do PC0.

---

## Next Hop

Next Hop significa:

```text
Próximo salto
```

É o próximo roteador para o qual o pacote deve ser enviado.

No Router0:

```text
Destino:
192.168.20.0/24

Next Hop:
10.0.0.2
```

No Router1:

```text
Destino:
192.168.10.0/24

Next Hop:
10.0.0.1
```

O Next Hop não representa o destino final.

Ele representa apenas o próximo roteador no caminho.

---

## Tabela de Roteamento

A tabela de roteamento pode ser consultada utilizando:

```bash
show ip route
```

No Router0, de forma simplificada:

```text
C 192.168.10.0/24

C 10.0.0.0/30

S 192.168.20.0/24 via 10.0.0.2
```

No Router1:

```text
C 10.0.0.0/30

C 192.168.20.0/24

S 192.168.10.0/24 via 10.0.0.1
```

Onde:

```text
C
→ Connected

S
→ Static
```

Uma rota `C` é aprendida automaticamente porque a rede está diretamente conectada ao roteador.

Uma rota `S` foi configurada manualmente.

---

## Fluxo da Comunicação

Quando PC0 deseja acessar PC1:

```text
PC0
192.168.10.10
        |
        v
Destino:
192.168.20.10
        |
        v
Destino está fora
da rede local
        |
        v
Gateway:
192.168.10.1
        |
        v
Router0
        |
        v
Consulta tabela
de roteamento
        |
        v
Encontra:
192.168.20.0/24
via 10.0.0.2
        |
        v
Router1
        |
        v
192.168.20.0/24
diretamente conectada
        |
        v
PC1
192.168.20.10
```

---

## Caminho de Retorno

Para a comunicação funcionar, o pacote de resposta também precisa encontrar um caminho de volta.

O fluxo de retorno é:

```text
PC1
        |
        v
Gateway:
192.168.20.1
        |
        v
Router1
        |
        v
Rota estática:
192.168.10.0/24
via 10.0.0.1
        |
        v
Router0
        |
        v
PC0
```

Por isso é importante configurar tanto a rota de ida quanto a rota de retorno.

---

## Teste de Conectividade

Antes de testar a comunicação completa, a infraestrutura foi validada por partes.

### PC0 até Router0

```bash
ping 192.168.10.1
```

### Router0 até Router1

```bash
ping 10.0.0.2
```

### PC1 até Router1

```bash
ping 192.168.20.1
```

Depois das rotas estáticas configuradas:

```bash
ping 192.168.20.10
```

Esse teste valida o caminho completo:

```text
PC0
↓
Router0
↓
Router1
↓
PC1
```

---

## Tracert

O comando:

```bash
tracert 192.168.20.10
```

permite observar os saltos realizados até o destino.

O caminho esperado é aproximadamente:

```text
PC0
↓
Router0
↓
Router1
↓
PC1
```

No Linux, o equivalente comum é:

```bash
traceroute
```

---

## ARP

Mesmo com rotas estáticas, a comunicação Ethernet dentro de cada trecho continua utilizando ARP.

Por exemplo, PC0 precisa descobrir o MAC do gateway:

```text
192.168.10.1
```

Router0 também pode precisar descobrir o MAC da interface:

```text
10.0.0.2
```

e Router1 pode precisar descobrir o MAC do PC1.

Portanto:

```text
Roteamento
→ decide para onde o pacote deve seguir

ARP
→ descobre o MAC necessário para a entrega local
```

---

## Diferença entre os Laboratórios

### Lab 01

```text
PC
↓
Switch
↓
PC
```

Os dispositivos estavam na mesma rede.

Não havia necessidade de roteamento.

### Lab 02

```text
PC
↓
Switch
↓
Router
↓
Switch
↓
PC
```

Foram utilizadas duas redes e um roteador.

As duas redes estavam diretamente conectadas ao mesmo roteador.

### Lab 03

```text
PC
↓
Switch
↓
Router
↓
Router
↓
Switch
↓
PC
```

Agora existem dois roteadores.

Cada roteador conhece apenas suas redes diretamente conectadas.

Para alcançar redes remotas, foram utilizadas rotas estáticas.

---

## Principais Comandos Utilizados

### Modo privilegiado

```bash
enable
```

### Modo de configuração

```bash
configure terminal
```

### Configuração de interface

```bash
interface gigabitEthernet 0/0
```

### Configuração de IP

```bash
ip address ENDERECO MASCARA
```

### Ativação da interface

```bash
no shutdown
```

### Criação de rota estática

```bash
ip route REDE_DESTINO MASCARA NEXT_HOP
```

### Verificação das interfaces

```bash
show ip interface brief
```

### Verificação da tabela de roteamento

```bash
show ip route
```

### Teste de conectividade

```bash
ping ENDERECO_IP
```

### Visualização do caminho

```bash
tracert ENDERECO_IP
```

---

## Boas Práticas

Durante este laboratório foram aplicadas algumas boas práticas:

- Planejar o endereçamento IP antes da configuração.
- Utilizar uma rede específica para o enlace entre roteadores.
- Validar cada trecho da infraestrutura antes de testar a comunicação completa.
- Verificar a tabela de roteamento após criar uma rota.
- Configurar sempre um caminho de ida e retorno.
- Utilizar `show ip interface brief` para diagnóstico rápido.
- Utilizar `show ip route` para validar as decisões de roteamento.
- Utilizar `ping` e `tracert` para troubleshooting.

---

## Troubleshooting

Caso PC0 não consiga alcançar PC1, algumas verificações devem ser feitas.

### Verificar os gateways

```text
PC0:
192.168.10.1

PC1:
192.168.20.1
```

### Verificar interfaces

```bash
show ip interface brief
```

As interfaces utilizadas devem estar:

```text
up / up
```

### Testar o enlace entre roteadores

```bash
ping 10.0.0.2
```

ou no sentido contrário:

```bash
ping 10.0.0.1
```

### Verificar as rotas

```bash
show ip route
```

Router0 deve possuir rota para:

```text
192.168.20.0/24
```

Router1 deve possuir rota para:

```text
192.168.10.0/24
```

### Validar caminho de retorno

Uma rota de ida sem uma rota de retorno não é suficiente para completar a comunicação.

---

## O que Aprendi

Com este laboratório foi possível compreender como um roteador consegue alcançar redes que não estão diretamente conectadas a ele.

As redes diretamente conectadas são adicionadas automaticamente à tabela de roteamento quando as interfaces são configuradas.

Redes remotas precisam de um caminho conhecido.

Neste laboratório, esse caminho foi configurado manualmente através de rotas estáticas.

Também foi possível compreender o conceito de Next Hop, que representa o próximo roteador para o qual o pacote deve ser encaminhado.

Outro ponto importante foi entender que uma comunicação precisa possuir caminho tanto de ida quanto de retorno.

O fluxo principal estudado foi:

```text
Host envia para o gateway
↓
roteador consulta a tabela de roteamento
↓
encontra uma rota estática
↓
envia para o próximo salto
↓
segundo roteador recebe
↓
rede de destino diretamente conectada
↓
host de destino
```

---

## Tecnologias e Conceitos Utilizados

- Cisco Packet Tracer
- Cisco IOS
- IPv4
- CIDR
- Subnetting
- Ethernet
- ARP
- ICMP
- Switching
- Routing
- Static Routing
- Next Hop
- Gateway padrão
- Tabela de roteamento

---

## Próximos Passos

A partir deste laboratório, os próximos estudos podem aprofundar:

- Rota padrão
- Múltiplos roteadores
- Roteamento dinâmico
- RIP
- OSPF
- DHCP
- VLANs
- Inter-VLAN Routing
- NAT
