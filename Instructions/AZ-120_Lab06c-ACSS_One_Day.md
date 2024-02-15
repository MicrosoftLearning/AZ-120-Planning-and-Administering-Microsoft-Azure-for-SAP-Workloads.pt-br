---
lab:
  title: 06c – Visão geral da implantação do Centro do Azure para soluções SAP
  module: Design and implement an infrastructure to support SAP workloads on Azure
---

# Módulo AZ 1006: Projetar e implementar uma infraestrutura para dar suporte a cargas de trabalho SAP no Azure
# Laboratório do curso de um dia do AZ-1006 6c: Visão geral da implantação com o Centro do Azure para soluções SAP

Tempo estimado: 60 minutos

Todas as tarefas neste laboratório do curso de 1 dia do AZ-1006 são realizadas no portal do Azure

## Objetivos

Depois de realizar este laboratório, você será capaz de:

- Implantar a infraestrutura que hospeda cargas de trabalho SAP no Azure usando o Centro do Azure para soluções SAP

>**Importante**: os pré-requisitos implementados neste exercício *não* se destinam a representar as práticas recomendadas para implantar cargas de trabalho SAP no Azure usando o Centro do Azure para soluções SAP. Sua finalidade é minimizar o tempo, o custo e os recursos necessários para avaliar a mecânica de implantação de cargas de trabalho SAP no Azure usando o Centro do Azure para soluções SAP e executando tarefas de gerenciamento e manutenção pós-implantação. A implementação dos pré-requisitos inclui as seguintes atividades:

- Criar uma identidade gerenciada atribuída pelo usuário do Microsoft Entra a ser usada pelo Centro do Azure para soluções SAP para acesso ao Armazenamento do Microsoft Azure durante sua implantação.
- Conceder à identidade gerenciada atribuída pelo usuário do Microsoft Entra, que é usada para realizar a implantação, acesso à assinatura do Azure
- Criar a rede virtual do Azure que hospeda todas as máquinas virtuais do Azure incluídas na implantação.

Essas atividades correspondem às seguintes tarefas deste exercício:

- Tarefa 1: criar uma identidade gerenciada atribuída pelo usuário do Microsoft Entra
- Tarefa 2: configurar atribuições de função controle de acesso baseado em função (RBAC) do Azure para a identidade gerenciada atribuída pelo usuário do Microsoft Entra ID
- Tarefa 3: criar a rede virtual do Azure

## Instruções

### Exercício 1: implementar os pré-requisitos mínimos para avaliar a implantação de cargas de trabalho SAP no Azure usando o Centro do Azure para soluções SAP

Duração: 15 minutos

Nesse exercício, você implementa os pré-requisitos mínimos para avaliar a implantação de cargas de trabalho SAP no Azure usando o Centro do Azure para soluções SAP. Isso inclui as seguintes atividades:

>**Importante**: os pré-requisitos implementados neste exercício *não* se destinam a representar as práticas recomendadas para implantar cargas de trabalho SAP no Azure usando o Centro do Azure para soluções SAP. Sua finalidade é minimizar o tempo, o custo e os recursos necessários para avaliar a mecânica de implantação de cargas de trabalho SAP no Azure usando o Centro do Azure para soluções SAP e executando tarefas de gerenciamento e manutenção pós-implantação.

A implementação dos pré-requisitos inclui as seguintes atividades:

- Criar uma identidade gerenciada atribuída pelo usuário do Microsoft Entra a ser usada pelo Centro do Azure para soluções SAP para acesso ao Azure durante sua implantação.
- Criar a rede virtual do Azure que hospeda todas as máquinas virtuais do Azure incluídas na implantação.

Essas atividades correspondem às seguintes tarefas deste exercício:

- Tarefa 1: criar uma identidade gerenciada atribuída pelo usuário do Microsoft Entra
- Tarefa 2: criar a rede virtual do Azure

#### Tarefa 1: criar uma identidade gerenciada atribuída pelo usuário do Microsoft Entra

Nessa tarefa, crie uma identidade gerenciada atribuída pelo usuário do Microsoft Entra a ser usada pelo Centro do Azure para soluções SAP para acesso ao Armazenamento do Microsoft Azure durante sua implantação.

1. No computador do laboratório, inicie um navegador da Web, navegue até o portal do Azure em `https://portal.azure.com` e autentique-se usando uma conta da Microsoft ou uma conta do Microsoft Entra ID com a função Proprietário na assinatura do Azure que você usa nesse laboratório.
1. Na janela do navegador da Web que exibe o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Identidades Gerenciadas**.
1. Na página **Identidades gerenciadas**, selecione **+ Criar**.
1. Na guia **Noções básicas** da página **Criar Identidade Gerenciada Atribuída ao Usuário**, especifique as seguintes configurações e selecione **Examinar + Criar**:

   |Configuração|Valor|
   |---|---|
   |Assinatura|O nome da assinatura do Azure usada nesse laboratório|
   |Grupo de recursos|O nome de um **novo** grupo de recursos **acss-infra-RG**|
   |Region|o nome da região do Azure que você usa para a implantação do ACSS|
   |Nome|**acss-infra-MI**|

1. Na guia **Revisar**, aguarde a conclusão do processo de validação e selecione **Criar**.
1. Aguarde a conclusão do processo de provisionamento e selecione **Ir para o recurso** para se preparar para a próxima tarefa. O provisionamento deve levar apenas alguns segundos.

