# Ch04 — Inventários Dinâmicos

## Tipos de Inventário

| Tipo | Descrição |
|---|---|
| **Manual** | Hosts/grupos criados manualmente na UI |
| **Dynamic (Source)** | Sincronizado de um provedor externo (VMware, AWS, etc.) |
| **Smart** | *(Deprecated)* Filtro de hosts baseado em fatos armazenados |
| **Constructed** | Agrega múltiplos inventários com filtros e grupos construídos |

## Adicionar Fonte Dinâmica a um Inventário

```
Automation Execution → Infrastructure → Inventories
  → Selecionar inventário → Sources → Create source
  → Tipo de fonte → Credential → Source Variables
```

## Fontes Dinâmicas Disponíveis

| Fonte | Coleção/Plugin | Credencial |
|---|---|---|
| Amazon Web Services EC2 | `amazon.aws.aws_ec2` | AWS credential |
| Google Compute Engine | `google.cloud.gcp_compute` | GCP credential |
| Microsoft Azure | `azure.azcollection.azure_rm` | Azure credential |
| VMware vCenter | `community.vmware.vmware_vm_inventory` | VMware credential |
| VMware ESXi | `vmware.vmware.vms` | VMware credential |
| Red Hat Satellite 6 | `theforeman.foreman.foreman` | Satellite credential |
| OpenStack | `openstack.cloud.openstack` | OpenStack credential |
| Red Hat Virtualization | `ovirt` | RHV credential |
| Red Hat Ansible Automation Platform | `awx.awx.tower` | AAP credential |
| Red Hat Lightspeed | Lightspeed plugin | Lightspeed credential |
| **Terraform State** | `cloud.terraform.terraform_state` | Terraform backend credential |
| OpenShift Virtualization | `kubevirt.core.kubevirt` | K8s/OCP Bearer Token |
| Sourced from a Project | SCM custom plugin | Conforme plugin |

## VMware vCenter — Configuração

```
Fonte: VMware vCenter
Credential: (credencial VMware com host/user/pass do vCenter)
Source Variables:
```

```yaml
# Variáveis opcionais para VMware
plugin: community.vmware.vmware_vm_inventory
filters:
  - runtime.powerState == "poweredOn"    # somente VMs ligadas
strict: false
with_nested_properties: true
keyed_groups:
  - key: config.guestId
    prefix: ''
    separator: ''
```

> VMs ligadas agrupadas por guestId (ex: `rhel9_64Guest`, `windows9_64Guest`)

## AWS EC2 — Configuração Mínima

```yaml
plugin: amazon.aws.aws_ec2
filters:
  instance-state-name:
    - running
keyed_groups:
  - key: tags.Name
    prefix: tag
  - key: placement.region
    prefix: ''
    separator: ''
```

## Red Hat Satellite 6 — Configuração

```yaml
plugin: theforeman.foreman.foreman
group_prefix: foreman_
want_facts: true
want_params: true
legacy_hostvars: true
validate_certs: false
keyed_groups:
  - key: foreman['environment_name'] | lower
    prefix: foreman_environment_
    separator: ''
  - key: foreman['location_name'] | lower
    prefix: foreman_location_
    separator: ''
  - key: foreman['content_facet_attributes']['lifecycle_environment_name'] | lower
    prefix: foreman_lifecycle_environment_
    separator: ''
```

## Terraform State — Hosts do Terraform no AAP

```yaml
# Source Variables
backend_type: s3    # deve bater com o backend credential configurado

# Requer EE com binário Terraform instalado
# Requer credencial: Terraform backend configuration
```

> Não editar manualmente inventários gerenciados pelo Terraform no AAP — causa drift.

## Constructed Inventory — Agrupar Múltiplos Inventários

```yaml
# Ao criar inventário do tipo Constructed:
# Input Inventories: [inventario-aws, inventario-vmware, inventario-azure]

# Source Variables (source_vars):
plugin: constructed
strict: false
groups:
  linux_hosts: ansible_os_family == "RedHat"
  windows_hosts: ansible_os_family == "Windows"
  prod_hosts: "'producao' in group_names"

# limit: restringir quais hosts incluir
# Exemplo: limit=linux_hosts
```

## Smart Inventory (Deprecated — usar Constructed)

```
⚠️  Smart Inventories estão deprecated e serão removidos em versão futura.
Migrar para Constructed Inventories.

# Filtro de exemplo (host_filter):
/api/v2/hosts?host_filter=ansible_facts__ansible_processor_vcpus=8
```

## Update Options nas Fontes

| Opção | Descrição |
|---|---|
| **Overwrite** | Remover hosts/grupos que não existem mais na fonte |
| **Overwrite Variables** | Sobrescrever variáveis existentes com dados da fonte |
| **Update on Launch** | Sincronizar a fonte antes de cada job (mais lento) |
| **Update on Project Update** | Sincronizar quando o projeto associado é atualizado |
