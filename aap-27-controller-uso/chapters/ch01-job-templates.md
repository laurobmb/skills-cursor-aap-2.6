# Ch01 — Job Templates: Configuração Completa

## Campos do Job Template

| Campo | Descrição | Prompt on Launch |
|---|---|---|
| **Name** | Nome do Job Template | N/A |
| **Job type** | `Run` (executar) ou `Check` (dry run) | Sim |
| **Inventory** | Inventário a usar | Sim (passo separado) |
| **Project** | Projeto (repo Git) | N/A |
| **Source control branch** | Override de branch (requer projeto com branch override) | Sim |
| **Playbook** | Arquivo `.yml` dentro do projeto | N/A |
| **Execution Environment** | Container image para executar o job | Sim (passo separado) |
| **Credentials** | Uma ou mais credenciais | Sim (passo separado) |
| **Forks** | Paralelismo (0 = padrão Ansible = 5) | Sim |
| **Limit** | Host pattern: `a:b`, `a:!b`, `a:b:&c` | Sim |
| **Verbosity** | Normal / Verbose / More Verbose / Debug | N/A |
| **Job slicing** | Número de slices (>1 gera workflow automaticamente) | Sim |
| **Timeout** | Segundos antes de cancelar (0 = padrão global) | Sim |
| **Show changes** | Exibir diff do que mudou | Sim |
| **Instance groups** | Grupo de instâncias preferido para execução | Sim |
| **Job tags** | Executar apenas as tasks com essas tags | Sim |
| **Skip tags** | Pular tasks com essas tags | Sim |
| **Extra variables** | Variáveis adicionais (YAML/JSON) | Sim |
| **Labels** | Etiquetas para organização e filtro | Sim |
| **Concurrent jobs** | Permitir execução simultânea do mesmo JT | N/A |

## Job Types

```
Run:   executa o playbook normalmente
Check: dry run — mostra mudanças sem aplicar
       (tarefas sem suporte a check mode são puladas)
```

## Forks e Capacidade

```
forks=0     → usa padrão do Ansible (5 forks)
forks=N     → N conexões simultâneas

Regra de ouro:
- Inventários grandes: aumentar forks (melhor paralelismo)
- Tarefas CPU-intensivas: manter forks baixo
- Jobs com gather_facts: cuidado com forks altos (alto consumo de memória)
```

## Job Slicing (Escala Horizontal)

Job slicing divide o inventário em N partes e executa jobs em paralelo:

```
job_slice_count=4 + inventory de 400 hosts
→ Cria automaticamente um Workflow com 4 jobs de 100 hosts cada

# Considerações:
- Não usar em playbooks que orquestram entre hosts (ex: haproxy backends)
- Slicing implica allow_simultaneous (ignora configuração do JT)
- Se o limit fizer slices ficarem sem hosts → esses slices falham
```

## Surveys (Formulários de Input)

```
Job Template → Survey → Create survey question

Tipos de pergunta:
- Text       → string livre
- Textarea   → texto multi-linha
- Password   → oculto nos logs
- Integer    → número inteiro
- Float      → número decimal
- Multiple Choice (single select)
- Multiple Choice (multiple select)

Cada resposta vira uma extra_var no job.
extra_vars com !unsafe → não expandem variáveis aninhadas (segurança).
```

## Extra Variables — Precedência

```
(maior → menor prioridade)
1. Job launch extra variables    (sobrescrevem tudo ao lançar)
2. Job template extra variables  (definidas no JT)
3. Survey defaults               (valores padrão do survey)
4. Group/host vars do inventário
5. Playbook vars
6. Role defaults
```

## Webhooks SCM → Disparar JT

```
Job Template → Enable webhook → GitHub / GitLab
→ Webhook URL: https://aap.empresa.org/api/controller/v2/job_templates/<id>/github/
→ Webhook Key: chave gerada (configurar como secret no GitHub)

Ao receber push/PR → Controller lança o JT automaticamente.
PAT do GitHub como credencial → Controller posta status de volta ao repo.
```

## Via API — Lançar Job Template

```bash
# Lançar JT com extra_vars
curl -X POST https://aap.empresa.org/api/controller/v2/job_templates/<id>/launch/ \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "extra_vars": {
      "target_env": "producao",
      "app_version": "2.1.0"
    },
    "limit": "webservers",
    "credential": 5
  }'

# Verificar status do job
curl https://aap.empresa.org/api/controller/v2/jobs/<job_id>/ \
  -H "Authorization: Bearer <token>" | jq '.status'
```

## Via CaC — Criar Job Template

```yaml
ansible.platform.job_template:
  name: "Patching RHEL - Producao"
  organization: "Infraestrutura"
  project: "automacoes-infra"
  playbook: "playbooks/patch.yml"
  inventory: "inventario-linux-prod"
  execution_environment: "ee-supported-rhel9:latest"
  credentials:
    - "satellite-prod"
    - "vault-machine"
  forks: 10
  timeout: 3600
  verbosity: 1
  job_slice_count: 4
  allow_simultaneous: false
  survey_enabled: true
  extra_vars:
    patch_type: "security"
  controller_host: "https://aap.empresa.org"
  controller_oauthtoken: "{{ gateway_token }}"
```
