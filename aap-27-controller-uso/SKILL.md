---
name: aap-27-controller-uso
description: >-
  Referência técnica de uso do Automation Controller no AAP 2.7. Cobre Job Templates
  (todos os campos: forks, limit, verbosity, timeout, job slicing, surveys, extra_vars,
  instance groups, tags/skip tags, concurrent jobs, webhooks SCM), Workflow Job Templates
  (visualizer, convergence nodes, extra vars entre jobs, artifacts/set_stats), Projects
  (SCM config, branch override, webhooks de projeto), Inventários Dinâmicos (Smart,
  Constructed, AWS EC2, Azure, VMware vCenter, Red Hat Satellite, GCE, OpenStack,
  Terraform State, OpenShift Virtualization), Notificações (email, Slack, webhook,
  PagerDuty), e Instance/Container Groups (job runtime behavior, capacity limits).
  Use quando precisar configurar um Job Template completo, montar um Workflow com
  convergência, criar inventário dinâmico do VMware ou AWS, adicionar survey a um JT,
  configurar notificações, ou entender como os jobs são distribuídos nos instance groups.
disable-model-invocation: true
---

# AAP 2.7 — Automation Controller: Uso e Configuração

## Índice

| Capítulo | Conteúdo |
|---|---|
| [ch01-job-templates](chapters/ch01-job-templates.md) | Todos os campos de JT, surveys, job slicing, extra_vars, webhooks |
| [ch02-workflows](chapters/ch02-workflows.md) | Workflow visualizer, convergência, extra_vars/artifacts, cenários |
| [ch03-projects](chapters/ch03-projects.md) | Projects SCM, branch override, webhook de projeto, update schedule |
| [ch04-inventories](chapters/ch04-inventories.md) | Smart, Constructed, AWS EC2, Azure, VMware, Satellite, Terraform State |
| [ch05-notifications](chapters/ch05-notifications.md) | Tipos de notificação, configuração, trigger events |

## Uso Rápido

- **Configurar Job Template com survey e forks:** → ch01
- **Criar workflow com aprovação manual e convergência:** → ch02
- **Inventário dinâmico do VMware vCenter:** → ch04
- **Inventário dinâmico do Terraform State:** → ch04
- **Notificação no Slack quando job falha:** → ch05
- **Job slicing para escalar em inventário grande:** → ch01
