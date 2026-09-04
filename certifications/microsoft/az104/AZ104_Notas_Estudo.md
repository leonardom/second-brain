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
Tenant (Entra ID)
 └─ Management Groups
      └─ Subscriptions
            └─ Resource Groups
                  └─ Resources
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

Management Group é um contêiner lógico usado para organizar e governar múltiplas assinaturas do Azure (Azure Subscriptions). Ele permite que você aplique políticas de governança (Azure Policies), controles de acesso (Azure RBAC) e limites de conformidade em larga escala. Qualquer assinatura colocada dentro de um Management Group herda automaticamente todas as regras definidas para aquele grupo.

💡 Exemplo Prático de Uso

Imagine que você é o Arquiteto de Nuvem de uma empresa global chamada TechCorp, que possui 20 assinaturas do Azure divididas entre diferentes departamentos (Finanças, RH, Engenharia) e ambientes (Produção, Desenvolvimento). Sem os Management Groups, você teria que configurar as permissões e regras de segurança manualmente em cada uma das 20 assinaturas. Com os Management Groups, você cria uma árvore hierárquica para automatizar isso. A Estrutura da Hierarquia

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

Policies sao herdadas.

Pode ser aplicadas:
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

# 6. Resource Groups

Um Resource Group (Grupo de Recursos) é um contêiner lógico dentro de uma Azure Subscription usado para agrupar e gerenciar recursos do Azure que compartilham o mesmo ciclo de vida.

No Azure, todo recurso (como uma máquina virtual, um banco de dados SQL ou uma rede virtual) deve pertencer a obrigatoriamente um único Resource Group. Ele funciona como uma "pasta organizada" que facilita a implementação, a exclusão, o monitoramento de custos e o controle de acesso desses recursos de forma conjunta.

### 💡 Exemplo Prático de Uso: 

O Sistema de E-commerceImagine que você está  implantando um sistema de E-commerce corporativo composto por:

- 1 Banco de dados SQL (para armazenar os pedidos e produtos)
- 2 Máquinas Virtuais (para hospedar o site)
- 1 Conta de Armazenamento (Storage Account para as imagens dos produtos)
- 1 Rede Virtual (VNet para conectar tudo com segurança)

Em vez de deixar esses componentes espalhados e misturados com outros sistemas da empresa, você cria um Resource Group chamado rg-ecommerce-prod.

O benefício prático: 

Quando a equipe de marketing pedir um relatório de custos do e-commerce, você não precisa somar cada recurso manualmente. Basta olhar o custo total gerado pelo rg-ecommerce-prod. Da mesma forma, se o projeto for descontinuado, você exclui o Resource Group e todos os recursos internos são deletados juntos automaticamente, evitando custos fantasmas com recursos esquecidos.

### 🛡️ Boas Recomendações e Práticas de Uso


#### 1. Agrupe pelo Ciclo de Vida (A regra de ouro)

Coloque no mesmo Resource Group apenas recursos que são criados, atualizados e excluídos juntos

- O erro comum: Criar um único Resource Group chamado rg-empresa-geral e colocar tudo dentro dele.
- A prática correta: Se você tem um ambiente de teste que será deletado na próxima sexta-feira, coloque todos os componentes dele em um grupo separado (ex: rg-projetoX-test).

#### 2. Separe por Ambiente (Ambientes isolados)

Nunca misture recursos de Produção com recursos de Desenvolvimento ou Homologação dentro do mesmo Resource Group.

- Isso evita que um desenvolvedor delete acidentalmente um banco de dados de produção ao tentar limpar o ambiente de testes.
- Facilita a aplicação de permissões de acesso (RBAC) mais rígidas para produção e mais flexíveis para desenvolvimento.

#### 3. Use e abuse de Tags (Etiquetas)

As tags são pares de chave-valor aplicados aos Resource Groups para facilitar a organização financeira e de governança. Sempre defina tags como:

- `Ambiente`: Produção / Dev / QA
- `CentroDeCusto`: TI_1020 / Marketing_3040
- `DonoDoProjeto`: equipe_engenharia@empresa.com

#### 4. Aplique Locks (Bloqueios de Recursos) para Segurança

Para Resource Groups críticos de produção, aplique um Resource Lock do tipo `CanNotDelete` (Não Excluir). Isso impede que qualquer usuário — até mesmo um administrador com função de Owner — delete o grupo ou seus recursos por engano sem antes remover manualmente o bloqueio.

#### 5. Adote um Padrão de Nomenclatura Claro

