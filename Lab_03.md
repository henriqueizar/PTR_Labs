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

flowchart TB
    R["📡 Router R1<br/>G0/0: 192.168.10.1/24<br/>G0/1: 192.168.20.1/24"] --- S1["🔀 SW1"]
    R --- S2["🔀 SW2"]

    S1 --- SRC["💻 Host Origem<br/>192.168.10.10/24"]
    S2 --- RCV["💻 Host Receptor<br/>192.168.20.10/24"]