#### Tarefa 2: configurar o colaborador de assinatura do controle de acesso baseado em função (RBAC) do Azure

1. Continue no portal do Azure, na página de visão geral da Identidade Gerenciada para **acss-infra-MI**, dando continuidade à última tarefa realizada.
1. No menu da página de Identidade Gerenciada **acss-infra-MI**, escolha as **Atribuições de função do Azure**.
1. No painel **Atribuições de função do Azure**, selecione **+Adicionar atribuição de função**.

1. Na guia **Adicionar atribuição de função**, dentro do painel de **Adicionar atribuição de função**, especifique as seguintes configurações e selecione **Salvar**:

   |Configuração|Valor|
   |---|---|
   |Escopo|**Assinatura**|
   |Assinatura|O nome da assinatura do Azure usada nesse laboratório|
   |Função|**Colaborador**|

#### Tarefa 3: configurar atribuições de função de controle de acesso baseado em função (RBAC) do Azure para a conta de usuário do Microsoft Entra ID que será usada para executar a implantação

##### Adicionar Identidade: "Função de serviço do Centro do Azure para soluções SAP"

1. No portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Assinaturas**.
1. Na página **Assinaturas**, selecione a entrada que representa a assinatura do Azure que você usará para este laboratório. 
1. Na página que exibe as propriedades da assinatura do Azure, selecione **Controle de acesso (IAM)**.
1. Na página **Controle de acesso (IAM),** selecione **+ Adicionar** e, no menu suspenso, selecione **Adicionar atribuição de função**.
1. Na guia **Função** da página **Adicionar atribuição de função**, na listagem **Funções de função de trabalho**, pesquise e selecione a entrada **função de serviço** do Centro do Azure para soluções SAP e selecione **Avançar**.
1. Na guia **Membros** da página **Adicionar atribuição de função**, para** atribuir acesso**, selecione **Identidade Gerenciada** e clique em + **Selecionar membros**.
1. No painel **Selecionar identidades gerenciadas**, especifique as seguintes configurações e depois clique em **Selecionar**:

   |Configuração|Valor|
   |---|---|
   |Assinatura|O nome da assinatura do Azure usada nesse laboratório|
   |Identidade gerenciada|**Identidade gerenciada atribuída pelo usuário**|
   |Selecionar|**acss-infra-RG/subscriptions/...**|

1. De volta à guia **Membros**, selecione **Examinar + atribuir**.
1. Na guia **Revisão + atribuição**, selecione **Examinar + atribuir**.

##### Adicionar Identidade: "Administrador do Centro do Azure para soluções SAP

1. Na página **Controle de acesso (IAM),** selecione **+ Adicionar** e, no menu suspenso, selecione **Adicionar atribuição de função**.
1. Na guia **Função** da página **Adicionar atribuição de função**, na listagem **Funções de função de trabalho**, pesquise e selecione a entrada **Administrador do Centro do Azure para soluções SAP** e selecione **Avançar**.
1. Na guia **Membros**
   - em **Atribuir acesso a**, selecione **Usuário, Grupo ou Entidade de Serviço**
   - clique **em + Selecionar membros**.
1. No painel **Selecionar membros**, na caixa **Selecionar texto**, digite o nome da conta de usuário do Microsoft Entra ID usada para acessar a assinatura do Azure que você está usando para este laboratório, selecione-a na lista de resultados correspondentes à sua entrada e clique em **Selecionar**.
1. De volta à guia **Membros**, selecione **Examinar + atribuir**.
1. Na guia **Revisão + atribuição**, selecione **Examinar + atribuir**.

##### Adicionar Identidade: "Operador de Identidade Gerenciada"

1. Na página **Controle de acesso (IAM),** selecione **+ Adicionar** e, no menu suspenso, selecione **Adicionar atribuição de função**.
1. Na guia **Função** da página **Adicionar atribuição de função**, na listagem **Funções de função de trabalho**, pesquise e selecione a entrada **Operador de Identidade Gerenciada** e selecione **Avançar**.
1. Na guia **Membros**
   - em **Atribuir acesso a**, selecione **Usuário, Grupo ou Entidade de Serviço**
   - clique **em + Selecionar membros**.
1. No painel **Selecionar membros**, na caixa **Selecionar texto**, digite o nome da conta de usuário do Microsoft Entra ID usada para acessar a assinatura do Azure que você está usando para este laboratório, selecione-a na lista de resultados correspondentes à sua entrada e clique em **Selecionar**.
1. De volta à guia **Membros**, selecione **Examinar + atribuir**.
1. Na guia **Revisão + atribuição**, selecione **Examinar + atribuir**.




#### Tarefa 4: criar a rede virtual

Nessa tarefa, você cria a rede virtual do Azure que hospeda todas as máquinas virtuais do Azure incluídas na implantação. Além disso, na rede virtual, você cria as seguintes sub-redes:

- bastion
- aplicativo – destinado a hospedar o aplicativo SAP e servidores de instância do Centro para Serviços SA
- db: destinado à hospedagem da camada de banco de dados SAP

