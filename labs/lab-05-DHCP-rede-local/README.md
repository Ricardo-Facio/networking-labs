# Lab 05 — DHCP em uma Rede Local

## Objetivo

Configurar um roteador Cisco como servidor DHCP para fornecer automaticamente configurações de rede aos computadores de uma LAN.

## Conceitos praticados

- DHCP
- Cliente DHCP
- Servidor DHCP
- Endereço IP dinâmico
- Endereço IP estático
- Pool DHCP
- Exclusão de endereços
- Máscara de sub-rede
- Gateway padrão
- DNS
- Configuração de interfaces

## Topologia

PC0 \
PC1 --- Switch0 --- Router0
PC2 /

A rede utilizada foi:

`192.168.50.0/24`

## Endereçamento da infraestrutura

Router0:

- Interface: `G0/0`
- IP: `192.168.50.1`
- Máscara: `255.255.255.0`

Esse endereço também funciona como gateway padrão dos clientes DHCP.

## Configuração da interface do roteador

```text
enable
configure terminal

interface gigabitEthernet 0/0
ip address 192.168.50.1 255.255.255.0
no shutdown
end
