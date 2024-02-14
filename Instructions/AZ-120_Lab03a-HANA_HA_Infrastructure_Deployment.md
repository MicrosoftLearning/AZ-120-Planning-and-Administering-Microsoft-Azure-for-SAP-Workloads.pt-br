---
lab:
  title: 04a – Implementar a arquitetura SAP em VMs do Azure que executam o Linux
  module: Module 04 - Deploy SAP on Azure
---

# Módulo 4 do AZ 120: Implantar SAP no Azure
# Laboratório 4a: Implementa a arquitetura SAP em VMs do Azure que executam o Linux

Tempo estimado: 100 minutos

Todas as tarefas neste laboratório são executadas do portal do Azure (incluindo a sessão do Cloud Shell Bash)  

   > **Observação**: Quando não estiver usando o Cloud Shell, a máquina virtual do laboratório deverá ter a CLI do Azure instalada [**https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows**](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows).

Arquivos de laboratório: nenhum

## Cenário
  
Em preparação para a implantação do SAP NetWeaver no Azure, a Adatum Corporation deseja implementar uma demonstração que ilustrará a implementação altamente disponível do SAP NetWeaver em VMs do Azure executando a distribuição SUSE do Linux.

## Objetivos
  
Depois de realizar este laboratório, você será capaz de:

-   Provisionar recursos do Azure necessários para dar suporte a uma implantação do SAP NetWeaver altamente disponível

-   Configure o sistema operacional de VMs do Azure que executam o Linux para dar suporte a uma implantação de SAP NetWeaver altamente disponível

-   Configure o clustering em VMs do Azure que executam o Linux para dar suporte a uma implantação do SAP NetWeaver altamente disponível

## Requisitos

-   Uma assinatura do Microsoft Azure com o número suficiente de vCPUs Dsv3 disponíveis (quatro VMs Standard_D2s_v3 com 2 vCPUS cada e duas VMs Standard_D4s_v3 com 4 vCPUs cada) em uma região do Azure que oferece suporte a zonas de disponibilidade

-   Um computador de laboratório com um navegador da Web compatível com o Azure Cloud Shell e acesso ao Azure


## Exercício 1: provisione os recursos do Azure necessários para dar suporte a implantações SAP NetWeaver altamente disponíveis

Duração: 30 minutos

Neste exercício, você implantará os componentes de computação de infraestrutura do Azure necessários para configurar o clustering do Linux. Isso envolverá a criação de um par de VMs do Azure que executam o Linux SUSE no mesmo conjunto de disponibilidade.

### Tarefa 1: Crie uma rede virtual que hospedará uma implantação SAP NetWeaver altamente disponível.

1. No computador do laboratório, inicie um navegador da Web e navegue até o portal do Azure em **https://portal.azure.com**.

1. Se solicitado, entre com a conta corporativa, de estudante ou pessoal da Microsoft com a função de proprietário ou colaborador na assinatura do Azure a qual você usará para este laboratório e a função de Administrador Global no locatário do Azure AD associado à sua assinatura.

1. No portal do Azure, inicie uma sessão do Bash no Cloud Shell. 

    > **Observação**: Se esta for a primeira vez que você está iniciando o Cloud Shell na assinatura atual do Azure, você será solicitado a criar um compartilhamento de arquivos do Azure para persistir arquivos do Cloud Shell. Nesse caso, aceite os padrões, o que resultará na criação de uma conta de armazenamento em um grupo de recursos gerado automaticamente.

