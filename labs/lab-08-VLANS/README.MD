# Lab 08 — VLANs e Segmentação de Rede

## Objetivo

Criar e configurar VLANs em um switch Cisco para segmentar uma rede local em diferentes redes lógicas.

O laboratório demonstra como dispositivos conectados ao mesmo switch físico podem ser separados em VLANs diferentes, criando domínios de broadcast independentes.

## Conceitos praticados

- VLAN
- Switch gerenciável
- Segmentação de rede
- Portas access
- Domínio de broadcast
- Endereçamento IPv4
- Comunicação dentro da mesma VLAN
- Isolamento entre VLANs

## Topologia

Foi utilizado:

- 1 switch Cisco 2960
- 4 computadores

```text
PC0 ─ Fa0/1
PC1 ─ Fa0/2
          \
         Switch0
          /
PC2 ─ Fa0/3
PC3 ─ Fa0/4
```

## VLANs utilizadas

Foram criadas duas VLANs:

```text
VLAN 10 → TI
VLAN 20 → ADMIN
```

Cada VLAN representa uma rede lógica independente.

## Endereçamento

### VLAN 10 — TI

Rede:

```text
192.168.10.0/24
```

| Dispositivo | IP |
|---|---|
| PC0 | 192.168.10.10 |
| PC1 | 192.168.10.20 |

### VLAN 20 — ADMIN

Rede:

```text
192.168.20.0/24
```

| Dispositivo | IP |
|---|---|
| PC2 | 192.168.20.10 |
| PC3 | 192.168.20.20 |

Nenhum gateway foi configurado, pois este laboratório não utiliza roteamento entre VLANs.

## Criação da VLAN 10

No switch:

```text
enable
configure terminal

vlan 10
name TI
exit
```

O comando:

```text
vlan 10
```

cria ou acessa a VLAN 10.

O comando:

```text
name TI
```

define um nome para facilitar sua identificação.

## Criação da VLAN 20

```text
vlan 20
name ADMIN
exit
```

## Configuração das portas da VLAN 10

PC0 e PC1 foram conectados às portas Fa0/1 e Fa0/2.

As duas interfaces foram configuradas simultaneamente:

```text
interface range fastEthernet 0/1 - 2
switchport mode access
switchport access vlan 10
exit
```

### switchport mode access

Define a interface como uma porta access.

Portas access normalmente são utilizadas para conectar dispositivos finais e pertencem a uma única VLAN.

### switchport access vlan 10

Associa as interfaces selecionadas à VLAN 10.

Assim:

```text
Fa0/1 → VLAN 10
Fa0/2 → VLAN 10
```

## Configuração das portas da VLAN 20

```text
interface range fastEthernet 0/3 - 4
switchport mode access
switchport access vlan 20
exit
```

Resultado:

```text
Fa0/3 → VLAN 20
Fa0/4 → VLAN 20
```

## Verificação das VLANs

Para visualizar as VLANs configuradas:

```text
show vlan brief
```

Esse comando permite verificar:

- número da VLAN;
- nome;
- estado;
- interfaces associadas.

A configuração esperada é semelhante a:

```text
VLAN 10 → TI → Fa0/1, Fa0/2
VLAN 20 → ADMIN → Fa0/3, Fa0/4
```

## Testes de comunicação

### Comunicação dentro da VLAN 10

Do PC0:

```text
ping 192.168.10.20
```

A comunicação funciona porque PC0 e PC1 pertencem à mesma VLAN e à mesma sub-rede.

```text
PC0
VLAN 10
   ↓
Switch
   ↓
PC1
VLAN 10
```

## Comunicação dentro da VLAN 20

Do PC2:

```text
ping 192.168.20.20
```

PC2 e PC3 também conseguem se comunicar por pertencerem à mesma VLAN.

## Comunicação entre VLANs

Do PC0:

```text
ping 192.168.20.10
```

A comunicação falha.

Isso ocorre porque:

```text
PC0 → VLAN 10

PC2 → VLAN 20
```

As VLANs representam redes lógicas distintas e o switch utilizado neste laboratório não está realizando roteamento entre elas.

Para que VLANs diferentes se comuniquem, é necessário algum dispositivo realizando roteamento, como:

- roteador;
- switch Layer 3.

Esse conceito será explorado posteriormente em roteamento entre VLANs.

## Domínio de broadcast

Cada VLAN cria um domínio de broadcast separado.

Assim, um broadcast gerado na VLAN 10 permanece dentro da VLAN 10.

```text
VLAN 10

PC0
 ↓
broadcast
 ↓
PC1
```

O broadcast não é encaminhado automaticamente para dispositivos da VLAN 20.

Essa separação reduz o domínio de broadcast e permite uma maior organização lógica da rede.

## Uso de VLANs em empresas

VLANs são muito utilizadas para separar setores ou tipos de dispositivos sem exigir um switch físico diferente para cada grupo.

Exemplo:

```text
VLAN 10 → TI
VLAN 20 → Financeiro
VLAN 30 → Administrativo
VLAN 40 → Visitantes
VLAN 50 → Câmeras
```

Assim, uma mesma infraestrutura física pode suportar diversas redes lógicas.

## Troubleshooting

Caso dispositivos da mesma VLAN não consigam se comunicar:

1. Verificar os endereços IP.
2. Verificar a máscara de rede.
3. Confirmar as portas utilizadas no switch.
4. Executar:

```text
show vlan brief
```

5. Confirmar se as interfaces estão associadas à VLAN correta.
6. Verificar se as portas estão configuradas como access.

Caso dispositivos de VLANs diferentes não consigam se comunicar, isso é esperado neste laboratório, pois ainda não existe roteamento entre VLANs.

## Boas práticas

- Utilizar nomes claros para as VLANs.
- Documentar quais portas pertencem a cada VLAN.
- Manter cada VLAN associada a uma sub-rede IP própria.
- Evitar colocar todos os dispositivos em uma única VLAN.
- Utilizar VLANs para separar setores, funções e níveis de acesso.
- Validar a configuração utilizando comandos de verificação.

## Aprendizados

Neste laboratório foi possível compreender:

- o conceito de VLAN;
- como criar VLANs em um switch Cisco;
- como associar portas a VLANs;
- o funcionamento de portas access;
- como um switch físico pode ser dividido em várias redes lógicas;
- que cada VLAN cria um domínio de broadcast separado;
- que dispositivos da mesma VLAN conseguem se comunicar diretamente;
- que dispositivos de VLANs diferentes precisam de roteamento para se comunicar;
- como VLANs podem ser utilizadas para segmentar setores de uma empresa.
