# Laboratório 03 - PIM-DM em topologia controlada

## Objetivos:
- montar uma topologia controlada de multicast no PNetLab;
- configurar interfaces IP em roteador e hosts;
- habilitar o roteamento multicast com ip multicast-routing;
- ativar o PIM-DM nas interfaces do roteador;
- gerar tráfego multicast de teste;
- verificar a operação básica do encaminhamento multicast;
- analisar a relação entre a tabela unicast e a tabela multicast;
- validar o funcionamento do grupo multicast configurado.

## Topologia

- *1 roteador* - Cisco IOL L3
- *2 switch Ethernet* - Cisco IOL L2 IRON
- *2 PCS* - Linux tinycore-6.4

<img width="568" height="424" alt="Captura de Tela 2026-04-27 às 16 35 11" src="https://github.com/user-attachments/assets/691731c4-62ac-41ac-a8d6-3448e52df6e1" />

<img width="640" height="192" alt="Captura de Tela 2026-04-27 às 17 38 20" src="https://github.com/user-attachments/assets/bf8974e9-a4cc-4815-9cc4-6d3ec39e692c" />

### Resultado da montagem

<img width="311" height="343" alt="Captura de Tela 2026-05-01 às 10 48 47" src="https://github.com/user-attachments/assets/a1606769-1f79-4098-a329-f8ee09b385e9" />

## Configuração do roteador
### Configuração básica das interfaces
Primeiramente foram configuradas as interfaces e0/0 e e0/1 do roteador

<img width="599" height="521" alt="Captura de Tela 2026-04-21 às 09 27 47" src="https://github.com/user-attachments/assets/61f36e62-61d0-4547-bad1-024d66e8e4eb" />

E posteriormente habilitado o multicast e o PIM-DM

<img width="599" height="261" alt="Captura de Tela 2026-04-21 às 09 28 59" src="https://github.com/user-attachments/assets/46cc26d6-770d-446c-bbe9-994d7fadba97" />

## Configuração dos hosts

### Origem

<img width="670" height="104" alt="image" src="https://github.com/user-attachments/assets/e46461e3-346b-48f5-9eee-1036ffc7f363" />

### Receptor

<img width="483" height="91" alt="image" src="https://github.com/user-attachments/assets/a85ec234-f954-4322-9f74-b82a839dc7e1" />


## Geração de tráfego multicast

### Problema encontrado:
Para essa etapa, é necessário que a máquina linux, tanto receptor quanto origem, tenha suporte para ferramentas como iperf. 

<img width="251" height="44" alt="image" src="https://github.com/user-attachments/assets/d977ff22-ca1e-4716-9bdc-9cc4740e238c" />

Como pode ser visto na imagem acima, a máquina Linux que havia sido escolhida (tinycore) não continha iperf instalado, e, depois de uma extensa busca por imagens diferentes de máquinas Linux(tinycore, webserver, debian, etc), foi constatado que no PNetLab, nenhuma máquina linux tinha o iperf instalado. Portanto, observou-se que seria necessário instalar o iperf na máquina:

<img width="789" height="218" alt="image" src="https://github.com/user-attachments/assets/87d2c633-4958-44f9-b1f8-17ef767b8333" />

No entanto, houve um erro quanto a resolução do archive.ubuntu.com, e isso era esperado, já que não há acesso externo à internet. Portanto, foi necessáro contornar esse problema, conectando um módulo NET diretamente nas máquinas, para entao conseguir instalar o iperf, por meio do comando sudo apt install iperf iperf3 socat -y.

### Módulo NET

<img width="473" height="452" alt="image" src="https://github.com/user-attachments/assets/5140739d-2d9b-423e-b0af-a07d6605b55d" />

E depois rodou-se o comando mencionado:

<img width="604" height="287" alt="image" src="https://github.com/user-attachments/assets/e9a3ca26-7617-486e-a412-a2b91e17a55f" />

E o mesmo foi feito no receptor. Com isso, o iperf foi devidamente instalado, e o multicast poderá ser testado.

### Testes Multicast

Receptor pronto e esperando: 


<img width="425" height="126" alt="image" src="https://github.com/user-attachments/assets/979c56f1-2367-4345-a7be-bb02ee0403f1" />

SUCESSO! O cliente está conectado ao grupo 239.1.1.1 e enviando dados a uma taxa constante, de 1.05 Mbits/s, aproximadamente:
<img width="539" height="319" alt="image" src="https://github.com/user-attachments/assets/bc7b5310-4c2d-4e89-8377-1106e276c58e" />

## Comandos de verificação no roteador:


<img width="539" height="319" alt="image" src="https://github.com/user-attachments/assets/c5a85427-cf73-47c7-8a6f-a5c8fabb4641" />
Interfaces e0/0 e e0/1 em estado up/up, e PIM habilitado nas interfaces.


<img width="539" height="319" alt="image" src="https://github.com/user-attachments/assets/e16465a7-33f9-4c19-ba04-b70efe98fc2c" />
Existência de entradas multicast em show ip mroute

<img width="539" height="319" alt="image" src="https://github.com/user-attachments/assets/57b9469b-0dc4-487c-9061-8f5e3ff9baed" />
Coerência entre interface de entrada, interface de saída e origem do tráfego.

## Funcionamento e Validação
- o tráfego multicast sai da rede de origem e alcança a rede do receptor;
- o roteador mantém informações multicast específicas para o grupo configurado;
- o funcionamento do PIM depende da base de encaminhamento unicast já estabelecida;
- a topologia controlada permite visualizar o comportamento inicial do PIM-DM com baixa complexidade.

## Conclusão
Esta atividade apresenta o primeiro cenário prático de multicast IP com PIM-DM em ambiente controlado no PNetLab. O laboratório permite o entendimento de como o roteador passa a tratar tráfego multicast, formando a base conceitual e operacional necessária para topologias mais complexas nas próximas práticas.
