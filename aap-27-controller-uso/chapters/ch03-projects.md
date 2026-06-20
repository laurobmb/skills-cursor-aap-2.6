# Ch03 — Projects e SCM

## Criar um Project

```
Automation Content › Projects → Create Project

Campos:
  Name: nome descritivo
  Organization: organização dona do projeto
  Source Control Type: Git | SVN | Archive (tar.gz)
  Source Control URL: https://github.com/empresa/automacoes.git
  Source Control Branch/Tag/Commit: main (ou específico)
  Source Control Credential: credencial SSH/token do repo
  Options:
    ☐ Clean                 → remover arquivos modificados localmente
    ☐ Delete on Update      → deletar pasta local e re-clonar
    ☐ Track Submodules      → atualizar submodules Git
    ☑ Update Revision on Launch → sync antes de cada job (mais lento)
    ☐ Allow Branch Override → permitir que JTs sobrescrevam a branch
```

## Allow Branch Override

Quando habilitado, Job Templates usando esse projeto podem especificar uma branch diferente:

```
Project → Allow Branch Override ✓

Job Template → Source Control Branch: feature/nova-automacao
(campo só aparece se o projeto permite override)
```

**Cuidado:** branch não-default é sempre buscada do remote antes do job iniciar — sem cache local. Use commit hashes ou tags para garantir versões fixas:

```
main        → sempre pega a última versão do remote (não determinístico)
v2.1.0      → tag fixa = sempre a mesma versão ✓
abc123def   → commit hash = sempre o mesmo commit ✓
```

## Git Refspec — Buscar PRs

```ini
# Buscar todos os PRs do GitHub no project
Source control refspec: refs/pull/*:refs/remotes/origin/pull/*

# Buscar PR específico
Source control refspec: refs/pull/62/head:refs/remotes/origin/pull/62/head

# Então no Job Template → Source Control Branch: pull/62/head
```

## Webhook de Projeto (SCM Webhook)

```
Project → Edit → Enable webhook → GitHub / GitLab
→ Webhook URL: https://aap.empresa.org/api/controller/v2/projects/<id>/github/
→ Webhook Key: copiar e configurar no GitHub

Ao receber push → Controller lança Project Update (sync)
Se Update Revision on Launch = false, o JT pode usar a versão nova em seguida.
```

## Update Schedule

```
Project → Schedules → Create Schedule

Exemplo: sync diário às 2h
  Frequency: Daily
  Start: 02:00
  Timezone: America/Sao_Paulo
```

## Via CaC — Criar Project

```yaml
ansible.platform.project:
  name: "automacoes-infra"
  organization: "Infraestrutura"
  scm_type: git
  scm_url: "https://github.com/empresa/automacoes-infra.git"
  scm_branch: main
  scm_credential: "github-token"
  scm_update_on_launch: false   # true = mais lento mas sempre atual
  allow_override: true
  controller_host: "https://aap.empresa.org"
  controller_oauthtoken: "{{ gateway_token }}"
```