1. No computador do laboratório, na janela do navegador da Web que exibe o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Redes virtuais**.
1. Na página **Redes virtuais**, selecione **+ Criar**.
1. Na guia **Noções básicas** da página **Criar rede virtual**, especifique as seguintes configurações e selecione **Avançar**:

   |Configuração|Valor|
   |---|---|
   |Assinatura|O nome da assinatura do Azure usada nesse laboratório|
   |Grupo de recursos|**acss-infra-RG**|
   |Nome da rede virtual|**acss-infra-VNET**|
   |Region|o nome da mesma região do Azure que você usou na tarefa anterior desse exercício|

1. Na guia **Segurança**, em **Azure Bastion**, marque a **caixa de seleção** para **Habilitar o Azure Bastion**.
1. Especifique as seguintes configurações e selecione **Avançar**.
   |Configuração|Valor|
   |---|---|
   |Nome do host do Azure Bastion|**acss-infra-vnet-bastion**|
   |Endereço IP público do Azure Bastion|**(Novo) acss-infra-bastion**|

1. Na guia **Endereços IP**, especifique as seguintes configurações e selecione **adicionar**:

   |Configuração|Valor|
   |---|---|
   |Espaço de endereços IP|**10.0.0.0/16 (65,536 addresses)**|

1. Na lista de sub-redes, selecione o ícone da lixeira para **excluir** a **sub-rede padrão**.

1. Selecione **+Adicionar uma sub-rede**.
1. No painel **Adicionar uma sub-rede**, especifique as seguintes configurações e selecione **Adicionar** (deixe as demais com seus valores padrão):

   |Configuração|Valor|
   |---|---|
   |Nome|**acss-admin**|
   |Endereço inicial|**10.0.0.0**|
   |Tamanho|**/24 (256 endereços)**|

1. Selecione **+Adicionar uma sub-rede**.
1. No painel **Adicionar uma sub-rede**, especifique as seguintes configurações e selecione **Adicionar** (deixe as demais com seus valores padrão):

   |Configuração|Valor|
   |---|---|
   |Nome|**app**|
   |Endereço inicial|**10.0.2.0**|
   |Tamanho|**/24 (256 endereços)**|

1. Selecione **+Adicionar uma sub-rede**.
1. No painel **Adicionar uma sub-rede**, especifique as seguintes configurações e selecione **Adicionar** (deixe as demais com seus valores padrão):

   |Configuração|Valor|
   |---|---|
   |Nome|**db**|
   |Endereço inicial|**10.0.3.0**|
   |Tamanho|**/24 (256 endereços)**|

1. Na guia **Endereços IP**, selecione **Examinar + criar**:
1. Na guia **Examinar + criar**, aguarde a conclusão do processo de validação e selecione **Criar**.

   >**Observação**: aguarde cerca de 3 minutos para que o processo de provisionamento prossiga parcialmente antes de prosseguir para a próxima tarefa. O provisionamento completo pode levar 25 minutos para o Azure Bastion, portanto, **não vamos esperar**.

### Exercício 2: implantar a infraestrutura que hospedará cargas de trabalho SAP no Azure usando o Centro do Azure para soluções SAP

Duração: 20 minutos

Nesse exercício, você executará a implantação do Centro do Azure para soluções SAP. Isso inclui a atividade a seguir:

- Usar o Centro do Azure para soluções SAP para implantar a infraestrutura capaz de hospedar cargas de trabalho SAP em uma assinatura do Azure.

Essa atividade corresponde à seguinte tarefa desse exercício:

- Tarefa 1: criar Instância Virtual para soluções SAP (VIS)

>**Observação**: após a implantação bem-sucedida, você pode continuar com a instalação de software SAP usando o Centro do Azure para soluções SAP. No entanto, a instalação do software SAP não está incluída nesse laboratório.

>**Observação**: para obter informações sobre como instalar o software SAP usando o Centro do Azure para soluções SAP, confira a documentação do Microsoft Learn que descreve como [Obter mídia de instalação SAP](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/get-sap-installation-media) e [Instalar software SAP](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/install-software).

#### Tarefa 1: criar Instância Virtual para soluções SAP (VIS)

1. No computador do laboratório, na janela do Microsoft Edge que exibe o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione o **Centro do Azure para Soluções SAP**.
1. Na página **Visão geral \| do Centro do Azure para Soluções SAP**, selecione** Criar um novo sistema SAP**.
1. Na guia **Básico** da página **Criar Instância Virtual para soluções SAP**, especifique as seguintes configurações e selecione **Avançar: Máquinas virtuais**

   |Configuração|Valor|
   |---|---|
   |Assinatura|O nome da assinatura do Azure usada nesse laboratório|
   |Grupo de recursos|**acss-infra-RG**|
   |Nome (SID)|**VI1**|
   |Region|o nome da região do Azure que hospeda a implantação SAP registrada pelo ACSS ou outra região na mesma região geográfica|
   |Tipo de ambiente|**Não produção**|
   |Produto SAP|**S/4HANA**|
   |Backup de banco de dados|**HANA**|
   |Método de escala HANA|**Escalar verticalmente (recomendado)**|
   |Tipo de implantação|**Distribuído**|
   |Rede virtual|**acss-infra-VNET**|
   |Sub-rede do aplicativo|**app**|
   |Sub-rede do banco de dados|**db**|
   |Opções de imagem do sistema operacional do aplicativo|**Usar uma imagem do marketplace**|
   |Imagem do sistema operacional do aplicativo|**Red Hat Enterprise Linux 8.2 para aplicativos SAP - x64 Gen2 mais recente**|
   |Opções de imagem do sistema operacional de banco de dados|**Usar uma imagem do marketplace**|
   |Imagem do sistema operacional do banco de dados|**Red Hat Enterprise Linux 8.2 para aplicativos SAP - x64 Gen2 mais recente**|
   |Opção de transporte do SAP|**Criar um novo diretório de transporte SAP**|
   |Grupo de recursos de transporte|**acss-infra-RG**|
   |Nome da conta de armazenamento|*em branco*|
   |Tipo de autenticação|**SSH público**|
   |Nome de Usuário|**contososapadmin**|
   |Origem de chave pública SSH|**Gerar novo par de chaves**|
   |Nome do par de chaves|**contosovi1key**|
   |SQP FQDN|**sap.contoso.com**|
   |Fonte de Identidade gerenciada|**Usar a identidade gerenciada atribuída ao usuário existente**|
   |Nome da identidade gerenciada|**acss-infra-MI**|

