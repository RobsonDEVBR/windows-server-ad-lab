Este repositório documenta a implementação de um ambiente de rede corporativa utilizando **Windows Server 2025**, focado na gestão centralizada de identidades e segurança.

##  Objetivo do Projeto
Estabelecer uma fundação sólida de diretório (Active Directory) que permita escalabilidade, automação de políticas e controle rígido de acessos (RBAC - Role-Based Access Control).

##  Implementação Técnica

### 1. Estruturação de Unidades Organizacionais (OUs)
Implementei uma hierarquia de OUs personalizada para fugir dos containers padrão do Windows, permitindo a aplicação futura de **GPOs (Group Policy Objects)** de forma granular.

* **Matriz-Caxambu (Raiz)**: Unidade principal da organização.
* **ADM / TI**: Segmentação por departamentos para aplicação de políticas distintas.
* **COMPUTADORES**: Destinado ao gerenciamento de ativos de hardware e Hardening de estações de trabalho.

> **Evidência:** ![Estrutura de OUs](img/Arvore1.png)
> **Evidência:** ![Estrutura de OUs](img/Arvore.png)

### 2. Gestão de Grupos e Segurança (Metodologia AGDLP)
Adotei a estratégia de **Grupos de Segurança Globais** para organizar usuários por função, preparando o ambiente para o padrão AGDLP.

* **Grupo Criado**: `G_TI_AcessoFull` (Escopo: Global).
* **Ação**: O usuário `robson.silva` foi movido para a OU correspondente e vinculado ao grupo de acesso, garantindo que suas permissões sejam gerenciadas via grupo e não de forma individual.

> **Evidência:** ![Membros do Grupo](img/MembroDe.png)
> **Evidência:** ![Membros do Grupo](img/MembroGTI.png)


## 🚀 Próximos Passos
- [ ] Implementação de GPOs de segurança (Bloqueio de USB e Painel de Controle).
- [ ] Configuração de Servidor de Arquivos (File Server) com permissões NTFS.
- [ ] Automação de criação de usuários via PowerShell.