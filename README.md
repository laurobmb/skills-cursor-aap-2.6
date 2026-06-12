# Cursor Skills — Red Hat Ansible Automation Platform 2.6

Skills para o [Cursor IDE](https://cursor.sh) com conteúdo extraído da documentação oficial do **Red Hat Ansible Automation Platform 2.6**, organizados para consulta direta durante o desenvolvimento e implantação.

---

## O que são Cursor Skills?

Skills são arquivos `SKILL.md` que ensinam o agente de IA do Cursor a responder perguntas técnicas usando conteúdo real — sem alucinações e sem precisar abrir PDFs. O agente carrega apenas os capítulos relevantes para cada pergunta.

---

## Skills disponíveis

| Branch | Skill | Conteúdo |
|---|---|---|
| `v2.6` | `aap-26-instalacao` | Arquitetura, topologias, instalação containerizada, Operator/OpenShift, rede e portas |
| `v2.6` | `aap-26-automacoes` | EDA, Event-Driven Ansible, Configuration as Code, Execution Environments |
| `v2.6` | `aap-26-operacao` | Gestão de acesso, autenticação, hardening, troubleshooting |

---

## Instalação

> Todos os skills estão na branch `v2.6`.

### Instalar todos os skills de uma vez

```bash
# Clonar o repositório
git clone -b v2.6 git@github.com:laurobmb/skills-cursor-aap-2.6.git /tmp/aap-skills

# Copiar cada skill para o diretório de skills do Cursor
cp -r /tmp/aap-skills/aap-26-instalacao ~/.cursor/skills/
cp -r /tmp/aap-skills/aap-26-automacoes ~/.cursor/skills/
cp -r /tmp/aap-skills/aap-26-operacao ~/.cursor/skills/
```

### Instalar um skill específico

```bash
# Apenas o skill de instalação
git clone -b v2.6 --filter=blob:none --sparse git@github.com:laurobmb/skills-cursor-aap-2.6.git /tmp/aap-skills
cd /tmp/aap-skills
git sparse-checkout set aap-26-instalacao
cp -r aap-26-instalacao ~/.cursor/skills/
```

### Manter atualizado

```bash
cd /tmp/aap-skills
git pull origin v2.6
cp -r aap-26-instalacao ~/.cursor/skills/
cp -r aap-26-automacoes ~/.cursor/skills/
cp -r aap-26-operacao ~/.cursor/skills/
```

---

## Como usar no Cursor

Após instalar, abra o Cursor e use o agente normalmente. Os skills são carregados automaticamente quando o contexto é relevante, ou podem ser invocados explicitamente:

```
Usando aap-26-instalacao, quais são os requisitos mínimos de IOPS para o PostgreSQL?

Usando aap-26-instalacao, quais portas preciso liberar no firewall para o Automation Mesh?

Usando aap-26-automacoes, como configurar um Rulebook para receber webhook do GLPI?

Usando aap-26-operacao, como configurar LDAP no Platform Gateway?
```

---

## Estrutura de cada skill

```
aap-26-<nome>/
├── SKILL.md              # Core: modelos mentais, componentes, índice
├── chapters/             # Capítulos carregados sob demanda
│   ├── ch01-*.md
│   ├── ch02-*.md
│   └── ...
├── glossary.md           # Termos técnicos com definições precisas
├── patterns.md           # Padrões de decisão arquitetural
└── cheatsheet.md         # Tabelas rápidas: portas, versões, variáveis
```

---

## Origem do conteúdo

Gerado a partir da documentação oficial Red Hat (AAP 2.6):

- *Planning your installation*
- *Containerized installation*
- *Installing on OpenShift Container Platform*
- *Tested deployment models*
- *Using automation decisions (EDA)*
- *Configuration as Code*
- *Creating and using Execution Environments*
- *Access management and authentication*
- *Hardening and compliance*
- *Troubleshooting Ansible Automation Platform*

---

## Contribuindo

Pull requests são bem-vindos para corrigir imprecisões ou adicionar conteúdo de novos guias do AAP 2.6.
