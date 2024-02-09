---
lab:
  title: 01b – Implementar o clustering do Windows em VMs do Azure
  module: Module 01 - Explore the foundations of IaaS for SAP on Azure
---

# Módulo 1 do AZ 120: Explorar os conceitos básicos do IaaS para SAP no Azure
# Laboratório 1b: Implementar o clustering do Windows em VMs do Azure

Tempo estimado: 120 minutos

Todas as tarefas neste laboratório são executadas no portal do Azure (incluindo a sessão do PowerShell do Cloud Shell)  

   > **Observação**: Quando não estiver usando o Cloud Shell, a máquina virtual do laboratório deve ter o módulo do Az PowerShell instalado [**https://docs.microsoft.com/en-us/powershell/azure/install-az-ps-msi**](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps-msi).

Arquivos de laboratório: nenhum

## Cenário
  
Em preparação para a implantação do SAP NetWeaver no Azure, com o SQL Server como o sistema de gerenciamento de banco de dados, a Adatum Corporation deseja explorar o processo de implementação de clustering em VMs do Azure que executam o Windows Server 2022.

## Objetivos
  
Depois de realizar este laboratório, você será capaz de:

-   Provisionar os recursos de computação do Azure necessários para dar suporte a implantações SAP NetWeaver altamente disponíveis.

-   Configure o sistema operacional de VMs do Azure executando o Windows Server 2022 para dar suporte a uma implantação do SAP NetWeaver altamente disponível.

-   Provisionar os recursos de rede do Azure necessários para dar suporte a implantações SAP NetWeaver altamente disponíveis.

## Requisitos

-   Uma assinatura do Microsoft Azure com o número suficiente de VCPUs DSv2 e Dsv3 disponíveis (uma VM Standard_DS1_v2 com 1 vCPU e quatro VMs Standard_D4s_v3 com 4 vCPUs cada) na região do Azure que você pretende usar para este laboratório

-   Um computador de laboratório com um navegador da Web compatível com o Azure Cloud Shell e acesso ao Azure

> **Observação**: Certifique-se de que a região do Azure escolhida para implantação dos seus recursos dê suporte a zonas de disponibilidade. Para obter a lista dessas regiões, confira (https://docs.microsoft.com/en-us/azure/availability-zones/az-overview). Considere usar o **Leste dos EUA** ou o **Leste dos EUA 2**.

## Exercício 1: provisione os recursos de computação do Azure necessários para dar suporte a implantações SAP NetWeaver altamente disponíveis

Duração: 50 minutos

Neste exercício, você implantará os componentes de computação de infraestrutura do Azure necessários para configurar o clustering de failover em VMs do Azure que executam o Windows Server 2022. Isso envolverá a implantação de um par de controladores de domínio do Active Directory, seguido por um par de VMs do Azure que executam o Windows Server 2022. Cada par de VMs será colocado em zonas de disponibilidade separadas dentro da mesma rede virtual. Para automatizar a implantação de controladores de domínio, você usará um modelo de início rápido do Azure Resource Manager disponível em <https://aka.ms/az120-1bdeploy>

### Tarefa 1: Implantar um par de VMs do Azure executando controladores de domínio do Active Directory altamente disponíveis usando um modelo Bicep

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
    $rgName = 'az12001b-ad-RG'
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
    $deploymentName = 'az1201b-' + $(Get-Date -Format 'yyyy-MM-dd-hh-mm')
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

    - Navegue até a folha **az1201b-ad-RG** e, no menu de navegação vertical no lado esquerdo, na seção **Configurações**, selecione **Implantações**.

    - Na folha **az1201b-ad-RG \| Implantações**, selecione a implantação que começa com o prefixo **az1201b** e, na folha de implantação, selecione **Reimplantar**.

    - Na folha **Implantação personalizada**, na caixa de texto **Senha do administrador**, insira a mesma senha usada durante a implantação original, selecione **Examinar + criar** e, em seguida, selecione **Criar**.

    - Não aguarde a conclusão da reimplantação, prossiga para a próxima tarefa. A reimplantação deve levar cerca de 3 minutos.

### Tarefa 2: Implantar um par de VMs do Azure executando o Windows Server 2022 em diferentes zonas de disponibilidade

1. No computador de laboratório, no portal do Azure, navegue até a folha **Máquinas virtuais**, clique em **+ Criar** e, no menu suspenso, selecione a **máquina virtual do Azure**.

1. Na folha **Criar uma máquina virtual**, inicie o provisionamento de um Datacenter do Windows Server 2022 **: Edição Azure – VM do Azure Gen2** com as seguintes configurações (deixe todas as outras com os valores padrão):
    
    | Configuração | Valor |
    |   --    |  --   |
    | **Assinatura** | *o nome da sua assinatura do Azure*  |
    | **Grupo de recursos** | *o nome de um novo grupo de recursos* **az12001b-cl-RG** |
    | **Nome da máquina virtual** | **az12001b-cl-vm0** |
    | **Região** | *a mesma região do Azure em que você implantou as VMs do Azure na tarefa anterior* |
    | **Opções de disponibilidade** | **Zona de disponibilidade** |
    | **Zona de disponibilidade** | **Zona 1** |
    | **Imagem** | *selecione* **Datacenter do Windows Server 2022: Edição do Azure – Gen2** |
    | **Tamanho** | **D4s v3 Standard** |
    | **Nome de usuário** | *o mesmo nome de usuário especificado ao implantar o modelo do Bicep anteriormente neste exercício* |
    | **Senha** | *a mesma senha que você especificou ao implantar o modelo do Bicep anteriormente neste exercício* |
    | **Portas de entrada públicas** | **Permitir portas selecionadas** |
    | **Portas de entrada selecionadas** | **RDP (3389)** |
    | **Deseja usar uma licença existente do Windows Server?** | **Não** |
    | **Tipo de disco de SO** | **SSD Premium** |
    | **Rede virtual** | **adVNET** |
    | **Nome da sub-rede** | *uma nova sub-rede chamada* **clSubnet** |
    | **Intervalo de endereços da sub-rede** | **10.0.1.0/24** |
    | **Endereço IP público** | *um novo endereço IP chamado* **az12001b-cl-vm0-ip** |
    | **Grupo de segurança de rede da NIC** | **Basic**  |
    | **Habilitar a rede acelerada** | **Ativado** |
    | **Opções de balanceamento de carga** | **Nenhuma** |
    | **Habilitar identidade gerenciada atribuída pelo sistema** | **Desativado** |
    | **Fazer logon com o Azure AD** | **Desativado** |
    | **Habilitar o desligamento automático** | **Desativado** |
    | **Opção de orquestração de patch** | **Atualizações manuais** |
    | **Diagnóstico de inicialização** | **Desabilitar** |
    | **Extensões** | *Nenhuma* |
    | **Marcas** | *Nenhuma* |

1. Não aguarde a conclusão do provisionamento, continue para a próxima etapa.

1. Provisione outro **Datacenter do Windows Server 2022: Edição Azure – VM do Azure Gen2** com as seguintes configurações:
     
    | Configuração | Valor |
    |   --    |  --   |
    | **Assinatura** | *o nome da sua assinatura do Azure*  |
    | **Grupo de recursos** | *o nome do grupo de recursos usado ao implantar o primeiro **Datacenter do Windows Server 2022: Edição do Azure – VM do Azure Gen2** nesta tarefa* |
    | **Nome da máquina virtual** | **az12001b-cl-vm1** |
    | **Região** | *a mesma região do Azure em que você implantou o primeiro Datacenter do Windows Server 2022 **: Edição do Azure – VM do Azure Gen2** nesta tarefa* |
    | **Opções de disponibilidade** | **Zona de disponibilidade** |
    | **Zona de disponibilidade** | **Zona 2** |
    | **Imagem** | *selecione* **Datacenter do Windows Server 2022: Edição do Azure – Gen2** |
    | **Tamanho** | **D4s v3 Standard** |
    | **Nome de usuário** | *o mesmo nome de usuário especificado ao implantar o modelo do Bicep anteriormente neste exercício* |
    | **Senha** | *a mesma senha que você especificou ao implantar o modelo do Bicep anteriormente neste exercício* |
    | **Portas de entrada públicas** | **Permitir portas selecionadas** |
    | **Portas de entrada selecionadas** | **RDP (3389)** |
    | **Deseja usar uma licença existente do Windows Server?** | **Não** |
    | **Tipo de disco de SO** | **SSD Premium** |
    | **Rede virtual** | **adVNET** |
    | **Nome da sub-rede** | **clSubnet** |
    | **Endereço IP público** | *um novo endereço IP chamado* **az12001b-cl-vm1-ip** |
    | **Grupo de segurança de rede da NIC** | **Basic**  |
    | **Habilitar a rede acelerada** | **Ativado** |
    | **Opções de balanceamento de carga** | **Nenhuma** |
    | **Fazer logon com o Azure AD** | **Desativado** |
    | **Habilitar o desligamento automático** | **Desativado** |
    | **Opção de orquestração de patch** | **Atualizações manuais** |
    | **Diagnóstico de inicialização** | **Desabilitar** |
    | **Extensões** | *Nenhuma* |
    | **Marcas** | *Nenhuma* |

1. Aguarde a conclusão do provisionamento. Isso deve levar alguns minutos.

### Tarefa 3: Criar e configurar discos de VMs do Azure

1. No portal do Azure, inicie uma sessão do PowerShell no Cloud Shell. 

1. No painel do Cloud Shell, execute o seguinte comando para definir o valor da variável `$resourceGroupName` como o nome do grupo de recursos que contém os recursos provisionados na tarefa anterior:

    ```
    $resourceGroupName = 'az12001b-cl-RG'
    ```

1. No painel do Cloud Shell, execute o seguinte comando para criar o primeiro conjunto de 4 discos gerenciados que você anexará à primeira VM do Azure implantada na tarefa anterior:

    ```
    $location = (Get-AzResourceGroup -Name $resourceGroupName).Location
    
    $zone = (Get-AzVM -ResourceGroupName $resourceGroupName -Name 'az12001b-cl-vm0').Zones

    $diskConfig = New-AzDiskConfig -Location $location -DiskSizeGB 128 -AccountType Premium_LRS -OsType Windows -CreateOption Empty -Zone $zone

    for ($i=0;$i -lt 4;$i++) {New-AzDisk -ResourceGroupName $resourceGroupName -DiskName az12001b-cl-vm0-DataDisk$i -Disk $diskConfig}
    ```

1. No painel do Cloud Shell, execute o seguinte comando para criar o segundo conjunto de 4 discos gerenciados que você anexará à segunda VM do Azure implantada na tarefa anterior:

    ```
    $zone = (Get-AzVM -ResourceGroupName $resourceGroupName -Name 'az12001b-cl-vm1').Zones
    
    $diskConfig = New-AzDiskConfig -Location $location -DiskSizeGB 128 -AccountType Premium_LRS -OsType Windows -CreateOption Empty -Zone $zone
        
    for ($i=0;$i -lt 4;$i++) {New-AzDisk -ResourceGroupName $resourceGroupName -DiskName az12001b-cl-vm1-DataDisk$i -Disk $diskConfig}
    ```

1. No portal do Azure, navegue até a folha da primeira VM do Azure provisionada na tarefa anterior (**az12001b-cl-vm0**).

1. Na folha **az12001b-cl-vm0**, navegue até a folha **az12001b-cl-vm0 – discos**.

1. Na folha **az12001b-cl-vm0 – discos**, anexe discos de dados com as seguintes configurações ao az12001b-cl-vm0:
   
   | Configuração | Valor |
   |   --    |  --   |
   | **LUN** | **0** |
   | **Nome do disco** | **az12001b-cl-vm0-DataDisk0** |
   | **Grupo de recursos** | *o nome do grupo de recursos que você usou ao implantar o par de VMs do Azure do **Datacenter do Windows Server 2022** na tarefa anterior* |
   | **CACHE DE HOST** | **Somente leitura** |

1. Repita a etapa anterior para anexar os três discos restantes com o prefixo **az12001b-cl-vm0-DataDisk** (para um total de 4). Atribua o número LUN correspondente ao último caractere do nome do disco. Para o último disco (LUN **3**), defina o CACHE DE HOST como **Nenhum**.

1. Salve suas alterações. 

1. No portal do Azure, navegue até a folha da segunda VM do Azure provisionada na tarefa anterior (**az12001b-cl-vm1**).

1. Na folha **az12001b-cl-vm1**, navegue até a folha **az12001b-cl-vm1 – discos**.

1. Na folha **az12001b-cl-vm1 – discos**, anexe discos de dados com as seguintes configurações ao az12001b-cl-vm1:
     
   | Configuração | Valor |
   |   --    |  --   |
   | **LUN** | **0** |
   | **Nome do disco** | **az12001b-cl-vm1-DataDisk0** |
   | **Grupo de recursos** | *o nome do grupo de recursos que você usou ao implantar o par de VMs do Azure do **Datacenter do Windows Server 2022** na tarefa anterior* |
   | **CACHE DE HOST** | **Somente leitura** |

1. Repita a etapa anterior para anexar os três discos restantes com o prefixo **az12001b-cl-vm1-DataDisk** (para um total de 4). Atribua o número LUN correspondente ao último caractere do nome do disco. Para o último disco (LUN **3**), defina o CACHE DE HOST como **Nenhum**.

1. Salve suas alterações. 

> **Resultado**: Depois de concluir este exercício, você terá provisionado os recursos de computação do Azure necessários para dar suporte a implantações do SAP NetWeaver altamente disponíveis.


## Exercício 2: Configurar o sistema operacional de VMs do Azure executando o Datacenter do Windows Server 2022 para dar suporte a uma instalação do SAP NetWeaver altamente disponível

Duração: 40 minutos

### Tarefa 1: Ingresse as VMs do Datacenter do Windows Server 2022 no domínio do Active Directory.

   > **Observação**: Antes de iniciar essa tarefa, certifique-se de que a implantação do modelo iniciada na última tarefa do exercício anterior tenha sido concluída com êxito. 

1. No Portal do Azure, navegue até a folha da rede virtual **adVNET**, que foi provisionada automaticamente no primeiro exercício deste laboratório.

1. Exiba a folha **adVNET – servidores DNS** e observe que a rede virtual está configurada com os endereços IP privados atribuídos aos controladores de domínio implantados no primeiro exercício deste laboratório como seus servidores DNS.

1. No portal do Azure, inicie uma sessão do PowerShell no Cloud Shell. 

1. No painel do Cloud Shell, execute o seguinte comando para definir o valor da variável `$resourceGroupName` como o nome do grupo de recursos que contém o par de VMs do Azure do **Datacenter do Windows Server 2022: Edição Azure – Gen2** provisionado no exercício anterior:

    ```
    $resourceGroupName = 'az12001b-cl-RG'
    ```

1. No painel do Cloud Shell, execute o seguinte comando para ingressar nas VMs do Azure do Windows Server 2022 implantadas na segunda tarefa do exercício anterior no domínio **adatum.com** do Active Directory (substitua os espaços reservados `<username>` e `<password>` pelo nome e a senha da conta de usuário administrativo especificada ao implantar o modelo Bicep no primeiro exercício deste laboratório):

    ```
    $location = (Get-AzureRmResourceGroup -Name $resourceGroupName).Location

    $settingString = '{"Name": "adatum.com", "User": "adatum.com\\<username>", "Restart": "true", "Options": "3"}'

    $protectedSettingString = '{"Password": "<password>"}'

    $vmNames = @('az12001b-cl-vm0','az12001b-cl-vm1')

    foreach ($vmName in $vmNames) { Set-AzVMExtension -ResourceGroupName $resourceGroupName -ExtensionType 'JsonADDomainExtension' -Name 'joindomain' -Publisher "Microsoft.Compute" -TypeHandlerVersion "1.0" -Vmname $vmName -Location $location -SettingString $settingString -ProtectedSettingString $protectedSettingString }
    ```

1. Aguarde até que o script seja concluído antes de prosseguir para a próxima tarefa.


### Tarefa 2: Configure o armazenamento em VMs do Azure que executam o Windows Server 2022 para dar suporte a uma instalação do SAP NetWeaver altamente disponível.

1. No Portal do Azure, navegue até a folha da máquina virtual **az12001b-cl-vm0**, que você provisionou no primeiro exercício deste laboratório.

1. Na folha **az12001b-cl-vm0**, conecte-se ao sistema operacional convidado da máquina virtual usando a Área de Trabalho Remota. Quando solicitado a autenticar-se, forneça as credenciais da conta de usuário administrativo especificada ao implantar o modelo Bicep no primeiro exercício deste laboratório. 

    > **Observação**: Certifique-se de entrar usando a conta de domínio **ADATUM**, em vez da conta no nível do sistema operacional (ou seja, certifique-se de que o nome de usuário seja precedido pelo prefixo **ADATUM\\**.

1. Na sessão do RDP para az12001b-cl-vm0, no Gerenciador do Servidor, navegue até o nó **Serviços de arquivo e armazenamento** -> **Servidores**. 

1. Navegue até o a exibição de **pools de armazenamento** e verifique se você vê todos os discos anexados à VM do Azure no exercício anterior.

1. Use o **Assistente para novos pools de armazenamento** para criar um novo pool de armazenamento com as seguintes configurações:
     
    | Configuração | Valor |
    |   --    |  --   |
    | **Nome** | **Pool de armazenamento de dados** |
    | **Discos físicos** | *selecione os 3 discos com números de disco correspondentes aos três primeiros números LUN (0-2) e defina sua alocação como* **Automática** |

    > **Observação**: Use a entrada na coluna **Chassis** para identificar o número **LUN**.

1. Use o **Assistente para Novo Disco Virtual** para criar um novo disco virtual com as seguintes configurações:
     
    | Configuração | Valor |
    |   --    |  --   |
    | **Nome do disco virtual** | **Disco Virtual de Dados** |
    | **Layout de armazenamento** | **Simples** |
    | **Provisionamento** | **Fixo** |
    | **Tamanho** | **Tamanho máximo** |

1. Use o **Assistente para Novo Volume** para criar um novo volume com as seguintes configurações:
    
    | Configuração | Valor |
    |   --    |  --   |
    | **Servidor e disco** | *aceite os valores padrão* |
    | **Tamanho** | *aceite os valores padrão* |
    | **Letra da unidade** | **M** |
    | **Sistema de Arquivos** | **ReFS** |
    | **Tamanho da unidade de alocação** | **Default** |
    | **Rótulo de volume** | **Dados** |

1. De volta ao modo de exibição **Pools de armazenamento**, use o **Assistente para Novos Pools de Armazenamento** para criar um novo pool de armazenamento com as seguintes configurações:
    
    | Configuração | Valor |
    |   --    |  --   |
    | **Nome** | **Pool de armazenamento de logs** |
    | **Discos físicos** | *selecione o último de 4 discos e defina sua alocação como* **Automática** |

1. Use o **Assistente para Novo Disco Virtual** para criar um novo disco virtual com as seguintes configurações:
    
    | Configuração | Valor |
    |   --    |  --   |
    | **Nome do disco virtual** | **Disco Virtual de Logs** |
    | **Layout de armazenamento** | **Simples** |
    | **Provisionamento** | **Fixo** |
    | **Tamanho** | **Tamanho máximo** |

1. Use o **Assistente para Novo Volume** para criar um novo volume com as seguintes configurações:
    
    | Configuração | Valor |
    |   --    |  --   |
    | **Servidor e disco** | *aceite os valores padrão* |
    | **Tamanho** | *aceite os valores padrão* |
    | **Letra da unidade** | **L** |
    | **Sistema de Arquivos** | **ReFS** |
    | **Tamanho da unidade de alocação** | **Default** |
    | **Rótulo de volume** | **Log** |

1. Repita a etapa anterior nesta tarefa para configurar o armazenamento em az12001b-cl-vm1.

### Tarefa 3: Prepare-se para a configuração do clustering de failover em VMs do Azure que executam o Windows Server 2022 para dar suporte a uma instalação do SAP NetWeaver altamente disponível.

1. Na sessão do RDP para az12001b-cl-vm0, inicie uma sessão do ISE do Windows PowerShell e instale os recursos de clustering de failover e ferramentas administrativas remotas no az12001b-cl-vm0 e az12001b-cl-vm1 executando o seguinte:

    ```
    $nodes = @('az12001b-cl-vm1', 'az12001b-cl-vm0')

    Invoke-Command $nodes {Install-WindowsFeature Failover-Clustering -IncludeAllSubFeature -IncludeManagementTools} 

    Invoke-Command $nodes {Install-WindowsFeature RSAT -IncludeAllSubFeature -Restart} 
    ```

    > **Observação**: Isso resultará na reinicialização do sistema operacional convidado de ambas as VMs do Azure.

1. No computador do laboratório, no portal do Azure, clique em **+ Criar um recurso**.

1. Na folha **Novo**, inicie a criação de uma nova **Conta de armazenamento** com as seguintes configurações:
    
    | Configuração | Valor |
    |   --    |  --   |
    | **Assinatura** | *o nome da sua assinatura do Azure* |
    | **Grupo de recursos** | *o nome do grupo de recursos que contém o par de VMs do Azure do **Datacenter do Windows Server 2022** provisionadas por você no exercício anterior* |
    | **Nome da conta de armazenamento** | *qualquer nome exclusivo que consista entre 3 e 24 letras e dígitos* |
    | **Localidade** | *a mesma região do Azure em que você implantou as VMs do Azure no exercício anterior* |
    | **Desempenho** | **Standard** |
    | **Redundância** | **Armazenamento com redundância local (LRS)** |
    | **Método de conectividade** | **Ponto de extremidade público (todas as redes)** |
    | **Exigir transferência segura para operações de API REST** | **Enabled** |
    | **Compartilhamentos de arquivos grandes** | **Desabilitado** |
    | **Exclusão temporária para blobs, contêineres e arquivos** | **Desabilitado** |
    | **Namespace hierárquico** | **Desabilitado** |

### Tarefa 4: Configure o clustering de failover em VMs do Azure que executam o Windows Server 2022 para dar suporte a uma instalação do SAP NetWeaver altamente disponível.

1. No Portal do Azure, navegue até a folha da máquina virtual **az12001b-cl-vm0**, que você provisionou no primeiro exercício deste laboratório.

1. Na folha **az12001b-cl-vm0**, conecte-se ao sistema operacional convidado da máquina virtual usando a Área de Trabalho Remota. Quando solicitado a autenticar-se, forneça as credenciais da conta de usuário administrativo especificada ao implantar o modelo Bicep no primeiro exercício deste laboratório.

1. Na sessão do RDP para az12001b-cl-vm0, no menu **Ferramentas** no Gerenciador do Servidor, inicie o **Centro Administrativo do Active Directory**.

1. No Centro Administrativo do Active Directory, crie uma nova unidade organizacional chamada **Clusters** na raiz do domínio adatum.com.

1. No Centro Administrativo do Active Directory, mova as contas de computador do **az12001b-cl-vm0** e **az12001b-cl-vm1** do contêiner **Computadores** para a unidade organizacional **Clusters**. 

1. Na sessão do RDP para az12001b-cl-vm0, inicie uma sessão do ISE do Windows PowerShell e crie um novo cluster executando o seguinte:

    ```
    $nodes = @('az12001b-cl-vm0','az12001b-cl-vm1')

    New-Cluster -Name az12001b-cl-cl0 -Node $nodes -NoStorage -StaticAddress 10.0.1.6
    ```

1. Na sessão do RDP para az12001b-cl-vm0, alterne para o console do **Centro Administrativo do Active Directory**.

1. No Centro Administrativo do Active Directory, navegue até a unidade organizacional **Clusters** e exiba a janela **Propriedades**. 

1. Na unidade organizacional **Clusters**, janela **Propriedades**, navegue até a seção **Extensões** e exiba a guia **Segurança**. 

1. Na guia **Segurança**, clique no botão **Avançado** para abrir a janela **Configurações avançadas de segurança para clusters**. 

1. Na guia **Permissões** da janela **Configurações avançadas de segurança para clusters**, clique em **Adicionar**.

1. Na janela **Entrada de permissão para clusters**, clique em **Selecionar entidade de segurança**

1. Na caixa de diálogo **Selecionar usuário, conta de serviço ou grupo**, clique em **Tipos de objeto**, habilite a caixa de seleção ao lado da entrada **Computadores** e clique em **OK**. 

1. Na caixa de diálogo **Selecionar usuário, computador, conta de serviço ou grupo**, **Insira o nome do objeto a ser selecionado**, digite **az12001b-cl-cl0** e clique em **OK**.

1. Na janela **Entrada de permissão para clusters**, certifique-se de que **Permitir** apareça na lista suspensa **Tipo**. Em seguida, na lista suspensa **Aplica-se a**, selecione **Este objeto e todos os objetos descendentes**. Na lista **Permissões**, selecione as caixas de seleção **Criar objetos do computador** e **Excluir objetos do computador** e clique em **OK** duas vezes.

1. Na sessão do ISE do Windows PowerShell, instale o módulo Az PowerShell executando o seguinte:

    ```
    Install-PackageProvider -Name NuGet -Force

    Install-Module -Name Az -Force
    ```

1. Na sessão do ISE do Windows PowerShell, autentique-se usando suas credenciais do Azure AD executando o seguinte:

    ```
    Add-AzAccount
    ```

    > **Observação**: Quando solicitado, entre com a conta Microsoft corporativa ou de estudante ou pessoal com a função de proprietário ou colaborador para a assinatura do Azure que você está usando para este laboratório.

1. Na sessão do ISE do Windows PowerShell, defina o quorum da testemunha de nuvem do novo cluster executando o seguinte:

    ```
    $resourceGroupName = 'az12001b-cl-RG'

    $cwStorageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName

    $cwStorageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $cwStorageAccountName).Value[0]

    Set-ClusterQuorum -CloudWitness -AccountName $cwStorageAccountName -AccessKey $cwStorageAccountKey
    ```

1. Para verificar a configuração resultante, na sessão do RDP para az12001b-cl-vm0, no menu **Ferramentas** no Gerenciador do Servidor, inicie o **Gerenciador de cluster de failover**.

1. No console do **Gerenciador de cluster de failover**, examine a configuração do cluster **az12001b-cl-cl0**, incluindo os seus nós, bem como as configurações de testemunha e de rede. Observe que o cluster não tem nenhum armazenamento compartilhado.

1. Encerre a sessão do RDP para az12001b-cl-vm0.

> **Resultado**: Depois de concluir este exercício, você terá configurado o sistema operacional de VMs do Azure que executam o Windows Server 2022 para dar suporte a uma instalação do SAP NetWeaver altamente disponível


## Exercício 3: provisione os recursos de rede do Azure necessários para dar suporte a implantações SAP NetWeaver altamente disponíveis

Duração: 30 minutos

Neste exercício, você implementará balanceadores de carga do Azure Load Balancer para acomodar instalações clusterizadas do SAP NetWeaver.

### Tarefa 1: Configure as VMs do Azure para facilitar a instalação do balanceamento de carga.

   > **Observação**: Como você configurará um par do Azure Load Balancer de SKU Stardard, primeiro será necessário remover os endereços IP públicos associados aos adaptadores de rede de duas VMs do Azure que servirão como o pool de back-end com balanceamento de carga.

1. No computador de laboratório, no portal do Azure, navegue até a folha da VM do Azure **az12001b-cl-vm0**. 

1. Na folha **az12001b-cl-vm0**, navegue até a folha do endereço IP público **az12001b-cl-vm0-ip** associado ao adaptador de rede.

1. Na folha **az12001b-cl-vm0-ip**, primeiro desassocie o endereço IP público do adaptador de rede e exclua-o.

1. No portal do Azure, navegue até a folha da VM do Azure **az12001b-cl-vm1**. 

1. Na folha **az12001b-cl-vm1**, navegue até a folha do endereço IP público **az12001b-cl-vm1-ip** associado ao adaptador de rede.

1. Na folha **az12001b-cl-vm1-ip**, primeiro desassocie o endereço IP público do adaptador de rede e exclua-o.

1. No portal do Azure, navegue até a folha da VM do Azure **az12001a-vm0**.

1. Na folha **az12001a-vm0**, navegue até a folha **Rede**. 

1. Na folha **az12001a-vm0 – rede**, navegue até a interface de rede da az12001a-vm0. 

1. Na folha da interface de rede da az12001a-vm0, navegue até a folha de configurações de IP e, a partir daí, exiba sua folha **ipconfig1**.

1. Na folha **ipconfig1**, defina a atribuição de endereço IP privado como **Estática** e salve a alteração.

1. No portal do Azure, navegue até a folha da VM do Azure **az12001a-vm1**.

1. Na folha **az12001a-vm1**, navegue até a folha **Rede**. 

1. Na folha **az12001a-vm1 – rede**, navegue até a interface de rede da az12001a-vm1. 

1. Na folha da interface de rede da az12001a-vm1, navegue até a sua folha de configurações de IP e, a partir daí, exiba sua folha **ipconfig1**.

1. Na folha **ipconfig1**, defina a atribuição de endereço IP privado como **Estática** e salve a alteração.

### Tarefa 2: Criar e configurar a manipulação do tráfego de entrada de balanceadores de carga do Azure Load Balancer

1. No portal do Azure, clique em **Criar um recurso**.

1. Na folha **Novo**, inicie a criação de um novo Azure Load Balancer com as seguintes configurações:
    
    | Configuração | Valor |
    |   --    |  --   |
    | **Assinatura** | *o nome da sua assinatura do Azure* |
    | **Grupo de recursos** | *o nome do grupo de recursos que contém o par de VMs do Azure do **Datacenter do Windows Server 2022** que você provisionou no primeiro exercício deste laboratório* |
    | **Nome** | **az12001b-cl-lb0** |
    | **Região** | a mesma região do Azure em que você implantou VMs do Azure no primeiro exercício deste laboratório* |
    | **SKU** | **Standard** |
    | **Tipo** | **Interna** |
    | **Nome do IP de front-end** | **frontend-ip1** |
    | **Rede virtual** | **adVNET** |
    | **Sub-rede** | **clSubnet** |
    | **Atribuição de endereço IP** | **Estático** |
    | **Endereço IP** | **10.0.1.240** |
    | **Zona de disponibilidade** | **Com redundância de zona** |

1. Aguarde até que o balanceador de carga seja provisionado e navegue até a sua folha no portal do Azure.

1. Na folha **az12001b-cl-lb0**, adicione um pool de back-end com as seguintes configurações:
    
    | Configuração | Valor |
    |   --    |  --   |
    | **Nome** | **az12001b-cl-lb0-bepool** |
    | **Rede virtual** | **adVNET** |
    | **Configuração do pool de back-end** | **Endereço IP** |
    | **Endereço IP** | **10.0.1.4** Nome do recurso **az1201b-cl-vm0** |
    | **Endereço IP** | **10.0.1.5** Nome do recurso **az1201b-cl-vm1** |

1. Na folha **az12001b-cl-lb0**, adicione uma investigação de integridade com as seguintes configurações:
    
    | Configuração | Valor |
    |   --    |  --   |
    | **Nome** | **az12001b-cl-lb0-hprobe** |
    | **Protocolo** | **TCP** |
    | **Porta** | **59999** |
    | **Intervalo** | **5** *segundos* |
    | **Limite não íntegro** | **2** *falhas consecutivas* |

1. Na folha **az12001b-cl-lb0**, adicione uma regra de balanceamento de carga de rede com as seguintes configurações:
     
    | Configuração | Valor |
    |   --    |  --   |
    | **Nome** | **az12001b-cl-lb0-lbruletcp1433** |
    | **Versão IP** | **IPv4** |
    | **Endereço IP de front-end** | **10.0.1.240 (LoadBalancerFrontEnd)** |
    | **Portas de HA** | **Desabilitado** |
    | **Protocolo** | **TCP** |
    | **Porta** | **1433** |
    | **Porta de back-end** | **1433** |
    | **Pool de back-end** | **az12001b-cl-lb0-bepool (2 máquinas virtuais)** |
    | **Investigação de integridade** | **az12001b-cl-lb0-hprobe (TCP:59999)** |
    | **Persistência de sessão** | **Nenhuma** |
    | **Tempo limite de ociosidade (minutos)** | **4** |
    | **Redefinição de TCP** | **Desabilitado** |
    | **IP flutuante (retorno de servidor direto)** | **Enabled** |

### Tarefa 3: Criar e configurar balanceadores de carga do Azure Load Balancer que manipulam tráfego de saída

1. No Portal do Azure, inicie uma sessão do PowerShell no Cloud Shell. 

1. No painel do Cloud Shell, execute o seguinte comando para definir o valor da variável `$resourceGroupName` como o nome do grupo de recursos que contém o par de VMs do Azure do **Datacenter do Windows Server 2022** que você provisionou no primeiro exercício deste laboratório:

    ```
    $resourceGroupName = 'az12001b-cl-RG'
    ```

1. No painel do Cloud Shell, execute o seguinte comando para criar o endereço IP público a ser usado pelo segundo balanceador de carga:

    ```
    $location = (Get-AzResourceGroup -Name $resourceGroupName).Location

    $pipName = 'az12001b-cl-lb0-pip'

    az network public-ip create --resource-group $resourceGroupName --name $pipName --sku Standard --location $location
    ```

1. No painel do Cloud Shell, execute o seguinte comando para criar o segundo balanceador de carga:

    ```
    $lbName = 'az12001b-cl-lb1'

    $lbFeName = 'az12001b-cl-lb1-fe'

    $lbBePoolName = 'az12001b-cl-lb1-bepool'
   
    $pip = Get-AzPublicIpAddress -ResourceGroupName $resourceGroupName -Name $pipName

    $feIpconfiguration = New-AzLoadBalancerFrontendIpConfig -Name $lbFeName -PublicIpAddress $pip

    $bePoolConfiguration = New-AzLoadBalancerBackendAddressPoolConfig -Name $lbBePoolName

    New-AzLoadBalancer -ResourceGroupName $resourceGroupName -Location $location -Name $lbName -Sku Standard -BackendAddressPool $bePoolConfiguration -FrontendIpConfiguration $feIpconfiguration
    ```

1. Feche o painel do Cloud Shell.

1. No portal do Azure, navegue até a folha que exibe as propriedades do Azure Load Balancer **az12001b-cl-lb1**.

1. Na folha **az12001b-cl-lb1**, clique em **Pools de back-end**.

1. Na folha **az12001b-cl-lb1 – pools de back-end**, clique em **az12001b-cl-lb1-bepool**.

1. Na folha **az12001b-cl-lb1-bepool**, especifique as seguintes configurações e clique em **Salvar**:
    
    | Configuração | Valor |
    |   --    |  --   |
    | **Rede virtual** | **adVNET (4 VM)** |
    | **Máquina virtual** | **az12001b-cl-vm0**  ENDEREÇO IP: **ipconfig1** |
    | **Máquina virtual** | **az12001b-cl-vm1**  ENDEREÇO IP: **ipconfig1** |

1. Na folha **az12001b-cl-lb1**, clique em **Investigações de integridade**.

1. Na folha **az12001b-cl-lb1 – investigações de integridade**, adicione uma investigação de integridade com as seguintes configurações:
    
    | Configuração | Valor |
    |   --    |  --   |
    | **Nome** | **az12001b-cl-lb1-hprobe** |
    | **Protocolo** | **TCP** |
    | **Porta** | **80** |
    | **Intervalo** | **5** *segundos* |
    | **Limite não íntegro** | **2** *falhas consecutivas* |

1. Na folha **az12001b-cl-lb1**, clique em **Regras de balanceamento de carga**.

1. Na folha **az12001b-cl-lb1 – regras de balanceamento de carga**, adicione uma regra de balanceamento de carga de rede com as seguintes configurações:
    
    | Configuração | Valor |
    |   --    |  --   |
    | **Nome** | **az12001b-cl-lb1-lbharule** |
    | **Versão IP** | **IPv4** |
    | **Endereço IP de front-end** | *aceitar o valor padrão* |
    | **Portas de HA** | **Desabilitado** |
    | **Protocolo** | **TCP** |
    | **Porta** | **80** |
    | **Porta de back-end** | **80** |
    | **Pool de back-end** | **az12001b-cl-lb1-bepool (2 máquinas virtuais)** |
    | **Investigação de integridade** | **az12001b-cl-lb1-hprobe (TCP:80)** |
    | **Persistência de sessão** | **Nenhuma** |
    | **Tempo limite de ociosidade (minutos)** | **4** |
    | **Redefinição de TCP** | **Desabilitado** |
    | **IP flutuante (retorno de servidor direto)** | **Desabilitado** |

### Tarefa 4: Implantar um jump host

   > **Observação**: Como duas VMs clusterizados do Azure não são mais diretamente acessíveis da Internet, você implantará uma VM do Azure executando o Datacenter do Windows Server 2022 que servirá como um jump host. 

1. No computador de laboratório, no portal do Azure, navegue até a folha **Máquinas virtuais**, clique em **+ Criar** e, no menu suspenso, selecione a **máquina virtual do Azure**.

1. Na folha **Criar uma máquina virtual**, inicie o provisionamento de um Datacenter do Windows Server 2022 **: Edição Azure – VM do Azure Gen2** com as seguintes configurações:
     
    | Configuração | Valor |
    |   --    |  --   |
    | **Assinatura** | *o nome da sua assinatura do Azure*  |
    | **Grupo de recursos** | *o nome do grupo de recursos que contém o par de VMs do Azure do **Datacenter do Windows Server 2022: Edição Azure – Gen2** provisionadas no primeiro exercício deste laboratório* |
    | **Nome da máquina virtual** | **az12001b-vm2** |
    | **Região** | *a mesma região do Azure em que você implantou as VMs do Azure no primeiro exercício deste laboratório* |
    | **Opções de disponibilidade** | **Nenhuma redundância de infraestrutura necessária** |
    | **Imagem** | *selecione* **Datacenter do Windows Server 2022: Edição do Azure – Gen2** |
    | **Tamanho** | **DS1 v2 Standard*** ou similar* |
    | **Nome de usuário** | *o mesmo nome de usuário especificado ao implantar o modelo Bicep no primeiro exercício deste laboratório* |
    | **Senha** | *a mesma senha que você especificou ao implantar o modelo Bicep no primeiro exercício deste laboratório* |
    | **Portas de entrada públicas** | **Permitir portas selecionadas** |
    | **Portas de entrada selecionadas** | **RDP (3389)** |
    | **Deseja usar uma licença existente do Windows Server?** | **Não** |
    | **Tipo de disco de SO** | **HDD Standard** |
    | **Rede virtual** | **adVNET** |
    | **Nome da sub-rede** | *uma nova sub-rede chamada* **bastionSubnet** |
    | **Intervalo de endereços da sub-rede** | **10.0.255.0/24** |
    | **Endereço IP público** | *um novo endereço IP chamado* **az12001b-vm2-ip** |
    | **Grupo de segurança de rede da NIC** | **Basic**  |
    | **Portas de entrada públicas** | **Permitir portas selecionadas** |
    | **Portas de entrada selecionadas** | **RDP (3389)** |
    | **Habilitar a rede acelerada** | **Desativado** |
    | **Opções de balanceamento de carga** | **Nenhuma** |
    | **Habilitar identidade gerenciada atribuída pelo sistema** | **Desativado** |
    | **Fazer logon com o Azure AD** | **Desativado** |
    | **Habilitar o desligamento automático** | **Desativado** |
    | **Diagnóstico de inicialização** | **Desabilitar** |
    | **Habilitar o diagnóstico de convidado do sistema operacional** | **Desativado** |
    | **Extensões** | *Nenhuma* |
    | **Marcas** | *Nenhuma* |

1. Aguarde a conclusão do provisionamento. Isso deve levar alguns minutos.

1. Conecte-se à VM do Azure recém-provisionada via RDP. 

1. Na sessão do RDP para az12001b-vm2, certifique-se de que você possa estabelecer a sessão do RDP para az12001b-cl-vm0 e az12001b-cl-vm1 por meio dos seus endereços IP privados (10.0.1.4 e 10.0.1.5, respectivamente). 

> **Resultado**: Depois de concluir este exercício, você terá provisionado os recursos de rede do Azure necessários para dar suporte a implantações do SAP NetWeaver altamente disponíveis

## Exercício 4: remova recursos de laboratório

Duração: 10 minutos

Neste exercício, você removerá os recursos provisionados neste laboratório.

#### Tarefa 1: Abrir o Cloud Shell

1. Na parte superior do portal, clique no ícone do **Cloud Shell** para abrir o painel do Cloud Shell e escolha o PowerShell como o shell.

1. No painel do Cloud Shell, execute o seguinte comando para definir o valor da variável `$resourceGroupName` como o nome do grupo de recursos que contém o par de VMs do Azure do **Datacenter do Windows Server 2022** que você provisionou no primeiro exercício deste laboratório:

    ```
    $resourceGroupNamePrefix = 'az12001b-'
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
