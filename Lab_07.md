# Laboratório 07 - Configuração dos Provedores (BGP Externo)

**Observação:** Este laboratório é continuação do **Laboratório 06**

## Objetivo

Configurar os roteadores **ISP1**, **ISP2** e **ISP3** para permitir o funcionamento completo do cenário de **BGP externo**.

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
        R1["R1"]

        SW1 --- R1
        LAN1 --- SW1
    end

    %% ===== AS 100 =====
    subgraph AS100["AS 100"]
        direction TB
        ISP1["ISP1"]
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
    end

    %% ===== Links =====
    R1 ---|10.1.0.0 /30| ISP1
    R1 ---|10.1.0.4 /30| ISP1 
    R1 ---|10.2.0.0 /30| ISP2
    ISP1 ---|191.1.0.0 /30| ISP3
    ISP2 ---|191.2.0.0 /30| ISP3

    %% ===== Cores dos nós =====
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

    %% ===== Estilo dos AS =====
    style AS1000 fill:#eff6ff,stroke:#1d4ed8,stroke-width:2px,stroke-dasharray: 8 6
    style AS100 fill:#f0fdf4,stroke:#16a34a,stroke-width:2px,stroke-dasharray: 8 6
    style AS200 fill:#fffbeb,stroke:#d97706,stroke-width:2px,stroke-dasharray: 8 6
    style AS300 fill:#fef2f2,stroke:#dc2626,stroke-width:2px,stroke-dasharray: 8 6

```

Considerando a topologia do laboratório 06:

- **R1** pertence ao **AS 1000**
- **ISP1** pertence ao **AS 100**
- **ISP2** pertence ao **AS 200**
- **ISP3** pertence ao **AS 300**

Os enlaces considerados são:

- **R1 ↔ ISP1**
  - rede **10.1.0.0/30**
  - rede **10.1.0.4/30**

- **R1 ↔ ISP2**
  - rede **10.2.0.0/30**

- **ISP1 ↔ ISP3**
  - rede **191.1.0.0/30**

- **ISP2 ↔ ISP3**
  - rede **191.2.0.0/30**

Também será utilizada a loopback:

- **ISP1:** `10.10.10.10/32`

E os prefixos externos anunciados por **ISP3**:

- `181.0.0.0/8`
- `182.0.0.0/8`
- `183.0.0.0/8`
- `184.0.0.0/8`
- `185.0.0.0/8`

---

## 3. Configuração do ISP3

Configure primeiro o **ISP3**, pois ele representa a rede externa com os prefixos de teste.

```bash
ISP3> enable
ISP3# configure terminal
ISP3(config)# no ip domain lookup
ISP3(config)# interface g0/0
ISP3(config-if)# ip address 191.1.0.2 255.255.255.252
ISP3(config-if)# no shut
ISP3(config-if)# interface g0/1
ISP3(config-if)# ip address 191.2.0.2 255.255.255.252
ISP3(config-if)# no shut
ISP3(config-if)# interface loopback 1
ISP3(config-if)# ip address 181.0.0.1 255.0.0.0
ISP3(config-if)# interface loopback 2
ISP3(config-if)# ip address 182.0.0.1 255.0.0.0
ISP3(config-if)# interface loopback 3
ISP3(config-if)# ip address 183.0.0.1 255.0.0.0
ISP3(config-if)# interface loopback 4
ISP3(config-if)# ip address 184.0.0.1 255.0.0.0
ISP3(config-if)# interface loopback 5
ISP3(config-if)# ip address 185.0.0.1 255.0.0.0
ISP3(config-if)# end
```

### BGP no ISP3

```bash
ISP3> enable

ISP3# configure terminal

ISP3(config)# router bgp 300

ISP3(config-router)# neighbor 191.1.0.1 remote-as 100

ISP3(config-router)# neighbor 191.1.0.1 password SENHA

ISP3(config-router)# neighbor 191.2.0.1 remote-as 200

ISP3(config-router)# neighbor 191.2.0.1 password SENHA

ISP3(config-router)# network 181.0.0.0 mask 255.0.0.0

ISP3(config-router)# network 182.0.0.0 mask 255.0.0.0

ISP3(config-router)# network 183.0.0.0 mask 255.0.0.0

ISP3(config-router)# network 184.0.0.0 mask 255.0.0.0

ISP3(config-router)# network 185.0.0.0 mask 255.0.0.0

