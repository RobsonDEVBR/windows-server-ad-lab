## 🛠️ Tecnologias e Ambiente

![Windows Server](https://img.shields.io/badge/Windows_Server_2025-0078D4?style=for-the-badge&logo=windows&logoColor=white)
![Active Directory](https://img.shields.io/badge/Active_Directory-003366?style=for-the-badge&logo=microsoft&logoColor=white)
![Hyper-V](https://img.shields.io/badge/Hyper--V-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)
![PowerShell](https://img.shields.io/badge/PowerShell-5391FE?style=for-the-badge&logo=powershell&logoColor=white)

| Componente | Detalhe |
|---|---|
| Hipervisor | Hyper-V (Windows 10 — host) |
| Servidor | Windows Server 2025 (Domain Controller) |
| Cliente | Windows 10 (estação ingressada no domínio) |
| Serviços | AD DS, DNS, GPO, File Server (NTFS) |
| Automação | PowerShell |
##  Objetivo do Projeto
Lab montado para praticar a estruturação de um AD do zero — OUs, grupos de segurança, GPOs e File Server — simulando o ambiente de uma empresa com sede em Caxambu. Toda decisão técnica está documentada com o raciocínio por trás dela.

##  Implementação Técnica

### 1. Estruturação de Unidades Organizacionais (OUs)
Implementei uma hierarquia de OUs personalizada para fugir dos containers padrão do Windows, permitindo a aplicação futura de **GPOs (Group Policy Objects)** de forma granular.

* **Matriz-Caxambu (Raiz)**: Unidade principal da organização.
* **ADM / TI**: Segmentação por departamentos para aplicação de políticas distintas.
* **COMPUTADORES**: Destinado ao gerenciamento de ativos de hardware e Hardening de estações de trabalho.

>**Evidência:** ![Estrutura de OUs](img/Arvore1.png)
> **Evidência:** ![Estrutura de OUs](img/Arvore.png)

### 2. Gestão de Grupos e Segurança (Metodologia AGDLP)
Adotei a estratégia de **Grupos de Segurança Globais** para organizar usuários por função, preparando o ambiente para o padrão AGDLP.

* **Grupo Criado**: `G_TI_AcessoFull` (Escopo: Global).
* **Ação**: O usuário `robson.silva` foi movido para a OU correspondente e vinculado ao grupo de acesso, garantindo que suas permissões sejam gerenciadas via grupo e não de forma individual.

> **Evidência:** ![Membros do Grupo](img/MembroDe.png)
> **Evidência:** ![Membros do Grupo](img/MembroGTI.png)

###  Implementação de Políticas de Grupo (GPO) - Hardening
Para elevar o nível de segurança e padronização das estações de trabalho, implementei a primeira política de restrição no domínio.

* **Política:** Bloqueio de acesso ao Painel de Controle e Configurações do Sistema.
* **Escopo:** Aplicada especificamente à OU `TI` para fins de homologação.
* **Objetivo:** Impedir que usuários finais realizem alterações críticas no sistema operacional, reduzindo o número de chamados por desconfiguração e aumentando a integridade do ambiente.
* **Validação:** Após a execução do comando `gpupdate /force` na estação cliente, a restrição foi aplicada com sucesso, bloqueando tentativas de acesso administrativo por parte do usuário comum.

> **Evidência:** ![Criando Políticas](img/CriandoPoliticaTI.png)
> **Evidência:** ![Proibindo Acesso ao Painel e Configurações](img/ProibindoAcessoPainelEConf.png)
> **Evidência:** ![Abrindo Painel](img/AbrindoPainel.png)
> **Evidência:** ![Painel Bloqueado](img/PainelEConfBloq.png)

###  Bloqueio de Dispositivos Removíveis (USB)
Para mitigar riscos de vazamento de dados (DLP) e infecções por malware, apliquei uma política de nível de máquina para o bloqueio de armazenamento externo.

* **Política:** *All Removable Storage classes: Deny all access* (Bloqueio total de portas USB para armazenamento).
* **Escopo:** Aplicada à OU `COMPUTADORES`. O objeto *Computer* da estação Windows 10 foi movido do container padrão para a OU correspondente para receber a diretiva.
* **Objetivo:** Garantir que a restrição de hardware seja aplicada fisicamente à estação de trabalho, independentemente do nível de privilégio do usuário que realizar o logon.

> **Evidência:** ![Máquina na OU de Computadores](img/USB1.png)
> **Evidência:** ![Regra de USB Ativa no Servidor](img/USB2.png)
> **Evidência:** ![Bloqueio ao Tentar Acessar Unidade CD/DVD](img/USB3.png)

###  Servidor de Arquivos (File Server) e Segurança NTFS
Para garantir a confidencialidade e integridade dos dados da empresa, implementei um Servidor de Arquivos utilizando a metodologia de acesso de menor privilégio.

* **Arquitetura de Permissões:** Utilização do modelo de "Duas Portas". Acesso amplo na camada de Compartilhamento (Share) e restrição granular na camada de Segurança (NTFS).
* **Quebra de Herança:** A herança de diretórios foi desabilitada para garantir que pastas departamentais tenham controle de acesso exclusivo.
* **Aplicação do AGDLP na Prática:** O acesso não é dado ao usuário, mas sim ao grupo de segurança. A pasta `TI_Confidencial` foi restrita para aceitar modificações apenas do grupo global `G_TI_AcessoFull`.

> **Evidência:** ![Criando C:\Arquivo_Matriz\TI_Confidencial](img/NTFS1.png)
* **Criando C:\Arquivo_Matriz\TI_Confidencial**
> **Evidência:** ![Adicionando Grupo G_TI_AcessoFull](img/NTFS2.png)
* **Adicionando Grupo G_TI_AcessoFull**
> **Evidência:** ![Adicionando Permissão Modify permitindo trabalhar dentro da pasta mas sem controle total"Full Control"Para que o Usuário não controle as senhas e segurança](img/NTFS3.png)
* **Adicionando Permissão Modify permitindo trabalhar dentro da pasta mas sem controle total"Full Control"Para que o Usuário não controle as senhas e segurança**
> **Evidência:** ![Grupo Adicionado Usuários Removidos e Herança desabilitada](img/NTFS4.png)
* **Grupo Adicionado Usuários Removidos e Herança desabilitada**
> **Evidência:** ![Acesso Mapeado Windows 10 na rede](img/NTFS5.png)
* **Acesso Mapeado Windows 10 na rede**

###  Automação de Identidades via PowerShell (Scripting)
Criar usuário por usuário na interface gráfica não escala. Escrevi um script que lê um CSV simulando uma exportação do RH e cria as contas automaticamente nas OUs certas.

 **[Clique aqui para ver o script completo e a planilha CSV base na pasta /Scripts](./Scripts/)**

* **A Lógica de Negócio:** O script lê o CSV linha por linha, verifica o campo Departamento e joga o usuário na OU correspondente (ADM ou TI). Se o departamento não bater com nenhuma OU cadastrada, o script pula e registra o erro.
* **Segurança por Padrão:** As senhas temporárias são geradas via *SecureString* e a política de "Exigir alteração de senha no primeiro logon" é forçada via código para todas as novas identidades.
> **Evidência 1 - Criação Arquivo csv pasta Script:** > ![Script PowerShell Executando](img/PS1.png)
> **Evidência 2 - Preenchendo Dados:** > ![Script PowerShell Executando](img/PS2.png)
> **Evidência 3 - Utilizando Script PowerShell via CMD :** > ![Usuários Criados nas OUs](img/PS3.png)
> **Evidência 4 - Resultado no Active Directory Pasta ADM:** > ![Usuários Criados nas OUs](img/PS4.png)
> **Evidência 5 - Resultado no Active Directory Pasta TI:** > ![Usuários Criados nas OUs](img/PS5.png)
##  Próximos Passos (Mês 1 — Em andamento)
- [x] Implementação de GPOs de segurança (Bloqueio de USB e Painel de Controle).
- [x] Configuração de Servidor de Arquivos (File Server) com restrição NTFS.
- [x] Automação de criação de usuários via PowerShell.
- [ ] Implementação de Política de Senhas e Bloqueio de Conta via GPO.
- [ ] Configuração de Auditoria de Eventos de Logon (Logs de Segurança).
- [ ] Liberação de Acesso Remoto (RDP) restrito via GPO para o Grupo de TI.