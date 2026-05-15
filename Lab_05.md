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
<img width="1381" height="518" alt="image" src="https://github.com/user-attachments/assets/1d40c32e-d460-46a3-a7dd-185fbfe123de" />



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
<img width="885" height="423" alt="image" src="https://github.com/user-attachments/assets/588a7ee5-9207-425f-906c-f1e18c6ff287" />

Importante: O roteiro original especifica interfaces GigabitEthernet (g0/0, g0/1) e FastEthernet (f0/0, f0/1) para os roteadores. Entretanto, o modelo de roteador disponível no PNetLab possui apenas interfaces Ethernet genéricas (e0/0 a e0/3).

## Verificação inicial sem roteamento dinâmico
- conectividade apenas entre redes diretamente conectadas;
- ausência de rotas remotas;
- tabela de rotas contendo apenas rotas conectadas.
SHOW IP ROUTE:

<img width="448" height="336" alt="image" src="https://github.com/user-attachments/assets/4e884f8f-07a8-4cc3-9329-e88ba795619e" />

<img width="810" height="445" alt="image" src="https://github.com/user-attachments/assets/d56b6f0b-401d-44b1-a95e-92e4be9ce614" />


Os outros Routers mostraram resultados similares.


## Configuração do RIP
Como o cenário usa a faixa 172.16.0.0/16, pode-se anunciar a rede maior 172.16.0.0 no processo RIP.
Exemplo configuração RIP no Router-RJ:
<img width="447" height="180" alt="image" src="https://github.com/user-attachments/assets/636d3ba6-4bc5-405f-8ebd-df1cbed3e952" />

## Verificação do RIP
A verficação foi feita em todos os roteadores. A seguir está a verificação do Router-SP, com função de sumarizar.
SHOW IP ROUTE(letra R indica caminho aprendido pelo RIP):

<img width="743" height="249" alt="image" src="https://github.com/user-attachments/assets/a1f4ebd8-5759-4d04-b518-ca643394154e" />