ISP3(config-router)# end
```

---

## Configuração do ISP1

O **ISP1** possui dois enlaces físicos com o **R1**, além de uma loopback usada como vizinho BGP.

### Interfaces do ISP1

```bash
ISP1> enable
ISP1# configure terminal
ISP1(config)# no ip domain lookup
ISP1(config)# interface loopback 0
ISP1(config-if)# ip address 10.10.10.10 255.255.255.255
ISP1(config-if)# no shut
ISP1(config-if)# interface g0/0
ISP1(config-if)# ip address 10.1.0.2 255.255.255.252
ISP1(config-if)# no shut
ISP1(config-if)# interface g0/1
ISP1(config-if)# ip address 10.1.0.6 255.255.255.252
ISP1(config-if)# no shut
ISP1(config-if)# interface g0/2
ISP1(config-if)# ip address 191.1.0.1 255.255.255.252
ISP1(config-if)# no shut
ISP1(config-if)# end
```

### BGP no ISP1

```bash
ISP1> enable
ISP1# configure terminal
ISP1(config)# router bgp 100
ISP1(config-router)# neighbor 11.11.11.11 remote-as 1000
ISP1(config-router)# neighbor 11.11.11.11 password SENHA
ISP1(config-router)# neighbor 11.11.11.11 ebgp-multihop 2
ISP1(config-router)# neighbor 11.11.11.11 update-source Loopback0
ISP1(config-router)# neighbor 191.1.0.2 remote-as 300
ISP1(config-router)# neighbor 191.1.0.2 password SENHA
ISP1(config-router)# network 10.10.10.10 mask 255.255.255.255
ISP1(config-router)# exit
ISP1(config)# ip route 11.11.11.11 255.255.255.255 Ethernet0/0
ISP1(config)# ip route 11.11.11.11 255.255.255.255 Ethernet0/1
ISP1(config)# end
```

---

## Configuração do ISP2

O **ISP2** possui um enlace direto com o **R1** e um enlace com o **ISP3**.

### Interfaces do ISP2

```bash
ISP2> enable
ISP2# configure terminal
ISP2(config)# no ip domain lookup
ISP2(config)# interface g0/0
ISP2(config-if)# ip address 10.2.0.2 255.255.255.252
ISP2(config-if)# no shut
ISP2(config-if)# interface g0/1
ISP2(config-if)# ip address 191.2.0.1 255.255.255.252
ISP2(config-if)# no shut
ISP2(config-if)# end
```

### BGP no ISP2

```bash
ISP2> enable

ISP2# configure terminal

ISP2(config)# router bgp 200

ISP2(config-router)# neighbor 10.2.0.1 remote-as 1000

ISP2(config-router)# neighbor 10.2.0.1 password SENHA

ISP2(config-router)# neighbor 191.2.0.2 remote-as 300

ISP2(config-router)# neighbor 191.2.0.2 password SENHA

ISP2(config-router)# end
```

---


## Verificação

Após concluir as configurações:

```bash
Router# show ip bgp summary

Router# show ip bgp