1. Na guia **Máquinas virtuais**, especifique as seguintes configurações:

   |Configuração|Valor|
   |---|---|
   |Gerar Recomendação baseada em|**Padrão de Desempenho do Aplicativo SAP (SAPS): selecione essa opção para fornecer o SAPS para a camada de aplicativo e o tamanho da memória do banco de dados e clique em Gerar Recomendações**|
   |SAPS para camada de aplicativo|**1000**|
   |Tamanho da memória do banco de dados (GiB)|**128**|

1. Selecione **Gerar recomendação**.
1. Examine o tamanho e o número de VMs para as máquinas virtuais ASCS, aplicativo e banco de dados.

   >**Observação**: se necessário, ajuste os tamanhos recomendados selecionando o link **Ver todos os Tamanhos** para cada conjunto de máquinas virtuais e escolhendo um tamanho alternativo. Por padrão, o tipo de implantação distribuída com alta disponibilidade, bem como a camada de aplicativo SAPS e o tamanho de memória do banco de dados especificados acima, resultam nas seguintes recomendações mínimas de SKU de VM:
   - 1 x Standard_D4ds_v4 para as VMs ASCS (4 vCPUs e 16 GiB de memória cada)
   - 1 x Standard_D4ds_v4 para as VMs do aplicativo (4 vCPUs e 16 GiB de memória cada)
   - 1 x Standard_E16ds_v5 para as VMs de banco de dados (16 vCPUs e 128 GiB de memória cada)

   >**Observação**: se necessário, você pode solicitar aumento de cota selecionando o link **Cota de Solicitação** para um SKU específico de máquinas virtuais e enviando uma solicitação de aumento de cota. O processamento de uma solicitação normalmente leva alguns minutos.

   >**Observação**: o Centro do Azure para soluções SAP impõem o uso de SKUs de VM com suporte do SAP durante a implantação.

1. Na guia **Máquinas virtuais**, na seção **Discos de dados**, selecione o link **Exibir e personalizar a configuração**.
1. Na página de **Configuração do disco de banco de dados**, examine a configuração recomendada sem fazer alterações e selecione **Fechar**.
1. De volta à guia **Máquinas virtuais**, selecione **Avançar: Visualizar arquitetura**.
1. Na guia **Visualizar Arquitetura**, examine o diagrama que ilustra a arquitetura recomendada e selecione **Examinar + criar**.
1. Na guia **Revisar + criar**, aguarde a conclusão do processo de validação, marque a caixa de seleção para confirmar se você tem uma cota ampla disponível na região de implantação para evitar executar o erro "Cota Insuficiente" e selecione **Criar**.
1. Na janela pop-up **Gerar novo par de chaves**, selecione **Baixar chave privada e criar recurso**.

   >**Observação**: a chave privada necessária para se conectar às VMs do Azure incluídas na implantação será baixada para o computador do qual você está executando este laboratório.

   >**Observação**: aguarde apenas cerca de 3 minutos para que o processo de provisionamento prossiga parcialmente antes de prosseguir para a próxima tarefa. O provisionamento completo pode levar 25 minutos, mas **não vamos esperar**.

   >**Observação**: após a implantação, você pode continuar a instalar o software SAP usando o Centro do Azure para soluções SAP. Nesse laboratório, você irá explorar os recursos do Centro do Azure para soluções SAP sem instalar o software SAP.

### Exercício 3: explorar uma VIS para cargas de trabalho do SAP no Azure usando o Centro do Azure para soluções SAP

Duração: 25 minutos

Neste exercício, você verá as propriedades e funções de uma VIS no Centro do Azure para soluções SAP. Você também se conectará a uma VM criada pelo ACSS e explorará os diretórios criados.

#### Tarefa 1: examinar a página VIS do ACSS