Mantenha os nomes previsíveis para que qualquer analista entenda o que há dentro do contêiner logo de cara. Um padrão muito utilizado pela indústria é:`rg-[NomeDoProjeto/Aplicativo]-[Ambiente]-[RegiãoOpcional]`
- Exemplo: `rg-portalrh-dev-eastus`
---

# 7. Virtual Networks (VNet)

Por padrão, duas VNets diferentes não se comunicam. Mesmo que elas estejam na mesma assinatura, na mesma região ou dentro do mesmo Resource Group, o tráfego entre elas é totalmente isolado por motivos de segurança.

Para estabelecer a comunicação entre duas VNets, você precisa configurar explicitamente um mecanismo de conectividade. Existem três formas principais de fazer isso, dependendo do seu cenário:

#### 1. Azure VNet Peering (A melhor e mais comum opção)

O VNet Peering (Emparelhamento de Redes) conecta duas VNets diretamente através da rede de backbone de alta velocidade da Microsoft.

- Como funciona: Uma vez configurado, as duas VNets passam a funcionar como se fossem uma única rede unificada. O tráfego de dados é criptografado, privado e possui baixíssima latência (atraso) e altíssima largura de banda.
- Tipos:
  - VNet Peering Regional: Conecta VNets que estão na mesma região do Azure (ex: ambas em East US).
  - Global VNet Peering: Conecta VNets localizadas em regiões geográficas diferentes (ex: uma em East US e outra no Brazil South).
- ⚠️ Regra de Ouro: Para que o Peering funcione, as duas VNets não podem ter espaços de endereço IP sobrepostos (ex: se a VNet A usa 10.0.0.0/16, a VNet B não pode usar o mesmo bloco).

#### 2. Arquitetura Hub-and-Spoke com Azure Firewall / NVA

Se você tem muitas VNets (por exemplo, 10 ou mais), criar Peering entre todas elas (malha total) se torna difícil de gerenciar. A recomendação da Microsoft é o modelo Hub-and-Spoke:
- A VNet Hub (Central): Funciona como o ponto central de conectividade. Ela geralmente abriga um Azure Firewall ou um appliance de segurança (NVA).
- As VNets Spokes (Raios): São as VNets de workloads (ex: uma VNet para o RH, outra para Finanças). Cada Spoke faz um VNet Peering apenas com a VNet Hub.
- A Comunicação: Se a VNet-Spoke-RH precisar falar com a VNet-Spoke-Finanças, o tráfego é direcionado para a VNet Hub, inspecionado pelo Firewall e depois encaminhado ao destino.

#### 3. VPN Gateway (VNet-to-VNet)

Esta opção conecta as duas VNets simulando uma conexão de internet segura (túnel VPN IPsec/IKE) através de Azure VPN Gateways.

- Quando usar: É uma alternativa antiga ou usada em cenários muito específicos, como quando você precisa criptografar o tráfego com chaves de criptografia customizadas de nível governamental, ou para conectar VNets em nuvens públicas diferentes (ex: Azure Comercial conectando com Azure Government).
- Desvantagem: É mais cara (você paga pelo preço por hora do gateway) e a velocidade é limitada pela capacidade do gateway (geralmente bem menor que o Peering).

## Address Space
```text
10.0.0.0/16
```

### 🎨 Exemplo Prático de Address Space (Sem Sobreposição)

Para organizar uma empresa de médio porte, podemos utilizar o bloco privado 10.0.0.0/8 (que oferece milhões de IPs) e fatiá-lo em blocos menores (/16) para cada VNet. Depois, dividimos cada VNet em sub-redes ainda menores (/24).Aqui está um desenho técnico perfeito que você pode usar como modelo:

```text
[ Bloco Pai da Empresa: 10.0.0.0/8 ]
  │
  ├──► VNet HUB (Central) ────────────────► Address Space: 10.0.0.0/16
  │     ├── Subnet-Firewall ──────────────► 10.0.1.0/24
  │     └── Subnet-VPN-Gateway ───────────► 10.0.2.0/24
  │
  ├──► VNet Spoke Produção (Prod) ────────► Address Space: 10.1.0.0/16
  │     ├── Subnet-Prod-Web ──────────────► 10.1.1.0/24
  │     └── Subnet-Prod-DB ───────────────► 10.1.2.0/24
  │
  └──► VNet Spoke Desenvolvimento (Dev) ──► Address Space: 10.2.0.0/16
        ├── Subnet-Dev-Web ───────────────► 10.2.1.0/24
        └── Subnet-Dev-DB ────────────────► 10.2.2.0/24

```

### Capacidade de IPs por bloco:

