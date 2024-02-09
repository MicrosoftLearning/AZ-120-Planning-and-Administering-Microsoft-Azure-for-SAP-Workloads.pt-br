---
lab:
  title: 04b – implementar a arquitetura SAP em VMs do Azure que executam o Windows
  module: Module 04 - Deploy SAP on Azure
---

# Módulo 4 do AZ 120: Implantar SAP no Azure
# Laboratório 4b: Implementar a arquitetura SAP em VMs do Azure que executam o Windows

Tempo estimado: 150 minutos

Todas as tarefas neste laboratório são executadas no portal do Azure (incluindo uma sessão do PowerShell do Cloud Shell)  

   > **Observação**: Quando não estiver usando o Cloud Shell, a máquina virtual do laboratório deve ter o módulo do Az PowerShell instalado [**https://docs.microsoft.com/en-us/powershell/azure/install-az-ps-msi**](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps-msi).

Arquivos de laboratório: nenhum

## Cenário
  
Em preparação para a implantação do SAP NetWeaver no Azure, a Adatum Corporation deseja implementar uma demonstração que ilustrará a implementação altamente disponível do SAP NetWeaver em VMs do Azure executando o Windows Server 2016.

## Objetivos
  
Depois de realizar este laboratório, você será capaz de:

-   Provisionar recursos do Azure necessários para dar suporte a uma implantação do SAP NetWeaver altamente disponível

-   Configurar o sistema operacional de VMs do Azure que executam o Windows para dar suporte a uma implantação do SAP NetWeaver altamente disponível

-   Configurar o clustering em VMs do Azure que executam o Windows para dar suporte a uma implantação do SAP NetWeaver altamente disponível

## Requisitos

-   Uma assinatura do Microsoft Azure com número suficiente de VCPUs Dsv3 disponíveis (quatro VMs Standard_D2s_v3 com 2 vCPUs e seis VMs Standard_D4s_v3 com 4 vCPUs cada) em uma região do Azure que dá suporte a zonas de disponibilidade

-   Um computador de laboratório com um navegador da Web compatível com o Azure Cloud Shell e acesso ao Azure


## Exercício 1: provisione os recursos do Azure necessários para dar suporte a implantações SAP NetWeaver altamente disponíveis

Duração: 60 minutos

Neste exercício, você implantará os componentes de computação de infraestrutura do Azure necessários para configurar o clustering do Windows. Isso envolverá a criação de um par de VMs do Azure que executam o Windows Server 2016 no mesmo conjunto de disponibilidade.

### Tarefa 1: Implantar um par de VMs do Azure que executam controladores de domínio do Active Directory altamente disponíveis usando um modelo do Azure Resource Manager

1. No computador do laboratório, inicie um navegador da Web e navegue até o portal do Azure em **https://portal.azure.com**.

1. Se solicitado, entre com a conta Microsoft corporativa ou de estudante ou pessoal com a função de proprietário ou colaborador para a assinatura do Azure que você usará para este laboratório.

1. No portal do Azure, inicie uma sessão do PowerShell no Cloud Shell. 

    > **Observação**: Se esta for a primeira vez que você está iniciando o Cloud Shell na assinatura atual do Azure, você será solicitado a criar um compartilhamento de arquivos do Azure para persistir arquivos do Cloud Shell. Nesse caso, aceite os padrões, o que resultará na criação de uma conta de armazenamento em um grupo de recursos gerado automaticamente.

1. No painel do Cloud Shell, execute os seguintes comandos para criar um clone superficial do repositório que hospeda o modelo Bicep que você usará para implantação de um par de VMs do Azure que executam controladores de domínio do Active Directory altamente disponíveis, e defina o diretório atual como o local desse modelo e seu arquivo de parâmetro:

    ```
    cd $HOME
    rm ./azure-quickstart-templates -rf
    git clone --depth 1 https://github.com/polichtm/azure-quickstart-templates
    cd ./azure-quickstart-templates/application-workloads/active-directory/active-directory-new-domain-ha-2-dc-zones/
    ```

1. No painel do Cloud Shell, execute o seguinte comando para definir o valor da variável `$rgName` como `az12001b-ad-RG`:

    ```
    $rgName = 'az12003b-ad-RG'
    ```

1. No painel do Cloud Shell, execute o seguinte comando para definir o valor da variável `$location` para o nome das regiões do Azure que dão suporte a zonas de disponibilidade e onde você pretende implantar as VMs do laboratório (substitua o espaço reservado `<Azure_region>` pelo nome da região):

    ```
    $location = '<Azure_region>'
    ```

1. No painel do Cloud Shell, execute o seguinte comando para criar um grupo de recursos chamado **az12001b-ad-RG** na região do Azure escolhida:

    ```
    New-AzResourceGroup -Name $rgName -Location $location
    ```

1. No painel do Cloud Shell, execute o seguinte comando para definir o valor da variável `$deploymentName`:

    ```
    $deploymentName = 'az1203b-' + $(Get-Date -Format 'yyyy-MM-dd-hh-mm')
    ```

1. No painel do Cloud Shell, execute os seguintes comandos para definir o nome e a senha da conta de usuário administrativo (substitua os espaços reservados `<username>` e `<password>` pelo nome e o valor da senha da conta de usuário administrativo, respectivamente):

    ```
    $adminUsername = '<username>'
    $adminPassword = ConvertTo-SecureString '<password>' -AsPlainText -Force
    ```

    > **Observação**: Certifique-se de que a senha atenda aos requisitos de complexidade aplicáveis à implantação de VMs do Azure que executam o Windows (ela deve conter pelo menos 12 caracteres contendo letras minúsculas e maiúsculas, números e caracteres especiais).

1. No painel do Cloud Shell, execute o seguinte comando para executar a implantação:

    ```
    New-AzResourceGroupDeployment -Name $deploymentName -ResourceGroupName $rgName -TemplateFile .\main.bicep -TemplateParameterFile .\azuredeploy.parameters.json -adminUsername $adminUsername -adminPassword $adminPassword -c
    ```

1. Examine a saída do comando e verifique se ele não inclui erros e avisos. Quando solicitado, pressione a tecla **Enter** para continuar com a implantação.

    > **Observação**: A implantação deve levar cerca de 30 minutos. Aguarde a conclusão da implantação antes de prosseguir para a próxima tarefa.

    > **Observação**: Se a implantação falhar com um erro, incluindo a instrução `PowerShell DSC resource MSFT_xADDomainController failed to execute Set-TargetResource functionality with error message: Domain 'adatum.com' could not be found`, use as seguintes etapas para corrigir esse problema:

    - No portal do Azure, navegue até a folha da VM **adBDC**, no menu de navegação vertical no lado esquerdo, na seção **Configurações**, selecione **Extensões + aplicativos**, no painel **Extensões + aplicativos**, selecione **PrepareBDC** e, no painel **Preparar BDC**, selecione **Desinstalar**. 

    - Navegue de volta para a folha de VM **AdBDC** e reinicie a VM do Azure.

    - Navegue até a folha **az1203b-ad-RG**, no menu de navegação vertical no lado esquerdo, seção **Configurações**, selecione **Implantações**.

    - Na folha **az1203b-ad-RG \| Implantações**, selecione a implantação que começa com o prefixo **az1203b** e, na folha de implantação, selecione **Reimplantar**.

    - Na folha **Implantação personalizada**, na caixa de texto **Senha do administrador**, insira a mesma senha usada durante a implantação original, selecione **Examinar + criar** e, em seguida, selecione **Criar**.

    - Não aguarde a conclusão da reimplantação, prossiga para a próxima tarefa. A reimplantação deve levar cerca de 3 minutos.

### Tarefa 2: Provisione sub-redes que hospedarão VMs do Azure que executam a implantação do SAP NetWeaver altamente disponível e o cluster S2D.

1. No portal do Azure, navegue até a folha do grupo de recursos **az12003b-ad-RG**.

1. Na folha do grupo de recursos **az12003b-ad-RG**, na lista de recursos, localize a rede virtual **adVNET** e clique em sua entrada para exibir a folha **adVNET**. 

1. Na folha **adVNET**, navegue até a folha **adVNET – sub-redes**. 

1. Na folha **adVNET – sub-redes**, crie uma nova sub-rede com as seguintes configurações:

    -   Nome: **sapSubnet**

    -   Intervalos de endereços (bloco CIDR): **10.0.1.0/24**

1. Na folha **adVNET – sub-redes**, crie uma nova sub-rede com as seguintes configurações:

    -   Nome: **s2dSubnet**

    -   Intervalos de endereços (bloco CIDR): **10.0.2.0/24**

1. No portal do Azure, inicie uma sessão do PowerShell no Cloud Shell. 

    > **Observação**: Se esta for a primeira vez que você está iniciando o Cloud Shell na assinatura atual do Azure, você será solicitado a criar um compartilhamento de arquivos do Azure para persistir arquivos do Cloud Shell. Nesse caso, aceite os padrões, o que resultará na criação de uma conta de armazenamento em um grupo de recursos gerado automaticamente.

1. No painel do Cloud Shell, execute o seguinte comando para definir o valor da variável `$resourceGroupName` como o nome do grupo de recursos que contém os recursos provisionados na tarefa anterior:

    ```
    $resourceGroupName = 'az12003b-ad-RG'
    ```

1. No painel do Cloud Shell, execute o seguinte comando para identificar a rede virtual criada na tarefa anterior:

    ```
    $vNetName = 'adVNet'

    $subnetName = 'sapSubnet'
    ```

1. No painel do Cloud Shell, execute o seguinte comando para identificar a ID de recurso da sub-rede recém-criada:

    ```
    $vNet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name $vNetName
    
    (Get-AzVirtualNetworkSubnetConfig -Name $subnetName -VirtualNetwork $vNet).Id
    ```

1. Copie o valor resultante para área de transferência. Isso será necessário na próxima tarefa.

### Tarefa 3: Implantar VMs do Azure de provisionamento de modelos do Azure Resource Manager que executam o Windows Server 2016 que hospedarão uma implantação do SAP NetWeaver altamente disponível

1. No computador do laboratório, no portal do Azure, pesquise e selecione **Implantação do modelo (implantar usando modelo personalizado)**.

1. Na folha de **Implantação personalizada**, na lista suspensa **Modelo de início rápido (aviso de isenção de responsabilidade)**, digite **application-workloads/sap/sap-3-tier-marketplace-image-md** e clique em **Selecionar modelo**.

    > **Observação**: Use o Microsoft Edge ou um navegador de terceiros. Não use o Internet Explorer.

1. Na folha **SAP NetWeaver de 3 camadas (disco gerenciado)**, selecione **Editar modelo**.

1. Na folha **Editar modelo**, aplique a seguinte alteração e selecione **Salvar**:

    -   na linha **197**, substitua `"dbVMSize": "Standard_E8s_v3",` por `"dbVMSize": "Standard_D4s_v3",`

1. De volta à folha **SAP NetWeaver de 3 camadas (disco gerenciado)**, especifique as seguintes configurações, clique em **Examinar + criar** e clique em **Criar** para iniciar a implantação:
    
    | Configuração | Valor |
    |   --    |  --   |
    | **Assinatura** | *o nome da sua assinatura do Azure*  |
    | **Grupo de recursos** | *o nome de um novo grupo de recursos* **az12003b-sap-RG** |
    | **Localidade** | *a mesma região do Azure que você especificou na primeira tarefa deste exercício* |
    | **ID do sistema SAP** | **I20** |
    | **Tipo de pilha** | **ABAP** |
    | **Tipo de sistema operacional** | **Windows Server 2016 Datacenter** |
    | **Dbtype** | **SQL** |
    | **Tamanho do sistema SAP** | **Demonstração** |
    | **Disponibilidade do sistema** | **HA** |
    | **Nome de Usuário do Administrador** | **Aluno** |
    | **Tipo de Autenticação** | **password** |
    | **Senha ou chave de administrador** | *a mesma senha que você especificou anteriormente neste laboratório* |
    | **ID da sub-rede** | *o valor copiado para área de transferência na tarefa anterior* |
    | **Zonas de Disponibilidade** | **1,2** |
    | **Localidade** | **[resourceGroup().location]** |
    | **Localização de _artifacts** | **https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/application-workloads/sap/sap-3-tier-marketplace-image-md/** |
    | **_token SAS de localização de artefatos** | *deixar em branco* |

1. Não aguarde a conclusão da implantação, prossiga para a próxima tarefa. 

### Tarefa 4: Implantar o cluster do Servidor de Arquivos de Escalabilidade Horizontal (SOFS)

Nesta tarefa, você implantará o cluster do Servidor de Arquivos de Escalabilidade Horizontal (SOFS) que hospedará um compartilhamento de arquivos para os servidores SAP do ASCS usando um modelo de início rápido do Azure Resource Manager do GitHub disponível em [**https://github.com/polichtm/301-storage-spaces-direct-md**](https://github.com/polichtm/301-storage-spaces-direct-md). 

1. No computador do laboratório, inicie um navegador e navegue até [**https://github.com/polichtm/301-storage-spaces-direct-md**](https://github.com/polichtm/301-storage-spaces-direct-md). 

    > **Observação**: Use o Microsoft Edge ou um navegador de terceiros. Não use o Internet Explorer.

1. Na página intitulada **Usar discos gerenciados para criar um cluster de Servidor de Arquivos de Escalabilidade Horizontal (SOFS) dos Espaços de Armazenamento Diretos (S2D) com o Windows Server 2016**, clique em **Implantar no Azure**. Isso redirecionará o seu navegador automaticamente para o portal do Azure e exibirá a folha **Implantação personalizada**.

1. Na folha **Implantação personalizada**, especifique as seguintes configurações, clique em **Examinar + criar** e clique em **Criar** para iniciar a implantação:
    
    | Configuração | Valor |
    |   --    |  --   |
    | **Assinatura** | *o nome da sua assinatura do Azure*  |
    | **Grupo de recursos** | *o nome de um novo grupo de recursos* **az12003b-s2d-RG** |
    | **Região** | *a mesma região do Azure em que você implantou VMs do Azure nas tarefas anteriores deste exercício* |
    | **Prefixo de nome** | **i20** |
    | **Tamanho da VM** | **D4s Standard\_v3** |
    | **Habilitação da rede acelerada** | **true** |
    | **SKU de imagem** | **2016-Datacenter-Server-Core** |
    | **Contagem de VMs** | **2** |
    | **Tamanho do disco da VM** | **128** |
    | **Contagem de discos de VM** | **3** |
    | **Nome de Domínio Existente** | **adatum.com** |
    | **Nome de Usuário do Administrador** | **Aluno** |
    | **Senha de administrador** | *a mesma senha que você especificou anteriormente neste laboratório* |
    | **Nome RG da rede virtual existente** | **az12003b-ad-RG** |
    | **Nome da rede virtual existente** | **adVNet** |
    | **Nome da sub-rede existente** | **s2dSubnet** |
    | **Nome do Sofs** | **sapglobalhost** |
    | **Nome do compartilhamento** | **sapmnt** |
    | **Data da atualização agendada** | **Domingo** |
    | **Horário da atualização agendada** | **03h00** |
    | **Antimalware em tempo real habilitado** | **false** |
    | **Antimalware agendado habilitado** | **false** |
    | **Horário agendado do antimalware** | **120** |
    | **Localização de _artifacts** | **https://raw.githubusercontent.com/polichtm/301-storage-spaces-direct-md/master** |
    | **_token SAS de localização de artefatos** | **Deixe o valor padrão** |

1. A implantação pode levar cerca de 20 minutos. Não aguarde a conclusão da implantação, prossiga para a próxima tarefa.

    > **Observação**: Se a implantação falhar com a mensagem de erro de **conflito** durante a implantação do componente i20-s2d-1/s2dPrep ou i20-s2d-0/s2dPrep, use as seguintes etapas para corrigir esse problema:

       - No portal do Azure, navegue até a máquina virtual **i20-s2d-0** e, no menu de navegação vertical, na seção **Operações**, selecione **Executar comando**; no painel **Executar script de comando**, na caixa de texto **Script do PowerShell**, insira o script a seguir e selecione o botão **Executar** (substitua o espaço reservado `<password>` pela senha especificada anteriormente neste laboratório):

       ```
       $domain = 'adatum.com'
       $password = '<password>' | ConvertTo-SecureString -asPlainText -Force
       $username = "Student@$domain" 
       $credential = New-Object System.Management.Automation.PSCredential($username,$password)
       Add-Computer -DomainName $domain -Credential $credential -Restart -Force
       ```

       - Navegue até a folha da máquina virtual **i20-s2d-1** e, no menu de navegação vertical, na seção **Operações**, selecione **Executar comando**; no painel **Executar script de comando**, na caixa de texto **Script do PowerShell**, insira o script a seguir e selecione o botão **Executar** (substitua o espaço reservado `<password>` pela senha especificada anteriormente neste laboratório):

       ```
       $domain = 'adatum.com'
       $password = '<password>' | ConvertTo-SecureString -asPlainText -Force
       $username = "Student@$domain" 
       $credential = New-Object System.Management.Automation.PSCredential($username,$password)
       Add-Computer -DomainName $domain -Credential $credential -Restart -Force
       ```
       
       - Executar novamente as etapas da tarefa atual na versão inicial

### Tarefa 5: Implantar um jump host

   > **Observação**: Como as VMs do Azure implantadas na tarefa anterior não são acessíveis da Internet, você implantará uma VM do Azure executando o Datacenter do Windows Server 2016 que servirá como um jump host. 

1. No computador do laboratório, na interface do portal do Azure, clique em **+ Criar um recurso**.

1. Na folha **Novo**, inicie a criação de uma nova VM do Azure com base na imagem do **Datacenter do Windows Server 2019 – Gen1**.

1. Provisione uma VM do Azure com as seguintes configurações:
    
    | Configuração | Valor |
    |   --    |  --   |
    | **Assinatura** | *o nome da sua assinatura do Azure*  |
    | **Grupo de recursos** | *o nome de um novo grupo de recursos* **az12003b-dmz-RG** |
    | **Nome da máquina virtual** | **az12003b-vm0** |
    | **Região** | *a mesma região do Azure em que você implantou VMs do Azure nas tarefas anteriores deste exercício* |
    | **Opções de disponibilidade** | **Nenhuma redundância de infraestrutura necessária** |
    | **Imagem** | *selecione* **Datacenter do Windows Server 2019 – Gen2** |
    | **Tamanho** | **Standard D2s_v3** |
    | **Nome de usuário** | **Aluno** |
    | **Senha** | *a mesma senha que você especificou anteriormente neste laboratório* |
    | **Portas de entrada públicas** | **Permitir portas selecionadas** |
    | **Portas de entrada selecionadas** | **RDP (3389)** |
    | **Deseja usar uma licença existente do Windows Server?** | **Não** |
    | **Tipo de disco de SO** | **HDD Standard** |
    | **Rede virtual** | **adVNET** |
    | **Nome da sub-rede** | *uma nova sub-rede chamada* **dmzSubnet** |
    | **Intervalo de endereços da sub-rede** | **10.0.255.0/24** |
    | **Endereço IP público** | *um novo endereço IP chamado* **az12003b-vm0-ip** |
    | **Grupo de segurança de rede da NIC** | **Basic**  |
    | **Portas de entrada públicas** | **Permitir portas selecionadas** |
    | **Portas de entrada selecionadas** | **RDP (3389)** |
    | **Habilitar a rede acelerada** | **Desativado** |
    | **Opções de balanceamento de carga** | **Nenhuma** |
    | **Habilitar identidade gerenciada atribuída pelo sistema** | **Desativado** |
    | **Fazer logon com o Azure AD** | **Desativado** |
    | **Habilitar o desligamento automático** | **Desativado** |
    | **Opções de orquestração de patch** | **Atualizações manuais** |
    | **Diagnóstico de inicialização** | **Desabilitar** |
    | **Habilitar o diagnóstico de convidado do sistema operacional** | **Desativado** |
    | **Extensões** | *Nenhuma* |
    | **Marcas** | *Nenhuma* |
   
1. Aguarde a conclusão do provisionamento. Isso deve levar alguns minutos.

> **Resultado**: Depois de concluir este exercício, você terá provisionado os recursos do Azure necessários para dar suporte a implantações do SAP NetWeaver altamente disponíveis


## Exercício 2: configure o sistema operacional de VMs do Azure que executam o Windows para dar suporte a uma implantação de SAP NetWeaver altamente disponível

Duração: 60 minutos

Neste exercício, você configurará o sistema operacional de VMs do Azure executando o Windows Server para acomodar uma implantação do SAP NetWeaver altamente disponível.

### Tarefa 1: Ingresse as VMs do Azure do Windows Server 2016 no domínio do Active Directory.

   > **Observação**: Antes de iniciar essa tarefa, certifique-se de que as implantações de modelo iniciadas no exercício anterior tenham sido concluídas com êxito. 

1. No portal do Azure, navegue até a folha da rede virtual chamada **adVNET**, que foi provisionada automaticamente no primeiro exercício deste laboratório.

1. Exiba a folha **adVNET – servidores DNS** e observe que a rede virtual está configurada com os endereços IP privados atribuídos aos controladores de domínio implantados no primeiro exercício deste laboratório como seus servidores DNS.

1. No portal do Azure, inicie uma sessão do PowerShell no Cloud Shell. 

1. No painel do Cloud Shell, execute o seguinte comando para definir o valor da variável `$resourceGroupName` como o nome do grupo de recursos que contém os recursos provisionados na tarefa anterior:

    ```
    $resourceGroupName = 'az12003b-sap-RG'
    ```

1. No painel do Cloud Shell, execute o comando a seguir para unir as VMs do Azure do Windows Server implantadas na terceira tarefa do exercício anterior ao domínio **adatum.com** do Active Directory (substitua o espaço reservado `<password>` pela senha especificada anteriormente neste laboratório):

    ```
    $location = (Get-AzResourceGroup -Name $resourceGroupName).Location

    $settingString = '{"Name": "adatum.com", "User": "adatum.com\\Student", "Restart": "true", "Options": "3"}'

    $protectedSettingString = '{"Password": "<password>"}'

    $vmNames = @('i20-ascs-0','i20-ascs-1','i20-db-0','i20-db-1','i20-di-0','i20-di-1')

    foreach ($vmName in $vmNames) { Set-AzVMExtension -ResourceGroupName $resourceGroupName -ExtensionType 'JsonADDomainExtension' -Name 'joindomain' -Publisher "Microsoft.Compute" -TypeHandlerVersion "1.0" -Vmname $vmName -Location $location -SettingString $settingString -ProtectedSettingString $protectedSettingString }
    ```

### Tarefa 2: Examine a configuração de armazenamento das VMs do Azure da camada de banco de dados.

1. No computador de laboratório, no portal do Azure, navegue até a folha **az12003b-vm0**.

1. Na folha **az12003b-vm0**, conecte-se à VM do Azure az12003b-vm0 via Área de Trabalho Remota. Quando solicitado, forneça as seguintes credenciais:

    -   Faça logon como: **aluno**

    -   Senha: *a mesma senha especificada anteriormente por você neste laboratório*

1. Da sessão do RDP para az12003b-vm0, use a Área de Trabalho Remota para se conectar à VM do Azure **i20-db-0.adatum.com**. Quando solicitado, forneça as seguintes credenciais:

    -   Faça logon como: **ADATUM\\Aluno**

    -   Senha: *a mesma senha especificada anteriormente por você neste laboratório*

1. Use a Área de Trabalho Remota para se conectar à VM do Azure **i20-db-1.adatum.com** com as mesmas credenciais.

1. Na sessão do RDP para i20-db-0.adatum.com, use os Serviços de Arquivo e Armazenamento no Gerenciador do Servidor para examinar a configuração de disco. Observe que um único disco de dados foi configurado por meio de montagens de volume para fornecer armazenamento para arquivos de banco de dados e de log. 

1. Na sessão do RDP para i20-db-1.adatum.com, use os Serviços de Arquivo e Armazenamento no Gerenciador do Servidor para examinar a configuração do disco. Observe que um único disco de dados foi configurado por meio de montagens de volume para fornecer armazenamento para arquivos de banco de dados e de log. 


### Tarefa 3: Prepare-se para a configuração do clustering de failover em VMs do Azure que executam o Windows Server 2016 para dar suporte a uma instalação do SAP NetWeaver altamente disponível.

1. Na sessão do RDP para i20-db-0.adatum.com, inicie uma sessão do ISE do Windows PowerShell e instale os recursos de clustering de failover e ferramentas administrativas remotas executando o seguinte no par de servidores do ASCS e BD que se tornarão nós dos clusters do ASCS e SQL Server, respectivamente:

    ```
    $nodes = @('i20-ascs-0','i20-ascs-1','i20-db-0','i20-db-1')

    Invoke-Command $nodes {Install-WindowsFeature Failover-Clustering -IncludeAllSubFeature -IncludeManagementTools} 

    Invoke-Command $nodes {Install-WindowsFeature RSAT -IncludeAllSubFeature -Restart} 
    ```

    > **Observação**: Isso pode resultar na reinicialização do sistema operacional convidado de todas as quatro VMs do Azure.

1. No computador do laboratório, no portal do Azure, clique em **+ Criar um recurso**.

1. Na folha **Novo**, inicie a criação de uma nova **Conta de armazenamento** com as seguintes configurações:
    
    | Configuração | Valor |
    |   --    |  --   |
    | **Assinatura** | *o nome da sua assinatura do Azure* |
    | **Grupo de recursos** | *o nome do grupo de recursos no qual você implantou as VMs do Azure que hospedarão a implantação do SAP NetWeaver altamente disponível* |
    | **Nome da conta de armazenamento** | *qualquer nome exclusivo que consista entre 3 e 24 letras e dígitos* |
    | **Localidade** | *a mesma região do Azure em que você implantou as VMs do Azure no exercício anterior* |
    | **Desempenho** | **Standard** |
    | **Redundância** | **Armazenamento com redundância local (LRS)** |
    | **Método de conectividade** | **Ponto de extremidade público (todas as redes)** |
    | **Exigir transferência segura para operações de API REST** | **Enabled** |
    | **Compartilhamentos de arquivos grandes** | **Desabilitado** |
    | **Exclusão temporária para blobs, contêineres e arquivos** | **Desabilitado** |
    | **Namespace hierárquico** | **Desabilitado** |


### Tarefa 4: Configure o clustering de failover em VMs do Azure que executam o Windows Server 2016 para dar suporte a uma camada de banco de dados altamente disponível da instalação do SAP NetWeaver.

1. Se necessário, da sessão do RDP ao az12003b-vm0, use a Área de Trabalho Remota para se conectar novamente à VM do Azure **i20-db-0.adatum.com**. Quando solicitado, forneça as seguintes credenciais:

    -   Faça logon como: **ADATUM\\Aluno**

    -   Senha: *a mesma senha especificada anteriormente por você neste laboratório*

1. Na sessão do RDP para i20-db-0.adatum.com, no Gerenciador do Servidor, navegue até o modo de exibição **Servidor local** e desative a **Configuração de segurança reforçada do IE**.

1. Na sessão do RDP para i20-db-0.adatum.com, no menu **Ferramentas** no Gerenciador do Servidor, inicie o **Centro Administrativo do Active Directory**.

1. No Centro Administrativo do Active Directory, crie uma nova unidade organizacional chamada **Clusters** na raiz do domínio adatum.com.

1. No Centro Administrativo do Active Directory, mova as contas de computador do i20-db-0 e i20-db-1 do contêiner **Computadores** para a unidade organizacional **Clusters**.

1. Na sessão do RDP para i20-db-0, inicie uma sessão do ISE do Windows PowerShell e crie um novo cluster executando o seguinte:

    ```
    $nodes = @('i20-db-0','i20-db-1')

    New-Cluster -Name az12003b-db-cl0 -Node $nodes -NoStorage -StaticAddress 10.0.1.15
    ```

1. Na sessão do RDP para i20-db-0.adatum.com, alterne para o console do **Centro Administrativo do Active Directory**.

1. No Centro Administrativo do Active Directory, navegue até a unidade organizacional **Clusters** e exiba a janela **Propriedades**. 

1. Na unidade organizacional **Clusters**, janela **Propriedades**, navegue até a seção **Extensões** e exiba a guia **Segurança**. 

1. Na guia **Segurança**, clique no botão **Avançado** para abrir a janela **Configurações avançadas de segurança para clusters**. 

1. Na guia **Permissões** da janela **Configurações avançadas de segurança para clusters**, clique em **Adicionar**.

1. Na janela **Entrada de permissão para clusters**, clique em **Selecionar entidade de segurança**

1. Na caixa de diálogo **Selecionar usuário, conta de serviço ou grupo**, clique em **Tipos de objeto**, habilite a caixa de seleção ao lado da entrada **Computadores** e clique em **OK**. 

1. Na caixa de diálogo **Selecionar usuário, computador, conta de serviço ou grupo**, **insira o nome do objeto a ser selecionado**, digite **az12003b-db-cl0** e clique em **OK**.

1. Na janela **Entrada de permissão para clusters**, certifique-se de que **Permitir** apareça na lista suspensa **Tipo**. Em seguida, na lista suspensa **Aplica-se a**, selecione **Este objeto e todos os objetos descendentes**. Na lista **Permissões**, selecione as caixas de seleção **Criar objetos do computador** e **Excluir objetos do computador** e clique em **OK** duas vezes.

1. Na sessão do ISE do Windows PowerShell, instale o módulo Az PowerShell executando o seguinte:

    ```
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    
    Install-PackageProvider -Name NuGet -Force

    Install-Module -Name Az -Force
    ```

1. Na sessão do ISE do Windows PowerShell, autentique-se usando suas credenciais do Azure AD executando o seguinte:

    ```
    Add-AzAccount
    ```

    > **Observação**: Quando solicitado, entre com a conta Microsoft corporativa ou de estudante ou pessoal com a função de proprietário ou colaborador para a assinatura do Azure que você está usando para este laboratório.

1. Na sessão do ISE do Windows PowerShell, execute o seguinte comando para definir o valor da variável `$resourceGroupName` como o nome do grupo de recursos que contém a conta de armazenamento que você provisionou na tarefa anterior:

    ```
    $resourceGroupName = 'az12003b-sap-RG'
    ```

1. Na sessão do ISE do Windows PowerShell, execute o seguinte para definir o quorum da testemunha de nuvem do novo cluster:

    ```
    $cwStorageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName

    $cwStorageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $cwStorageAccountName).Value[0]

    Set-ClusterQuorum -CloudWitness -AccountName $cwStorageAccountName -AccessKey $cwStorageAccountKey
    ```

1. Para verificar a configuração resultante, na sessão do RDP para i20-db-0.adatum.com, no menu **Ferramentas** no Gerenciador do Servidor, inicie o **Gerenciador de cluster de failover**.

1. No console do **Gerenciador de cluster de failover**, examine a configuração do cluster **az12003b-db-cl0**, incluindo os seus nós e configurações de testemunha e rede. Observe que o cluster não tem nenhum armazenamento compartilhado.


### Tarefa 6: Configure o clustering de failover em VMs do Azure que executam o Windows Server 2016 para dar suporte a uma camada do ASCS altamente disponível da instalação do SAP NetWeaver.

> **Observação**: Certifique-se de que a implantação do cluster S2D iniciada na tarefa 4 do exercício 1 tenha sido concluída com êxito antes de iniciar esta tarefa.

1. Da sessão do RDP para az12003b-vm0, use a Área de Trabalho Remota para se conectar à VM do Azure **i20-ascs-0.adatum.com**. Quando solicitado, forneça as seguintes credenciais:

    -   Faça logon como: **ADATUM\\Aluno**

    -   Senha: *a mesma senha especificada anteriormente por você neste laboratório*

1. Na sessão do RDP para i20-ascs-0.adatum.com, no Gerenciador do Servidor, navegue até o modo de exibição **Servidor local** e desative a **Configuração de segurança reforçada do IE**.

1. Na sessão do RDP para i20-ascs-0.adatum.com, no menu **Ferramentas** no Gerenciador do Servidor, inicie o **Centro Administrativo do Active Directory**.

1. No Centro Administrativo do Active Directory, navegue até o contêiner **Computadores**. 

1. No Centro Administrativo do Active Directory, mova as contas de computador do i20-ascs-0 e i20-ascs-1 do contêiner **Computadores** para a unidade organizacional **Clusters**.

1. Na sessão do RDP para i20-ascs-0.adatum.com, inicie uma sessão do ISE do Windows PowerShell e crie um novo cluster executando o seguinte:

    ```
    $nodes = @('i20-ascs-0','i20-ascs-1')

    New-Cluster -Name az12003b-ascs-cl0 -Node $nodes -NoStorage -StaticAddress 10.0.1.16
    ```

1. Na sessão do RDP para i20-ascs-0.adatum.com, alterne para o console do **Centro Administrativo do Active Directory**.

1. No Centro Administrativo do Active Directory, navegue até a unidade organizacional **Clusters** e exiba a janela **Propriedades**. 

1. Na unidade organizacional **Clusters**, janela **Propriedades**, navegue até a seção **Extensões** e exiba a guia **Segurança**. 

1. Na guia **Segurança**, clique no botão **Avançado** para abrir a janela **Configurações avançadas de segurança para clusters**. 

1. Na guia **Permissões** da janela **Configurações avançadas de segurança para computadores**, clique em **Adicionar**.

1. Na janela **Entrada de permissão para clusters**, clique em **Selecionar entidade de segurança**

1. Na caixa de diálogo **Selecionar usuário, conta de serviço ou grupo**, clique em **Tipos de objeto**, habilite a caixa de seleção ao lado da entrada **Computadores** e clique em **OK**. 

1. Na caixa de diálogo **Selecionar usuário, computador, conta de serviço ou grupo**, **insira o nome do objeto a ser selecionado**, digite **az12003b-ascs-cl0** e clique em **OK**.

1. Na janela **Entrada de permissão para clusters**, certifique-se de que **Permitir** apareça na lista suspensa **Tipo**. Em seguida, na lista suspensa **Aplica-se a**, selecione **Este objeto e todos os objetos descendentes**. Na lista **Permissões**, selecione as caixas de seleção **Criar objetos do computador** e **Excluir objetos do computador** e clique em **OK** duas vezes.

1. Na sessão do ISE do Windows PowerShell, instale o módulo Az PowerShell executando o seguinte:

    ```
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    
    Install-PackageProvider -Name NuGet -Force

    Install-Module -Name Az -Force
    ```

1. Na sessão do ISE do Windows PowerShell, autentique-se usando suas credenciais do Azure AD executando o seguinte:

    ```
    Add-AzAccount
    ```

    > **Observação**: Quando solicitado, entre com a conta Microsoft corporativa ou de estudante ou pessoal com a função de proprietário ou colaborador para a assinatura do Azure que você está usando para este laboratório.

1. Na sessão do ISE do Windows PowerShell, execute o seguinte comando para definir o valor da variável `$resourceGroupName` como o nome do grupo de recursos que contém a conta de armazenamento que você provisionou anteriormente neste exercício:

    ```
    $resourceGroupName = 'az12003b-sap-RG'
    ```

1. Na sessão do ISE do Windows PowerShell, execute o seguinte para definir o quorum da testemunha de nuvem do cluster:

    ```
    $cwStorageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName

    $cwStorageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $cwStorageAccountName).Value[0]

    Set-ClusterQuorum -CloudWitness -AccountName $cwStorageAccountName -AccessKey $cwStorageAccountKey
    ```

1. Para verificar a configuração resultante, dentro da sessão RDP para i20-ascs-0.adatum.com, no menu **Ferramentas** no Gerenciador do Servidor, inicie o **Gerenciador de cluster de failover**.

1. No console do **Gerenciador de cluster de failover**, examine a configuração do cluster **az12003b-ascs-cl0**, incluindo os seus nós e configurações de testemunha e de rede. Observe que o cluster não tem nenhum armazenamento compartilhado.


### Tarefa 7: Definir permissões no compartilhamento \\\\GLOBALHOST\\sapmnt

Nesta tarefa, você definirá permissões de nível de compartilhamento no compartilhamento **\\\\GLOBALHOST\\sapmnt**. 

> **Observação**: Por padrão, as permissões de controle total são concedidas somente à conta ADATUM\Aluno. 

1. Na sessão da Área de Trabalho Remota para i20-ascs-0.adatum.com, na janela **ISE do Windows PowerShell**, execute o seguinte:

    ```
    $remoteSession = New-CimSession -ComputerName SAPGLOBALHOST

    Grant-SmbShareAccess -Name sapmnt -AccountName 'ADATUM\Domain Admins' -AccessRight Full -CimSession $remoteSession -Force
   
    ```

### Tarefa 8: Configurar pré-requisitos do sistema operacional para instalar o ASCS do SAP NetWeaver e componentes de banco de dados

1. Na sessão da Área de Trabalho Remota para i20-ascs-0.adatum.com, na sessão do ISE do Windows PowerShell, execute o seguinte para configurar as entradas do Registro necessárias para facilitar a instalação de componentes do ASCS do SAP e o uso de nomes virtuais:

    ```
    $nodes = ('i20-db-0','i20-db-1')

    Invoke-Command $nodes {
        $registryPath = 'HKLM:\SYSTEM\CurrentControlSet\Services\lanmanworkstation\parameters'
        $registryEntry = 'DisableCARetryOnInitialConnect'
        $registryValue = 1
        New-ItemProperty -Path $registryPath -Name $registryEntry -Value $registryValue -PropertyType DWORD -Force
    }

    Invoke-Command $nodes {
        $registryPath = 'HKLM:\SYSTEM\CurrentControlSet\Control\LSA'
        $registryEntry = 'DisableLoopbackCheck'
        $registryValue = 1
        New-ItemProperty -Path $registryPath -Name $registryEntry -Value $registryValue -PropertyType DWORD -Force
    }

    Invoke-Command $nodes {
        $registryPath = 'HKLM:\SYSTEM\CurrentControlSet\Services\lanmanserver\parameters'
        $registryEntry = 'DisableStrictNameChecking'
        $registryValue = 1
        New-ItemProperty -Path $registryPath -Name $registryEntry -Value $registryValue -PropertyType DWORD -Force
    }
    ```

> **Resultado**: Depois de concluir este exercício, você terá configurado o sistema operacional de VMs do Azure que executam o Windows para dar suporte a uma implantação do SAP NetWeaver altamente disponível


## Exercício 3: remova recursos de laboratório

Duração: 10 minutos

Neste exercício, você removerá os recursos provisionados neste laboratório.

#### Tarefa 1: Abrir o Cloud Shell

1. Na parte superior do portal, clique no ícone do **Cloud Shell** para abrir o painel do Cloud Shell e escolha o PowerShell como o shell.

1. No painel do Cloud Shell, execute o seguinte comando para definir o valor da variável `$resourceGroupName` como o nome do grupo de recursos que contém o par de VMs do Azure do **Datacenter do Windows Server 2019** provisionadas no primeiro exercício deste laboratório:

    ```
    $resourceGroupNamePrefix = 'az12003b-'
    ```

1. No painel do Cloud Shell, execute o seguinte comando para listar todos os grupos de recursos criados neste laboratório:

    ```
    Get-AzResourceGroup | Where-Object {$_.ResourceGroupName -like "$resourceGroupNamePrefix*"} | Select-Object ResourceGroupName
    ```

1. Verifique se a saída contém apenas os grupos de recursos que você criou neste laboratório. Esses grupos serão excluídos na próxima tarefa.

#### Tarefa 2: Excluir grupos de recursos

1. No painel do Cloud Shell, execute o comando a seguir para excluir os grupos de recursos criados neste laboratório

    ```
    Get-AzResourceGroup | Where-Object {$_.ResourceGroupName -like "$resourceGroupNamePrefix*"} | Remove-AzResourceGroup -Force  
    ```

1. Feche o painel do Cloud Shell.

> **Resultado**: Depois de concluir este exercício, você terá removido os recursos usados neste laboratório.
