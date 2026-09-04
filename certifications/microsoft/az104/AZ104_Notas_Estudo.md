# AZ-104 - Microsoft Azure Administrator

## Estratégia para Passar na AZ-104

A AZ-104 é uma certificação prática focada na administração do Azure. É fundamental compreender identidade, governança, computação, redes, armazenamento, monitoramento e backup.

---

# 1. Microsoft Entra ID (Azure AD)

## Tipos de Usuário
- **Member**: usuário interno do tenant.
- **Guest**: usuário externo convidado via B2B (emails microsoft).

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

#### P1 (Entra ID P1 is designed for standard hybrid cloud environments requiring core security and management)
- Conditional Access: Enforce rules based on user, location, device state, or application.
- Self-Service Password Reset (SSPR): Allow cloud and on-premises users to reset their own passwords.
- Dynamic Groups: Automatically manage group memberships based on user attributes
- Inclusions: Often bundled in Microsoft Entra pricing plans like Microsoft 365 Business Premium and E3

#### P2 (Entra ID P2 includes all P1 features and layers on automated security intelligence for higher-risk or heavily regulated organizations)
- Entra ID Protection: Continuously monitor real-time sign-in and user risk signals (such as impossible travel or leaked credentials)
- Risk-Based Conditional Access: Automatically block or challenge sign-ins dynamically calculated as high risk.
- Privileged Identity Management (PIM): Enforce just-in-time, time-bound, and approval-based admin access instead of permanent administrative roles.
- Access Reviews: Run recurring automated reviews to audit and certify user permissions or group memberships.
- Inclusions: Bundled within Microsoft 365 E5 or available as a standalone license.

## SSPR (Self-Service Password Reset)
Acesso:
- None
- Selected (Groups)
- All

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

## Entra ID Tenant - Subscriptions

As Azure Subscriptions (assinaturas do Azure) e o Entra ID (antigo Azure Active Directory) possuem uma relação de dependência direta, onde o Entra ID atua como o sistema de identidade e controle de acesso para a assinatura.

1. Relação de Confiança (1 para Muitos)
- Uma assinatura confia em apenas um diretório: Cada Azure Subscription está vinculada a uma única instância do Entra ID. Ela confia nesse diretório para autenticar usuários, serviços e dispositivos.
- Um Entra ID pode ter várias assinaturas: Um único diretório do Entra ID pode estar associado a múltiplas assinaturas do Azure (por exemplo: Assinatura de Produção, Assinatura de Desenvolvimento e Assinatura de Testes).

2. Ciclo de Vida e Vinculação
- Mudança de Diretório: É possível alterar o diretório do Entra ID ao qual uma assinatura está vinculada. Quando você move uma assinatura para um novo Entra ID, todas as atribuições de controle de acesso (Azure RBAC) são perdidas, pois os usuários do diretório antigo não existem no novo. No entanto, os recursos físicos (como VMs e dados) continuam intactos.
- Faturamento vs. Identidade: O proprietário da conta de faturamento (quem paga pela assinatura) pode ser um usuário externo, mas o acesso aos recursos internos da assinatura sempre dependerá dos usuários cadastrados no Entra ID associado.

## Um Management Group

Management Group é um contêiner lógico usado para organizar e governar múltiplas assinaturas do Azure (Azure Subscriptions).Ele permite que você aplique políticas de governança (Azure Policies), controles de acesso (Azure RBAC) e limites de conformidade em larga escala. Qualquer assinatura colocada dentro de um Management Group herda automaticamente todas as regras definidas para aquele grupo.

💡 Exemplo Prático de Uso

Imagine que você é o Arquiteto de Nuvem de uma empresa global chamada TechCorp, que possui 20 assinaturas do Azure divididas entre diferentes departamentos (Finanças, RH, Engenharia) e ambientes (Produção, Desenvolvimento).Sem os Management Groups, você teria que configurar as permissões e regras de segurança manualmente em cada uma das 20 assinaturas. Com os Management Groups, você cria uma árvore hierárquica para automatizar isso.A Estrutura da Hierarquia

```text
                  [ Root Management Group ]
                              │
                      [ TechCorp-Root ]
                              │
            ┌─────────────────┴─────────────────┐
            ▼                                   ▼
    [ MG-Produção ]                      [ MG-Sandbox / Dev ]
     (Alta Segurança)                     (Ambiente Livre)
            │                                   │
    ┌───────┴───────┐                           ├───────────────────┐
    ▼               ▼                           ▼                   ▼
[Sub-Finanças]  [Sub-RH]                [Sub-Dev-Engenharia]  [Sub-Testes-RH]
```

### Como a governança é aplicada na prática:

