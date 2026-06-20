# Ch03 — Automation Mesh em OCP (Operator)

## Adicionar Execution/Hop Node via UI

```
Automation Execution → Infrastructure → Instances → Add instance

Campos:
  Host name: FQDN ou IP público (DNS privado causa problemas — usar IP)
  Description: (opcional)
  Listener port: 27199 (padrão receptor)
  Instance type: execution | hop
  
Opções:
  ☑ Enable instance         → disponível para rodar jobs
  ☐ Managed by policy       → deixar para política automática de instance groups
  Peers from control nodes: ☑ (execução direto do control) | ☐ (via hop)
```

> Após salvar → **Associate peers** → **Download Bundle** (tar.gz com certs e install bundle)

## Instalar o Bundle no Novo Nó

```bash
# No nó remoto: extrair bundle
tar -xzf instance-bundle.tar.gz
cd instance-bundle/

# Editar inventory.yml
all:
  hosts:
    remote-execution:
      ansible_host: 10.0.1.50         # IP do novo nó
      ansible_user: ec2-user
      ansible_ssh_private_key_file: ~/.ssh/id_rsa

# Instalar receptor collection
ansible-galaxy install -r requirements.yml

# Abrir porta do receptor
sudo firewall-cmd --permanent --zone=public --add-port=27199/tcp
sudo firewall-cmd --reload

# Executar playbook de instalação
ansible-playbook -i inventory.yml install_receptor.yml
```

> Requer openssl: `openssl -v` (se não existir: `sudo dnf install -y openssl`)

## Topology Viewer

```
Automation Execution → Infrastructure → Topology View

Controles:
  zoom-in / zoom-out / expand (fit to screen) / reset view
  Click em nó → ver detalhes (hostname, tipo, status)
  Click no link hostname → página de detalhes do nó

Status por cor:
  Verde   = Ready
  Amarelo = Unavailable
  Vermelho = Error
  
Link statuses:
  Established = conexão ativa entre nós
  Adding      = nó sendo adicionado à topologia
  Removing    = nó sendo removido
```

## Mesh Ingress — Para Redes Sem Inbound

Quando a rede impede conexões inbound para os control nodes, usa-se Mesh Ingress:
um hop node dentro do cluster K8s com um Route/URL para acesso externo.

```yaml
# Criar arquivo oc_meshingress.yml
apiVersion: automationcontroller.ansible.com/v1alpha1
kind: AutomationControllerMeshIngress
metadata:
  name: mesh-ingress-datacenter-b
  namespace: aap
spec:
  deployment_name: aap-controller  # nome do CR do controller
```

```bash
oc apply -f oc_meshingress.yml
```

**Configurar o execution node para conectar ao Mesh Ingress:**
```
Instance type: execution
Listener port: (vazio)
Options:
  ☑ Enable instance
  ☐ Peers from control nodes   ← IMPORTANTE: não marcar
→ Associate peer: selecionar o hop node criado pelo Mesh Ingress
→ Download Bundle → Instalar
```

## Pull Secret para EE em Execution Nodes Remotos

```bash
# Criar secret no namespace do controller
oc create secret generic ee-pull-secret \
  --from-literal=username=<rhn-username> \
  --from-literal=password=<rhn-password> \
  --from-literal=url=registry.redhat.io

# Referenciar no CR do Controller
oc edit automationcontrollers aap-controller -o yaml
# Adicionar:
spec:
  ee_pull_credentials_secret: ee-pull-secret
```

## Remover Instância

```
Automation Execution → Infrastructure → Instances
→ Selecionar instância → Remove

Notas:
- Jobs ativos nessa instância terminam antes da remoção
- Se VM for deletada antes de remover no UI → mostra erro
- Reinstalar bundle somente se topologia mudar (hop nodes alterados)
```

## Upgrade de Receptor

```bash
# Verificar versão atual
receptor --version

# Atualizar
sudo dnf update ansible-runner receptor -y

# Reiniciar serviço
sudo systemctl restart receptor

# Verificar
receptor --version
```

> Red Hat recomenda atualizar receptor após qualquer atualização do control plane.
