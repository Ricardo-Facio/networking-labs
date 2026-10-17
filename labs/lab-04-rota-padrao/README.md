# Lab 04 — Rota Padrão

## Objetivo

Compreender e configurar uma rota padrão em um roteador Cisco, entendendo como um roteador encaminha pacotes para destinos que não possuem uma rota mais específica em sua tabela de roteamento.

## Conceitos praticados

- Endereçamento IPv4
- Máscara de sub-rede
- Gateway padrão
- Interfaces de roteadores
- Redes diretamente conectadas
- Tabela de roteamento
- Rotas estáticas
- Rota padrão
- Next hop
- Comunicação entre redes

## Topologia

PC0 --- Switch0 --- Router0 --- Router1 --- Switch1 --- Server0

Foram utilizadas três redes:

- Rede interna: `192.168.10.0/24`
- Rede entre os roteadores: `10.0.0.0/30`
- Rede externa simulada: `200.0.0.0/24`

## Endereçamento

| Dispositivo | Interface | Endereço IP | Máscara | Gateway |
|---|---|---|---|---|
| PC0 | Fa0 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| Router0 | G0/0 | 192.168.10.1 | 255.255.255.0 | - |
| Router0 | G0/1 | 10.0.0.1 | 255.255.255.252 | - |
| Router1 | G0/0 | 10.0.0.2 | 255.255.255.252 | - |
| Router1 | G0/1 | 200.0.0.1 | 255.255.255.0 | - |
| Server0 | Fa0 | 200.0.0.10 | 255.255.255.0 | 200.0.0.1 |

## Por que foi utilizada uma rede /30 entre os roteadores?

A comunicação entre Router0 e Router1 precisa de apenas dois endereços IP utilizáveis.

A rede:

`10.0.0.0/30`

possui quatro endereços:

- `10.0.0.0` — endereço da rede
- `10.0.0.1` — Router0
- `10.0.0.2` — Router1
- `10.0.0.3` — broadcast

Por isso, uma rede /30 é adequada para esse tipo de enlace ponto a ponto.

## Configuração do Router0

```text
enable
configure terminal

interface gigabitEthernet 0/0
ip address 192.168.10.1 255.255.255.0
no shutdown
exit

interface gigabitEthernet 0/1
ip address 10.0.0.1 255.255.255.252
no shutdown
end
