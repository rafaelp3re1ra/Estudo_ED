# Guia Completo: OSPF, EIGRP, RIP v1/v2 e VLSM com Configurações

## 📁 Sumário

- Introdução ao Roteamento Dinâmico
- RIP v1 e v2
  - Características
  - Comandos de Configuração
  - Autenticação
- EIGRP
  - Características
  - Comandos de Configuração
  - Autenticação
- OSPF
  - Características
  - Comandos de Configuração
  - Autenticação
  - Virtual Links
  - Tipos de Rotas (E1, E2, N1, N2)
- VLSM
- Integração e Comparativo
- Configurações Específicas por Exemplo
  - Política de Roteamento (PBR)
  - Redistribuição EIGRP → OSPF
  - Ajuste de Métrica OSPF
  - Configuração de OSPFv3 IPv6

---

## 🌐 Introdução ao Roteamento Dinâmico

O roteamento dinâmico permite que os routers atualizem automaticamente as tabelas de roteamento com base em alterações na topologia da rede. Existem três protocolos principais: **RIP**, **EIGRP** e **OSPF**.

---

## 🗓️ RIP v1 e v2

### Características

| Versão | Classless | VLSM | Autenticação  | Tipo                | Métrica        |
| ------- | --------- | ---- | --------------- | ------------------- | --------------- |
| RIP v1  | Não      | Não | Não            | Vetor de distância | Saltos (max 15) |
| RIP v2  | Sim       | Sim  | Sim (MD5/Texto) | Vetor de distância | Saltos (max 15) |

### Comandos Básicos de Configuração

```bash
router rip
 version 2
 network 192.168.1.0
 no auto-summary
```

### Autenticação no RIP v2

```bash
interface FastEthernet0/0
 ip rip authentication mode md5
 ip rip authentication key-chain RIP_KEYS

key chain RIP_KEYS
 key 1
  key-string senha123
```

---

## 🚀 EIGRP (Cisco Proprietário)

### Características

- Vetor de distância avançado (DUAL)
- Rápida convergência
- Métrica composta (largura de banda, atraso, etc.)
- Suporte a VLSM/CIDR
- Usa Multicast 224.0.0.10

### Comandos Básicos de Configuração

```bash
router eigrp 100
 network 192.168.0.0
 no auto-summary
```

Para EIGRP IPv6:

```bash
ipv6 router eigrp 100
 router-id 1.1.1.1
 no shutdown

interface GigabitEthernet0/0
 ipv6 eigrp 100
```

### Autenticação

```bash
interface FastEthernet0/0
 ip authentication mode eigrp 100 md5
 ip authentication key-chain eigrp_keys

key chain eigrp_keys
 key 1
  key-string senha123
```

---

## 🔍 OSPF

### Características

- Protocolo de estado de enlace
- Suporta áreas (backbone é a área 0)
- Convergência muito rápida
- Métrica baseada em custo (bandwidth)
- Suporte a VLSM/CIDR
- Usa Multicast 224.0.0.5 e 224.0.0.6

### Comandos Básicos de Configuração

```bash
router ospf 1
 router-id 1.1.1.1
 network 192.168.1.0 0.0.0.255 area 0
```

Para OSPF IPv6:

```bash
ipv6 router ospf 1
 router-id 1.1.1.1

interface GigabitEthernet0/0
 ipv6 ospf 1 area 0
```

### Autenticação no OSPF

**Simples (plaintext)**

```bash
interface FastEthernet0/0
 ip ospf authentication
 ip ospf authentication-key senha123
```

**MD5**

```bash
interface FastEthernet0/0
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 senha123
```

### Virtual Links

```bash
router ospf 1
 area <intermediaria> virtual-link <router-id do destino>
```

### Tipos de Rotas

- **E1/E2**: rotas externas redistribuídas
- **N1/N2**: rotas externas em áreas NSSA
  - E1/N1: inclui custo interno OSPF
  - E2/N2: custo fixo (default = 20)

---

## 💰 VLSM (Variable Length Subnet Mask)

Permite o uso de sub-redes com tamanhos diferentes dentro da mesma rede principal.

### Exemplo:

Rede principal: `192.168.1.0/24`

- /30 para links ponto a ponto
- /28 para redes pequenas
- /24 para LANs grandes

---

