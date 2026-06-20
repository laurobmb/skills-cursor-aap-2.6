# Glossário — AAP 2.7 Mesh e Backup

| Termo | Definição |
|---|---|
| **Automation Mesh** | Rede overlay peer-to-peer sobre TLS que distribui execução de automação entre nós geograficamente dispersos |
| **Hybrid Node** | Nó do control plane que executa tanto funções de controle quanto de execução de jobs. Tipo padrão |
| **Control Node** | Nó apenas de controle: project updates, management jobs. Não executa jobs regulares |
| **Execution Node** | Nó apenas de execução: roda `ansible-playbook` com podman isolation. Não tem acesso ao banco de controle |
| **Hop Node** | Nó de roteamento de tráfego (similar a jump host). Não executa automation. Usado para atravessar redes restritas |
| **Receptor** | Daemon de comunicação de rede que implementa o mesh usando TLS/TCP. Porta padrão 27199 |
| **Instance Group** | Agrupamento lógico de instâncias para controle de onde jobs são executados |
| **Container Group** | Tipo especial de Instance Group que executa jobs em pods K8s/OCP (container groups não usam o algoritmo de capacidade normal) |
| **receptor_peers** | Variável do inventory (containerizado) que define conexões peer-to-peer. Substituiu `peers` do installer RPM |
| **receptor_type** | Variável do inventory (containerizado) que define o tipo de nó. Substituiu `node_type` |
| **Topology Viewer** | Ferramenta visual na UI do Controller para ver o estado e conexões do mesh |
| **Mesh Ingress** | CR K8s que cria um hop node dentro do cluster com Route URL público para permitir conexão de execution nodes externos sem inbound |
| **Install Bundle** | Arquivo tar.gz gerado ao adicionar um nó via UI. Contém certificados TLS, CA, receptor config e playbook de instalação |
| **AnsibleAutomationPlatformBackup** | Custom Resource do Operator para criar backup de um deployment AAP em OCP |
| **AnsibleAutomationPlatformRestore** | Custom Resource do Operator para restaurar um deployment AAP em OCP a partir de um backup |
| **backup_dir** | Variável de inventory que define o diretório de destino dos backups (padrão: `./backups/`) |
| **Peers** | Relacionamento de conectividade entre nós do mesh. Após estabelecida, comunicação é bidirecional |
| **Control Plane** | Conjunto de serviços persistentes do AAP: web server, API, task dispatcher, project updates |
| **Execution Plane** | Conjunto de nós que executam jobs. Podem ser geograficamente separados do control plane |