| Bloco CIDR | Total IPs  | IPs Utilizaveis | Uso comum | 
| ---------- | ---------- | --------------- | --------- |
| /8         | 16.777.216 | 16.777.211      | Bloco pai (ex: 10.0.0.0/8) |
| /16        | 65.536     | 65.531          | Tamanho ideal e padrão para uma VNet |
| /20        | 4.096      | 4.091           | VNets médias ou sub-redes muito grandes (como clusters AKS).|
| /21        | 2.048      |2.043            | Sub-redes robustas para grandes frotas de máquinas virtuais.|
| /22        | 1.024      | 1.019           | Sub-redes de infraestrutura geral.|
| /23        | 512        | 507             | Excelente tamanho para sub-redes de servidores de aplicação.|
/ 24         | 256        | 251             | O tamanho mais comum e recomendado para Sub-redes de produção.|
| /25        | 128        | 123             | Sub-redes menores (ex: banco de dados isolado).|
| /26        | 64         | 59              | Sub-redes para serviços específicos e controlados.
| /27        | 32         | 27              | O tamanho mínimo recomendado para a sub-rede do Azure Firewall.|
| /28        | 16         | 11              | Muito usado para a sub-rede do VPN Gateway (GatewaySubnet).|
| /29        | 8          | 3               | O menor bloco que o Azure permite criar como sub-rede.|

> ℹ️ **Nota:** Blocos menores como /30, /31 e /32 possuem menos de 5 IPs totais, portanto não podem ser criados como sub-redes utilizáveis dentro de uma VNet do Azure.

> ‼️**Importante:** Enderecos reservados no Azure: `0`, `1`, `2`, `4`


## Subnets
```text
10.0.1.0/24
10.0.2.0/24
```

Por padrão, todas as sub-redes (subnets) dentro da mesma rede virtual (VNet) do Azure se comunicam de forma automática e direta.

Isso acontece porque o Azure cria rotas do sistema (System Routes) integradas assim que a VNet é criada. Essas rotas padrão direcionam o tráfego entre qualquer endereço IP de todas as sub-redes daquela VNet sem a necessidade de nenhuma configuração adicional (como gateways ou roteadores virtuais).

## VNet Peering
- Comunicação entre VNets
- Sem transitividade
- Pode ser Regional ou Global Peering (tem custo)

## Hub and Spoke
- Hub: serviços compartilhados
- Spoke: aplicações isoladas

---

# 8. NSG (Network Security Group)

O Network Security Group (NSG - Grupo de Segurança de Rede) funciona como um firewall virtual para os seus recursos no Azure. Ele contém uma lista de regras de segurança que permitem ou bloqueiam o tráfego de rede (de entrada ou de saída) com base em IPs, portas e protocolos.

Pode ser associado a:
- Subnets
- NIC (Network Interface Card)

Não pode ser associado diretamente à VNet.

### ⚙️ Como Funcionam as Regras do NSG

As regras de um NSG são processadas em uma ordem lógica estrita baseada em prioridade:

- **Número de Prioridade (100 a 4096)**: As regras são lidas de forma sequencial, do menor número para o maior. Assim que uma regra corresponde ao tráfego, o processamento para — qualquer outra regra conflitante que esteja mais abaixo na lista é completamente ignorada.
- **A Combinação dos 5 Elementos**: Para que o tráfego ative uma regra, ele precisa bater com 5 critérios específicos: IP de Origem, Porta de Origem, IP de Destino, Porta de Destino e o Protocolo (TCP, UDP ou ICMP).

### 🚨 Regras Padrão (A Rede de Segurança)

Todo NSG já vem de fábrica com regras padrão ocultas. Você não pode deletá-las, mas elas possuem a menor prioridade possível (65000+), o que significa que você pode substituí-las facilmente criando regras próprias.

Os comportamentos padrão mais importantes de lembrar são:

1. **Permitir Entrada da VNet (AllowVNetInbound)**: Toda a comunicação entre sub-redes da mesma VNet é liberada por padrão.
2. **Permitir Entrada do Load Balancer (AllAzureLoadBalanceInbound)**: Permite que os balanceadores de carga do Azure testem a saúde dos seus recursos.
3. **Bloquear Tudo da Internet (Inbound)**: Qualquer tráfego vindo da internet pública para dentro da sua rede é bloqueado por padrão.
4. **Permitir Saída para a Internet (Outbound)**: Seus recursos internos têm permissão para falar com a internet por padrão.

### 💡 Exemplo Prático: Protegendo um Sistema Web + Banco de Dados

