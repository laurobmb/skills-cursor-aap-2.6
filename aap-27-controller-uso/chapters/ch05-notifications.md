# Ch05 — Notificações

## Tipos de Notificação

| Tipo | Protocolo | Quando usar |
|---|---|---|
| **Email** | SMTP | Alertas internos por email |
| **Slack** | API Slack | Integração com canais da equipe |
| **Webhook** | HTTP POST | Integração com qualquer sistema |
| **PagerDuty** | API PagerDuty | Alertas de on-call/incidentes |
| **Grafana** | API Grafana | Anotações em dashboards |
| **HipChat** | API HipChat | (legado) |
| **IRC** | IRC | Chat interno |
| **Mattermost** | API Mattermost | Chat open-source |
| **Rocket.Chat** | API Rocket.Chat | Chat self-hosted |
| **Twilio** | API Twilio | SMS |

## Criar Notificação

```
Access Management → Notifications → Create notification

1. Selecionar tipo
2. Preencher campos específicos do tipo
3. Salvar
4. Testar clicando em "Test"
```

## Associar Notificação a um Job Template

```
Job Template → Notifications tab → Enable para:
  - Started  (ao iniciar o job)
  - Success  (ao completar com sucesso)
  - Failure  (ao falhar)
  - Approval required (workflow: aguardando aprovação)
  - Approved  (workflow: aprovado)
  - Denied    (workflow: negado)
  - Timed out (workflow: aprovação expirou)
```

## Webhook — Configuração

```yaml
# Notificação do tipo Webhook
Type: Webhook
Target URL: https://meu-sistema.empresa.org/aap-events
HTTP Method: POST
HTTP Headers:
  Authorization: Bearer meu-token-interno
  Content-Type: application/json
Username: (opcional, para basic auth)
Password: (opcional, para basic auth)
```

**Payload enviado (exemplo):**
```json
{
  "job_id": 123,
  "name": "Deploy App",
  "status": "failed",
  "url": "https://aap.empresa.org/#/jobs/123",
  "created_by": "joao.silva",
  "started": "2025-01-15T10:00:00Z",
  "finished": "2025-01-15T10:05:30Z",
  "extra_vars": {}
}
```

## Slack — Configuração

```
Type: Slack
Token: xoxb-xxxxxxxxxxxx-xxxxxxxxxxxxxxxx (Bot Token do Slack)
Destination Channels: #aap-alerts, #infra-prod
Notification Color: #e03e2f (vermelho para falha)
```

## Email — Configuração

```
Type: Email
Host: smtp.empresa.org
Port: 587
Sender Email: aap@empresa.org
Recipients: time-infra@empresa.org
TLS: enabled
Recipient list: [ops@empresa.org, devops@empresa.org]
```

## Via CaC — Criar Notificação

```yaml
ansible.platform.notification_template:
  name: "Slack - Alertas Infraestrutura"
  organization: "Infraestrutura"
  notification_type: slack
  notification_configuration:
    token: "{{ vault_slack_token }}"
    channels:
      - "#aap-alertas"
    hex_color: ""
  controller_host: "https://aap.empresa.org"
  controller_oauthtoken: "{{ gateway_token }}"

# Associar ao JT
ansible.platform.job_template:
  name: "Deploy App"
  # ... outros campos ...
  notification_templates_error:
    - "Slack - Alertas Infraestrutura"
  notification_templates_success:
    - "Slack - Alertas Infraestrutura"
```
