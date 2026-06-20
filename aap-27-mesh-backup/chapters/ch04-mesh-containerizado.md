# Ch04 — Automation Mesh: Containerizado e VM

## Adicionar Nó ao Mesh (Containerizado)

```bash
# Ir ao diretório do installer
cd ansible-automation-platform-containerized-setup-<versão>/

# 1. Editar inventory adicionando o novo nó
vi inventory

# 2. Re-executar o setup
ansible-playbook -i inventory ansible.containerized_installer.install
```

## Inventory — Execution Node Simples

```ini
[automationcontroller]
aap-ctrl-1.empresa.org

[execution_nodes]
aap-exec-1.empresa.org

[execution_nodes:vars]
receptor_type=execution
# Sem receptor_peers = o control plane inicia conexão com o execution node
```

## Inventory — Hop Node para Site Remoto

```ini
[automationcontroller]
aap-ctrl-1.empresa.org

[execution_nodes]
# Nó local
aap-exec-local.empresa.org receptor_type=execution

# Hop node (relay para site remoto)
aap-hop-dmz.empresa.org receptor_type=hop

# Nó de execução remoto
aap-exec-remoto.empresa.org receptor_type=execution

# Peers: exec-remoto → hop → controller
# Nota: receptor_peers = hostnames separados por vírgula
aap-hop-dmz.empresa.org receptor_type=hop receptor_peers=aap-ctrl-1.empresa.org
aap-exec-remoto.empresa.org receptor_type=execution receptor_peers=aap-hop-dmz.empresa.org

[instance_group_local]
aap-exec-local.empresa.org

[instance_group_remoto]
aap-exec-remoto.empresa.org
```

## Inventory — Outbound-Only (Execution → Controller)

```ini
[automationcontroller]
aap-ctrl-1.empresa.org
aap-ctrl-2.empresa.org

[execution_nodes]
aap-exec-[1:3].empresa.org

[execution_nodes:vars]
receptor_type=execution
# Execution nodes iniciam conexão AO controller (não o inverso)
receptor_peers=aap-ctrl-1.empresa.org,aap-ctrl-2.empresa.org
```

## CA Personalizada para o Mesh

```ini
# Adicionar no inventory antes de instalar:
[all:vars]
mesh_ca_keyfile=/tmp/mesh_CA.key
mesh_ca_certfile=/tmp/mesh_CA.crt

# Re-executar installer para aplicar a CA
ansible-playbook -i inventory ansible.containerized_installer.install
```

> CA customizada é copiada para `/etc/receptor/tls/ca/` em cada nó do mesh.

## Instance Groups — Associar Nós no UI

```
Automation Execution → Infrastructure → Instance Groups → Create group

Criar Instance Group → associar execution nodes específicos
→ Job Template → Instance Groups → selecionar o grupo

Usar para:
- Reservar nós para workloads específicos (ex: "producao" vs "dev")
- Isolar execução por região/datacenter (ex: "instance_group_datacenter_b")
```

## Configuração de Instance Group via CaC

```yaml
ansible.platform.instance_group:
  name: "datacenter-remoto"
  max_concurrent_jobs: 10
  max_forks: 50
  controller_host: "https://aap.empresa.org"
  controller_oauthtoken: "{{ gateway_token }}"
```