1. Depois que a implantação da VIS terminar, visualize a página da **Instância Virtual para soluções SAP VI1** e explore as informações disponíveis no menu de página Instância Virtual para soluções SAP, incluindo:

    1. **Visão geral**
        - A guia **Introdução** exibe opções para "Instalar software SAP" e "Confirmar software SAP já instalado".
        - A guia **Propriedades** exibe as máquinas virtuais implantadas.
        - A guia **Monitoramento** exibe a "Utilização da CPU da VM do Servidor Central + Aplicativo", "Consumo de IOPS de disco de banco de dados" e "Utilização da CPU da VM do banco de dados".
    1. **Monitorando** > **Quality Insights**  na página Insights de qualidade:
        - **Máquinas Virtuais**: explore Lista de Computação, Extensões de Computação e Disco de Computação+SO.
        - **Verificações de Configuração**: explore as opções de subitem: Rede Acelerada, IP Público, Backup e Load Balancer.
    1. **Gerenciamento de Custos** > **Análise de custos**
        - Expanda itens na coluna ** Recurso**.

#### Tarefa 2: conectar-se à VM de banco de dados e revisar a configuração do ACSS

1. No Portal do Azure, selecione as Máquinas Virtuais e selecione a VM de banco de dados criada no ACSS, **vi1dbvm**.
1. Selecione Conectar > Azure Bastion e escolha as seguintes configurações e selecione **conectar**:
    - Tipo de Autenticação **Chave Privada SSH de Arquivo Local**
    - Nome de Usuário **contososapadmin** 
    - Arquivo Local ***chave privada que você baixou***
    
1. No prompt do Bash, insira: `mount` e localize a saída da seguinte maneira para o mapeamento:

```output
/dev/sdb1 on /mnt type ext4 (rw,relatime,x-systemd.requires=cloud-init.service,_netdev)
/dev/mapper/vg_sap-lv_usrsap on /usr/sap type xfs (rw,relatime,attr2,inode64,noquota)
/dev/mapper/vg_hana_shared-lv_hana_shared on /hana/shared type xfs (rw,relatime,attr2,inode64,noquota)
/dev/mapper/vg_hana_backup-lv_hana_backup on /hana/backup type xfs (rw,relatime,attr2,inode64,noquota)
/dev/mapper/vg_hana_data-lv_hana_data on /hana/data type xfs (rw,relatime,attr2,inode64,sunit=512,swidth=2048,noquota)
/dev/mapper/vg_hana_log-lv_hana_log on /hana/log type xfs (rw,relatime,attr2,inode64,sunit=128,swidth=384,noquota)
```

1. No prompt, insira: `more fstab` e encontre uma saída parecida com a seguinte, que corresponde ao mapeamento do compartilhamento SAP Media (`sapmedia`):

```output
10.100.1.8:/vi2nfs9fbec656c6a60a7/sapmedia on /usr/sap/install type nfs4 (rw,relatime,vers=4.1,rsize=65536,wsize=65536,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.100.1.6,local_
lock=none,addr=10.100.1.8)
```

1. No prompt, insira: `cd /usr/sap/install` para navegar até o diretório que mapeia para `sapmedia`.

1. Retorne à página da VIS do ACSS e examine os seguintes itens:
1. Interface do usuário do Azure: Link privado > interface do usuário do Azure de pontos de extremidade privados: Ver a instalação do software ACSS na interface do usuário

### Exercício 4 (opcional): manter cargas de trabalho SAP no Azure usando o Centro do Azure para soluções SAP

Duração: 20 minutos

Nesse exercício, você examinará o gerenciamento pós-implantação e o monitoramento de cargas de trabalho SAP usando o Centro do Azure para soluções SAP. Isso inclui as seguintes atividades:

- Revisar os pré-requisitos para backup de cargas de trabalho SAP gerenciadas pelo Centro do Azure para soluções SAP
- Revisar os pré-requisitos para recuperação de desastre de cargas de trabalho SAP gerenciadas pelo Centro do Azure para soluções SAP
- Revisar as opções de monitoramento disponíveis para cargas de trabalho SAP gerenciadas pelo Centro do Azure para soluções SAP
- Excluir todos os recursos do Azure provisionados neste laboratório.

Essas atividades correspondem às seguintes tarefas deste exercício:

- Tarefa 1: Revisar os pré-requisitos para backup de cargas de trabalho SAP gerenciadas pelo Centro do Azure para soluções SAP
- Tarefa 2: Revisar os pré-requisitos para recuperação de desastre de cargas de trabalho SAP gerenciadas pelo Centro do Azure para soluções SAP
- Tarefa 3: examinar as opções de monitoramento para cargas de trabalho SAP gerenciadas pelo Centro do Azure para soluções SAP
- Tarefa 4: excluir os recursos do Azure provisionados nesse laboratório

#### Tarefa 1: Revisar os pré-requisitos para backup de cargas de trabalho SAP gerenciadas pelo Centro do Azure para soluções SAP

>**Observação**: ao configurar o Backup do Azure no nível de recurso da VIS no Centro do Azure para soluções SAP, você pode, em uma etapa, habilitar o backup para VMs do Azure que hospedam o banco de dados, os servidores de aplicativos e a instância do Serviços do Centro do SAP e para o BD HANA. Para o backup do BD HANA, o Centro do Azure para soluções SAP executa automaticamente o script de pré-registro de backup.

