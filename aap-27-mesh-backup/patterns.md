# Patterns — AAP 2.7 Mesh e Backup

## Pattern 1: Mesh para Datacenter Remoto com Hop Node

```
Cenário: Executar automação em site B (baixa latência, rede restrita).
Solução: Execution node no site B conectando via hop node no DMZ.
```

```ini
# inventory (containerizado)
[automationcontroller]
aap-ctrl.hq.empresa.org

[execution_nodes]
# Local HQ
aap-exec-hq.hq.empresa.org receptor_type=execution

# Hop no DMZ
aap-hop-dmz.hq.empresa.org receptor_type=hop receptor_peers=aap-ctrl.hq.empresa.org

# Execution remoto Site B
aap-exec-siteb.siteb.empresa.org receptor_type=execution receptor_peers=aap-hop-dmz.hq.empresa.org

[instance_group_hq]
aap-exec-hq.hq.empresa.org

[instance_group_site_b]
aap-exec-siteb.siteb.empresa.org

# Re-executar installer:
# ansible-playbook -i inventory ansible.containerized_installer.install
```

## Pattern 2: Mesh Ingress para OCP com Execution Node na Nuvem

```
Cenário: Controller em OCP on-premise, execution nodes em AWS/Azure.
Solução: Mesh Ingress cria Route externo para os clouds nodes se conectarem.
```

```bash
# 1. Criar Mesh Ingress no OCP
cat > mesh-ingress.yaml << 'EOF'
apiVersion: automationcontroller.ansible.com/v1alpha1
kind: AutomationControllerMeshIngress
metadata:
  name: mesh-ingress-aws
  namespace: aap
spec:
  deployment_name: aap-controller
EOF
oc apply -f mesh-ingress.yaml

# 2. No AAP UI: Adicionar execution node (AWS EC2)
#    Host name: 10.0.1.50  (IP privado da EC2)
#    Instance type: execution
#    Listener port: 27199
#    Peers from control nodes: ☐ (vai peer com o hop ingress)
#    Associate peer: selecionar "mesh-ingress-aws"

# 3. Download Bundle → copiar para EC2 → instalar
scp bundle.tar.gz ec2-user@10.0.1.50:~/
ssh ec2-user@10.0.1.50
tar -xzf bundle.tar.gz
ansible-playbook -i inventory.yml install_receptor.yml
```

## Pattern 3: Backup Antes de Upgrade com Validação

```bash
#!/bin/bash
# pre-upgrade-backup.sh

INSTALLER_DIR="/opt/aap/ansible-automation-platform-containerized-setup-2.7.0"
BACKUP_DIR="/data/backups/aap-$(date +%Y%m%d-%H%M%S)"

echo "=== Iniciando backup pré-upgrade ==="
cd "$INSTALLER_DIR"

ansible-playbook -i inventory ansible.containerized_installer.backup \
  -e backup_dir="$BACKUP_DIR" \
  -e use_archive_compression=true \
  -e use_db_compression=true

if [ $? -eq 0 ]; then
  echo "✓ Backup concluído em: $BACKUP_DIR"
  ls -lh "$BACKUP_DIR"
  echo "=== Pode prosseguir com o upgrade ==="
else
  echo "✗ ERRO no backup! Não fazer upgrade."
  exit 1
fi
```

## Pattern 4: Instance Groups por Ambiente (dev/prod/dmz)

```ini
# inventory — usando nomes que o AAP reconhece como Instance Groups
[execution_nodes]
# Ambiente DEV
aap-exec-dev-1.empresa.org receptor_type=execution
aap-exec-dev-2.empresa.org receptor_type=execution

# Ambiente PROD (servidores mais robustos)
aap-exec-prod-1.empresa.org receptor_type=execution
aap-exec-prod-2.empresa.org receptor_type=execution

# DMZ
aap-exec-dmz.empresa.org receptor_type=execution
aap-hop-dmz.empresa.org receptor_type=hop receptor_peers=aap-ctrl.empresa.org

# Instance groups automáticos (instance_group_ prefix)
[instance_group_dev]
aap-exec-dev-1.empresa.org
aap-exec-dev-2.empresa.org

[instance_group_producao]
aap-exec-prod-1.empresa.org
aap-exec-prod-2.empresa.org

[instance_group_dmz]
aap-exec-dmz.empresa.org
```

```yaml
# CaC: forçar execução em production no instance group correto
ansible.platform.job_template:
  name: "Deploy Producao"
  instance_groups:
    - "instance_group_producao"
```
