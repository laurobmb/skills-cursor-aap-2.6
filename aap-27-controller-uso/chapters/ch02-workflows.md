# Ch02 — Workflow Job Templates e Visualizer

## Conceitos de Workflow

Workflows encadeiam Job Templates, syncs de projeto/inventário e outros workflows em um único job rastreável.

```
Recursos que podem ser nodes de um workflow:
- Job Templates
- Workflow Job Templates (aninhamento)
- Project syncs
- Inventory source syncs
```

## Tipos de Conexão entre Nodes

| Tipo | Quando o próximo node executa |
|---|---|
| **On Success** | Se o node pai completou com sucesso |
| **On Failure** | Se o node pai falhou |
| **Always** | Independente do resultado (root node = sempre Always) |

## Workflow Visualizer

```
Templates → Create workflow template → Workflow Visualizer

1. Adicionar primeiro node (Start)
2. Clicar em "+" no node para adicionar o próximo
3. Definir tipo de conexão: On Success / On Failure / Always
4. Repetir para cada etapa
```

## Convergência (Múltiplos jobs → 1 próximo)

```
    [Job A]──────┐
                 ├──[Convergence Node]──[Job D]
    [Job B]──────┘

Job D só executa quando A e B terminam (com sucesso).
Todos os ancestrais devem completar antes de triggers convergirem.
```

## Aprovação Manual em Workflows

```yaml
# No Workflow Visualizer: adicionar node do tipo "Approval"
# Alguém precisa aprovar ou rejeitar no UI antes do workflow continuar

Casos de uso:
- Aprovação de deploy em produção
- Validação humana antes de ação destrutiva
```

## Extra Variables e Artifacts entre Jobs

```yaml
# Job 1: publicar artifact para downstream
- name: Publicar resultado para workflow
  set_stats:
    data:
      deploy_url: "https://app.empresa.org/{{ version }}"
      deploy_success: true

# Job 2 (downstream): herda a variável
- debug:
    msg: "Deploy em {{ deploy_url }}"
```

**Precedência de variáveis em workflow:**
```
1. Workflow job launch extra variables
2. Workflow job template extra variables
3. Workflow job template survey (defaults)
4. Variáveis herdadas de jobs upstream (set_stats/artifacts)
5. Job template extra variables
6. Job template survey defaults
```

## Cenários e Considerações

```
✓ Convergência: múltiplos jobs convergindo em um
✓ Workflows dentro de workflows (aninhamento)
✓ Inventory no nível do workflow (aplicado a todos os JTs com ask_inventory_on_launch)
✓ Schedules no workflow (não nos JTs individuais)
✓ Artefatos de jobs upstream passados downstream via set_stats
✓ Rodar múltiplos workflows simultaneamente

⚠️  set_stats com chaves duplicadas em convergência: comportamento indefinido (usar chaves únicas)
⚠️  Workflows recursivos: Controller para ao detectar loop
⚠️  JTs sem inventory padrão + prompt_inventory=false: não podem ser adicionados a workflow
```

## Via CaC — Criar Workflow

```yaml
ansible.platform.workflow_job_template:
  name: "Pipeline Deploy Completo"
  organization: "Infraestrutura"
  ask_inventory_on_launch: false
  extra_vars:
    env: producao
  controller_host: "https://aap.empresa.org"
  controller_oauthtoken: "{{ gateway_token }}"

# Nodes do workflow (exemplo simplificado)
ansible.platform.workflow_job_template_node:
  workflow_job_template: "Pipeline Deploy Completo"
  unified_job_template: "Backup Pre-Deploy"
  identifier: "backup"

ansible.platform.workflow_job_template_node:
  workflow_job_template: "Pipeline Deploy Completo"
  unified_job_template: "Deploy Aplicacao"
  identifier: "deploy"
  success_nodes:
    - backup
```