1. No computador do laboratório, na janela do Microsoft Edge que exibe o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione o **Centro do Azure para Soluções SAP**.
1. Na página **Centro do Azure para soluções SAP \| Visão Geral**, no menu de navegação vertical no lado esquerdo, selecione **Instâncias virtuais para soluções SAP** e, na lista de instâncias virtuais, selecione a instância que você implantou no exercício anterior.
1. Na página da instância virtual, no menu de navegação vertical à esquerda, na seção **Operações**, selecione **Backup (versão prévia)**.
1. Observe a mensagem indicando que o backup não pode ser configurado, pois a instalação/registro de software SAP para esse sistema SAP não está concluída.

   >**Observação**: isso é esperado. Você não poderá configurar o backup dessa forma até que a instalação do software SAP seja concluída. No entanto, concluir a instalação também envolve pré-requisitos adicionais, incluindo a criação de cofres e políticas de backup, que examinaremos aqui. 

1. No computador do laboratório, na janela do Microsoft Edge que exibe o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione o **Centro de backup**. 
1. Na página **Centro de backup**, no menu de navegação vertical no lado esquerdo, na seção **Gerenciar**, selecione **Cofres**.
1. Na página **Centro de backup \| Cofres**, selecione **+ Cofre**.
1. Na página **Início: Criar Cofre**, examine os tipos de cofre disponíveis, verifique se o **Cofre de serviços de recuperação** (que dá suporte aos tipos de fonte de dados das **Máquinas virtuais do Azure** e **SAP HANA na VM do Azure**) está selecionado e selecione **Continuar**.
1. Na guia **Noções básicas** da página **Criar cofre dos Serviços de Recuperação**, especifique as seguintes configurações e selecione **Avançar: Redundância**

   |Configuração|Valor|
   |---|---|
   |Assinatura|O nome da assinatura do Azure usada nesse laboratório|
   |Grupo de recursos|O nome de um **novo** grupo de recursos **acss-mgmt-RG**|
   |Nome do cofre|**acss-backup-RSV**|
   |Region|o nome da região do Azure que hospeda a implantação SAP registrada pelo ACSS|

1. Na guia **Redundância**, especifique as seguintes configurações e selecione **Avançar: Propriedades do Cofre**

   |Configuração|Valor|
   |---|---|
   |Redundância do armazenamento de backup|**Com redundância geográfica**|
   |Restauração Entre Regiões|**Habilitar**|