1. Governança de Custos e Segurança no ambiente de Produção:

      Você aplica uma Azure Policy no nível do grupo `[ MG-Produção ]` que proíbe a criação de máquinas virtuais extremamente caras (ex: série G) e exige que o backup esteja ativado.
      
      - O resultado: As assinaturas `[Sub-Finanças]` e `[Sub-RH]` herdam essa regra instantaneamente. Nenhum desenvolvedor conseguirá estourar o orçamento nesses ambientes.

2. Liberdade controlada no ambiente de Desenvolvimento: 

      No grupo `[ MG-Sandbox / Dev ]`, você não aplica essa restrição de custos para permitir testes, mas aplica uma política que obriga o uso de Tags (ex: CriadoPor, Projeto) para rastrear os gastos.
      
      - O resultado: Os engenheiros têm liberdade para criar o que precisam nas assinaturas de desenvolvimento, mas a empresa ainda mantém o controle de quem é dono de cada recurso.


3. Controle de Acesso Centralizado (Azure RBAC):

      A equipe de Auditoria de TI precisa ler os dados de todas as assinaturas da empresa. Em vez de dar acesso a eles 20 vezes, você atribui a função de Reader (Leitor) no grupo `[ TechCorp-Root ]`.

      - O resultado: Eles ganham acesso de leitura a todas as assinaturas atuais e a qualquer nova assinatura que for criada no futuro.

## Administrative Unit

Uma Administrative Unit (AU - Unidade Administrativa) é um contêiner lógico do Entra ID utilizado para agrupar recursos exclusivamente de identidade, como usuários, grupos e dispositivos

Exemplo: Delegar ao gerente regional de TI o poder de resetar senhas apenas dos funcionários da filial de São Paulo.


🔑 Principais Roles do Entra ID (Foco em Identidade)

Essas funções controlam o diretório do Entra ID e são atribuídas no nível global da empresa ou dentro de Unidades Administrativas (AUs).

- Global Administrator (Administrador Global): Possui acesso total e irrestrito a todos os recursos administrativos do Entra ID e à maioria das configurações de serviços Microsoft (como M365). É a conta com maior privilégio no ambiente.

- User Administrator (Administrador de Usuários): Pode criar, excluir e gerenciar usuários e grupos. Também pode redefinir senhas de usuários comuns e de outros administradores de menor privilégio.

- Security Administrator (Administrador de Segurança): Gerencia políticas de segurança, Conditional Access (Acesso Condicional), monitora logs de auditoria e responde a alertas de ameaças à identidade.

- Application Administrator (Administrador de Aplicativos): Pode registrar aplicativos no Entra ID, configurar o logon único (SSO) e gerenciar permissões de APIs que os aplicativos utilizam.

- Billing Administrator (Administrador de Faturamento): Faz compras, gerencia assinaturas de licenças do Microsoft 365, monitora faturas e abre chamados de suporte relacionados a faturamento.

---

# 3. RBAC (Access Control IAM)

## Roles Principais

As permissões podem ser dadas desde o management group, subscription, resource group, ate resources. 

RBAC sao herdadas.

RBAC pode ser adicionados:
- Management Groups
- Subscriptions
- Resource Groups
- Resources

### Owner
- Controle total
- Pode delegar permissões

### Contributor
- Criar, alterar e remover recursos
- Não pode delegar permissões

### Reader
- Apenas leitura

### User Access Administrator (Administrador de Acesso do Usuário): 
- Uma função peculiar que não permite gerenciar recursos, mas dá permissão exclusiva para gerenciar o acesso de outros usuários (atribuir funções RBAC). - É ideal para equipes de Governança e Compliance

### Separação de Funções e Funções de Acesso (RBAC)

O gerenciamento de acessos é dividido em duas camadas distintas, embora integradas:
- Funções do Entra ID: Controlam o gerenciamento do próprio diretório (criar usuários, resetar senhas, gerenciar domínios). Exemplos: Global Administrator, User Administrator.
- Funções do Azure (Azure RBAC): Controlam o gerenciamento dos recursos dentro da assinatura (criar máquinas virtuais, bancos de dados, redes). Os usuários do Entra ID recebem essas permissões. Exemplos: Owner, Contributor, Reader.

---

# 4. Azure Policy

Usada para governança e conformidade.

Policys sao herdadas.

Pode ser aplicacas:
- Management Groups
- Subscriptions
- Resource Groups

Exemplos:
- Exigir tags
- Restringir regiões
- Restringir tipos de recursos

---

# 5. Tags

Tags NAO sao herdadas (podem ser aplicadas atraves de Policy)

Tags NAO podem ser adicionadas a Management Groups.

Podem ser adicionadas:
- Subscriptions
- Resource Groups
- Resources

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
