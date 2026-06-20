# Cheatsheet — AAP 2.7 Controller Uso

## Campos Obrigatórios de Job Template

```
Name + Job type + Inventory + Project + Playbook + EE
```

## Fórmula de Job Slicing

```
job_slice_count=N → inventory dividido em N partes
→ Gera Workflow automaticamente
→ Cada slice = 1 job separado
⚠️  Não usar em playbooks multi-host (haproxy, clustering)
```

## Workflow — Tipos de Conexão

```
On Success  → próximo executa se pai teve sucesso
On Failure  → próximo executa se pai falhou
Always      → próximo executa independente (root = sempre Always)
```

## Inventory Sources Disponíveis

```
AWS EC2 | GCE | Azure | VMware vCenter | VMware ESXi
Red Hat Satellite 6 | OpenStack | RHV | AAP
Terraform State | OpenShift Virtualization | Custom (SCM)
```

## Terraform State — Atenção

```
backend_type: s3    ← obrigatório nas Source Variables
Requer EE com binário Terraform
⚠️  Não editar o inventário no AAP (causa drift)
```

## Notificações — Trigger Events

```
Started | Success | Failure
Approval required | Approved | Denied | Timed out
```

## Notificação via API

```bash
# Associar notificação ao JT via API
curl -X POST .../api/controller/v2/job_templates/<id>/notification_templates_error/ \
  -H "Authorization: Bearer <token>" \
  -d '{"id": <notification_id>}'
```

## Extra Vars — Precedência

```
(maior) Job launch extra vars
        Job template extra vars
        Survey defaults
(menor) Group/host vars
```
