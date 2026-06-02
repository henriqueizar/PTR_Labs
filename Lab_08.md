# Laboratório 08 - Políticas BGP e Integração com OSPF

**Disciplina:** ENE0025 - Protocolos de Transporte e Roteamento
**Professor responsável:** Prof. Dr. Laerte Peotta de Melo
**Monitores:** Victor Lima dos Santos / Beatriz Silva Nascimento

**Observação:** Este laboratório é continuação do **Laboratório 07**.

---

## Objetivo

Aplicar **políticas de roteamento BGP** e integrar o **BGP** ao **OSPF** sobre o cenário do Lab 07, realizando:

- anúncio do prefixo público da empresa (`200.18.245.64/27`)
- preferência de saída via **ISP1** (caminho principal) e **ISP2** como contingência (atributo `weight`)
- **OSPF** representando o domínio interno do **AS 1000**
- propagação controlada de apenas a **rota default** do BGP para o OSPF (`default-information originate`)
- análise de redundância entre provedores

---

## Premissas do cenário

### Diagrama lógico

```mermaid
flowchart LR
    %% ===== AS 1000 =====
    subgraph AS1000["AS 1000"]
        direction TB
        PFX1["200.18.245.64 /27"]
        SW1["SW1"]
        LAN1["192.168.0.0 /24"]
        R1["R1Loopback1 11.11.11.11 /32"]

        LAN1 --- SW1
        SW1 --- R1
        PFX1 --- R1
    end

    %% ===== AS 100 =====
    subgraph AS100["AS 100"]
        direction TB
        ISP1["ISP1Loopback1 10.10.10.10 /32"]
    end

    %% ===== AS 200 =====
    subgraph AS200["AS 200"]
        direction TB
        ISP2["ISP2"]
    end

    %% ===== AS 300 =====
    subgraph AS300["AS 300"]
        direction TB
        ISP3["ISP3"]
        P181["181.0.0.0 /8"]
        P182["182.0.0.0 /8"]
        P183["183.0.0.0 /8"]
        P184["184.0.0.0 /8"]
        P185["185.0.0.0 /8"]

        ISP3 --- P181
        ISP3 --- P182
        ISP3 --- P183
        ISP3 --- P184
        ISP3 --- P185
    end

    %% ===== Links =====
    R1 ---|10.1.0.0 /30  e0/1| ISP1
    R1 ---|10.1.0.4 /30  e0/2| ISP1
    R1 ---|10.2.0.0 /30  e0/3| ISP2
    ISP1 ---|191.1.0.0 /30| ISP3
    ISP2 ---|191.2.0.0 /30| ISP3

    classDef empresa fill:#dbeafe,stroke:#1d4ed8,color:#111827,stroke-width:2px;
    classDef isp1 fill:#dcfce7,stroke:#16a34a,color:#111827,stroke-width:2px;
    classDef isp2 fill:#fef3c7,stroke:#d97706,color:#111827,stroke-width:2px;
    classDef isp3 fill:#fee2e2,stroke:#dc2626,color:#111827,stroke-width:2px;
    classDef prefixo fill:#f3f4f6,stroke:#6b7280,color:#111827,stroke-width:1.5px;
    classDef lan fill:#e0f2fe,stroke:#0284c7,color:#111827,stroke-width:1.5px;

    class R1,SW1 empresa;
    class ISP1 isp1;
    class ISP2 isp2;
    class ISP3 isp3;
    class PFX1,P181,P182,P183,P184,P185 prefixo;
    class LAN1 lan;

    style AS1000 fill:#eff6ff,stroke:#1d4ed8,stroke-width:2px,stroke-dasharray: 8 6
    style AS100 fill:#f0fdf4,stroke:#16a34a,stroke-width:2px,stroke-dasharray: 8 6
    style AS200 fill:#fffbeb,stroke:#d97706,stroke-width:2px,stroke-dasharray: 8 6
    style AS300 fill:#fef2f2,stroke:#dc2626,stroke-width:2px,stroke-dasharray: 8 6
```

### Cabeamento

| Enlace        | Rede           | Interface R1 | Interface ISP |
|---------------|----------------|--------------|---------------|
| R1 ↔ SW1      | 192.168.0.0/24 | e0/0         | e0/0          |
| R1 ↔ ISP1 (1) | 10.1.0.0/30    | e0/1         | e0/0          |
| R1 ↔ ISP1 (2) | 10.1.0.4/30    | e0/2         | e0/1          |
| R1 ↔ ISP2     | 10.2.0.0/30    | e0/3         | e0/0          |
| ISP1 ↔ ISP3   | 191.1.0.0/30   | —            | e0/2 ↔ e0/0   |
| ISP2 ↔ ISP3   | 191.2.0.0/30   | —            | e0/1 ↔ e0/1   |

