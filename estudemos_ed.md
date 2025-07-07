# Exame Recurso 2024 - 03/07/2024

## Pergunta 2

### Configuração da Interface `fa0/1`

```plaintext
conf t
interface fa0/1
 ip address 20.20.20.222 255.255.255.248
 no shutdown
 exit
```

### Cofiguração OSPF com autenticação MD5

```p
int fa0/1
ip ospf message-digest-key 1 md5 ED
```

### Configuração RIP com chave

```
key chain rip_key
key 1
key-string ED
exit
exit
int s0/3
ip rip authentication key-chain rip_key
exit
```

### Protocolos de encaminhamento

#### OSPF

```
router ospf 1
router-id 9.9.9.2
network 20.20.20.216 0.0.0.7 area 2
network 20.20.20.128 0.0.0.3 area 2
network 20.20.20.204 0.0.0.3 area 2
```

#### RIP

```
router rip
ver 2
network 20.20.20.126
no auto-summary
exit
```

### Redistribute

#### OSPF

```
router ospf 1
redistribute rip subnets
exit
```

#### RIP

```
router rip
redistribute ospf 1 metric 2
exit
```

### Rota para ISP

```
ip route 0.0.0.0 0.0.0.0 100.100.100.1
router ospf 1
default-information originate metric 10 metric-type 1
exit
```

## Pergunta 3

### Tráfego PC-PC através do router Rx

```
access-list 101 permit ip 20.20.20.208 0.0.0.7 20.20.20.216 0.0.0.7 -> para as sub-redes inteiras
access-list 101 permit ip host <ip> host <ip> -> unicast para unicast
route-map PBR_PCx_PCx permit 10
match ip address 101
set ip next hop <ip do router>
exit
inter fa0/1
ip policy route-map PBR_PCx_PCx
exit
```

## Pergunta 4

R5-R4 e R2-R4

```
router ospf <cr>
   area <aréa-comum> virtual-link <router-id-parceiro>
```

É preciso fazer isso nos 2 routers. Neste exame temos o R4 na área 2 e área 0 e o R5 e R2 na ára 1, logo em R2 e R5:

```
router ospf 1
   area 1 virtual-link 9.9.9.4
```

Para R4 seria igual mas com os id's do R2 e R5.


## Pergunta 5

- **O** – rota intra-área (mesma área)
- **O IA** – rota inter-área (entre áreas OSPF)
- **N1/N2** – rotas externas redistribuídas (tipo 1 ou 2)
- **C** – rotas diretamente conectadas
- **S** – rotas estáticas
- **R** – rotas aprendidas via RIP

Depois era ver o exercício 1 e colocar os IP's de acordo.


## Pergunta 6

Com uma prefix-list.

```
ip prefix-list BLOQUEAR_ROUTER_R6 seq 5 deny <prefixo>
ip prefix-list BLOQUEAR_ROUTER_R6 seq 10 permit 0.0.0.0/0 le 32

router rip
  distribute-list prefix BLOQUEAR_ROUTER_R6 in <interface>
```

## Pergunta 7

RIPv1 não suporta VLSM, o que iria causar um grande problema nas rotas ao ignorar as máscaras de rede, pois é classful. Também envia atualizações por broadcast, o que gera tráfego desnecessário. 

## Pergunta 8

Sim, deve estar desativada. Quando está ativada pode causar (e geralmente causa) perda de rotas específicas, rotas incorretas, esconder sub-redes menores.

```
no auto-summary
```

## Pergunta 9

- **Área Stub**:

  - Não aceita rotas externas (tipo 5)
  - Recebe rotas inter-área
- **Área Totally Stub**:

  - Não aceita nem rotas externas nem inter-área
  - Só recebe rota default
- Ativação:

  - No router interno:
    ```bash
    area <id> stub
    ```
  - No ABR (router de fronteira da área):
    ```bash
    area <id> stub no-summary
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

## Pergunta 10

### IPv6 com EIGRPv6

#### Planeamento:

- Cada ligação IPv6 deve ter um prefixo /64
- Endereçamento IPv6 pode usar prefixos ULA (`fd00::/8`) ou globais (`2001::/16`)

#### Ativação:

```bash
ipv6 unicast-routing

interface <interface>
  ipv6 address <prefixo>/<64>
  ipv6 eigrp <ASN>

ipv6 router eigrp <ASN>
  eigrp router-id <IPv4-formato>
  no shutdown
```

- Verificar rotas com:
  ```bash
  show ipv6 route
  ```
