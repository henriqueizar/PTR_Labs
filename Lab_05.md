# Laboratório 05 - Roteamento Dinâmico com RIP e OSPF
## Objetivos
- Configurar interfaces IP em roteadores Cisco;
- Ativar roteamento dinâmico com RIP;
- Ativar roteamento dinâmico com OSPF;
- Verificar tabelas de roteamento;
- Validar conectividade entre redes remotas;
- Comparar as características básicas de RIP e OSPF.

## Topologia
A topologia possui três roteadores interligando três unidades:

Router-RJ - Rio de Janeiro
Router-SP - São Paulo
Router-BH - Belo Horizonte
Cada unidade possui duas LANs locais, e os roteadores são interligados por duas redes WAN.
```mermaid
flowchart LR
  %% =========================
  %% Roteadores
  %% =========================
  RJR["Router-RJ<br/>Rio de Janeiro"]
  SPR["Router-SP<br/>São Paulo"]
  BHR["Router-BH<br/>Belo Horizonte"]

  RJR ---|WAN 172.16.100.0/24| SPR
  SPR ---|WAN 172.16.200.0/24| BHR

  %% =========================
  %% Rio de Janeiro
  %% =========================
  subgraph RJ["Rio de Janeiro"]
    direction TB
    RJR
    SWRJ10["SW-RJ-10<br/>172.16.10.0/24"]
    SWRJ20["SW-RJ-20<br/>172.16.20.0/24"]

    RJR --- SWRJ10
    RJR --- SWRJ20

    VPCRJ1["VPC-RJ-1<br/>172.16.10.1"]
    VPCRJ2["VPC-RJ-2<br/>172.16.10.2"]
    VPCRJ3["VPC-RJ-3<br/>172.16.20.1"]
    VPCRJ4["VPC-RJ-4<br/>172.16.20.2"]

    SWRJ10 --- VPCRJ1
    SWRJ10 --- VPCRJ2
    SWRJ20 --- VPCRJ3
    SWRJ20 --- VPCRJ4
  end

  %% =========================
  %% São Paulo
  %% =========================
  subgraph SP["São Paulo"]
    direction TB
    SPR
    SWSP30["SW-SP-30<br/>172.16.30.0/24"]
    SWSP40["SW-SP-40<br/>172.16.40.0/24"]

    SPR --- SWSP30
    SPR --- SWSP40

    VPCSP1["VPC-SP-1<br/>172.16.30.1"]
    VPCSP2["VPC-SP-2<br/>172.16.30.2"]
    VPCSP3["VPC-SP-3<br/>172.16.40.1"]
    VPCSP4["VPC-SP-4<br/>172.16.40.2"]

    SWSP30 --- VPCSP1
    SWSP30 --- VPCSP2
    SWSP40 --- VPCSP3
    SWSP40 --- VPCSP4
  end

  %% =========================
  %% Belo Horizonte
  %% =========================
  subgraph BH["Belo Horizonte"]
    direction TB
    BHR
    SWBH50["SW-BH-50<br/>172.16.50.0/24"]
    SWBH60["SW-BH-60<br/>172.16.60.0/24"]

    BHR --- SWBH50
    BHR --- SWBH60

    VPCBH1["VPC-BH-1<br/>172.16.50.1"]
    VPCBH2["VPC-BH-2<br/>172.16.50.2"]
    VPCBH3["VPC-BH-3<br/>172.16.60.1"]
    VPCBH4["VPC-BH-4<br/>172.16.60.2"]

    SWBH50 --- VPCBH1
    SWBH50 --- VPCBH2
    SWBH60 --- VPCBH3
    SWBH60 --- VPCBH4
  end

  %% =========================
  %% Cores dos equipamentos
  %% =========================
  classDef router fill:#dbeafe,stroke:#1d4ed8,color:#111827,stroke-width:2px;
  classDef switch fill:#dcfce7,stroke:#16a34a,color:#111827,stroke-width:2px;
  classDef host fill:#fef3c7,stroke:#d97706,color:#111827,stroke-width:1.5px;

  class RJR,SPR,BHR router;
  class SWRJ10,SWRJ20,SWSP30,SWSP40,SWBH50,SWBH60 switch;
  class VPCRJ1,VPCRJ2,VPCRJ3,VPCRJ4,VPCSP1,VPCSP2,VPCSP3,VPCSP4,VPCBH1,VPCBH2,VPCBH3,VPCBH4 host;

  %% =========================
  %% Blocos dos estados
  %% =========================
  style RJ fill:transparent,stroke:#1d4ed8,stroke-width:2px,stroke-dasharray: 8 6
  style SP fill:transparent,stroke:#16a34a,stroke-width:2px,stroke-dasharray: 8 6
  style BH fill:transparent,stroke:#d97706,stroke-width:2px,stroke-dasharray: 8 6

```