### ASNs e loopbacks

| Roteador | ASN  | Loopback         |
|----------|------|------------------|
| R1       | 1000 | 11.11.11.11/32   |
| ISP1     | 100  | 10.10.10.10/32   |
| ISP2     | 200  | —                |
| ISP3     | 300  | 181-185.0.0.1/8  |

---

## Etapa 1 — Configuração do OSPF interno no R1

O OSPF representa o domínio interno da empresa (LAN + loopback do R1).

```bash
R1> enable
R1# configure terminal
R1(config)# router ospf 10
R1(config-router)# router-id 11.11.11.11
R1(config-router)# network 192.168.0.0 0.0.0.255 area 0
R1(config-router)# network 11.11.11.11 0.0.0.0 area 0
R1(config-router)# end
```

---

## Etapa 2 — Configuração do BGP no R1

Sessões eBGP para **ISP1** (via loopback, com `ebgp-multihop`) e **ISP2** (vizinho direto). Inclui o `network` do bloco público e a rota Null0 que o BGP exige para originar a rede.

```bash
R1(config)# ip route 10.10.10.10 255.255.255.255 10.1.0.2
R1(config)# ip route 10.10.10.10 255.255.255.255 10.1.0.6
R1(config)# ip route 200.18.245.64 255.255.255.224 Null0

R1(config)# router bgp 1000
R1(config-router)# bgp router-id 11.11.11.11
R1(config-router)# neighbor 10.10.10.10 remote-as 100
R1(config-router)# neighbor 10.10.10.10 password SENHA
R1(config-router)# neighbor 10.10.10.10 ebgp-multihop 2
R1(config-router)# neighbor 10.10.10.10 update-source Loopback1
R1(config-router)# neighbor 10.2.0.2 remote-as 200
R1(config-router)# neighbor 10.2.0.2 password SENHA
R1(config-router)# network 200.18.245.64 mask 255.255.255.224
R1(config-router)# end
```

> O lab original usa `Serial2/0` nas estáticas — como o cenário no PNetLab está em **Ethernet**, foi usado **next-hop IP**, alinhando com a lição do Lab 07 (rotas estáticas sobre Ethernet devem ter next-hop IP).

---

## Etapa 3 — Política BGP: ISP1 principal, ISP2 backup

O atributo **`weight`** é local ao roteador Cisco e é o **primeiro** critério do BGP best-path. Atribuir `weight 200` ao vizinho ISP1 e `weight 100` ao vizinho ISP2 faz com que **todas** as rotas via ISP1 venham preferidas, independentemente do AS-PATH.

```bash
R1(config)# router bgp 1000
R1(config-router)# neighbor 10.10.10.10 weight 200
R1(config-router)# neighbor 10.2.0.2 weight 100
R1(config-router)# end
```

---

## Etapa 4 — Integração BGP × OSPF (apenas rota default)

Em vez de redistribuir os 5 prefixos externos no OSPF (poluindo o domínio interno), apenas a **rota default** é propagada. O `default-information originate` injeta uma LSA Type-5 anunciando `0.0.0.0/0` no OSPF.

```bash
R1(config)# ip route 0.0.0.0 0.0.0.0 10.10.10.10
R1(config)# router ospf 10
R1(config-router)# default-information originate
R1(config-router)# end
```

A default aponta recursivamente para `10.10.10.10` (loopback do ISP1); enquanto qualquer um dos dois enlaces físicos com o ISP1 estiver UP, a recursão resolve e a default fica viva.

---

## Verificação

Após concluir as configurações, executamos os comandos de verificação em cada roteador.

### R1

#### `show ip bgp summary` em R1

```
<img width="880" height="389" alt="image" src="https://github.com/user-attachments/assets/227efa20-c0d8-4619-85e5-851c101daeee" />

```
Mostra as duas vizinhanças (ISP1 via loopback `10.10.10.10` e ISP2 direto `10.2.0.2`) em estado **Established**, ambas com **6 prefixos recebidos** e Up/Down >7 min.

#### `show ip bgp` em R1

<img width="872" height="430" alt="image" src="https://github.com/user-attachments/assets/4b5898bb-169a-4522-b2d6-428a56b07998" />


