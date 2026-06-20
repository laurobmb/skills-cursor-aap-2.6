# Ch02 — Automation Mesh: Topologias de Design

> **Nota de Sintaxe para Containerizado:**
> - Substituir `node_type` por `receptor_type`
> - Substituir `peers` por `receptor_peers`
> - `receptor_peers` = lista de hostnames separados por vírgula (não nomes de grupos)

## Topologia 1: Múltiplos Hybrid Nodes (Controle + Execução)

```ini
# Mais simples — nós do control plane fazem tudo
[automationcontroller]
aap_c_1.example.com
aap_c_2.example.com
aap_c_3.example.com

[automationcontroller:vars]
node_type=hybrid   # padrão
```

**Topology:**
```
aap_c_1 ←→ aap_c_2 ←→ aap_c_3
(todos automaticamente peered entre si)
```

## Topologia 2: Control Plane Separado + Execution Nodes

```ini
[automationcontroller]
aap_c_1.example.com
aap_c_2.example.com

[automationcontroller:vars]
node_type=control
peers=execution_nodes   # controles peeream com todos os execution nodes

[execution_nodes]
aap_e_1.example.com
aap_e_2.example.com
```

**Topology:**
```
aap_c_1 ←→ aap_c_2
   ↕              ↕
aap_e_1       aap_e_2
```

## Topologia 3: Execução Remota com Hop Node

Para execução em site remoto, DMZ ou rede com restrições.

```ini
[automationcontroller]
aap_c_1.example.com
aap_c_2.example.com

[automationcontroller:vars]
node_type=control
peers=instance_group_local

[execution_nodes]
aap_e_1.example.com          # local
aap_e_2.example.com          # local
aap_h_1.example.com node_type=hop  # hop para site remoto
aap_e_3.example.com          # remoto (acessível via hop)

[instance_group_local]
aap_e_1.example.com
aap_e_2.example.com

[hop]
aap_h_1.example.com

[hop:vars]
peers=automationcontroller   # hop se conecta ao control plane

[instance_group_remote]
aap_e_3.example.com

[instance_group_remote:vars]
peers=hop                    # remoto se conecta via hop
```

**Topology:**
```
aap_c_1 ←→ aap_c_2
   ↕              ↕        ↓
aap_e_1    aap_e_2    aap_h_1 → aap_e_3 (site remoto)
```

## Topologia 4: Multi-Hop (Remoto de Remoto)

Para automatizar em rede atrás de DMZ dentro de outra rede.

```ini
[automationcontroller]
aap_c_[1:3].example.com

[automationcontroller:vars]
node_type=control
peers=instance_group_local

[local_hop]
aap_h_1.example.com
aap_h_2.example.com

[local_hop:vars]
peers=automationcontroller

[remote_multi_hop]
aap_h_3.example.com

[remote_multi_hop:vars]
peers=local_hop

[instance_group_local]
aap_e_1.example.com
aap_e_2.example.com

[instance_group_remote]
aap_e_3.example.com

[instance_group_remote:vars]
peers=local_hop

[instance_group_multi_hop_remote]
aap_e_4.example.com

[instance_group_multi_hop_remote:vars]
peers=remote_multi_hop
```

## Topologia 5: Outbound-Only (Sem Inbound para o Controller)

Quando a rede não permite conexões inbound para os control nodes.

```ini
[automationcontroller]
controller-[1:2].example.com

[execution_nodes]
execution-[1:5].example.com

[execution_nodes:vars]
# Os execution nodes se conectam AO controller (não o inverso)
peers=automationcontroller
```

> Grupos que começam com `instance_group_` são automaticamente reconhecidos como Instance Groups no AAP UI.

## Equivalência Containerizado (receptor_peers)

```ini
# Containerizado: receptor_peers = hostnames separados por vírgula
[execution_nodes]
aap_e_3.example.com receptor_peers=aap_h_1.example.com
aap_h_1.example.com receptor_peers=aap_c_1.example.com,aap_c_2.example.com
```
