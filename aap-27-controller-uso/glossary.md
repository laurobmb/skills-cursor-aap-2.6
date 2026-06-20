# Glossário — AAP 2.7 Controller Uso

| Termo | Definição |
|---|---|
| **Job Template (JT)** | Definição reutilizável de um job Ansible: playbook + inventário + credenciais + parâmetros |
| **Workflow Job Template** | Cadeia de JTs e outros recursos executados em sequência ou paralelo com lógica condicional |
| **Job Slicing** | Divisão automática de um JT em múltiplos jobs menores, cada um rodando contra uma fração do inventário |
| **Survey** | Formulário de input no JT que coleta variáveis antes do lançamento |
| **Convergence Node** | Node de workflow que aguarda múltiplos nós pai completarem antes de executar |
| **Smart Inventory** | *(Deprecated)* Coleção de hosts filtrada por fatos armazenados; usar Constructed Inventory |
| **Constructed Inventory** | Inventário criado dinamicamente a partir de múltiplos inventários com filtros e grupos calculados |
| **SCM** | Source Control Management — Git, SVN etc. |
| **Branch Override** | Capacidade de um JT especificar uma branch diferente da branch padrão do projeto |
| **Git Refspec** | Mapeamento de refs do remote para refs locais (útil para buscar PRs do GitHub) |
| **Extra Variables** | Variáveis adicionais passadas ao Ansible via `--extra-vars`, sobrescrevem variáveis de inventário |
| **Instance Group** | Agrupamento lógico de instâncias do Controller com políticas de scheduling |
| **Container Group** | Tipo especial de Instance Group que roda jobs em pods Kubernetes/OCP |
| **Job Tags / Skip Tags** | Filtros para executar ou pular tasks específicas por tag Ansible |
| **Verbosity** | Nível de detalhe do output: Normal / Verbose / More Verbose / Debug |
| **Timeout** | Tempo máximo (segundos) antes de cancelar o job; 0 = padrão global |
| **Webhook SCM** | Disparador automático de job ao receber push/PR de um repositório Git |
| **Notification Template** | Configuração de canal de alerta: Slack, email, webhook, PagerDuty etc. |
| **Approval Node** | Node de workflow que pausa a execução aguardando aprovação manual |
| **Terraform State** | Fonte de inventário dinâmico que lê o arquivo de state do Terraform |
