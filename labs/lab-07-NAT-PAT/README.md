# Lab 07 — NAT/PAT (NAT Overload)

## Objetivo

Configurar e testar NAT/PAT em um roteador Cisco, permitindo que dispositivos de uma rede privada acessem uma rede externa utilizando o endereço IP da interface externa do roteador.

O laboratório simula um cenário semelhante ao de uma rede doméstica ou empresarial acessando a Internet através de um provedor.

## Conceitos praticados

- IPv4
- Endereços IP privados
- Endereços externos
- DHCP
- Gateway padrão
- Rota padrão
- NAT
- PAT
- NAT Overload
- Inside e Outside
- ACL utilizada pelo NAT
- Tabela de traduções NAT
- Comunicação entre redes
- Troubleshooting

## Topologia

```text
PC0 ──┐
      ├── Switch0 ── Router0 ── RouterISP ── Switch1 ── Server0
PC1 ──┘
```

O Router0 representa o roteador da rede interna.

O RouterISP representa, de forma simplificada, o roteador de uma operadora.

O Server0 representa um serviço localizado em uma rede externa.

## Redes utilizadas

### Rede interna

```text
192.168.10.0/24
```

### Enlace entre Router0 e RouterISP

```text
198.51.100.0/30
```

### Rede externa

```text
203.0.113.0/24
```

## Endereçamento

| Dispositivo | Interface | Endereço IP | Máscara | Gateway |
|---|---|---|---|---|
| PC0 | Fa0 | DHCP | 255.255.255.0 | 192.168.10.1 |
| PC1 | Fa0 | DHCP | 255.255.255.0 | 192.168.10.1 |
| Router0 | G0/0 | 192.168.10.1 | 255.255.255.0 | - |
| Router0 | G0/1 | 198.51.100.1 | 255.255.255.252 | - |
| RouterISP | G0/0 | 198.51.100.2 | 255.255.255.252 | - |
| RouterISP | G0/1 | 203.0.113.1 | 255.255.255.0 | - |
| Server0 | Fa0 | 203.0.113.10 | 255.255.255.0 | 203.0.113.1 |

## Configuração do Router0

### Interface da rede interna

```text
enable
configure terminal

interface gigabitEthernet 0/0
ip address 192.168.10.1 255.255.255.0
no shutdown
exit
```

Essa interface funciona como gateway padrão dos dispositivos da LAN.

### Interface externa

```text
interface gigabitEthernet 0/1
ip address 198.51.100.1 255.255.255.252
no shutdown
exit
```

Essa interface conecta o Router0 ao RouterISP.

## DHCP

O Router0 também foi utilizado como servidor DHCP para fornecer automaticamente configurações aos computadores da rede interna.

Foi criado um pool DHCP:

```text
ip dhcp pool LAN-INTERNA
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1
```

Os clientes recebem automaticamente:

- endereço IPv4;
- máscara de sub-rede;
- gateway padrão.

Endereços utilizados por equipamentos de infraestrutura podem ser excluídos da distribuição DHCP utilizando:

```text
ip dhcp excluded-address ENDERECO-INICIAL ENDERECO-FINAL
```

## Configuração do RouterISP

```text
enable
configure terminal

interface gigabitEthernet 0/0
ip address 198.51.100.2 255.255.255.252
no shutdown
exit

interface gigabitEthernet 0/1
ip address 203.0.113.1 255.255.255.0
no shutdown
end
```

O RouterISP possui conexão direta com:

```text
198.51.100.0/30
203.0.113.0/24
```

## Configuração do Server0

O servidor externo foi configurado com:

```text
IP Address:      203.0.113.10
Subnet Mask:     255.255.255.0
Default Gateway: 203.0.113.1
```

## Rota padrão

O Router0 conhece diretamente apenas as redes conectadas às suas interfaces.

Para alcançar outros destinos foi configurada uma rota padrão:

```text
configure terminal
ip route 0.0.0.0 0.0.0.0 198.51.100.2
end
```

A rota:

```text
0.0.0.0/0
```

é utilizada quando não existe uma rota mais específica para o destino.

O endereço:

```text
198.51.100.2
```

representa o próximo salto, neste caso o RouterISP.

## NAT

NAT significa:

```text
Network Address Translation
```

Sua função é realizar tradução de endereços IP entre redes.

Neste laboratório, os computadores utilizam endereços privados da rede:

```text
192.168.10.0/24
```

Ao acessar a rede externa, esses endereços são traduzidos pelo Router0.

## PAT / NAT Overload

PAT significa:

```text
Port Address Translation
```

É uma forma de NAT que permite que vários dispositivos internos compartilhem o mesmo endereço externo.

Nos equipamentos Cisco, essa configuração também é conhecida como:

```text
NAT Overload
```

O objetivo é permitir:

```text
vários IPs privados
        ↓
Router0
        ↓
um mesmo endereço externo
```

Neste laboratório, o endereço externo utilizado pelo Router0 é:

```text
198.51.100.1
```

## Definição das interfaces NAT

A interface conectada à LAN foi definida como lado interno:

```text
interface gigabitEthernet 0/0
ip nat inside
exit
```

A interface conectada ao RouterISP foi definida como lado externo:

```text
interface gigabitEthernet 0/1
ip nat outside
exit
```

Assim:

