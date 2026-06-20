---
name: aap-27-mesh-backup
description: >-
  Referência técnica de Automation Mesh e Backup/Restore no AAP 2.7. Cobre o conceito
  de Automation Mesh (plano de controle e execução, tipos de nó: hybrid, control, execution, hop),
  topologias de design (single node, minimal resilient, multi-hop, outbound-only), adição
  de nós de execução e hop em ambientes OCP (UI) e containerizados (inventory file),
  Mesh Ingress para redes sem inbound, topology viewer, e procedimentos completos de
  Backup e Restore para ambientes containerizados (backup.yml / restore.yml) e OCP
  (Operator CR AnsibleAutomationPlatformBackup). Use quando precisar expandir o mesh
  para um datacenter remoto, conectar nós através de hop (jump host), planejar alta
  disponibilidade de execução, executar backup antes de upgrade, ou restaurar um ambiente.
disable-model-invocation: true
---

# AAP 2.7 — Automation Mesh e Backup/Restore

## Índice

| Capítulo | Conteúdo |
|---|---|
| [ch01-mesh-conceitos](chapters/ch01-mesh-conceitos.md) | Conceitos, tipos de nó, planos de controle e execução, peers |
| [ch02-mesh-topologias](chapters/ch02-mesh-topologias.md) | Padrões de design: single node, multi-hop, outbound-only |
| [ch03-mesh-ocp](chapters/ch03-mesh-ocp.md) | Adicionar nós em OCP via UI, Mesh Ingress, topology viewer |
| [ch04-mesh-containerizado](chapters/ch04-mesh-containerizado.md) | Editar inventory file, receptor_peers, CA certificate |
| [ch05-backup-restore](chapters/ch05-backup-restore.md) | backup.yml / restore.yml (containerizado), AnsibleAutomationPlatformBackup (OCP) |

## Uso Rápido

- **Conceitos de plano de controle vs execução:** → ch01
- **Exemplo de inventory com hop node para site remoto:** → ch02 + ch04
- **Adicionar execution node em ambiente OCP:** → ch03
- **Mesh Ingress (rede sem inbound):** → ch03
- **Backup antes de upgrade (containerizado):** → ch05
- **Restore em novo ambiente (ambiente diferente):** → ch05