SHOW IP PROTOCOLS(RIP funcionando como esperado:

<img width="641" height="438" alt="image" src="https://github.com/user-attachments/assets/9f079a91-69c5-45f6-802d-c39d94a76e25" />


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

PING VPC-SP com as outras máquinas:

<img width="699" height="684" alt="image" src="https://github.com/user-attachments/assets/70290c65-21d7-4497-b525-7d028afe8eba" />


SHOW IP OSPF NEIGBOR(mostra vizinhos via OSPF):

<img width="780" height="118" alt="image" src="https://github.com/user-attachments/assets/2a39774c-63ff-4ff8-9c24-01a4e4921098" />


SHOW IP ROUTE(Letra "O" indica caminho aprendido por OSPF:

<img width="792" height="474" alt="image" src="https://github.com/user-attachments/assets/6a7514bc-191d-4a8e-a73c-2410033a635a" />


SHOW IP PROTOCOLS(OSPF rodando como esperado - pode ser verificado por Distance = 110, comparado ao RIP que é 120):

<img width="645" height="644" alt="image" src="https://github.com/user-attachments/assets/3ff9ed8d-6334-4b95-bb79-0d6df5202557" />


### Resultado obtido
- adjacência OSPF formada entre RJ-SP e SP-BH;
- rotas aprendidas com marcação O;
- conectividade total entre todas as LANs.

# Testes
## Teste de alcance 
A partir de um host do Rio de Janeiro, testar conectividade com São Paulo e Belo Horizonte.

A partir do PC-RJ-10-1:

<img width="613" height="723" alt="image" src="https://github.com/user-attachments/assets/5b91812b-da56-4276-af29-b77f851e5fe5" />


# Comparação orientada entre RIP e OSPF
Após concluir as duas configurações, responder:

## 1. Qual protocolo foi mais simples de configurar?
O RIP é mais simples de configurar, e isso é intencional, pois ele foi projetado para redes pequenas, diferentemente do OSPF, o qual é adequado para redes mais complexas.
## 2. Qual protocolo apresentou maior riqueza de informações operacionais?
O OSPF. Ele fornece comandos de diagnóstico muito mais detalhados:

**show ip ospf neighbor** mostra adjacências, estados, Dead Time, prioridades

<img width="613" height="723" alt="image" src="https://github.com/user-attachments/assets/22e32a45-fd2e-45d5-89ab-92acc5836542" />


**show ip ospf interface** detalha custos, tipos de rede, DR/BDR

<img width="795" height="500" alt="image" src="https://github.com/user-attachments/assets/9e7671f1-5b73-4416-931e-b1a7269b2b0a" />


**show ip ospf database** expõe a topologia completa conhecida (LSAs)

<img width="755" height="317" alt="image" src="https://github.com/user-attachments/assets/5d0546a9-6694-4bb1-ad67-b729ce387caf" />


O RIP basicamente só oferece **show ip protocols** e **show ip route**, com informações limitadas sobre contagem de saltos e timers. 

Essa diferença reflete a natureza dos protocolos: OSPF é link-state (cada roteador tem visão completa da topologia), enquanto RIP é distance-vector (cada roteador só conhece direção e distância, sem mapa topológico).
## 3. Qual a principal métrica do RIP?
O RIP escolhe rotas contando quantos roteadores o pacote precisa atravessar. Uma rede a 1 salto é preferida sobre uma a 2 saltos, independentemente da largura de banda, latência ou confiabilidade dos enlaces. Tem também um limite máximo de 15 saltos. Essa métrica simples é a maior limitação do RIP, porque ele pode escolher um caminho lento com 2 saltos em vez de um caminho rápido com 3 saltos.
## 5. Qual algoritmo é usado pelo OSPF?
**Dijkstra**
Cada roteador OSPF constrói um mapa completo da topologia e então executa o algoritmo de Dijkstra para calcular a árvore de caminhos mais curtos com ele mesmo como raiz. O "custo" de cada caminho é a soma dos custos das interfaces atravessadas, onde custo = 10^8 / largura de banda (em bps). Por exemplo, GigabitEthernet tem custo 1, FastEthernet tem custo 10. 
## 6. Qual protocolo tende a escalar melhor?
OSPF, pois ele não tem limite de saltos, su<img width="800" height="300" alt="4cac2632-2b45-4ecf-8b54-ee738e665b32" src="https://github.com/user-attachments/assets/2c4c3761-5555-4ce2-a2e7-565a83d601b0" />
porta hierarquia com áreas ( nesse experimento só foi usada a área 0, por ser introdutório), e tem rápida convergência.
## 7. Qual protocolo converge melhor em cenários maiores?
OSPF, como dito na última questão, tem uma rápida convergência. 

RIP: convergência lenta porque:

- Depende de propagação hop-by-hop (cada roteador espera receber atualização do vizinho)
- Updates periódicos a cada 30 segundos (não é sob demanda)
- Problemas como "counting to infinity" podem atrasar convergência

OSPF: convergência rápida porque:

- Detecta falhas via Hello packets (Dead Interval padrão = 40s, mas configurável para 1s)
- Propaga mudanças imediatamente via LSAs triggered updates (não espera timer)
- Cada roteador calcula independentemente (não depende de propagação sequencial)
- Em redes bem projetadas com áreas, mudanças em uma área não afetam outras

# Troubleshooting

## Situação 1 – Remover uma rede do anúncio OSPF em São Paulo
NO NETWORK:

<img width="739" height="146" alt="image" src="https://github.com/user-attachments/assets/4d096751-397a-4a55-bacd-6925a8943257" />


UNREACHABLE(perda de rota para a rede 172.16.40.0/24):

<img width="1006" height="190" alt="image" src="https://github.com/user-attachments/assets/10f5e9c1-6595-432a-8ea5-ccb17c8a7056" />


## Situação 2 – Interromper o enlace SP-BH
SHUTDOWN:
<img width="229" height="119" alt="image" src="https://github.com/user-attachments/assets/9dd5b15e-d333-447e-8eb1-bc5ed615265e" />
UNREACHABLE(perda de rota para BH - o teste foi feito de um PC do RJ pra um de BH):

<img width="780" height="464" alt="image" src="https://github.com/user-attachments/assets/b29998a7-7975-4116-ac88-2c5a91055cd4" />


## Situação 3 – Remover o RIP de um roteador
<img width="210" height="88" alt="image" src="https://github.com/user-attachments/assets/cfacc868-65ed-4641-9b1d-56ebecfee611" />
SHOW IP ROUTE:

<img width="999" height="190" alt="image" src="https://github.com/user-attachments/assets/7980c9d7-b5fb-4907-8871-5bd4ef076a50" />


A remoção do RIP via no router rip não gerou nenhum impacto, pois o protocolo já havia sido desativado anteriormente quando foi substituído pelo OSPF. Atualmente, toda a topologia opera somente com OSPF, pois ele é mais confiável (medido pela Distance = 110, comparado a Distance = 120 do RIP)
