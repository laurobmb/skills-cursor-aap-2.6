# Ch01 — Automation Mesh: Conceitos e Tipos de Nó

## O que é Automation Mesh?

Automation Mesh é uma rede overlay que distribui execução de automação entre nós via conexões peer-to-peer, usando TLS para criptografia. Resolve:

- Topologias de rede difíceis (firewalls, latência, sites remotos)
- Separação do plano de controle do plano de execução
- Escalabilidade horizontal de capacidade de execução independente da UI/API

## Planos: Controle vs Execução

### Plano de Controle
Executa serviços persistentes (web server, task dispatcher, project updates):

| Tipo de Nó | Funções |
|---|---|
| **Hybrid** (padrão) | Controle + execução de jobs. Nó padrão ao instalar |
| **Control** | Apenas controle: project updates, management jobs. Sem execução de jobs regulares |

> Em OCP/Operator: não existem hybrid/control separados. O control plane são container groups no cluster K8s.

### Plano de Execução
Executa apenas jobs de automação, sem funções de controle:

| Tipo de Nó | Funções |
|---|---|
| **Execution** | Executa jobs (`ansible-playbook` com podman isolation). Similar aos isolated nodes |
| **Hop** | Roteia tráfego entre nós (similar a jump host). Não executa automation |

## Variáveis de Tipo de Nó

```ini
# Containerizado: usar receptor_type
[execution_nodes]
exec1.empresa.org receptor_type=execution
hop1.empresa.org  receptor_type=hop

# RPM (deprecated): usar node_type
[execution_nodes]
exec1.empresa.org node_type=execution
hop1.empresa.org  node_type=hop

[automationcontroller]
ctrl1.empresa.org node_type=hybrid    # padrão
ctrl2.empresa.org node_type=control   # apenas controle
```

## Peers — Definição de Conexões

```ini
# Containerizado: receptor_peers (lista separada por vírgula de hostnames)
[execution_nodes]
exec1.empresa.org receptor_peers=hop1.empresa.org
hop1.empresa.org  receptor_peers=ctrl1.empresa.org

# RPM (deprecated): peers=
[automationcontroller]
ctrl1.empresa.org peers=exec1.empresa.org

[execution_nodes]
exec1.empresa.org peers=exec2.empresa.org
```

> Conexões são bidirecionais após estabelecidas. Evitar peers duplicados (A→B e B→A simultâneos).

## Criptografia e Segurança

- Comunicação via TLS/TCP (receptor) — FIPS compliant
- CA gerada automaticamente pelo installer se não fornecida
- Cada nó recebe certificado assinado pela CA do mesh
- Para CA customizada: variáveis `mesh_ca_keyfile` e `mesh_ca_certfile` no inventory

## Receptor — Porta Padrão

```
27199/TCP   → escuta receptor (pode ser customizado via listener_port)
```