1. No painel Cloud Shell, execute o seguinte comando para especificar a região do Azure que dá suporte a zonas de disponibilidade e onde você deseja criar recursos para este laboratório (substitua `<region>` pelo nome da região do Azure que oferece suporte a zonas de disponibilidade):

    ```cli
    LOCATION='<region>'
    ```

    > **Observação**: Considere o uso das regiões **Leste dos EUA** ou **Leste dos EUA 2** para a implantação de seus recursos. 

    > **Observação**: Certifique-se de usar a notação adequada para a região do Azure (nome curto que não inclui espaço, por exemplo, **eastus** em vez de **Leste dos EUA**)

    > **Observação**: Para identificar as regiões do Azure que dão suporte a zonas de disponibilidade, consulte [https://docs.microsoft.com/en-us/azure/availability-zones/az-region](https://docs.microsoft.com/en-us/azure/availability-zones/az-region)

1. No painel do Cloud Shell, execute o seguinte comando para definir o valor da variável `RESOURCE_GROUP_NAME` como o nome do grupo de recursos que contém os recursos provisionados na tarefa anterior:

    ```cli
    RESOURCE_GROUP_NAME='az12003a-sap-RG'
    ```

1. No painel do Cloud Shell, execute o seguinte comando para criar um grupo de recursos na região especificada:

    ```cli
    az group create --resource-group $RESOURCE_GROUP_NAME --location $LOCATION
    ```

1. No painel do Cloud Shell, execute o seguinte comando para criar uma rede virtual com uma única sub-rede no grupo de recursos que você criou:

    ```cli
    VNET_NAME='az12003a-sap-vnet'

    VNET_PREFIX='10.3.0.0/16'

    SUBNET_NAME='sapSubnet'

    SUBNET_PREFIX='10.3.0.0/24'

    az network vnet create --resource-group $RESOURCE_GROUP_NAME --location $LOCATION --name $VNET_NAME --address-prefixes $VNET_PREFIX --subnet-name $SUBNET_NAME --subnet-prefixes $SUBNET_PREFIX
    ```

1.  No painel do Cloud Shell, execute o seguinte comando para identificar a ID do recurso da sub-rede da rede virtual recém-criada e armazená-la na variável SUBNET_ID:

    ```cli
    SUBNET_ID=$(az network vnet subnet list --resource-group $RESOURCE_GROUP_NAME --vnet-name $VNET_NAME --query "[?name == '$SUBNET_NAME'].id" --output tsv)
    ```

### Tarefa 2: Implantar um modelo Bicep que provisiona VMs do Azure executando o SUSE do Linux, que hospedará uma implantação do SAP NetWeaver altamente disponível

1.  No computador de laboratório, no painel do Cloud Shell, execute os seguintes comandos para criar um clone superficial do repositório que hospeda o modelo Bicep que você usará para implantação de um par de VMs do Azure que irá hospedar uma instalação altamente disponível do SAP HANA, e defina o diretório atual como o local desse modelo e seu arquivo de parâmetro:

    ```
    cd $HOME
    rm ./azure-quickstart-templates -rf
    git clone --depth 1 https://github.com/polichtm/azure-quickstart-templates
    cd ./azure-quickstart-templates/application-workloads/sap/sap-3-tier-marketplace-image-md/
    ```

1.  No painel do Cloud Shell, execute os seguintes comandos para definir o nome da conta de usuário administrativa e sua senha:

    ```
    ADMINUSERNAME='student'
    ADMINPASSWORD='Pa55w.rd1234'
    ```

1.  No painel do Cloud Shell, execute o seguinte comando para identificar os recursos que serão incluídos na próxima implantação:

    ```
    DEPLOYMENT_NAME='az1203a-'$RANDOM
    az deployment group what-if --name $DEPLOYMENT_NAME --resource-group $RESOURCE_GROUP_NAME --template-file ./main.bicep --parameters ./azuredeploy.parameters03a.json --parameters adminUsername=$ADMINUSERNAME adminPasswordOrKey=$ADMINPASSWORD subnetId=$SUBNET_ID
    ```

1.  Examine a saída do comando e verifique se ele não inclui erros (ignore todos os avisos). Em seguida, no painel do Cloud Shell, execute o seguinte comando para executar a implantação:

    ```
    DEPLOYMENT_NAME='az1203a-'$RANDOM
    az deployment group create --name $DEPLOYMENT_NAME --resource-group $RESOURCE_GROUP_NAME --template-file ./main.bicep --parameters ./azuredeploy.parameters.json --parameters adminUsername=$ADMINUSERNAME adminPasswordOrKey=$ADMINPASSWORD subnetId=$SUBNET_ID
    ```

1.  Examine a saída do comando e verifique se ele não inclui erros e avisos. Quando solicitado, pressione a tecla **Enter** para continuar com a implantação.

1.  Não aguarde a conclusão da implantação, prossiga para a próxima tarefa. 

    > **Observação**: Se a implantação falhar com a mensagem de erro de **conflito** durante a implantação do componente CustomScriptExtension, use as seguintes etapas para corrigir esse problema:

       - no portal do Azure, no painel **Implantação**, examine os detalhes da implantação e identifique as VMs em que a instalação do CustomScriptExtension falhou

       - no portal do Azure, navegue até o painel da(s) VM(s) identificada(s) na etapa anterior, selecione **Extensões** e, no painel **Extensões**, remova a extensão CustomScript

       - Execute novamente a etapa anterior desta tarefa.

### Tarefa 3: Implantar um jump host

   > **Observação**: Como as VMs do Azure implantadas na tarefa anterior não são acessíveis da Internet, você implantará uma VM do Azure executando o Datacenter do Windows Server 2019 que servirá como um jump host. 

1. No computador do laboratório, no portal do Azure, clique em **+ Criar um recurso**.

1. Na folha **Novo**, inicie a criação de uma nova VM do Azure com base na imagem do **Datacenter do Windows Server 2019**.

1. Provisione uma VM do Azure com as seguintes configurações (deixe todas as outras com os valores padrão):

    | Configuração | Valor |
    |   --    |  --   |
    | **Assinatura** | *o nome da sua assinatura do Azure*  |
    | **Grupo de recursos** | *o nome de um novo grupo de recursos* **az12003a-dmz-RG** |
    | **Nome da máquina virtual** | **az12003a-vm0** |
    | **Região** | *a mesma região do Azure em que você implantou VMs do Azure nas tarefas anteriores deste exercício* |
    | **Opções de disponibilidade** | **Nenhuma redundância de infraestrutura necessária** |
    | **Imagem** | *selecione* **Datacenter do Windows Server 2019 – Gen2** |
    | **Tamanho** | **Standard D2s_v3** ou similar |
    | **Nome de usuário** | **Aluno** |
    | **Senha** | qualquer senha complexa de sua escolha |
    | **Portas de entrada públicas** | **Permitir portas selecionadas** |
    | **Portas de entrada selecionadas** | **RDP (3389)** |
    | **Deseja usar uma licença existente do Windows Server?** | **Não** |
    | **Tipo de disco de SO** | **HDD Standard** |
    | **Rede virtual** | **az12003a-sap-vnet** |
    | **Nome da sub-rede** | *uma nova sub-rede chamada* **bastionSubnet** |
    | **Intervalo de endereços da sub-rede** | **10.3.255.0/24** |
    | **Endereço IP público** | *um novo endereço IP chamado* **az12003a-vm0-ip** |
    | **Grupo de segurança de rede da NIC** | **Basic**  |
    | **Portas de entrada públicas** | **Permitir portas selecionadas** |
    | **Portas de entrada selecionadas** | **RDP (3389)** |
    | **Habilitar a rede acelerada** | **Ativado** |
    | **Opções de balanceamento de carga** | **Nenhuma** |
    | **Habilitar identidade gerenciada atribuída pelo sistema** | **Desativado** |
    | **Fazer logon com o Azure AD** | **Desativado** |
    | **Habilitar o desligamento automático** | **Desativado** |
    | **Opções de orquestração de patch** | **Atualizações manuais** |
    | **Diagnóstico de inicialização** | **Desabilitar** |
    | **Habilitar o diagnóstico de convidado do sistema operacional** | **Desativado** |
    | **Extensões** | *Nenhuma* |
    | **Marcas** | *Nenhuma* |

   > **Observação**: Lembre-se da senha especificada durante a implantação. Você precisará disso em uma etapa posterior deste laboratório.

1. Aguarde a conclusão do provisionamento. Isso deve levar alguns minutos.

> **Resultado**: Depois de concluir este exercício, você terá provisionado os recursos do Azure necessários para dar suporte a implantações do SAP NetWeaver altamente disponíveis


## Exercício 2: configure VMs do Azure que executam o Linux para dar suporte a uma implantação do SAP NetWeaver altamente disponível

Duração: 30 minutos

Neste exercício, você configurará as VMs do Azure executando o SUSE Linux Enterprise Server para acomodar uma implantação do SAP NetWeaver altamente disponível.

### Tarefa 1: Configure a rede das VMs do Azure da camada de banco de dados.

   > **Observação**: Antes de iniciar essa tarefa, certifique-se de que as implantações de modelo iniciadas no exercício anterior tenham sido concluídas com êxito. 

1. No computador de laboratório, no portal do Azure, navegue até a folha da VM do Azure **i20-db-0**.

1. Na folha **i20-db-0**, navegue até a folha **Rede**. 

1. Na folha **i20-db-0 - Rede**, navegue até a interface de rede do i20-db-0. 

1. Na folha da interface de rede da i20-db-0, navegue até a sua folha de configurações de IP e, a partir daí, exiba sua folha **ipconfig1**.

1. Na folha **ipconfig1**, defina o endereço IP privado como **10.3.0.20**, altere a atribuição para **Estática** e salve a alteração.

1. No portal do Azure, navegue até a folha da VM do Azure **i20-db-1**.

1. Na folha **i20-db-1**, navegue até a folha **Rede**. 

1. Na folha **i20-db-1 - Rede**, navegue até a interface de rede do i20-db-1. 

1. Na folha da interface de rede da i20-db-1, navegue até a sua folha de configurações de IP e, a partir daí, exiba sua folha **ipconfig1**.

1. Na folha **ipconfig1**, defina o endereço IP privado como **10.3.0.21**, altere a atribuição para **Estática** e salve a alteração.


### Tarefa 2: Conecte-se às VMs do Azure da camada de banco de dados.

1. No computador de laboratório, no portal do Azure, navegue até a folha **az12003a-vm0**.

1. Na folha **az12003a-vm0**, conecte-se à VM do Azure az12003a-vm0 via Área de Trabalho Remota. Quando solicitado a autenticar, insira o nome de usuário e a senha definidos durante a implantação dessa VM.

1. Na sessão do RDP para az12003a-vm0, no Gerenciador do Servidor, navegue até o modo de exibição **Servidor local** e desative a **Configuração de segurança reforçada do IE**.

1. Na sessão RDP para az12003a-vm0, baixe e instale o PuTTY de [**https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html**](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html).

1. Use o PuTTY para se conectar via SSH à VM do Azure **i20-db-0**. Confirme o alerta de segurança e, quando solicitado, forneça as seguintes credenciais:

    -   Faça logon como: **aluno**

    -   Senha: **Pa55w.rd1234**

1. Use o PuTTY para se conectar via SSH à VM do Azure **i20-db-1** com as mesmas credenciais.


### Tarefa 3: Examine a configuração de armazenamento das VMs do Azure da camada de banco de dados.

1. Na sessão SSH do PuTTY para a VM do Azure i20-db-0, execute o seguinte comando para elevar privilégios: 

    ```
    sudo su -
    ```

1. Se a senha for solicitada, digite **Pa55w.rd1234** e pressione a tecla **Enter**. 

1. Na sessão SSH para i20-db-0, verifique se todos os volumes relacionados ao SAP HANA (incluindo **/usr/sap**, **/hana/shared**, **/hana/backup**, **/hana/data** e **/hana/logs**) estão montados adequadamente executando:

    ```
    df -h
    ```

1. Repita as etapas anteriores na VM do Azure i20-db-1.


### Tarefa 4: Habilitar o acesso SSH sem senha entre nós

1. Na sessão SSH para i20-db-0, gere a chave SSH sem senha executando:

    ```
    ssh-keygen
    ```

1. Quando solicitado, pressione **Enter** três vezes e exiba a chave executando: 

    ```
    cat /root/.ssh/id_rsa.pub
    ```

1. Copie o valor da chave na Área de Transferência.

1. Na sessão SSH para i20-db-1, crie o arquivo **/root/.ssh/authorized\_keys** no editor vi executando:

    ```
    vi /root/.ssh/authorized_keys
    ```

1. No editor vi, cole a chave gerada no i20-db-0.

1. Salve as alterações e feche o editor.

1. Na sessão SSH para i20-db-1, gere a chave SSH sem senha executando:

    ```
    ssh-keygen
    ```

1. Quando solicitado, pressione **Enter** três vezes e exiba a chave executando: 

    ```
    cat /root/.ssh/id_rsa.pub
    ```

1. Copie o valor da chave na Área de Transferência.

1. Na sessão SSH para i20-db-0, crie o arquivo **/root/.ssh/authorized\_keys** no editor vi executando:

    ```
    vi /root/.ssh/authorized_keys
    ```

1. No editor vi, cole a chave gerada no i20-db-1 a partir de uma nova linha.

1. Salve as alterações e feche o editor.

1. Para verificar se a configuração foi bem-sucedida, na sessão SSH para i20-db-0, estabeleça uma sessão SSH como **raiz** de i20-db-0 para i20-db-1 executando: 

    ```
    ssh root@i20-db-1
    ```

1. Quando perguntado se deseja continuar se conectando, digite `yes` e pressione a tecla **Enter**. 

1. Verifique se você não foi solicitado a fornecer a senha.

1. Feche a sessão SSH de i20-db-0 para i20-db-1 executando: 

    ```
    exit
    ```

1. Na sessão SSH para i20-db-1, estabeleça uma sessão SSH como **raiz** de i20-db-1 para i20-db-0 executando: 

    ```
    ssh root@i20-db-0
    ```

1. Quando perguntado se deseja continuar se conectando, digite `yes` e pressione a tecla **Enter**. 

1. Verifique se você não foi solicitado a fornecer a senha.

1. Feche a sessão SSH do i20-db-1 para o i20-db-0 executando: 

    ```
    exit
    ```

### Tarefa 5: Adicionar pacotes YaST, atualizar o sistema operacional Linux e instalar extensões de HA

1. Na sessão SSH para i20-db-0, execute o seguinte para iniciar o YaST:

    ```
    yast
    ```

1. No **Centro de Controle do YaST**, selecione **Software -\> Produtos de Complemento** e pressione **Enter**. Isso carregará o **Gerenciador de Pacotes**.

1. Na tela **Produtos de Complemento Instalados**, verifique se o **Módulo de Nuvem Pública** já está instalado. Em seguida, pressione **F9** duas vezes para retornar ao prompt do shell.

1. Na sessão SSH para i20-db-0, execute o seguinte para atualizar o sistema operacional (quando solicitado, digite **y** e pressione a tecla **Enter**):

    ```
    zypper update
    ```

1. Na sessão SSH para i20-db-0, execute o seguinte para instalar os pacotes exigidos pelos recursos de cluster (quando solicitado, digite **y** e pressione a tecla **Enter**):

    ```
    zypper in socat
    ```

1. Na sessão SSH para i20-db-0, execute o seguinte para instalar o componente azure-lb exigido pelos recursos de cluster:

    ```
    zypper in resource-agents
    ```

1. Na sessão SSH para i20-db-0, abra o arquivo **/etc/systemd/system.conf** no editor vi executando:

    ```
    vi /etc/systemd/system.conf
    ```

1. No editor vi, substitua `#DefaultTasksMax=512` por `DefaultTasksMax=4096`. 

    > **Observação**: Em alguns casos, o Pacemaker pode criar muitos processos, atingindo o limite padrão imposto em seu número e disparando um failover. Essa alteração aumenta o número máximo de processos permitidos.

1. Salve as alterações e feche o editor.

1. Na sessão SSH para i20-db-0, execute o seguinte para ativar a alteração de configuração:

    ```
    systemctl daemon-reload
    ```

1. Na sessão SSH para i20-db-0, execute o seguinte para instalar o pacote de agentes de cerca:

    ```
    zypper install fence-agents
    ```

1. Na sessão SSH para i20-db-0, execute o seguinte para instalar o SDK do Python do Azure exigido pelo agente de cerca (quando solicitado, digite **y** e pressione a tecla **Enter**):

    ```
    SUSEConnect -p sle-module-public-cloud/12/x86_64
    zypper install python-azure-mgmt-compute
    ```

1. Repita as etapas anteriores nesta tarefa em i20-db-1.

> **Resultado**: Depois de concluir este exercício, você terá configurado o sistema operacional de VMs do Azure que executam o Linux para dar suporte a uma implantação do SAP NetWeaver altamente disponível

## Exercício 3: configure o clustering em VMs do Azure que executam o Linux para dar suporte a uma implantação do SAP NetWeaver altamente disponível

Duração: 30 minutos

Neste exercício, você irá configurar o clustering em VMs do Azure que executam o Linux para dar suporte a uma implantação do SAP NetWeaver altamente disponível.

### Tarefa 1: Configurar o clustering

1. Na sessão RDP para az12003a-vm0, na sessão SSH baseada em PuTTY para i20-db-0, execute o seguinte para iniciar a configuração de um cluster de HA no i20-db-0:

    ```
    ha-cluster-init -u
    ```

1. Quando solicitado, forneça as seguintes respostas:

    -   Deseja continuar assim mesmo (s/n)?: **s**

    -   Endereço do ring0 [10.3.0.20]: **INSERIR**

    -   Porta para o ring0 [5405]: **INSERIR**

    -   Você deseja usar SBD (s/n)?: **n**

    -   Você deseja configurar um endereço IP virtual (s/n)?: **n**

    > **Observação**: A configuração de clustering gera uma conta **hacluster** com sua senha definida como **linux**. Você a alterará posteriormente nesta tarefa.

1. Na sessão RDP para az12003a-vm0, na sessão SSH baseada em PuTTY para i20-db-1, execute o seguinte para ingressar no cluster de HA no i20-db-0 do i20-db-1:

    ```
    ha-cluster-join
    ```

1. Quando solicitado, forneça as seguintes respostas:

    -   Deseja continuar assim mesmo (s/n)? **y**

    -   Endereço IP ou nome do host do nó existente (por exemplo: 192.168.1.1) \[\]: **i20-db-0**

    -   Endereço do ring0 [10.3.0.21]: **INSERIR**

1. Na sessão SSH baseada em PuTTY como i20-db-0, execute o seguinte para definir a senha da conta **hacluster** como **Pa55w.rd1234** (digite a nova senha quando solicitado): 

    ```
    passwd hacluster
    ```

1. Repita a etapa anterior no i20-db-1.

### Tarefa 2: Examinar a configuração corosync

1. Dentro da sessão RDP para az12003a-vm0, na sessão SSH baseada em PuTTY para i20-db-0, abra o arquivo **/etc/corosync/corosync.conf** executando:

    ```
    vi /etc/corosync/corosync.conf
    ```

1. No editor vi, observe a entrada `transport: udpu` e a seção `nodelist`:
    ```
    [...]
       interface { 
           [...] 
       }
       transport:      udpu
    } 
    nodelist {
       node {
         ring0_addr:     10.3.0.20
         nodeid:     1
       }
       node {
         ring0_addr:     10.3.0.21
         nodeid:     2
       } 
    }
    logging {
        [...]
    ```

1. No editor vi, substitua a entrada `token: 5000` por `token: 30000`.

    > **Observação**: Essa alteração permite a manutenção da preservação da memória. Para obter mais informações, consulte a [documentação da Microsoft sobre manutenção de máquinas virtuais no Azure](https://docs.microsoft.com/en-us/azure/virtual-machines/maintenance-and-updates#maintenance-that-doesnt-require-a-reboot)

1. Salve as alterações e feche o editor.

1. Repita as etapas anteriores no i20-db-1.


### Tarefa 3: Identificar o valor da ID da assinatura do Azure e a ID do locatário do Azure AD

1. No computador do laboratório, na janela do navegador, no portal do Azure em **https://portal.azure.com**, verifique se você está conectado com a conta de usuário que tem a função de Administrador Global no locatário do Azure AD associado à sua assinatura.

1. No portal do Azure, inicie uma sessão do Bash no Cloud Shell. 

1. No painel do Cloud Shell, execute o seguinte comando para identificar a id da sua assinatura do Azure e a id do locatário do Azure AD correspondente:

    ```cli
    az account show --query '{id:id, tenantId:tenantId}' --output json
    ```

1. Copie os valores resultantes para o Bloco de notas. Isso será necessário na próxima tarefa.


### Tarefa 4: Criar um aplicativo do Azure AD para o dispositivo STONITH

1. No portal do Azure, navegue até o painel Azure Active Directory.

1. No painel do Azure Active Directory, navegue até o painel **Registros de aplicativo** e clique em **+ Novo registro**:

1. Na folha **Registrar um aplicativo**, especifique as seguintes configurações e clique em **Registrar**:

    -   Nome: **Aplicativo Stonith**

    -   Tipo de conta compatível: **Contas somente neste diretório organizacional**

1. No painel **Aplicativo Stoneth**, copie o valor da **ID do Aplicativo (cliente)** para o Bloco de notas. Isso será referido como **login_id** mais adiante neste exercício:

1. No painel **Aplicativo Stoneth**, clique em **Certificados e segredos**.

1. No painel **Aplicativo Stonith – Certificados e segredos**, clique em **+ Novo segredo do cliente**.

1. No painel **Adicionar um segredo do cliente**, na caixa de texto **Descrição**, digite **Chave do aplicativo STONITH**, na seção **Expira**, deixe o padrão **Recomendado: 6 meses** e clique em **Adicionar**.

1. Copie o **Valor** resultante para o Bloco de notas (esta entrada é exibida apenas uma vez, depois de clicar em **Adicionar**). Isso será referido como **senha** mais adiante neste exercício:


### Tarefa 5: Conceder permissões a VMs do Azure à entidade de serviço do aplicativo STONITH 

1. No portal do Azure, navegue até a folha da VM do Azure **i20-db-0**

1. No painel **i20-db-0**, exiba o painel **i20-db-0 - Controle de acesso (IAM)**.

1. No painel **i20-db-0 - Controle de acesso (IAM)**, adicione uma atribuição de função com as seguintes configurações:

    -   Função: **Colaborador da Máquina Virtual**

    -   Atribuir acesso a: **Usuário, grupo ou entidade de serviço do Azure AD**

    -   Select: **Aplicativo Stonith**

1. Repita as etapas anteriores para atribuir ao aplicativo Stonith a função de Colaborador de Máquina Virtual à VM do Azure **i20-db-1**


### Tarefa 6: Configurar o dispositivo de cluster STONITH 

1. Na sessão RDP para az12003a-vm0, alterne para a sessão SSH baseada em PuTTY para i20-db-0.

1. Na sessão RDP para az12003a-vm0, na sessão SSH baseada em PuTTY para i20-db-0, execute os seguintes comandos (substitua os espaços reservados `subscription_id`, `tenant_id`, `login_id,` e `password` pelos valores identificados na Tarefa 4 do Exercício 3:

    ```
    crm configure property stonith-enabled=true

    crm configure property concurrent-fencing=true

    crm configure primitive rsc_st_azure stonith:fence_azure_arm \
      params subscriptionId="subscription_id" resourceGroup="az12003a-sap-RG" tenantId="tenant_id" login="login_id" passwd="password" \
      pcmk_monitor_retries=4 pcmk_action_limit=3 power_timeout=240 pcmk_reboot_timeout=900 \
      op monitor interval=3600 timeout=120

    sudo crm configure property stonith-timeout=900
    ```

### Tarefa 7: Revisar a configuração de cluster em VMs do Azure que executam o Linux usando o Hawk

1. Na sessão RDP para az12003a-vm0, inicie o Internet Explorer e navegue até **https://i20-db-0:7630**. Isso deve exibir a página de entrada do SUSE Hawk.

   > **Observação**: Ignore a mensagem **Este site não é seguro**.

1. Na página de entrada do SUSE Hawk, faça logon usando as seguintes credenciais:

    -   Nome de usuário: **hacluster**

    -   Senha: **Pa55w.rd1234**

1. Verifique se o status do cluster é íntegro. Se você estiver vendo uma mensagem indicando que um dos dois nós de cluster não está limpo, reinicie esse nó no portal do Azure.

> **Resultado**: Depois de concluir este exercício, você terá configurado o clustering em VMs do Azure que executam o Linux para dar suporte a uma implantação do SAP NetWeaver altamente disponível


## Exercício 4: remova recursos de laboratório

Duração: 10 minutos

Neste exercício, você removerá os recursos provisionados neste laboratório.

#### Tarefa 1: Abrir o Cloud Shell

1. Na parte superior do portal, clique no ícone do **Cloud Shell** para abrir o painel do Cloud Shell e escolha o Bash como o shell.

1. No painel do Cloud Shell, execute o seguinte comando para definir o valor da variável `RESOURCE_GROUP_PREFIX` como o prefixo do nome do grupo de recursos que contém os recursos provisionados neste laboratório:

    ```cli
    RESOURCE_GROUP_PREFIX='az12003a-'
    ```

1. No painel do Cloud Shell, execute o seguinte comando para listar todos os grupos de recursos criados neste laboratório:

    ```cli
    az group list --query "[?starts_with(name,'$RESOURCE_GROUP_PREFIX')]".name --output tsv
    ```

1. Verifique se a saída contém apenas o grupo de recursos que você criou neste laboratório. Esse grupo de recursos com todos os seus recursos será excluído na próxima tarefa.

#### Tarefa 2: Excluir grupos de recursos

1. No painel do Cloud Shell, execute o comando a seguir para excluir o grupo de recursos e seus recursos.

    ```cli
    az group list --query "[?starts_with(name,'$RESOURCE_GROUP_PREFIX')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

1. Feche o painel do Cloud Shell.

> **Resultado**: Depois de concluir este exercício, você terá removido os recursos usados neste laboratório.