1. Na guia **Propriedades do Cofre**, examine a configuração **Habilitar imutabilidade** sem habilitá-la e selecione **Avançar: Rede**
1. Na guia **Rede**, aceite a opção padrão para **Permitir o acesso público de todas as redes** e selecione **Examinar + criar**
1. Neste exercício, **não** selecione **Criar**, pois estamos apenas revisando.
1. Na guia **Revisar + criar**, aguarde a conclusão do processo de validação e retorne à página Centro do Azure para soluções SAP (as configurações de backup serão perdidas).  

   >**Observação**: o recurso [Backup (versão prévia)](https://learn.microsoft.com/azure/sap/center-sap-solutions/acss-backup-integration) da interface do usuário do Centro do Azure para soluções SAP se tornará um método preferencial para concluir a configuração de backup depois que ele for liberado para *Disponibilidade Geral*, saindo do estágio de *Versão Prévia*.

   >**Observação**: ao configurar o backup no nível da VIS na interface do Centro do Azure para soluções SAP, você poderá aproveitar o cofre existente e suas políticas.

   >**Observação**: depois que o backup for configurado no nível da VIS, você poderá monitorar o status dos trabalhos de backup das VMs do Azure e do BD HANA da interface da VIS no portal do Azure.

#### Tarefa 2: Revisar os pré-requisitos para recuperação de desastre de cargas de trabalho SAP gerenciadas pelo Centro do Azure para soluções SAP

>**Observação**: embora o serviço do Centro do Azure para soluções SAP possua redundância de zona, não há failover iniciado pela Microsoft no caso de uma interrupção de região. Para corrigir esse cenário, você deve configurar a recuperação de desastre para sistemas SAP implantados usando o Centro do Azure para soluções SAP seguindo as diretrizes descritas na [Visão geral de recuperação de desastres e nas diretrizes de infraestrutura para a carga de trabalho SAP](https://learn.microsoft.com/en-us/azure/sap/workloads/disaster-recovery-overview-guide), que envolve o uso do Azure Site Recovery (ASR). Nessa tarefa, você percorrerá o processo de implementação de uma solução de recuperação de desastre baseada em ASR que depende dessas diretrizes.

>**Observação**: o ASR é a solução recomendada para servidores de aplicativos e instâncias de serviço do Centro do SAP. Para servidores de banco de dados, você deve considerar usar a funcionalidade de replicação nativa deles.

1. No computador do laboratório, na janela do Microsoft Edge que exibe o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Cofres dos Serviços de Recuperação**.
1. Na página **Cofres dos Serviços de Recuperação**, selecione **+ Criar**.
1. Na guia **Noções básicas** da página **Criar cofre dos Serviços de Recuperação**, especifique as seguintes configurações (deixe as demais com seus valores padrão) e selecione **Avançar: Redundância**.

   |Configuração|Valor|
   |---|---|
   |Assinatura|O nome da assinatura do Azure usada nesse laboratório|
   |Grupo de recursos|O nome de um **novo** grupo de recursos **acss-dr-RG**|
   |Nome do cofre|**acss-dr-RSV**|
   |Region|o nome da região do Azure emparelhada com a implantação SAP registrada no ACSS|

   >**Observação**: para identificar a região que é emparelhada com aquela que hospeda suas cargas de trabalho de produção, consulte a documentação do MS Learn que descreve [Regiões emparelhadas do Azure](https://learn.microsoft.com/en-us/azure/reliability/cross-region-replication-azure#azure-paired-regions).

1. Na guia **Redundância**, especifique as seguintes configurações e selecione **Avançar: Propriedades do Cofre**

   |Configuração|Valor|
   |---|---|
   |Redundância do armazenamento de backup|**Localmente redundante**|

1. Na guia **Propriedades do Cofre**, examine a configuração **Habilitar imutabilidade** sem habilitá-la e selecione **Avançar: Rede**
1. Na guia **Rede**, aceite a opção padrão para **Permitir o acesso público de todas as redes** e selecione **Examinar + criar**
1. Na guia **Revisar + criar**, aguarde a conclusão do processo de validação e selecione **Criar**.

   >**Observação**: não aguarde a conclusão do processo de provisionamento, mas prossiga para a próxima etapa. O provisionamento pode levar cerca de 2 minutos.

   >**Observação**: agora você configurará o ambiente de recuperação de desastre na região emparelhada na qual você criou o Cofre dos Serviços de Recuperação. Esse ambiente irá incluir uma rede virtual que hospedará réplicas das VMs do Azure atualmente hospedadas na região primária em que você provisionou a Instância Virtual para SAP. 

1. No computador do laboratório, na janela do Microsoft Edge que exibe o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Cofres dos Serviços de Recuperação**.
1. Na página **Cofres dos Serviços de Recuperação**, selecione **acss-dr-RSV**.
1. Na página **acss-dr-RSV**, no menu de navegação vertical no lado esquerdo, na seção **Introdução**, selecione **Site Recovery**.
1. Na página **acss-dr-RSV \| Site Recovery**, na seção **Máquinas virtuais do Azure**, selecione **1. Habilitar a replicação**. 
1. Na guia **Origem** da página **Habilitar replicação**, especifique as seguintes configurações e selecione **Avançar**:

   |Configuração|Valor|
   |---|---|
   |Região|o nome da região do Azure que hospeda sua Instância Virtual para SAP (VIS)|
   |Assinatura|O nome da assinatura do Azure usada nesse laboratório|
   |Grupo de recursos|**acss-vi-RG**|
   |Modelo de implantação da máquina virtual|**Resource Manager**|
   |Recuperação de desastre entre as zonas de disponibilidade|**Não**|

   >**Observação**: a **Recuperação de desastre entre zonas de disponibilidade** pode não ser configurável, dependendo se a região de origem dá suporte a zonas de disponibilidade.

1. Na guia **Máquinas virtuais**, selecione as quatro primeiras máquinas virtuais na lista (**vi1appvm0**, **vi1appvm1**, **vi1ascsvm0** e **vi1ascsvm0**) e selecione **Avançar**.

   >**Observação**: conforme mencionado anteriormente, a replicação baseada em ASR será aplicada aos servidores de aplicativos e às instâncias dos Serviços Centrais do SAP. Os servidores de banco de dados permanecerão sincronizados contando com a funcionalidade de replicação de banco de dados nativo.

1. Na guia **Configurações de Replicação**, execute as seguintes ações:

   1. Se necessário, na lista de seleção **Local de destino**, selecione a região do Azure na qual você criou o cofre dos Serviços de Recuperação **acss-dr-RSV**.
   1. Verifique se o nome da assinatura do Azure usada neste laboratório aparece na lista seleção de **Assinatura de destino**.
   1. Na lista de seleção **Grupo de recursos de destino**, selecione **acss-dr-RG**.
   1. Abaixo da lista de seleção **Rede virtual de failover**, selecione **Criar novo**.
   1. No painel **Criar rede virtual**, na caixa de texto **Nome**, digite **CONTOSO-VNET-asr**
   1. Na seção **Espaço de endereço**, na caixa de texto **Intervalo de endereços**, substitua a entrada padrão por **10.10.0.0/16**.
   1. Na seção **Sub-redes**, na caixa de texto **Nome da Sub-rede**, digite **aplicativo** e, na caixa de texto **Intervalo de endereços**, digite **10.10.0.0/24**.
   1. Abaixo da entrada de sub-rede recém-adicionada, na seção **Sub-redes**, na caixa de texto **Nome da sub-rede**, insira **db** e, na caixa de texto **Intervalo de endereços**, digite **10.10.2.0/24**.
   1. No painel **Criar rede virtual**, selecione **OK**.
   1. De volta à página **Habilitar replicação**, verifique se a entrada **(novo) aplicativo (10.10.0.0/24)** aparece na lista de seleção **Sub-rede de failover**.
   1. Na seção **Armazenamento**, selecione o link **Exibir/editar configuração de armazenamento**.
   1. Na página **Personalizar configurações de destino**, examine a configuração resultante, mas não faça alterações e selecione **Cancelar**.
   1. Na seção **Opções de disponibilidade**, selecione o link **Exibir/editar opções de disponibilidade**.
   1. Na página **Opções de disponibilidade**, você tem a opção de implementar grupos de posicionamento por proximidade para recursos de destino, mas não faça nenhuma alteração e selecione **Cancelar**.

   >**Observação**: você também tem a opção de configurar a reserva de capacidade.

   >**Observação**: você também tem a opção de definir as configurações da política de replicação.

   >**Importante**: observe que os espaços de endereço IP diferem entre a rede virtual nas regiões primária e secundária. Isso é intencional, pois permitirá conectar as duas redes virtuais juntas, o que é necessário para configurar a replicação entre servidores de banco de dados hospedados nas duas regiões. Essa conexão pode ser estabelecida usando o emparelhamento de rede virtual.

1. Na guia **Configurações de replicação** da página **Habilitar replicação**, selecione **Avançar**.
1. Na guia **Gerenciar** da página **Habilitar replicação**, selecione **Avançar**.
1. Na guia **Revisão** da página **Habilitar replicação**, examine as configurações. Neste exercício, **não habilitaremos** a replicação.

   >**Observação**: a replicação inicial geralmente leva um tempo considerável para ser concluída. Considerando o tempo limitado alocado para este laboratório, consulte o instrutor para obter orientações sobre as etapas extras a serem realizadas como parte dessa tarefa. Na ausência de orientações específicas, prossiga diretamente para a próxima tarefa.

   >**Observação**: neste momento, você pode provisionar o Azure Bastion e o Firewall do Azure, no entanto, você deve automatizar o provisionamento como parte do procedimento de failover de recuperação de desastre. Isso minimizará os encargos associados à manutenção do ambiente de recuperação de desastre. O mesmo deve se aplicar a outros componentes desse ambiente que espelham a configuração da Instância Virtual primária para SAP, como compartilhamentos de arquivos Premium do Azure e roteamento personalizado.

#### Tarefa 3: examinar as opções de monitoramento para cargas de trabalho SAP gerenciadas pelo Centro do Azure para soluções SAP

>**Observação**: assim como acontece com o backup, você não poderá experimentar totalmente os recursos de monitoramento do Centro do Azure para soluções SAP. Isso requer a instalação do software SAP ou o registro de uma instância já existente do Azure Monitor para soluções SAP. Em vez disso, nessa tarefa, você percorrerá a interface disponível na Instância Virtual para soluções SAP para identificar e examinar esses recursos.

1. No computador do laboratório, na janela do Microsoft Edge que exibe o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Instâncias Virtuais para soluções SAP**. 
1. Na página **Instâncias Virtuais para soluções SAP**, examine as informações de status resumidas para a instância **VI1**, incluindo os indicadores visuais de integridade e status gerais.
1. Na página **Instâncias Virtuais para soluções SAP**, selecione **VI1**.
1. Na página **VI1**, no menu de navegação vertical no lado esquerdo, selecione **Visão geral** e, no painel à direita, selecione **Monitoramento**.
1. Examine a telemetria de monitoramento exibida no painel de monitoramento.

   >**Observação**: o painel de monitoramento inclui grafos de utilização de vCPU e métricas para servidores de aplicativos, servidores de banco de dados e instâncias dos Serviços Centrais do SAP. Ele também inclui servidores de banco de dados de estatística de IOPS de disco de banco de dados. 

1. Na página **VI1**, no menu de navegação vertical à esquerda, na seção **Monitoramento**, selecione **Insights de qualidade**.
1. Na página **VI1 \| Insights de qualidade \| Pasta de trabalho 1**, analise a guia **Recomendação do Assistente**, que se destina a fornecer recomendações para otimizar a Instância Virtual para soluções SAP (VIS), instância central do servidor, instâncias do serviço de aplicativo e bancos de dados.

   >**Observação**: essas recomendações exigem a conclusão da instalação do software SAP.

1. Na página **VI1 \| Insights de qualidade \| Pasta de trabalho 1**, selecione a guia **Máquina Virtual** e examine o conteúdo das guias **Computação do Azure**, **Lista de Computação**, ** Extensões de Computação**, **Computação + Disco do SO** e **Computação + Discos de Dados**.

   >**Observação**: cada uma dessas guias deve incluir dados reais coletados das VMs do Azure que fazem parte da Instância Virtual para soluções SAP.

1. Na página **VI1 \| Insights de qualidade \| Pasta de trabalho 1**, selecione a guia **Verificações de Configuração** e examine o conteúdo das guias **Rede Acelerada**, **IP Público**, **Backup** e **Load Balancer**. Esse conteúdo fornece uma visão geral rápida das configurações relacionadas ao desempenho e à segurança dos componentes de computação e rede da Instância Virtual para soluções SAP. A guia **Load Balancer** inclui informações do **Monitor do Load Balancer** que exibem métricas do balanceador de carga de chave.
1. Na página **VI1**, no menu de navegação vertical à esquerda, na seção **Monitoramento**, selecione **Azure Monitor para soluções SAP**.
1. Na página **VI1 \| Azure Monitor para soluções SAP**, observe a mensagem informando que os Serviços de Mídia do Azure (AMS) não podem ser configurados, pois a instalação do software SAP\registration para VIS não está concluída.

   >**Observação**: depois de instalar o software SAP, você poderá integrá-lo a um recurso novo ou existente do Azure Monitor para soluções SAP. O Azure Monitor para soluções SAP depende dos recursos do Azure Monitor do Log Analytics e das pastas de trabalho para fornecer um monitoramento abrangente das cargas de trabalho SAP hospedadas em VMs do Azure, incluindo suporte para visualizações personalizadas, consultas e alertas.