Todas as rotas aparecem com **dois caminhos**: via ISP1 (AS-PATH `100 300`) e via ISP2 (`200 300`). O `*>` (best) está sempre no caminho via **ISP1**, comprovando a política weight. O prefixo `200.18.245.64/27` aparece originado localmente (`Weight 32768`).

#### `show ip route` em R1

<img width="874" height="497" alt="image" src="https://github.com/user-attachments/assets/a8c513d0-c6d0-419a-8d24-beb7d137c3eb" />


Confirma:

- `S* 0.0.0.0/0 via 10.10.10.10` — default route ativa.
- `S 10.10.10.10/32` com **dois next-hops** (`10.1.0.2` e `10.1.0.6`) — redundância L3 entre os dois enlaces físicos com o ISP1.
- 5 prefixos `B` (BGP) todos via `10.10.10.10` — política weight aplicada.
- `S 200.18.245.64 ... Null0` — rota dummy que sustenta o `network` no BGP.

#### `show ip ospf database` em R1

<img width="879" height="262" alt="image" src="https://github.com/user-attachments/assets/e5f56e48-ae07-4a8f-b781-7900b169b6ae" />


Apenas **uma LSA Type-5**, com Link ID `0.0.0.0` — a default route. Nenhum prefixo externo foi redistribuído. O domínio interno fica limpo, exatamente como a Etapa 4 propõe.

#### `show ip protocols | section ospf` em R1

<img width="877" height="317" alt="image" src="https://github.com/user-attachments/assets/616a43f3-fef0-4eba-9657-0389bd88e412" />

A linha `It is an autonomous system boundary router` confirma que o R1 virou ASBR (efeito direto do `default-information originate`).

#### `show running-config | section bgp` em R1

<img width="880" height="269" alt="image" src="https://github.com/user-attachments/assets/869f9329-f62e-4fec-bbf5-1c0095f215c7" />

Configuração final do BGP: política weight, sessão multihop por loopback e bloco público anunciado.

---

### ISP1

#### `show ip bgp summary` em ISP1

<img width="881" height="318" alt="image" src="https://github.com/user-attachments/assets/54acf6cf-21b3-4ec9-abc0-9aa912b2c866" />


Sessões com R1 (`11.11.11.11`) e ISP3 (`191.1.0.2`) em **Established**. Recebe **1** prefixo do R1 (`200.18.245.64/27`) e **5** prefixos do ISP3 (`181-185.0.0.0/8`).

#### `show ip bgp` em ISP1

<img width="800" height="321" alt="image" src="https://github.com/user-attachments/assets/00830395-65fd-4120-ace5-bfd9b23b033e" />

Origina sua loopback `10.10.10.10/32`, repassa os 5 prefixos do ISP3 e recebe o `/27` do R1.

#### `show ip route` em ISP1

<img width="891" height="411" alt="image" src="https://github.com/user-attachments/assets/84a50f51-9b71-45e9-80c5-6f72a22cd331" />

Rotas BGP (`B`) corretamente instaladas. As estáticas multipath para `11.11.11.11` sustentam a sessão eBGP multihop.

---

### ISP2

#### `show ip bgp summary` em ISP2

<img width="881" height="316" alt="image" src="https://github.com/user-attachments/assets/49d8e0ac-c092-4db4-8eac-636f42c604f8" />

Duas sessões eBGP UP. Recebe os 5 prefixos externos via ISP3 + a loopback do ISP1 + o `/27` da empresa.

#### `show ip bgp` em ISP2

<img width="880" height="469" alt="image" src="https://github.com/user-attachments/assets/85d17a76-485e-4eee-a744-833cfbb4a99f" />


Vê os prefixos externos por **dois caminhos** (direto via ISP3 ou indireto via R1→ISP1→ISP3) — o caminho direto vence por AS-PATH menor. Para o `/27` da empresa, o caminho direto (`10.2.0.1`) vence.

---

### ISP3

#### `show ip bgp summary` em ISP3

<img width="879" height="311" alt="image" src="https://github.com/user-attachments/assets/f65fa0bb-219b-4064-bd17-418008e4e5b2" />


Recebe **2** prefixos via ISP1 (`10.10.10.10/32` + `200.18.245.64/27`) e **1** via ISP2 (`200.18.245.64/27`).

#### `show ip bgp` em ISP3

<img width="808" height="347" alt="image" src="https://github.com/user-attachments/assets/9046324f-af44-4094-b093-fbc8998e5a8f" />


