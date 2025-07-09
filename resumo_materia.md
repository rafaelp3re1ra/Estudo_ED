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
  default-information originate
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