## 🔄 Integração e Comparativo

| Protocolo | Classless | Suporta VLSM | Autenticação  | Tipo de Protocolo   | Métrica          |
| --------- | --------- | ------------ | --------------- | ------------------- | ----------------- |
| RIPv1     | Não      | Não         | Não            | Vetor de Distância | Saltos (max 15)   |
| RIPv2     | Sim       | Sim          | Sim (MD5/Texto) | Vetor de Distância | Saltos            |
| EIGRP     | Sim       | Sim          | Sim (MD5)       | Dist. Avançado     | Métrica Composta |
| OSPF      | Sim       | Sim          | Sim (MD5/Texto) | Estado de Enlace    | Custo (bandwidth) |

---

## ⚙️ Configurações Específicas por Exemplo

### Política de Roteamento (PBR)

```bash
access-list 101 permit ip host 192.168.x.x host 2.2.2.2
route-map PC7_SWEB permit 10
 match ip address 101
 set ip next-hop <IP do R4>

interface FastEthernet0/1
 ip policy route-map PC7_SWEB
```

### Redistribuição EIGRP → OSPF

```bash
router ospf 1
 redistribute eigrp 100 subnets metric 1000 1 255 1 1500
```

### Ajuste de Métrica OSPF

```bash
interface FastEthernet0/0
 bandwidth 10000000
```

### NSSA → Totally NSSA

```bash
router ospf 1
 area 1 nssa no-summary
```

### OSPFv3 para IPv6

```bash
interface GigabitEthernet0/0
 ipv6 address 2001:db8:1::1/64
 ipv6 ospf 1 area 0

ipv6 router ospf 1
 router-id 1.1.1.1
```

Área stub:

* É uma área que não recebe rotas externas (de fora do OSPF).
* Ela recebe uma rota padrão para fora (default-route), mas não as rotas externas detalhadas.
* Serve para reduzir o tamanho da tabela de roteamento dentro da área.
* Não pode ter ASBR dentro.

Área tottaly stub:

* É uma área ainda mais restrita que a stub.
* Além de não receber rotas externas, ela também não recebe rotas inter-áreas (de outras áreas OSPF).
* Só tem rotas internas da área + uma rota padrão para fora (default-route)
* A cisco permite configurar com o comando area x stub no-summary

Área NSSA (Not-so-stuby Area)

* É uma área stub que pode importar rotas externas (por exemplo, de um protocolo externo como o RIP ou BGP)
* Permite que dentro da área exista um ASBR para injetar rotas externas, mas elas são convertidas em rotas do tipo Type 7 LSAs, que depois viram Type % quando chegam ao backbone.
* Serve para quando que uma área "stub", mas ainda precisa de importar rotas externas.

Diferença entre rotas E1 e E2 no OSPF:

Rotas externas são aquelas que vêm de fora do domínio OSPF, injetadas por um ASBR.

* **E1 (External Type 1)**
  * Custo total = custo interno até o ASBR + custo externo da rota.
  * Ou seja, o custo leva em conta todo o caminho dentro do OSPF até o ASBR + o custo externo.
  * É mais preciso e usado quando a distância interna faz diferença.
  * Exemplo: se o caminho interno até o ASBR ficar mais longo, o custo total aumenta.
* **E2 (External Type 2)**
  * Custo = custo externo da rota (valor fixo, definido na origem).
  * O custo interno até o ASBR não é considerado.
  * É o padrão no OSPF para rotas externas.
  * Independente do caminho até o ASBR, o custo externo permanece o mesmo.

| Termo               | Descrição                                              |
| ------------------- | -------------------------------------------------------- |
| Stub Area           | Não recebe rotas externas, recebe rota padrão          |
| Totally Stubby Area | Não recebe rotas externas nem inter-áreas, só default |
| NSSA                | Área stub que pode ter ASBR e rotas externas importadas |
| E1 Route            | Custo = interno + externo (mais preciso)                 |
| E2 Route            | Custo = externo fixo (padrão, mais simples)             |

- **O** – rota intra-área (mesma área)
- **O IA** – rota inter-área (entre áreas OSPF)
- **N1/N2** – rotas externas redistribuídas (tipo 1 ou 2)
- **C** – rotas diretamente conectadas
- **S** – rotas estáticas
- **R** – rotas aprendidas via RIP