Origina os 5 prefixos `/8`, recebe a loopback do ISP1 e o `/27` da empresa **por ambos os caminhos** (via ISP1 e via ISP2).

#### `show ip bgp 200.18.245.64 255.255.255.224` em ISP3

<img width="883" height="305" alt="image" src="https://github.com/user-attachments/assets/433f996a-4f87-4183-9270-41111819492a" />
<img width="883" height="305" alt="image" src="https://github.com/user-attachments/assets/c7fbadec-9818-4d25-8ab9-ea4bf6694964" />

Comprova o anúncio **ponta-a-ponta** do bloco público: o `200.18.245.64/27` chega ao backbone (AS 300) pelos dois caminhos. AS-PATHs têm o mesmo tamanho (`100 1000` vs `200 1000`), e o desempate fica no **router-id menor** (`1.1.1.1` vence `2.2.2.2`), por isso o ISP3 elege a path via ISP1 como **best**.

## Teste de falha (failover)
### Falha de 1 enlace (shutdown e0/1)

<img width="872" height="435" alt="image" src="https://github.com/user-attachments/assets/3d32f08c-659f-4357-acee-2654e0635a01" />
<img width="871" height="62" alt="image" src="https://github.com/user-attachments/assets/182d12bd-afd3-45b6-86c6-e200ef9561b0" />



A sessão eBGP multihop sobreviveu: o loopback 10.10.10.10 continua alcançável pelo segundo enlace (10.1.0.6), nada migrou. Os 5 prefixos /8 ainda via ISP1, default igual. Esse é exatamente o ganho do desenho com sessão por loopback + 2 enlaces físicos.
### Falha total do ISP1 (shutdown e0/2 + clear ip bgp 10.10.10.10)

<img width="877" height="722" alt="image" src="https://github.com/user-attachments/assets/1ae167d5-c9f3-43ec-a5a2-a5927540dc27" />
<img width="876" height="627" alt="image" src="https://github.com/user-attachments/assets/d0014f39-c489-42cc-b27f-2edec676d325" />
<img width="880" height="374" alt="image" src="https://github.com/user-attachments/assets/99a88f28-6ada-4c94-8aa5-d4c53b59ad66" />

Failover BGP comprovado: sessão com 10.10.10.10 em estado Idle, todos os 5 prefixos /8 migraram para best path via 10.2.0.2 (ISP2), e a tabela de rotas já mostra B ... via 10.2.0.2. O ISP2 assumiu o tráfego de saída em poucos segundos.
Detalhe técnico que vale documentar: a rota estática S 10.10.10.10/32 via 10.1.0.6 e a default S* 0.0.0.0/0 via 10.10.10.10 continuaram listadas (o IOS demora a invalidar estáticas com next-hop IP quando a interface vai admin-down). Isso confirma na prática a recomendação do relatório: em produção, adicionar uma default backup com distância maior — ip route 0.0.0.0 0.0.0.0 10.2.0.2 250 — para que a rota default migre junto com o BGP quando o ISP1 cai.
### Recuperação (no shutdown em ambas)

<img width="3809" height="1911" alt="image" src="https://github.com/user-attachments/assets/6ccc3a2d-df4f-4008-89d8-93330c9ac368" />
<img width="3809" height="1911" alt="image" src="https://github.com/user-attachments/assets/7244b2c4-12ac-4364-a270-0ed10b554320" />
<img width="875" height="305" alt="image" src="https://github.com/user-attachments/assets/7e9bec4c-7584-4b44-b707-9ef757b0433b" />
<img width="875" height="305" alt="image" src="https://github.com/user-attachments/assets/3ecfae89-b271-4ae1-b81c-01285caff645" />

Reconvergência: em ~30 s a sessão BGP reabriu (Up/Down 00:00:34), o multipath estático para 10.10.10.10/32 voltou com os dois next-hops, e o weight 200 puxou o best path de volta para ISP1 (visível agora explicitamente na coluna Weight do show ip bgp). Estado idêntico à Fase A, com write memory salvo.

**O que observar:**

- O atributo `weight` é **local** ao R1 — só afeta a saída do AS 1000.
- O ISP1 continua sendo o caminho enquanto **qualquer** um dos dois enlaces físicos estiver ativo, graças ao multipath estático para a loopback `10.10.10.10`.
- Em queda total do ISP1, o ISP2 assume — mas a default propagada via OSPF é perdida junto, o que motiva a configuração de uma default backup como descrito acima.

