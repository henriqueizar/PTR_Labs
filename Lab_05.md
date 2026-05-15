# Laboratório 05 - Roteamento Dinâmico com RIP e OSPF
## Objetivos
- Configurar interfaces IP em roteadores Cisco;
- Ativar roteamento dinâmico com RIP;
- Ativar roteamento dinâmico com OSPF;
- Verificar tabelas de roteamento;
- Validar conectividade entre redes remotas;
- Comparar as características básicas de RIP e OSPF.

# Topologia
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



# Configuração
## Hosts
<img width="584" height="588" alt="image" src="https://github.com/user-attachments/assets/376b9b43-e49b-49f4-8876-db9bf657244a" />

O endereço ip foi feito para os PCs, seguindo esse modelo, que está exmplificando o PC-BH-60-2
<img width="594" height="161" alt="image" src="https://github.com/user-attachments/assets/ae207957-9e65-45ed-a842-51aae5106938" />
IP ROUTE BH:
## Routers
Após isso, foi configuado cada um dos roteadores (RJ, SP, BH):
Exemplo configuração router-rj:
<img width="573" height="319" alt="image" src="https://github.com/user-attachments/assets/f3a207d3-bc85-4664-bd77-0a403d868060" />
EXEMPLO ROUTER BH:
Importante: O roteiro original especifica interfaces GigabitEthernet (g0/0, g0/1) e FastEthernet (f0/0, f0/1) para os roteadores. Entretanto, o modelo de roteador disponível no PNetLab possui apenas interfaces Ethernet genéricas (e0/0 a e0/3).

## Verificação inicial sem roteamento dinâmico
- conectividade apenas entre redes diretamente conectadas;
- ausência de rotas remotas;
- tabela de rotas contendo apenas rotas conectadas.
SHOW IP ROUTE:


Os outros Routers mostraram resultados similares.


## Configuração do RIP
Como o cenário usa a faixa 172.16.0.0/16, pode-se anunciar a rede maior 172.16.0.0 no processo RIP.
Exemplo configuração RIP no Router-RJ:
<img width="447" height="180" alt="image" src="https://github.com/user-attachments/assets/636d3ba6-4bc5-405f-8ebd-df1cbed3e952" />

## Verificação do RIP
A verficação foi feita em todos os roteadores. A seguir está a verificação do Router-SP, com função de sumarizar.
SHOW IP ROUTE(letra R indica caminho aprendido pelo RIP):

SHOW IP PROTOCOLS(RIP funcionando como esperado:

### Resultado Obtido
- presença de rotas C e L;
- ausência de rotas aprendidas dinamicamente;
- pings bem-sucedidos apenas para redes diretamente conectadas.

## Remoção do RIP
Em cada roteador foi executado:

<img width="221" height="130" alt="image" src="https://github.com/user-attachments/assets/8873421d-ef45-4e86-9b9c-70fb2062f40e" />

## Configuração do OSPF

<img width="633" height="205" alt="image" src="https://github.com/user-attachments/assets/f528ac01-e6d9-4692-81bb-11707baab22c" />
<img width="638" height="214" alt="image" src="https://github.com/user-attachments/assets/9a1eeaaf-7d00-46fe-82ea-c9e4d4059470" />
<img width="629" height="199" alt="image" src="https://github.com/user-attachments/assets/9b389dc4-6574-4ecb-b1c2-ca7c6735512a" />

## Verificação do OSPF
Novamente, a verficação foi feita em todos os roteadores. A seguir está a verificação do Router-SP, com função de sumarizar.
SHOW IP OSPF NEIGBOR(mostra vizinhos via OSPF):

SHOW IP ROUTE(Letra "O" indica caminho aprendido por OSPF:

SHOW IP PROTOCOLS(OSPF rodando como esperado - pode ser verificado por Distance = 110, comparado ao RIP que é 120):

### Resultado obtido
- adjacência OSPF formada entre RJ-SP e SP-BH;
- rotas aprendidas com marcação O;
- conectividade total entre todas as LANs.

# Testes
## Teste de alcance 
A partir de um host do Rio de Janeiro, testar conectividade com São Paulo e Belo Horizonte.
A partir do PC-RJ-10-1:

# Comparação orientada entre RIP e OSPF
Após concluir as duas configurações, responder:

1- Qual protocolo foi mais simples de configurar?
2.Qual protocolo apresentou maior riqueza de informações operacionais?
3. Qual a principal métrica do RIP?
4. Qual algoritmo é usado pelo OSPF?
5. Qual protocolo tende a escalar melhor?
6. Qual protocolo converge melhor em cenários maiores?

