# Ch05 — Backup e Restore

## Recomendações de Backup

Fazer backup:
- **Antes de qualquer upgrade** (AAP ou OCP)
- **Após mudanças de configuração significativas**
- **Semanalmente** (especialmente se auto-upgrades estão habilitados)

O backup inclui:
- Bancos de dados PostgreSQL (controller, hub, eda, gateway)
- Arquivos de configuração
- Arquivos de dados (automation hub, etc.)

---

## Ambiente Containerizado

### Backup

```bash
# Ir ao diretório do installer
cd /path/to/ansible-automation-platform-containerized-setup-<versão>/

# Backup simples (padrão salva em ./backups/)
ansible-playbook -i inventory ansible.containerized_installer.backup

# Customizar diretório de destino
ansible-playbook -i inventory ansible.containerized_installer.backup \
  -e backup_dir=/data/aap-backups/2025-01-15

# Excluir snapshots do hub para economizar espaço
ansible-playbook -i inventory ansible.containerized_installer.backup \
  -e hub_data_path_exclude="['*/.snapshots', '*/.snapshots/*']"
```

### Variáveis de Compressão (inventory)

```ini
[all:vars]
# Compressão global de arquivos
use_archive_compression=true

# Compressão de banco de dados
use_db_compression=true

# Por componente (opcional)
#controller_use_archive_compression=true
#eda_use_archive_compression=true
#hub_use_archive_compression=true
#gateway_use_archive_compression=true
```

### Restore — Mesmo Ambiente

```bash
# Restore padrão (usa ./backups/ como fonte)
ansible-playbook -i inventory ansible.containerized_installer.restore

# Backup em diretório diferente
ansible-playbook -i inventory ansible.containerized_installer.restore \
  -e backup_dir=/data/aap-backups/2025-01-15
```

### Restore — Ambiente Diferente (Hostnames Diferentes)

> Não recomendado — usar apenas como workaround.

```bash
# 1. Instalar AAP no novo ambiente com mesma topologia
# 2. Copiar os arquivos de backup do ambiente original para o novo
# 3. Renomear os arquivos para refletir os novos hostnames:

cd backups/
# Original: gateway_env1-gateway-node1.tar.gz
# Renomear: 
mv gateway_env1-gateway-node1.tar.gz gateway_env2-gateway-node1.tar.gz

# 4. Para enterprise topology:
# Garantir que o arquivo com o .db do componente seja listado PRIMEIRO no inventory
vi inventory
[automationgateway]
env2-gateway-node1    ← este deve vir primeiro (tem gateway.db)
env2-gateway-node2

# 5. Executar restore
ansible-playbook -i inventory ansible.containerized_installer.restore
```

---

## Ambiente OCP (Operator)

### Backup via CR

```yaml
# Criar AnsibleAutomationPlatformBackup
apiVersion: aap.ansible.com/v1alpha1
kind: AnsibleAutomationPlatformBackup
metadata:
  name: aap-backup-pre-upgrade
  namespace: aap
spec:
  deployment_name: aap-main    # nome do CR AnsibleAutomationPlatform
  # Opcional: armazenar em PVC específico
  # backup_pvc: aap-backup-pvc
  # backup_pvc_namespace: aap
  # db_management_pod_image: ...
```

```bash
oc apply -f aap-backup.yaml

# Verificar status
oc get ansibleautomationplatformbackup -n aap
oc describe ansibleautomationplatformbackup/aap-backup-pre-upgrade -n aap
```

### Restore via CR

```yaml
apiVersion: aap.ansible.com/v1alpha1
kind: AnsibleAutomationPlatformRestore
metadata:
  name: aap-restore
  namespace: aap
spec:
  deployment_name: aap-main
  backup_name: aap-backup-pre-upgrade
```

### Compressão de Backup (OCP)

```yaml
# No CR do AnsibleAutomationPlatform ou no backup spec:
spec:
  backup_resource_requirements:
    requests:
      cpu: "100m"
      memory: "128Mi"
  # Compressão de banco de dados
  postgres_extra_args:
    - '--compress=6'   # nível 1-9 de compressão
```

---

## Backup de Metrics Service

```bash
# Via installer (recomendado)
ansible-playbook -i inventory backup.yml

# Manual — backup do banco
pg_dump -h localhost -U metrics_service -F c -b -v \
  -f /backup/metrics_service_$(date +%Y%m%d).dump \
  metrics_service

# Manual — backup dos volumes
tar -czf /backup/automationmetrics_data_$(date +%Y%m%d).tar.gz \
  /var/lib/aap/volumes/automationmetrics
```

```bash
# Restore via installer (recomendado)
ansible-playbook -i inventory restore.yml

# Restore manual:
# 1. Parar serviços
systemctl --user stop automation-metrics-web.service \
                      automation-metrics-tasks.service \
                      automation-metrics-scheduler.service

# 2. Restaurar banco
psql -h localhost -U postgres -c "DROP DATABASE IF EXISTS metrics_service;"
psql -h localhost -U postgres -c "CREATE DATABASE metrics_service;"
pg_restore -h localhost -U metrics_service -d metrics_service -v /backup/metrics_service_TIMESTAMP.dump

# 3. Iniciar serviços
systemctl --user start automation-metrics-web.service \
                     automation-metrics-tasks.service \
                     automation-metrics-scheduler.service
```