---

## Observação sobre testes ICMP fim-a-fim

Ping para o **vizinho BGP** (`10.10.10.10`), que **é** anunciado e tem rota de retorno via estáticas multipath no ISP1, funciona com 100% success:

<img width="692" height="122" alt="image" src="https://github.com/user-attachments/assets/cde26aaa-ae94-415a-9520-f9169fc55a14" />

---

## Cenário final

Ao final do laboratório o cenário apresenta:

- sessão BGP entre **R1 e ISP1** (via loopback) em estado **Established**
- sessão BGP entre **R1 e ISP2** em estado **Established**
- sessão BGP entre **ISP1 e ISP3** em estado **Established**
- sessão BGP entre **ISP2 e ISP3** em estado **Established**
- 5 prefixos externos (`181-185.0.0.0/8`) na tabela do R1, todos via ISP1 (política weight)
- bloco público `200.18.245.64/27` anunciado e visto no AS 300 por ambos os caminhos
- LSA Type-5 (default) propagada no OSPF interno, **sem** poluição com os prefixos externos

---

## Respostas fas questões

1. **Papel do OSPF:** representar o domínio interno do AS 1000 (LAN + loopback), trocando prefixos internos e servindo como veículo para a rota default chegar nos elementos internos.
2. **Papel do BGP:** anunciar/receber prefixos entre sistemas autônomos (AS 1000 ↔ 100 ↔ 200 ↔ 300) e ser o ponto onde a política de saída é aplicada.
3. **Por que `200.18.245.64/27` é anunciado e `192.168.0.0/24` não:** o `/27` é o bloco público alocado à empresa (roteável na Internet); a `192.168.0.0/24` é RFC1918 (não roteável globalmente) — anunciá-la seria contraproducente.
4. **Vantagem de eBGP por loopback com ISP1:** sobrevive à perda de um dos dois enlaces físicos. Enquanto qualquer dos dois estiver ativo, a sessão continua UP.
5. **`update-source Loopback1`:** força o R1 a originar os pacotes BGP com endereço `11.11.11.11`, batendo com o `neighbor 11.11.11.11` configurado no ISP1.
6. **`ebgp-multihop 2`:** permite eBGP entre IPs **não diretamente conectados** (loopback R1 ↔ loopback ISP1) aumentando o TTL inicial para 2.
7. **Como ver qual ISP está preferido:** no `show ip bgp`, o caminho com `>` é o best — neste lab, todos os `181-185.0.0.0/8` mostram `*>` no next-hop `10.10.10.10` (ISP1).
8. **Por que só a default no OSPF:** mantém a LSDB enxuta, evita propagar mudanças da tabela de Internet para o interior e centraliza a política de saída no R1.
9. **Falha do ISP1 principal:** se cai 1 enlace, nada muda. Se caem os 2, a sessão eBGP morre, o ISP2 assume como best-path para todos os prefixos externos e a default original (recursiva via 10.10.10.10) some — daí a recomendação de adicionar uma default backup via ISP2.
10. **Diferença OSPF × BGP:** OSPF é IGP (link-state, dentro do AS) ideal para a rede interna; BGP é EGP (path-vector, entre ASes) e é o protocolo correto para a borda — cada um cumpre seu papel, e a integração entre eles deve ser seletiva.

---

## Conclusão

O laboratório demonstrou na prática como **políticas BGP** e **integração controlada com OSPF** materializam decisões operacionais comuns no mundo real:

- O atributo **`weight`** foi suficiente para implementar a política "ISP1 principal, ISP2 backup" sem nenhuma redistribuição extra.
- Sustentar a sessão eBGP por **loopback** com `ebgp-multihop 2` + multipath estático provou-se resistente à perda de um enlace físico.
- Propagar **apenas a default** para o OSPF mantém o domínio interno simples e centraliza a política no roteador de borda — evitando que prefixos da tabela de Internet sejam injetados no IGP.
- A integração **OSPF × BGP** ficou clara: cada protocolo cumpre o papel para o qual foi desenhado (IGP dentro, EGP fora) e a redistribuição é feita **seletiva e controlada**, não em massa.

As quatro vizinhanças BGP foram observadas em estado **Established**, a política `weight` foi confirmada pelo marcador `*>` em todos os prefixos externos, e a default route foi corretamente propagada no OSPF como uma única LSA Type-5 — completando todos os critérios de avaliação propostos.