Router# show ip route
```
### ISP1
<img width="882" height="499" alt="image" src="https://github.com/user-attachments/assets/b7df7c5a-0faa-4b93-ab1f-b74d953365b0" />
<img width="882" height="499" alt="image" src="https://github.com/user-attachments/assets/5f874458-5f09-4e53-97d7-723f06c16593" />
---

show ip bgp summary: Vizinhanças com R1 (11.11.11.11) e ISP3 (191.1.0.2) = Established (números 6 e 6)

show ip bgp: Todas as rotas aparecem, inclusive mostrando múltiplos caminhos para as redes 181-185.x:


* via 11.11.11.11 (vindo do R1)

*> via 191.1.0.2 (vindo do ISP3 — caminho escolhido)

Aparece também via R1→ISP2→ISP3 (path 1000 200 300)

### ISP2
<img width="875" height="646" alt="image" src="https://github.com/user-attachments/assets/6097883b-921c-47e1-bea9-a343abdb3264" />
<img width="883" height="643" alt="image" src="https://github.com/user-attachments/assets/cdbbee48-3d60-4cb6-a785-d1c3cf8bb4e6" />
---

show ip route: Rotas BGP (181-185.x e 200.18.245.64/27) marcadas com B ✅

show ip bgp summary: Vizinhanças com R1 (10.2.0.1) e ISP3 (191.2.0.2) = Established (números 1 e 6)

show ip bgp: Recebendo rotas do ISP3 e da empresa ✅

### ISP3
<img width="879" height="653" alt="image" src="https://github.com/user-attachments/assets/1aff097c-94e9-42fe-a6d6-b60795b2ff2d" />
<img width="878" height="493" alt="image" src="https://github.com/user-attachments/assets/cc460b54-88be-4fb9-8986-5b529e815e8d" />
---

show ip bgp summary: Vizinhanças com R1 (11.11.11.11) e ISP3 (191.1.0.2) = Established (números 6 e 6)

show ip bgp: Todas as rotas aparecem, inclusive mostrando múltiplos caminhos para as redes 181-185.x:


* via 11.11.11.11 (vindo do R1)
* 
*> via 191.1.0.2 (vindo do ISP3 — caminho escolhido)
  
Aparece também via R1→ISP2→ISP3 (path 1000 200 300)



---

## Cenário final:

Ao final, o cenário apresenta:

- sessão BGP entre **R1 e ISP1** em estado **Established**
- sessão BGP entre **R1 e ISP2** em estado **Established**
- sessão BGP entre **ISP1 e ISP3** em estado **Established**
- sessão BGP entre **ISP2 e ISP3** em estado **Established**

No **R1**:
### PROBLEMA ENCONTRADO:
<img width="800" height="590" alt="image" src="https://github.com/user-attachments/assets/8aed1af0-84a1-4cdb-ae3c-2c3136ee26d8" />
<img width="877" height="675" alt="image" src="https://github.com/user-attachments/assets/c838795c-148f-4e2d-8f8f-150b180b681a" />

A vizinhança BGP com ISP1 inicialmente permaneceu em estado Active devido a rotas estáticas duplicadas no ISP1: duas rotas corretas com next-hop IP (10.1.0.1 e 10.1.0.5) e duas rotas incorretas especificando apenas interface de saída (Ethernet0/0 e Ethernet0/1), causando falha na conectividade entre loopbacks.
A solução consistiu em remover as rotas estáticas configuradas apenas com interface de saída (no ip route 11.11.11.11 255.255.255.255 Ethernet0/0 e no ip route 11.11.11.11 255.255.255.255 Ethernet0/1), mantendo apenas as rotas com next-hop IP explícito.

<img width="809" height="456" alt="image" src="https://github.com/user-attachments/assets/66b0509b-cc75-4766-bb06-69d886cc8d39" />

Após a correção, o ping entre loopbacks passou a funcionar (success rate 100%) e a vizinhança BGP convergiu para o estado Established, permitindo a troca de prefixos entre R1 e ISP1.
Este problema reforça a importância de usar next-hop IP em rotas estáticas sobre redes Ethernet, conforme já observado no Lab 6.

<img width="788" height="234" alt="image" src="https://github.com/user-attachments/assets/e9f82419-34f5-4204-ac17-91c986ba7fb7" />


**show ip bgp summary:** em R1:
<img width="877" height="110" alt="image" src="https://github.com/user-attachments/assets/b70e6b5e-d388-438f-aa19-8e68762811bb" />

Mostra as duas vizinhanças (ISP1 e ISP2) em estado Established com números de prefixos recebidos.

**show ip bgp:** em R1:
<img width="870" height="488" alt="image" src="https://github.com/user-attachments/assets/33f88652-b19e-48e3-ab79-7307e5faa67f" />

Mostra todas as rotas aprendidas (181-185.x via ambos os ISPs) e o prefixo da empresa sendo anunciado.

**show ip route bgp:** em R1:
<img width="870" height="488" alt="image" src="https://github.com/user-attachments/assets/18277cf2-08d1-40ec-8cdd-4b0054c5b8c4" />

Mostra as rotas com marcação B instaladas na tabela, comprovando o multipath funcionando.

**show running-config | section bgp:** em R1:
<img width="526" height="301" alt="image" src="https://github.com/user-attachments/assets/13d869d5-6cc7-4a2a-8857-5364d40e6758" />

Mostra a configuração BGP completa: AS 1000, vizinhanças, network statements, multipath.

**show ip bgp | include 200.18.245.64** em ISP3:
Mostra que o prefixo da empresa chegou até o backbone (AS 300), demonstrando que o anúncio BGP funcionou ponta-a-ponta.



Como pode ser visto, aparecem rotas para:
- `181.0.0.0/8`
- `182.0.0.0/8`
- `183.0.0.0/8`
- `184.0.0.0/8`
- `185.0.0.0/8`

Nos provedores, aparece o prefixo da empresa:

- `200.18.245.64/27`

Observação sobre testes de conectividade fim-a-fim: Durante os testes adicionais, foram observados comportamentos inconsistentes de ping entre loopbacks distantes (R1 ↔ ISP3), apesar de todas as vizinhanças BGP estarem em estado Established, das rotas serem corretamente aprendidas e propagadas, e da conectividade entre nós adjacentes funcionar normalmente. Investigação extensiva apontou para possíveis limitações do emulador PNetLab no encaminhamento de pacotes em cenários BGP com múltiplos saltos e multipath habilitado.
