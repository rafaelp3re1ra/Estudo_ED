# Estudo para Encaminhamento de Dados

# Distância Administrativa

Define o grau de confiança/preferênica da fonte de uma determinada rota.

0 - maior grau de preferência

255 - menor grau de preferência

| Protocolo                  | Custo |
| -------------------------- | ----- |
| RIP                        | 120   |
| Internal EIGRP (dentro)    | 90    |
| External EIGRP (para fora) | 170   |
| Static                     | 1     |
| Connected                  | 0     |

---

# RIP

## RIPv1

RIPv1 não utiliza máscaras, é _classfull_.

**Split horizon** - Nunca se deve informar um router pelo qual se aprendeu alguma coisa.

O RIPv1 não suporta autenticação, VLSM nem sumarização. Não se deve utilizar.

## RIPv2

RIPv2 utiliza máscaras, é _classless_ (apesar de não ser necessário atribuir as máscaras quando se configura).

Tem um limite máximo de 15 saltos para prevenir _routing loops_.

Atualização periódica de 30 segundos.

### Configuração do RIPv2

```
conf t
router rip
  version 2
  network <ip>
```

### _Redistribute_ EIGRP

```
conf t
router rip
  redistribute eigrp <processo> metric <número de saltos (RIP aceita no máximo 15)>
```

Quanto mais baixa for a métrica, mais atraente é a rota

### _Redistribute_ OSPF

```
conf t
router rip
  redistribute ospf <processo> metric <número de saltos (RIP aceita no máximo 15)>
```

Quanto mais baixa for a métrica, mais atraente é a rota

### _Default-information originate_

```
conf t
router rip
  default-information originate
```

Serve para saber que _router_ é que comunica com o exterior.

### _Passive-interface_

```
conf t
router rip
  passive-interface <interface>
```

Evita que se envie tráfego RIP para que não quer. Utiliza-se sempre à saída da empresa porque o RISP não precisa de saber do RIP.

Utiliza-se também em todos os routers que estejam ligados a PC's.

---

# EIGRP

Na tabela de roteamento aparece "D" porque o EIGRP pode ser chamado de \*dual.

###### **Nota:** No EIGRP o _passive-interface_ e _default-information originate_ funciona igual

## Configuração do EIGRP

```
conf t
router eigrp 100
  network <ip> <wildcard>
```

O "100" é o número do sistema autónomo (EIGRP). Todos os _routers_ no sistema em questão têm de ter o mesmo número. É como se fossem áreas do OSPF.

## Métrica e cálculo de rotas

- **S** - Sucessor - Vizinho preferencial para o destino
- **FD** - _Feasible Distance_ - Menor custo para o destino
- **RD** - _Reported Distance_ - Distância para o destino conforme comunicada pelo vizinho
- **FS** - _Feasible Successor_ - Router vizinho de backup para o destino (caso o sucessor falhe)

Muito resumidamente, numa rede com EIGRP um router escolhe o vizinho para quem vai enviar a infromação com base no RD e FD.

Para evitar potenciais loops, um FS tem de apresentar uma RD inferior à FD.

Um _router_ só é FS se RD < FD.

## _Redistribute_ RIP

```
router eigrp 100
  redistribute rip metric <bandwidth> <delay> <reliability> <load> <MTU>
```

## _Redistribute_ OSPF

```
router eigrp 100
  redistribute ospf <processo OSPF> metric <bandwidth> <delay> <reliability> <load> <MTU>
```

| Campo       | Valor típico | Significado                              |
| ----------- | ------------ | ---------------------------------------- |
| Bandwidth   | 10000        | largura de banda em kbps (ex: 10 Mbps)   |
| Delay       | 100          | atraso em decenas de microssegundos      |
| Reliability | 255          | confiabilidade (255 = 100%)              |
| Load        | 1            | carga (1 = baixa)                        |
| MTU         | 1500         | tamanho máximo de unidade de transmissão |

---

# OSPF

###### **Nota:** No OSPF o _passive-interface_ funciona da mesma maneira que os anteriores

### Tipos de _routers_ no OSPF

- **DR** **-** _Designated router_ - Este router é quem trata de enviar as tabelas de roteamento para os restantes _routers_ na rede de que fazem parte. É o _router_ eleito para ser o ponto central de comunicação entre os _routers_ numa rede OSPF. Todos os _routers_ enviam as suas LSA's (mensagem OSPF, "notícia" sobre os links na rede) para o DR e o DR envia de volta a tabela completa para eles. Isto evita que cada _router_ inunde a rede com as suas próprias tabelas (comunicam todos com todos), podendo causar inconsistências e _loops_, o que acabar por melhorar o desempenho da rede.
- **BDR -** _Backup Designated Router_ - O mesmo que o anterior, mas é de _backup_. Assume a posição de DR caso o DR original falhe.
- **DROther -** _Outro_ - Este tipo é os _routers_ que não são DR, BDR, ABR nem ASBR. São os _routers_ que estão dentro da área.
- **ABR -** _Area Border Router_ - _Router_ que fica na fronteira entre a área 0 e outra área OSPF. Pelo menos 1 porta tem de pertencer à área 0.
- **ASBR -** _Autonomous System Boundary Router_ - É o _router_ que fica na fronteira e faz a redistribuição de outros protocolos de encaminhamento para o OSPF.
- **BR -** _Backbone router_ - Tem todas as interfaces dentro da área 0.
- **IR -** _Internal router_ - Tem todas as interfaces dentro de uma área qualquer que não a área 0.

