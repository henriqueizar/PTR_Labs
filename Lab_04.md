# Laboratório 04 - RIP e análise de convergência


Implementação do **RIPv2** em topologia com múltiplos roteadores, seguida de **análise de convergência após falha de enlace**.

## Objetivos

- configurar o **RIPv2** em uma topologia com três roteadores;
- anunciar redes diretamente conectadas;
- verificar tabelas de roteamento e parâmetros do protocolo RIP;
- observar o comportamento da rede após a queda de um enlace;
- medir e interpretar o **tempo de convergência**;
- analisar o impacto de mecanismos como **split horizon**, **poison reverse** e **triggered updates**;
- relacionar a prática com as limitações do RIP em redes maiores.

## Topologia 

<img width="1152" height="173" alt="image" src="https://github.com/user-attachments/assets/c97430dc-6535-4f05-8e56-3288e1a11bbb" />

<img width="681" height="361" alt="image" src="https://github.com/user-attachments/assets/d301d15c-9245-48c0-a355-b57a269508b7" />

## Montagem no PNetLab

- 3 roteadores Cisco L3 com suporte a RIP
- 2 hosts VPCs
- enlaces ponto a ponto entre os roteadores

<img width="664" height="140" alt="image" src="https://github.com/user-attachments/assets/897774c5-ca13-41cf-8b25-1023a68f3f7b" />

## Configuração básica nos Hosts

VPC1:
<img width="557" height="68" alt="image" src="https://github.com/user-attachments/assets/94001472-2330-4170-9299-f689aae5f1e9" />

VPC2:
<img width="546" height="73" alt="image" src="https://github.com/user-attachments/assets/ccbc1e36-9549-4569-9b58-67c547afe27f" />


## Configuração dos roteadores

R1:
<img width="593" height="443" alt="image" src="https://github.com/user-attachments/assets/b89415a8-c249-44fd-adc9-adc6cb594e5c" />
R2:
<img width="579" height="420" alt="image" src="https://github.com/user-attachments/assets/73f74bd5-2420-4a74-a58d-4cb846b7bc51" />
E similarmente para R3.

## Verificação

### show ip interface brief
Interfaces em estado up e endereçadas conforme a tabela:
<img width="897" height="124" alt="image" src="https://github.com/user-attachments/assets/fe5b0dbe-1d58-49ac-9a29-bb47e119e4ed" />

<img width="897" height="142" alt="image" src="https://github.com/user-attachments/assets/9d38eab9-eaa4-4177-be97-21544ba80eed" />

<img width="897" height="141" alt="image" src="https://github.com/user-attachments/assets/384d5b0a-42b7-4fbf-bb68-d4a4aa346937" />

### show ip route
Conexões dos roteadores:
R1:
<img width="892" height="400" alt="image" src="https://github.com/user-attachments/assets/2a983fba-b9ca-41e7-b3b6-e3491286b903" />

**IMPORTANTE**: As linhas que começam com C (connected) e L (local) indicam as redes que estao conectadas **fisicamente** ao roteador. Já as que começam com R, indicam que o roteador aprendeu o caminho para uma rede através do **protocolo RIP**, que acabou de ser configurado.

R2:
<img width="797" height="403" alt="image" src="https://github.com/user-attachments/assets/d740d67e-c0c9-453f-ad35-74d65e7fd492" />
R3:
<img width="790" height="420" alt="image" src="https://github.com/user-attachments/assets/5e911b3f-6c8a-4393-9fba-a34adcf40c60" />

### show ip protocols

Mostra que o protocolo RIP está configurado e se comportanto devidamente.
<img width="644" height="481" alt="image" src="https://github.com/user-attachments/assets/9342b41e-1fac-4a09-99e9-b010f9530b12" />


## Teste de Comunicação end-to-end:

<img width="612" height="158" alt="image" src="https://github.com/user-attachments/assets/fd83ed07-1f11-46a5-9aaa-2fbf6daf5eef" />

SUCESSO! O VPC 1 consegue conversar com o VPC 2, graças ao RIP.

## Experimento de convergência
**Etapa 1: Estável**

Com a rede funcionando, verificou-se:
- A rota para 192.168.30.0/24 em R1 ✔
  <img width="669" height="25" alt="image" src="https://github.com/user-attachments/assets/ed35813c-9788-476b-9f0e-7c5a37d81410" />

- A rota para 192.268.10.0/24 em R3 ✔

<img width="670" height="24" alt="image" src="https://github.com/user-attachments/assets/a56cedd3-133d-4965-beb6-1d77fc1f2c30" />

- Conectividade entre VPC1 e VPC2 ✔
<img width="605" height="158" alt="image" src="https://github.com/user-attachments/assets/3a5ffb9c-bc87-42aa-81c8-71411b64e9b2" />

**Etapa 2: Falha de Enlace**

Desativaram-se as interfaces entre R2 e R3:
<img width="893" height="216" alt="image" src="https://github.com/user-attachments/assets/fb0ddf53-43c5-4064-8edc-d17e0be8a6bd" />

Mesmo após 3 minutos, o RIP ainda insistia na rota que, na realidade, estava desativada.
<img width="671" height="24" alt="image" src="https://github.com/user-attachments/assets/0305a446-71c4-479f-96de-dc5046089ee5" />

Finalmente, após esse último timestampo de 3:05 (185 segundos), o próximo ip route mostrou que o RIP percebeu que a rota era inválida:
<img width="744" height="27" alt="image" src="https://github.com/user-attachments/assets/1ec45f5c-e1c0-4cca-962e-6737f74ff2b1" />

Internamente, o R1 agora considera que essa rede está a uma distância de 16 saltos, o que no RIP indica uma rede incalcançável

## Conclusão

Este laboratório amplia a complexidade das práticas anteriores ao introduzir um protocolo de roteamento dinâmico e a análise do comportamento da rede diante de falhas. A configuração do RIPv2 em uma topologia com múltiplos roteadores permite ao aluno compreender como rotas são propagadas, aprendidas e removidas.

A observação do processo de convergência evidencia limitações clássicas do RIP, especialmente em relação ao tempo de adaptação a mudanças de topologia. Com isso, a atividade estabelece uma base importante para o estudo comparativo com protocolos mais modernos e eficientes, como o OSPF, que serão explorados em etapas posteriores da disciplina.





