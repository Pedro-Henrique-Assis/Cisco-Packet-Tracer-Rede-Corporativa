# Rede Corporativa com VLAN, Trunk, STP e Roteamento Inter-VLAN

Simulação de uma rede corporativa de três andares no Cisco Packet Tracer, utilizando segmentação por VLAN, enlaces trunk 802.1Q, redundância com Spanning Tree Protocol e roteamento inter-VLAN via *router-on-a-stick*.

## Conteúdo do repositório

| Arquivo | Descrição |
|---|---|
| `Projeto1.pkt` | Projeto do Cisco Packet Tracer com a topologia completa e já configurada |
| `documentacao-tecnica.docx` | Documentação técnica com a fundamentação teórica e as referências normativas |
| `topologia.png` | Diagrama detalhado da topologia com todos os elementos identificados |
| `README.md` | Este arquivo |

**Vídeo explicativo:** [inserir link do YouTube aqui]

## Topologia

A rede é composta por um roteador, três comutadores e dezoito estações de trabalho:

- **router-f1** — Cisco 1941, com quatro subinterfaces (uma por VLAN) na interface Gi0/0
- **switch-f1** — Catalyst 2960 do Andar 1, definido como *Root Bridge* do STP (prioridade 4096)
- **switch-f2** — Catalyst 2960 do Andar 2, raiz secundária (prioridade 8192)
- **switch-f3** — Catalyst 2960 do Andar 3, instalado na expansão (prioridade padrão 32768)

Os três comutadores estão interligados dois a dois, formando um caminho redundante. O STP mantém uma das portas em estado de bloqueio para eliminar o laço, ativando-a automaticamente em caso de falha do enlace principal.

## VLANs

| VLAN | Nome | Rede | Gateway | Departamento |
|---|---|---|---|---|
| 2 | Sales | 172.16.2.0/24 | 172.16.2.1 | Vendas |
| 3 | HR | 172.16.3.0/24 | 172.16.3.1 | Recursos Humanos |
| 4 | Purchasing | 172.16.4.0/24 | 172.16.4.1 | Compras |
| 5 | Finance | 172.16.5.0/24 | 172.16.5.1 | Financeiro |
| 10 | Unused | — | — | Portas não utilizadas |

## Estações por andar

**Andar 1 (switch-f1)** — Sales1 (172.16.2.11), Sales2 (172.16.2.12), HR1 (172.16.3.11), HR2 (172.16.3.12), Fin1 (172.16.5.11), Fin2 (172.16.5.12), Pur1 (172.16.4.11)

**Andar 2 (switch-f2)** — Sales3 (172.16.2.21), Sales4 (172.16.2.22), HR3 (172.16.3.21), Pur2 (172.16.4.21), Pur3 (172.16.4.22), Fin3 (172.16.5.21), Fin4 (172.16.5.22)

**Andar 3 (switch-f3)** — Sales5 (172.16.2.31), Fin5 (172.16.5.31), Fin6 (172.16.5.32), Fin7 (172.16.5.33)

## Como executar localmente

### Pré-requisitos

Cisco Packet Tracer 8.0 ou superior. O download é gratuito e requer uma conta na Cisco Networking Academy:

- Cadastro: https://www.netacad.com/courses/packet-tracer
- Disponível para Windows, Linux e macOS

### Passos

1. Clone o repositório ou baixe o arquivo `Projeto1.pkt` diretamente.

```bash
git clone <url-do-repositorio>
```

2. Abra o Cisco Packet Tracer.

3. Vá em **File → Open** e selecione o arquivo `Projeto1.pkt`.

4. A topologia carrega já configurada. Aguarde alguns segundos até que todos os indicadores de porta fiquem verdes — o STP precisa convergir antes de a rede ficar operacional.

5. Confirme que o simulador está em modo **Realtime** (canto inferior direito). No modo *Simulation* os temporizadores do STP não avançam normalmente.

## Como testar

### Comunicação entre VLANs

Abra o **Desktop → Command Prompt** da estação Sales1 e execute:

```
ping 172.16.5.31
```

As respostas confirmam que o roteamento inter-VLAN está funcionando: o pacote sai da VLAN 2, atravessa o roteador e chega à VLAN 5, em outro andar.

### Verificação das configurações

Abra o **CLI** de qualquer comutador e execute:

```
enable
show vlan brief
show interfaces trunk
show spanning-tree
```

No switch-f3, o `show interfaces trunk` evidencia a porta bloqueada pelo STP: ela aparece entre as portas permitidas, mas sem encaminhar nenhuma VLAN.

No router-f1:

```
enable
show ip interface brief
```

As quatro subinterfaces (Gi0/0.2 a Gi0/0.5) devem aparecer como `up/up`.

### Teste de redundância

No CLI do switch-f3, derrube o enlace principal:

```
enable
configure terminal
interface FastEthernet0/1
 shutdown
end
show spanning-tree
```

Aguarde a convergência do STP (de 30 a 50 segundos no modo clássico) e verifique que a porta antes bloqueada passou ao estado *forwarding*. Teste novamente o ping para confirmar que a comunicação continua íntegra pelo caminho alternativo.

Para restaurar o estado original:

```
configure terminal
interface FastEthernet0/1
 no shutdown
end
```

## Tecnologias e normas de referência

- **IEEE 802.1Q** — VLAN e encapsulamento em enlaces trunk
- **IEEE 802.1D** — Spanning Tree Protocol
- **RFC 791** — Protocolo de Internet, base do roteamento inter-VLAN
- **RFC 1918** — Endereçamento IP privado
- **RFC 9542** — Relação entre especificações do IETF e parâmetros do IEEE 802

A fundamentação teórica completa está no arquivo `documentacao-tecnica.docx`.