### Critérios de seleção do DR e BDR

Cada _router_ possui, como parâmetros OSPF, uma prioridade e um identificador (ID) que se apresenta sob a forma de um endereço IP.

DR é então o router com a prioridade MAIS elevada.

BDR é o segundo _router_ com a prioridade mais elevada.

Quando as prioridades são idênticas:

1. É eleito para DR o _router_ com o ID mais elevado.
2. É eleito para BDR o _router_ com o segundo ID mais elevado.
3. Quando os _routers_ não têm esse ID, é usado o maior IP das várias interfaces _loopback_ configuradas.
4. Quando não existe qualquer interface _loopback_ configurada, é

## Configuração do OSPF

```
conf t
router ospf 1
  router-id 9.9.9.x #x é, normalmente, o número do router em questão
  network <rede> <wildcard> area x
```

## Exemplo comando _show ip ospf neighbor_

```
R1# show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.0.0.2        1     FULL/DR         00:00:39     192.168.1.2     FastEthernet0/0
10.0.0.3        1     FULL/BDR        00:00:37     192.168.1.3     FastEthernet0/0
10.0.0.4        1     FULL/DROTHER    00:00:38     192.168.1.4     FastEthernet0/0
```

## _Virtual-links_

```
conf t
router ospf 1
  area x1 virtual-link 9.9.9.x2
```

Este comando deve ser utilizado nos 2 _routers_ a ligar. O "x1" é a área que ambos têm em comum. O "x2" é o id do _router_.

## Rotas OSPF

- **O E1 -** Rota externa redistribuída para o OSPF. Rotas externas do tipo 1 (E1) somam-se as distâncias do interior com as do exterior. LSA tipo 5 (_AS External_)
- **O E2 -** Rotas externas do tipo 2 (E2) só conta do exterior. - LSA tipo 5 (_AS External_)
- **O IA** **-** _Inter Area_ - Rotas de outras áreas OSPF. - LSA tipo 1 (_router_) e 2 (_network_) - Ex.: 2 _routers_ na área 0 a trocarem rotas entre si.
- **O -** _Intra Area_ - Rotas da mesma área. - LSA tipo 3 (_summary_) e 4 (_ASBR summary_) - Ex.: Uma rota da área 0 recebida na área 1.
- \***\*O N1/N2** **–\*\*** NSSA Externas: rotas redistribuídas dentro de uma área NSSA. Rota externa vinda de uma NSSA - LSA tipo 7 (convertido para tipo 5 pelo ABR se necessário)
- N1 e N2 têm as mesmas métricas que E1 e E2, respetivamente.

Ordem de escolha das rotas para os routers:

1. O
2. O IA
3. N1
4. E1
5. N2
6. E2

## Tipos de área OSPF

### Área _Stub_

Aceita rotas **O IA**, mas não propaga rotas externas.

Os _routers_ (todos) dentro de uma área _stub_ precisam de

```
conf t
router ospf 1
  area x stub
```

para definir a área como _stub_.

Estas áreas não podem servir _virtual-links._ A área em questão e os seus _routers_ não podem ter _virtual-links_ a passar por ela ou configurados nos _routers_ desta. Não pode ser uma área de trânsito de _virtual-links_.

### Área _totally stub_

Aqui não há rotas **IA**. Não são propagadas rotas externas.

_No ABR:_

```
conf t
router ospf 1
  area x stub no-summary
```

_Nos restantes routers da área:_

```
conf t
router ospf 1
  area x stub
```

### _Not So Stuby Area -> NSSA_

É uma área fechada, mas permite na mesma algum tipo de saída para o exterior.

Aceita rotas **_inter-area_ (IA)**. São propagadas rotas como N1/N2.

_Em todos os routers da área:_

```
conf t
router ospf 1
  area x nssa
```

## _Totally NSSA_

Igual à NSSA, mas não recebe rotas **inter-area (IA)**. Apenas recebe **rota default** e permite redistribuir rotas externas como N1/N2.

_No ABR:_

```
conf t
router ospf 1
area 1 nssa no-summary
```

_Nos restantes routers da área:_

```
conf t
router ospf 1
  area x nssa
```

| Tipo de Área     | Aceita rotas IA (inter-area) | Aceita rotas externas (E1/E2) | Redistribui rotas externas | Usa rota default    | Permite virtual-links | Comando (ABR)            |
| ---------------- | ---------------------------- | ----------------------------- | -------------------------- | ------------------- | --------------------- | ------------------------ |
| **Normal**       | ✅ Sim                       | ✅ Sim                        | ✅ Sim                     | ❌ Não (por padrão) | ✅ Sim                | Nenhum especial          |
| **Stub**         | ✅ Sim                       | ❌ Não                        | ❌ Não                     | ✅ Sim              | ❌ Não                | `area x stub`            |
| **Totally Stub** | ❌ Não                       | ❌ Não                        | ❌ Não                     | ✅ Sim              | ❌ Não                | `area x stub no-summary` |
| **NSSA**         | ✅ Sim                       | ❌ Não (E1/E2)                | ✅ Sim (como N1/N2)        | ✅ Sim              | ❌ Não                | `area x nssa`            |
| **Totally NSSA** | ❌ Não                       | ❌ Não (E1/E2)                | ✅ Sim (como N1/N2)        | ✅ Sim              | ❌ Não                | `area x nssa no-summary` |

# Autenticação dos protocolos

## RIP

## EIGRP

## OSPF

# _Prefix-list & PBR_
