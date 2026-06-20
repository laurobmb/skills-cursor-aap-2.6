# Cheatsheet — AAP 2.7 Mesh e Backup

## Tipos de Nó

```
hybrid    → controle + execução (padrão no control plane)
control   → apenas controle (sem jobs regulares)
execution → apenas execução de jobs
hop       → apenas roteamento (jump host receptor)
```

## Containerizado: Variáveis de Inventory

```ini
receptor_type=execution|hop        # ao invés de node_type
receptor_peers=host1,host2         # ao invés de peers= (apenas hostnames, não grupos)
```

## Instalar Bundle em Novo Nó (OCP)

```bash
# Após Download Bundle da UI:
tar -xzf bundle.tar.gz && cd bundle/
# Editar inventory.yml → ansible_host, ansible_user, ansible_ssh_private_key_file
ansible-galaxy install -r requirements.yml
sudo firewall-cmd --permanent --zone=public --add-port=27199/tcp
ansible-playbook -i inventory.yml install_receptor.yml
```

## Mesh Ingress (Sem Inbound)

```yaml
apiVersion: automationcontroller.ansible.com/v1alpha1
kind: AutomationControllerMeshIngress
metadata:
  name: mesh-ingress-site-b
  namespace: aap
spec:
  deployment_name: aap-controller
```

> Execution node: Peers from control nodes = ☐, peer com o hop do Ingress

## Backup Containerizado

```bash
# Backup
ansible-playbook -i inventory ansible.containerized_installer.backup

# Restore (mesmo ambiente)
ansible-playbook -i inventory ansible.containerized_installer.restore
```

## Backup OCP

```bash
oc apply -f ansibleautomationplatformbackup.yaml
oc get ansibleautomationplatformbackup -n aap
```

## Quando Fazer Backup

```
OBRIGATÓRIO: Antes de qualquer upgrade
RECOMENDADO: Semanal + após mudanças significativas
```
