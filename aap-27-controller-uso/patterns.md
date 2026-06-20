# Patterns — AAP 2.7 Controller Uso

## Pattern 1: Job Template com Survey para Deploy Parametrizado

```
Cenário: Equipe precisa escolher a versão a deployar sem editar o playbook.
```

```yaml
# Job Template: Deploy App com Survey
Name: Deploy Aplicacao
Project: automacoes-infra
Playbook: playbooks/deploy.yml
Execution Environment: ee-supported-rhel9
Credentials: [vault-machine, vault-secrets]
Forks: 5
Timeout: 1800
Survey Enabled: true

# Perguntas do survey:
- variable: app_version
  question: "Versão para deploy"
  type: text
  required: true
  default: "latest"

- variable: target_env
  question: "Ambiente"
  type: multiplechoice
  choices: [dev, staging, producao]
  required: true
  default: dev

- variable: rollback_enabled
  question: "Habilitar rollback automático?"
  type: multiplechoice
  choices: ["yes", "no"]
  default: "yes"
```

## Pattern 2: Workflow de Patch com Aprovação Humana

```
Cenário: Aplicar patches em produção requerendo aprovação do gerente.
```

```
[Backup Pre-Patch] → ON SUCCESS →
[Teste Conectividade] → ON SUCCESS →
[Aprovação Gerente] → ON SUCCESS →    ← pausa aqui
[Apply Patches] → ON SUCCESS →
[Validação Pós-Patch] → ON FAILURE →
[Rollback]

                      ON FAILURE →
[Notificação Falha]
```

```yaml
# Node de Aprovação:
Node type: Approval
Name: "Aprovação Gerente de Infra"
Description: "Confirme que a janela de manutenção está aprovada"
Timeout: 86400  # 24 horas para aprovar
```

## Pattern 3: Inventário Dinâmico VMware + Filtro Terraform

```
Cenário: Ambientes híbridos com VMs no vCenter e infra no Terraform.
```

```yaml
# Constructed Inventory com 2 fontes:
Input Inventories:
  - inventario-vcenter    # source: VMware vCenter
  - inventario-terraform  # source: Terraform State

# Source Variables do Constructed:
plugin: constructed
groups:
  rhel_vms: "'linux' in group_names and 'vcenter' in source"
  terraform_managed: "'terraform' in group_names"
  prod_all: "'producao' in group_names"
```

## Pattern 4: Workflow de Deploy com Job Slicing

```
Cenário: Aplicar configuração em 1000+ servidores.
```

```yaml
# Job Template com slicing:
Name: Apply Baseline Config
Inventory: inventario-linux-producao
job_slice_count: 10  # 10 jobs de ~100 hosts cada

# Workflow pai:
[Sync Inventory] → [Apply Baseline Config (sliced × 10)] → [Validação]
```

## Pattern 5: Alerta Multi-Canal em Falhas de Produção

```yaml
# Notificações no JT de produção:
notification_templates_failure:
  - "Slack #aap-alertas"
  - "PagerDuty On-Call"
  - "Email time-infra"

notification_templates_success:
  - "Slack #aap-deploy-log"

# Webhook para sistema de ITSM (abrir chamado automaticamente):
notification_templates_failure:
  name: "Webhook GLPI"
  type: webhook
  url: "https://glpi.empresa.org/api/webhook/aap"
  method: POST
  headers:
    Content-Type: application/json
    X-AAP-Secret: "{{ vault_glpi_secret }}"
```
