# AZ-104 - Microsoft Azure Administrator

## Estratégia para Passar na AZ-104

A AZ-104 é uma certificação prática focada na administração do Azure. É fundamental compreender identidade, governança, computação, redes, armazenamento, monitoramento e backup.

---

# 1. Microsoft Entra ID (Azure AD)

## Tipos de Usuário
- **Member**: usuário interno do tenant.
- **Guest**: usuário externo convidado via B2B.

## Grupos
- **Security Group**: controle de permissões.
- **Microsoft 365 Group**: colaboração com Teams, Outlook e SharePoint.

## Membership Types
- Assigned
- Dynamic User
- Dynamic Device

## Licenças
- Free
- P1
- P2

## SSPR (Self-Service Password Reset)
Métodos recomendados:
- Microsoft Authenticator
- E-mail alternativo
- Telefone

---

# 2. Hierarquia Azure

```text
Tenant
 └─ Management Group
      └─ Subscription
            └─ Resource Group
                  └─ Resource
```

---

# 3. RBAC

## Roles Principais

### Owner
- Controle total
- Pode delegar permissões

### Contributor
- Criar, alterar e remover recursos
- Não pode delegar permissões

### Reader
- Apenas leitura

---

# 4. Azure Policy

Usada para governança e conformidade.

Exemplos:
- Exigir tags
- Restringir regiões
- Restringir tipos de recursos

---

# 5. Tags

Exemplo:

```text
Departamento=TI
Ambiente=Producao
CentroDeCusto=1234
```

Usos:
- Custos
- Governança
- Relatórios

---

# 6. Virtual Networks (VNet)

## Address Space
```text
10.0.0.0/16
```

## Subnets
```text
10.0.1.0/24
10.0.2.0/24
```

## VNet Peering
- Comunicação entre VNets
- Sem transitividade

## Hub and Spoke
- Hub: serviços compartilhados
- Spoke: aplicações isoladas

---

# 7. NSG (Network Security Group)

Pode ser associado a:
- Subnet
- NIC

Não pode ser associado diretamente à VNet.

---

# 8. Máquinas Virtuais

Recursos normalmente criados:
- NIC
- Disco
- NSG
- IP Público (opcional)

---

# 9. Load Balancer

Camada 4:
- TCP
- UDP

---

# 10. Application Gateway

Camada 7:
- HTTPS
- SSL Offload
- Path-Based Routing
- Multi-Domain

---

# 11. Azure Bastion

Acesso seguro via:
- RDP
- SSH

Subnet obrigatória:

```text
AzureBastionSubnet
```

---

# 12. Storage Account

## Replicação
- LRS
- ZRS
- GRS
- RA-GRS
- RA-GZRS

## Performance
- Standard
- Premium

---

# 13. Blob Storage

## Access Tiers
- Hot
- Cool
- Cold
- Archive

## Lifecycle Management

```text
Hot -> Cool -> Cold -> Archive
```

---

# 14. Azure Files

Protocolo:

```text
SMB (porta 445)
```

---

# 15. SAS vs Access Keys

## Access Key
- Acesso total ao Storage Account

## SAS Token
- Permissões específicas
- Data de expiração
- Escopo limitado

---

# 16. Private Endpoint

Benefícios:
- IP privado
- Mais segurança
- Tráfego interno Azure

---

# 17. DNS

## Público
Resolve nomes da Internet.

## Privado
Resolve nomes internos das VNets.

---

# 18. Backup

## Recovery Services Vault
- VMs
- Azure Files
- SQL em VM

## Backup Vault
- Azure Disks
- Blob Storage
- AKS

## Backup Center
Gerenciamento centralizado.

---

# 19. Azure Monitor

## Métricas
- CPU
- Memória
- Disco

## Logs
- Eventos
- Alterações
- Auditoria

---

# 20. Network Watcher

Ferramentas importantes:
- Topology
- Connection Troubleshoot
- NSG Diagnostics
- IP Flow Verify
- Packet Capture
- Next Hop

---

# 21. Containers

## Azure Container Instance (ACI)
- Containers sem cluster
- Execução rápida

## Azure Container Registry (ACR)
- Registry privado de imagens Docker

---

# Checklist Final

✅ Entra ID
✅ RBAC
✅ Azure Policy
✅ Management Groups
✅ VNets
✅ Peering
✅ NSG
✅ Storage Account
✅ Blob Storage
✅ Azure Files
✅ Private Endpoint
✅ DNS
✅ Backup
✅ Azure Monitor
✅ Network Watcher
✅ ACI
✅ ACR