```text
192.168.10.0/24
        ↓
G0/0
NAT INSIDE

Router0

G0/1
NAT OUTSIDE
        ↓
RouterISP
```

## Seleção da rede interna

Foi utilizada uma ACL para identificar quais endereços internos podem sofrer tradução:

```text
access-list 1 permit 192.168.10.0 0.0.0.255
```

A wildcard:

```text
0.0.0.255
```

corresponde à rede:

```text
192.168.10.0/24
```

Neste laboratório, a ACL é utilizada apenas para selecionar o tráfego que será processado pelo NAT.

## Configuração do NAT Overload

A configuração utilizada foi:

```text
ip nat inside source list 1 interface gigabitEthernet 0/1 overload
```

A sintaxe pode ser interpretada da seguinte forma:

```text
ip nat inside source
→ traduz endereços de origem do lado interno

list 1
→ utiliza os endereços selecionados pela ACL 1

interface gigabitEthernet 0/1
→ utiliza o endereço da interface externa

overload
→ permite que vários dispositivos compartilhem esse endereço
```

## Fluxo da comunicação

Um computador da LAN pode possuir um endereço como:

```text
192.168.10.x
```

Quando ele acessa:

```text
203.0.113.10
```

o fluxo ocorre de forma simplificada:

```text
PC
192.168.10.x
        ↓
Router0
        ↓
NAT/PAT
        ↓
198.51.100.1
        ↓
RouterISP
        ↓
Server0
203.0.113.10
```

Para a rede externa, a comunicação é representada pelo endereço externo do Router0.

Quando a resposta retorna, o Router0 consulta sua tabela de traduções e entrega os dados ao dispositivo correto da rede interna.

## Verificação das traduções

Para visualizar a tabela NAT:

```text
show ip nat translations
```

O comando apresenta informações como:

```text
Inside local
Inside global
Outside local
Outside global
```

### Inside Local

Representa o endereço utilizado pelo dispositivo dentro da rede privada.

Exemplo:

```text
192.168.10.x
```

### Inside Global

Representa o endereço utilizado externamente para representar aquele dispositivo.

Neste laboratório:

```text
198.51.100.1
```

Uma tradução pode ser interpretada como:

```text
192.168.10.x
        ↓
NAT/PAT
        ↓
198.51.100.1
```

## ICMP e a tabela NAT

Os testes foram realizados utilizando `ping`, que utiliza o protocolo ICMP.

Por isso, na tabela NAT podem aparecer entradas semelhantes a:

```text
icmp
```

Os números apresentados após os endereços nas traduções ICMP são identificadores utilizados para distinguir as comunicações.

Eles não representam portas TCP ou UDP.

## Estatísticas NAT

Outro comando utilizado para verificação foi:

```text
show ip nat statistics
```

Ele permite verificar informações como:

- quantidade de traduções;
- interfaces internas;
- interfaces externas;
- hits;
- misses.

## Testes realizados

Primeiro foi testada a comunicação entre cada segmento da topologia.

Do PC para o gateway:

```text
ping 192.168.10.1
```

Do Router0 para o RouterISP:

```text
ping 198.51.100.2
```

Do RouterISP para o servidor:

```text
ping 203.0.113.10
```

Depois foi testada a comunicação completa:

```text
ping 203.0.113.10
```

A comunicação bem-sucedida, juntamente com a tabela de traduções, confirmou o funcionamento do NAT/PAT.

## Comandos de verificação

### Interfaces

```text
show ip interface brief
```

### Tabela de roteamento

```text
show ip route
```

### DHCP

```text
show ip dhcp binding
```

```text
show ip dhcp pool
```

### NAT

```text
show ip nat translations
```

```text
show ip nat statistics
```

## Troubleshooting

Caso a comunicação com a rede externa não funcione, verificar em ordem:

1. Configuração IP dos clientes.
2. Gateway padrão dos clientes.
3. Estado das interfaces.
4. Comunicação entre Router0 e RouterISP.
5. Comunicação entre RouterISP e Server0.
6. Existência da rota padrão no Router0.
7. Interface interna configurada com `ip nat inside`.
8. Interface externa configurada com `ip nat outside`.
9. ACL utilizada pelo NAT.
10. Regra de NAT Overload.
11. Tabela de traduções NAT.

Essa abordagem permite identificar o problema por etapas em vez de alterar várias configurações ao mesmo tempo.

## Boas práticas

- Testar cada segmento da rede antes de configurar NAT.
- Verificar a tabela de roteamento antes de investigar NAT.
- Definir corretamente as interfaces `inside` e `outside`.
- Utilizar comandos de verificação depois da configuração.
- Não utilizar NAT como substituto de firewall.
- Documentar as redes internas e externas utilizadas.
- Realizar troubleshooting de forma progressiva.

## Aprendizados

Neste laboratório foi possível compreender:

- por que endereços privados não são utilizados diretamente como endereços públicos na Internet;
- o papel do NAT na tradução de endereços;
- a diferença entre NAT e PAT;
- como vários dispositivos podem compartilhar um único endereço externo;
- o conceito de NAT Overload;
- como definir interfaces internas e externas;
- como uma ACL pode selecionar os endereços utilizados pelo NAT;
- como uma rota padrão direciona o tráfego para o provedor;
- como analisar a tabela de traduções NAT;
- como validar o funcionamento do NAT/PAT através de testes e comandos Cisco.
