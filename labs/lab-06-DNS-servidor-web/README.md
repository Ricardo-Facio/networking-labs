# Lab 06 — DNS e Servidor Web Local

## Objetivo

Configurar uma rede local com DHCP, DNS e HTTP, permitindo que os computadores clientes recebam suas configurações de rede automaticamente e acessem um servidor web utilizando um nome em vez do endereço IP.

## Conceitos praticados

- DHCP
- DNS
- Registro A
- Endereço IP estático
- Endereço IP dinâmico
- Gateway padrão
- Servidor DNS
- Servidor HTTP
- Serviços de rede
- Portas
- Resolução de nomes
- Comunicação cliente-servidor

## Topologia

PC0, PC1 e PC2 estão conectados a um switch.

O switch está conectado ao Router0 e ao Server0.

```text
PC0 ──┐
PC1 ──┼── Switch0 ── Router0
PC2 ──┘       |
              |
           Server0