Imagine que você tem uma VNet com uma Sub-rede Web (pública) e uma Sub-rede de Banco de Dados (altamente confidencial).

```text
  [ Internet Pública ]
          │
          ▼ (Porta 443 liberada)
   [ Sub-rede Web ] ────► Associada ao NSG-Web
          │
          ▼ (Porta 1433 liberada)
   [ Sub-rede DB ] ──► Associada ao NSG-DB (Bloqueia o resto)

```

Para proteger esse ambiente, você configuraria os NSGs assim:

#### 1. Configuração do NSG-Web (Aplicado na Sub-rede Web)
- **Regra 100 (Entrada)**: Permitir Origem: Internet | Porta: 443 (HTTPS) | Destino: Any. (Permite que os clientes acessem seu site com segurança).
- **Regra 110 (Entrada)**: Permitir Origem: IP-do-Escritório | Porta: 22 (SSH) / 3389 (RDP) | Destino: Any. (Permite que a sua equipe de TI acesse o servidor para manutenção).

#### 2. Configuração do NSG-Banco (Aplicado na Sub-rede de Banco de Dados)
- **Regra 100 (Entrada)**: Permitir Origem: 10.1.1.0/24 (Faixa da Sub-rede Web) | Porta: 1433 (SQL) | Destino: Any. (Apenas os servidores web podem fazer consultas no banco).
- **Regra 200 (Entrada)**: Bloquear Origem: Any | Porta: Any | Destino: Any. (Garante o isolamento total do banco de dados contra qualquer outro tráfego externo ou da internet).

### 🛡️ Boas Práticas ao Usar NSGs

- **Associe no nível da Sub-rede**: Em vez de gerenciar NSGs colando-os em cada placa de rede de cada VM (o que gera caos administrativo), aplique-os na Sub-rede. Assim, qualquer nova VM criada ali já nasce protegida.

- **Deixe espaço entre as prioridades**: Crie suas regras pulando de 100 em 100 (ex: 100, 200, 300). Isso deixa janelas livres para você inserir regras novas no futuro (como uma regra 150) sem precisar refazer toda a numeração.

- **Utilize Application Security Groups (ASGs)**: Os ASGs permitem agrupar VMs sob uma etiqueta de texto (ex: "ServidoresWeb" ou "ServidoresDeBanco"). Em vez de digitar IPs brutos nas regras do NSG, você pode escrever regras limpas como: "Permitir tráfego do ASG-ServidoresWeb para o ASG-ServidoresDeBanco".

---

# 9. Máquinas Virtuais

Recursos normalmente criados:
- NIC (Network Interface Card)
- Disco
- NSG
- IP Público (PIP) (opcional)

---

# 10. Load Balancer

Camada 4:
- TCP
- UDP

---

# 11. Application Gateway

Camada 7:
- HTTPS
- SSL Offload
- Path-Based Routing
- Multi-Domain

---

# 12. Azure Bastion

Acesso seguro via:
- RDP
- SSH

Subnet obrigatória:

```text
AzureBastionSubnet
```

---

# 13. Storage Account

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

# 14. Blob Storage

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

# 15. Azure Files

Protocolo:

```text
SMB (porta 445)
```

---

# 16. SAS vs Access Keys

## Access Key
- Acesso total ao Storage Account

## SAS Token
- Permissões específicas
- Data de expiração
- Escopo limitado

---

# 17. Private Endpoint

Benefícios:
- IP privado
- Mais segurança
- Tráfego interno Azure

---

# 18. DNS

## Público
Resolve nomes da Internet.

## Privado
Resolve nomes internos das VNets.

---

# 19. Backup

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

# 20. Azure Monitor

## Métricas
- CPU
- Memória
- Disco

## Logs
- Eventos
- Alterações
- Auditoria

---

# 21. Network Watcher

Ferramentas importantes:
- Topology
- Connection Troubleshoot
- NSG Diagnostics
- IP Flow Verify
- Packet Capture
- Next Hop

---

# 22. Containers

## Azure Container Instance (ACI)
- Containers sem cluster
- Execução rápida

## Azure Container Registry (ACR)
- Registry privado de imagens Docker

---

# Checklist Final

- ✅ Entra ID
- ✅ RBAC
- ✅ Azure Policy
- ✅ Management Groups
- ✅ VNets
- ✅ Peering
- ✅ NSG
- ✅ Storage Account
- ✅ Blob Storage
- ✅ Azure Files
- ✅ Private Endpoint
- ✅ DNS
- ✅ Backup
- ✅ Azure Monitor
- ✅ Network Watcher
- ✅ ACI
- ✅ ACR
