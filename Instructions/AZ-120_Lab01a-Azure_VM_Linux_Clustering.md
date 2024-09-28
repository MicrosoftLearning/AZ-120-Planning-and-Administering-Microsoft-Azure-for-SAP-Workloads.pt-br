---
lab:
  title: 02a – Implementar o clustering do Linux em VMs do Azure
  module: Module 02 - Explore the foundations of IaaS for SAP on Azure
---

# Módulo 2 do AZ 120: Explorar os conceitos básicos do IaaS para SAP no Azure
# Laboratório 2a: Implementar o clustering do Linux em VMs do Azure

Tempo estimado: 90 minutos

Todas as tarefas neste laboratório são executadas do portal do Azure (incluindo a sessão do Cloud Shell Bash)  

   > **Observação**: Quando não estiver usando o Cloud Shell, a máquina virtual do laboratório deve ter a CLI do Azure instalada [**https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows**](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows) e incluir um cliente SSH, por exemplo, PuTTY, disponível em [**https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html**](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html).

Arquivos de laboratório: nenhum

## Cenário
  
Na preparação para a implantação de SAP HANA no Azure, a Adatum Corporation deseja explorar o processo de implementação de clustering em VMs do Azure que executam a distribuição SUSE do Linux.

## Objetivos
  
Depois de realizar este laboratório, você será capaz de:

- Provisionar os recursos de computação do Azure necessários para dar suporte a implantações SAP HANA altamente disponíveis

- Configurar o sistema operacional de VMs do Azure que executam o Linux para dar suporte a uma instalação de SAP HANA altamente disponível

- Provisionar os recursos de rede do Azure necessários para dar suporte a implantações SAP HANA altamente disponíveis

## Requisitos

- Uma assinatura do Microsoft Azure com o número suficiente de vCPUs DSv3 (2 x 4) e vCPUs DSv2 (1 x 1)

- Um computador de laboratório com um navegador da Web compatível com o Azure Cloud Shell e acesso ao Azure

## Exercício 1: provisione os recursos de computação do Azure necessários para dar suporte a implantações SAP HANA altamente disponíveis

Duração: 30 minutos

Neste exercício, você implantará os componentes de computação de infraestrutura do Azure necessários para configurar o clustering do Linux. Isso envolverá a criação de um par de VMs do Azure que executam o Linux SUSE no mesmo conjunto de disponibilidade e o provisionamento do Azure Bastion.

### Tarefa 1: Implantar VMs do Azure executando o SUSE do Linux

1. No computador do laboratório, inicie um navegador da Web e navegue até o portal do Azure em **https://portal.azure.com**.

1. Se solicitado, entre com a conta Microsoft corporativa ou de estudante ou pessoal com a função de proprietário ou colaborador para a assinatura do Azure que você usará para este laboratório.

1. No portal do Azure, use a caixa de texto **Pesquisar recursos, serviços e documentos** na parte superior da página do portal do Azure para pesquisar e navegar até a folha **Grupos de posicionamento por proximidade** e, na folha **Grupos de posicionamento por proximidade**, selecione **+ criar**.

1. Na guia **Básico** da folha **Criar Grupos de Posicionamento por Proximidade**, especifique as seguintes configurações e clique em **Revusar + criar** (deixe as outras com os valores padrão):

    | Configuração | Valor |
    |   --    |  --   |
    | **Assinatura** | o nome da sua assinatura do Azure |
    | Seção **Grupo de recursos** | selecione **Criar novo**, insira **az12001a-RG** e clique em **OK** |
    | **Nome do grupo de posicionamento por proximidade** | **az12001a-ppg** |
    | **Região** | a região do Azure onde você tem cotas de vCPU suficientes |
    | **Tamanhos de VM** | **D4s v3 Standard** |

   > **Observação**: Considere o uso das regiões **Leste dos EUA** ou **Leste dos EUA 2** para a implantação de seus recursos.

1. Na guia **Examinar + criar** da folha **Criar grupos de posicionamento por proximidade**, selecione **Criar**.

   > **Observação**: Aguarde a conclusão do provisionamento. Isso deve levar menos de um minuto.

1. No portal do Azure, use a caixa de texto **Pesquisar recursos, serviços e documentos** na parte superior da página do portal do Azure para pesquisar e navegar até a folha **de máquinas virtuais** e, na folha **Máquinas virtuais**, selecione **+ Criar** e, no menu suspenso, selecione **máquina virtual do Azure**.

1. Na guia **Básico** da folha **Criar uma máquina virtual**, especifique as seguintes configurações e selecione **Avançar: Discos >** (deixe todas as outras configurações com o valor padrão):

    | Configuração | Valor |
    |   --    |  --   |
    | **Assinatura** | o nome da sua assinatura do Azure |
    | **Grupo de recursos** | o nome do grupo de recursos criado anteriormente nesta tarefa |
    | **Nome da máquina virtual** | **az12001a-vm0** |
    | **Região** | o nome da região do Azure que você escolheu ao criar o grupo de posicionamento por proximidade |
    | **Opções de disponibilidade** | **Conjunto de disponibilidade** |
    | **Conjunto de disponibilidade** | um novo conjunto de disponibilidade chamado **az12001a-avset** com dois domínios de falha e cinco domínios de atualização |
    | **Tipo de segurança** | **Standard** |
    | **Imagem** | **SUSE Enterprise Linux for SAP 15 SP5 – BYOS – x64 Gen 2** |
    | **Executar com o Desconto de Spot do Azure** | desabilitado |
    | **Tamanho** | **Standard_D4s_v3** |
    | **Tipo de autenticação** | **Senha** |
    | **Nome de usuário** | **student** |
    | **Senha** | qualquer senha complexa de sua escolha |
   
    > **Observação**: Lembre-se da senha especificada durante a implantação. Você precisará disso em uma etapa posterior deste laboratório.

    > **Observação**: para localizar a imagem, clique no link **Ver todas as imagens**, na folha **Selecionar uma imagem**, na caixa de texto de pesquisa, digite **SUSE Enterprise Linux** e, na lista de resultados, clique em **SUSE Enterprise Linux for SAP 15 SP5 - BYOS** e selecione **Geração 2**.

1. Na guia **Discos** da folha **Criar uma máquina virtual **, especifique as seguintes configurações e selecione **Avançar: Rede >** (deixe todas as outras configurações com o valor padrão):

    | Configuração | Valor |
    |   --    |  --   |
    | **Tipo de disco de SO** | **SSD Premium (armazenamento com redundância local)**  |
    | **Gerenciamento de chaves** | **Chave de criptografia gerenciada pela plataforma** |

1. Na guia **Rede** da folha **Criar uma máquina virtual** na seção **Adaptador de rede** abaixo da caixa de texto **Rede virtual**, clique em **Criar**. 
1. No painel **Criar rede virtual**, especifique as seguintes configurações e clique em **OK**:

    | Configuração | Valor |
    |   --    |  --   |
    | **Nome** | **az12001a-RG-vnet** |
    | **Espaço de endereço** | **192.168.0.0/20** |
    | **Nome da sub-rede** | **subnet-0** |
    | **Intervalo de endereços** | **192.168.0.0/24** |

1. De volta à guia **Rede** da folha **Criar uma máquina virtual**, especifique as seguintes configurações e clique em **Avançar: Gerenciamento >** (deixe todas as outras configurações com o valor padrão):

    | Configuração | Valor |
    |   --    |  --   |
    | **Endereço IP público** | **Nenhuma** |
    | **Grupo de segurança de rede da NIC** | **Avançado**  |
    | **Habilitar a rede acelerada** | Habilitado |
    | **Opções de balanceamento de carga** | **Nenhuma** |
    
    > **Observação**: esta implantação tem regras NSG pré-configuradas.

1. Na guia **Gerenciamento** da folha **Criar uma máquina virtual**, especifique as seguintes configurações e selecione **Avançar: Monitoramento >** (deixe todas as outras configurações com o valor padrão):
   
   | Configuração | Valor |
   |   --    |  --   |
   | **Habilitar plano básico gratuitamente** | desabilitado |
   | **Habilitar identidade gerenciada atribuída pelo sistema** | desabilitado |
   | **Habilitar o desligamento automático** | desabilitado  |

   > **Observação**: A **configuração do plano básico para gratuidade** não estará disponível se você já tiver habilitado o Microsoft Defender para Nuvem em sua assinatura.

1. Na guia **Monitoramento** da folha **Criar uma máquina virtual**, selecione **Avançar: Avançado >** (deixe todas as outras configurações com o valor padrão)

1. Na guia **Avançado** da folha **Criar uma máquina virtual**, especifique as seguintes configurações e selecione **Examinar + criar** (deixe todas as outras configurações com o valor padrão):

   | Configuração | Valor |
   |   --    |  --   |
   | **Grupo de posicionamento por proximidade** | **az12001a-ppg** |

1. Na guia **Examinar + Criar** da folha **Criar uma máquina virtual**, selecione **Criar**.

   > **Observação**: Aguarde a conclusão do provisionamento. Isso deverá levar menos de três minutos.

1. No portal do Azure, use a caixa de texto **Pesquisar recursos, serviços e documentos** na parte superior da página do portal do Azure para pesquisar e navegar até a folha **de máquinas virtuais** e, na folha **Máquinas virtuais**, selecione **+ Criar** e, no menu suspenso, selecione **máquina virtual do Azure**.

1. Na guia **Básico** da folha **Criar uma máquina virtual**, especifique as seguintes configurações e selecione **Avançar: Discos >** (deixe todas as outras configurações com o valor padrão):
   
    | Configuração | Valor |
    |   --    |  --   |
    | **Assinatura** | o nome da sua assinatura do Azure |
    | **Grupo de recursos** | o nome do grupo de recursos criado anteriormente nesta tarefa |
    | **Nome da máquina virtual** | **az12001a-vm1** |
    | **Região** | o nome da região do Azure que você escolheu ao criar o grupo de posicionamento por proximidade |
    | **Opções de disponibilidade** | **Conjunto de disponibilidade** |
    | **Conjunto de disponibilidade** | **az12001a-avset** |
    | **Tipo de segurança** | **Standard** |
    | **Imagem** | **SUSE Enterprise Linux for SAP 15 SP5 – BYOS – x64 Gen 2** |
    | **Executar com o Desconto de Spot do Azure** | desabilitado |
    | **Tamanho** | **Standard_D4s_v3** |
    | **Tipo de autenticação** | **Senha** |
    | **Nome de usuário** | **student** |
    | **Senha** | A mesma senha especificada durante a primeira implantação |
   
    > **Observação**: para localizar a imagem, clique no link **Ver todas as imagens**, na folha **Selecionar uma imagem**, na caixa de texto de pesquisa, digite **SUSE Enterprise Linux** e, na lista de resultados, clique em **SUSE Enterprise Linux for SAP 15 SP5 - BYOS** e selecione **Geração 2**.

1. Na guia **Discos** da folha **Criar uma máquina virtual **, especifique as seguintes configurações e selecione **Avançar: Rede >** (deixe todas as outras configurações com o valor padrão):

    | Configuração | Valor |
    |   --    |  --   |
    | **Tipo de disco de SO** | **SSD Premium**  |
    | **Gerenciamento de chaves** | **Chave de criptografia gerenciada pela plataforma** |

1. Na guia **Rede** da folha **Criar uma máquina virtual**, especifique as seguintes configurações e selecione **Avançar: Gerenciamento >** (deixe todas as outras configurações com o valor padrão):
    
    | Configuração | Valor |
    |   --    |  --   |
    | **Rede virtual** | **az12001a-RG-vnet** |
    | **Sub-rede** | **subnet-0 (192.168.0.0/24)** |
    | **Endereço IP público** | **Nenhuma** |
    | **Grupo de segurança de rede da NIC** | **Avançado**  |
    | **Habilitar a rede acelerada** | **Ativado** |
    | **Opções de balanceamento de carga** | **Nenhuma** |

1. Na guia **Gerenciamento** da folha **Criar uma máquina virtual**, especifique as seguintes configurações e selecione **Avançar: Monitoramento >** (deixe todas as outras configurações com o valor padrão):
    
   | Configuração | Valor |
   |   --    |  --   |
   | **Habilitar plano básico gratuitamente** | desabilitado |
   | **Habilitar identidade gerenciada atribuída pelo sistema** | desabilitado |
   | **Habilitar o desligamento automático** | desabilitado |

   > **Observação**: A configuração do **plano básico para gratuito** não estará disponível se você já tiver selecionado o plano da Central de Segurança do Azure.

1.  Na guia **Monitoramento** da folha **Criar uma máquina virtual**, selecione **Avançar: Avançado >** (deixe todas as outras configurações com o valor padrão)

1.  Na guia **Avançado** da folha **Criar uma máquina virtual**, confirme a seguinte configuração e clique em **Revisar + criar** (deixe todas as outras configurações com o valor padrão):
    
   | Configuração | Valor |
   |   --    |  --   |
   | **Grupo de posicionamento por proximidade** | **az12001a-ppg** |

1.  Na guia **Examinar + Criar** da folha **Criar uma máquina virtual**, selecione **Criar**.

   > **Observação**: Aguarde a conclusão do provisionamento. Isso deverá levar menos de três minutos.


### Tarefa 2: Criar e configurar discos de VMs do Azure

1. No portal do Azure, inicie uma sessão do Bash no Cloud Shell. 

   > **Observação**: Se esta for a primeira vez que você está iniciando o Cloud Shell na assinatura atual do Azure, você será solicitado a criar um compartilhamento de arquivos do Azure para persistir arquivos do Cloud Shell. Nesse caso, aceite os padrões, o que resultará na criação de uma conta de armazenamento em um grupo de recursos gerado automaticamente.

1. No painel do Cloud Shell, execute o seguinte comando para definir o valor da variável `RESOURCE_GROUP_NAME` como o nome do grupo de recursos que contém os recursos provisionados na tarefa anterior:

   ```cli
   RESOURCE_GROUP_NAME='az12001a-RG'
   ```

1. No painel do Cloud Shell, execute o seguinte comando para criar o primeiro conjunto de 8 discos gerenciados que você anexará à primeira VM do Azure implantada na tarefa anterior:

   ```cli
   LOCATION=$(az group list --query "[?name == '$RESOURCE_GROUP_NAME'].location" --output tsv)

   for I in {0..7}; do az disk create --resource-group $RESOURCE_GROUP_NAME --name az12001a-vm0-DataDisk$I --size-gb 128 --location $LOCATION --sku Premium_LRS; done
   ```

1. No painel do Cloud Shell, execute o seguinte comando para criar o segundo conjunto de 8 discos gerenciados que você anexará à segunda VM do Azure implantada na tarefa anterior:

   ```cli
   for I in {0..7}; do az disk create --resource-group $RESOURCE_GROUP_NAME --name az12001a-vm1-DataDisk$I --size-gb 128 --location $LOCATION --sku Premium_LRS; done
   ```

1. Feche o painel do Cloud Shell.

1. No portal do Azure, navegue até a folha da primeira VM do Azure provisionada na tarefa anterior (**az12001a-vm0**).

1. Na folha **az12001a-vm0**, navegue até a folha **az12001a-vm0 \| Discos**.

1. Na folha **az12001a-vm0 \| Discos**, selecione **Anexar discos existentes** e anexe o disco de dados com as seguintes configurações a az12001a-vm0:
    
   | Configuração | Valor |
   |   --    |  --   |
   | **LUN** | **0** |
   | **Nome do disco** | **az12001a-vm0-DataDisk0** |
   | **Grupo de recursos** | o nome do grupo de recursos usado anteriormente nesta tarefa |
   | **CACHE DE HOST** | **Somente leitura** |

2. Repita a etapa anterior para anexar os 7 discos restantes com o prefixo **az12001a-vm0-DataDisk** (para um total de 8). Atribua o número LUN correspondente ao último caractere do nome do disco. Defina CACHE DE HOST do disco com LUN **1** como **Somente leitura** e, para todos os restantes, defina CACHE DE HOST como **Nenhum**.

3. Salve suas alterações. 

4. No portal do Azure, navegue até a folha da segunda VM do Azure provisionada na tarefa anterior (**az12001a-vm1**).

5. Na folha **az12001a-vm1**, navegue até a folha **az12001a-vm1 \| Discos**.

6. Na folha **az12001a-vm1 \| Discos**, anexe discos de dados com as seguintes configurações ao az12001a-vm1:
    
   | Configuração | Valor |
   |   --    |  --   |
   | **LUN** | **0** |
   | **Nome do disco** | **az12001a-vm1-DataDisk0** |
   | **Grupo de recursos** | o nome do grupo de recursos usado anteriormente nesta tarefa |
   | **CACHE DE HOST** | **Somente leitura** |

7. Repita a etapa anterior para anexar os 7 discos restantes com o prefixo **az12001a-vm1-DataDisk** (para um total de 8). Atribua o número LUN correspondente ao último caractere do nome do disco. Defina CACHE DE HOST do disco com LUN \ **1** como **Somente leitura** e, para todos os restantes, defina CACHE DE HOST como **Nenhum**.

8. Salve suas alterações. 

#### Tarefa 3: Provisionar o Azure Bastion 

> **Observação**: O Azure Bastion permite a conexão com as VMs do Azure (que você implantou na tarefa anterior deste exercício) sem usar pontos de extremidade públicos, ao mesmo tempo em que fornece proteção contra ataques de força bruta cujo alvo sejam credenciais de nível de sistema operacional.

> **Observação**: Para usar o Azure Bastion, certifique-se de que o navegador esteja com a funcionalidade de pop-up habilitada.

1. Na janela do navegador da Web que está exibindo o portal do Azure, abra outra guia e navegue até o [**portal do Azure**](https://portal.azure.com).
1. No portal do Azure, abra o painel do **Cloud Shell** selecionando no ícone da barra de ferramentas diretamente à direita da caixa de texto de pesquisa.
1. Na sessão do PowerShell no painel do Cloud Shell, execute o seguinte para adicionar uma sub-rede chamada **AzureBastionSubnet** à rede virtual chamada **az12001a-RG-vnet** criada anteriormente nesse exercício:

   ```powershell
   $resourceGroupName = 'az12001a-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az12001a-RG-vnet'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 192.168.15.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Feche o painel do Cloud Shell.
1. No portal do Azure, pesquise e selecione **Bastions** e, na folha **Bastions**, selecione **+ Criar**.
1. Na guia **Noções Básicas** da folha **Criar um Bastion**, especifique as seguintes configurações e selecione **Examinar + criar**:

   |Configuração|Valor|
   |---|---|
   |Assinatura|o nome da assinatura do Azure que você está usando neste laboratório|
   |Grupo de recursos|**az12001a-RG**|
   |Nome|**az12001a-bastion**|
   |Region|a mesma região do Azure na qual você implantou os recursos na tarefa anterior desse exercício|
   |Camada|**Basic**|
   |Rede virtual|**az12001a-RG-vnet**|
   |Sub-rede|**AzureBastionSubnet (192.168.15.0/24)**|
   |Endereço IP público|**Criar novo**|
   |Nome do IP público|**az12001a-RG-vnet-ip**|

1. Na guia **Revisar + criar** da folha **Criar um Bastion**, selecione **Criar**:

   > **Observação**: Aguarde a conclusão da implantação antes de prosseguir para a próxima tarefa desse exercício. A implantação pode levar cerca de cinco minutos.

> **Resultado**: Depois de concluir este exercício, você terá provisionado os recursos de computação do Azure necessários para dar suporte a implantações do SAP HANA altamente disponíveis.


## Exercício 2: configure o sistema operacional de VMs do Azure que executam o Linux para dar suporte a uma instalação de SAP HANA altamente disponível

Duração: 30 minutos

Neste exercício, você configurará o sistema operacional e o armazenamento em VMs do Azure que executam o SUSE Linux Enterprise Server para acomodar instalações clusterizadas do SAP HANA.

### Tarefa 1: Conectar-se a VMs do Linux do Azure

1. No seu computador de laboratório, no portal do Azure, pesquise e selecione **Máquinas virtuais** e, na folha **Máquinas virtuais**, selecione a entrada **az12001a-vm0**. Isso abrirá a folha **az12001a-vm0**.

1. Na folha **az12001a-vm0**, selecione **Conectar**, no menu suspenso, selecione **Conectar via Bastion**, na guia **Bastion** da **az12001a-vm0**, deixe o **Tipo de autenticação** definido como **Senha da VM**, forneça as credenciais definidas ao implantar a máquina virtual **az12001a-vm0**, deixe a caixa de seleção **Abrir em uma nova guia do navegador** habilitada e selecione **Conectar**.

1. Repita as duas etapas anteriores para se conectar via Bastion à VM do Azure **az12001a-vm1**.

### Tarefa 2: Configurar o armazenamento de VMs do Azure que executam Linux

1. Na sessão Bastion para a VM do Azure **az12001a-vm0**, execute o seguinte comando para elevar privilégios: 

   ```sh
   sudo su -
   ```

1. Execute o seguinte comando para identificar o mapeamento entre os dispositivos recém-conectados e seus números de LUN:
   
   ```sh
   lsscsi
   ```

1. Crie volumes físicos para 6 (de 8) discos de dados executando:
   
   ```sh
   pvcreate /dev/sdc
   pvcreate /dev/sdd
   pvcreate /dev/sde
   pvcreate /dev/sdf
   pvcreate /dev/sdg
   pvcreate /dev/sdh
   ```

1. Crie grupos de volumes executando:
   
   ```sh
   vgcreate vg_hana_data /dev/sdc /dev/sdd
   vgcreate vg_hana_log /dev/sde /dev/sdf
   vgcreate vg_hana_backup /dev/sdg /dev/sdh
   ```

1. Crie volumes lógicos executando:

   ```sh
   lvcreate -l 100%FREE -n hana_data vg_hana_data
   lvcreate -l 100%FREE -n hana_log vg_hana_log
   lvcreate -l 100%FREE -n hana_backup vg_hana_backup
   ```

   > **Observação**: Estamos criando um único volume lógico por cada grupo de volumes

1. Formate volumes lógicos executando:

   ```sh
   mkfs.xfs /dev/vg_hana_data/hana_data -m crc=1
   mkfs.xfs /dev/vg_hana_log/hana_log -m crc=1
   mkfs.xfs /dev/vg_hana_backup/hana_backup -m crc=1
   ```

   > **Observação**: A partir do SUSE Linux Enterprise Server 12, você tem a opção de usar o novo formato em disco (v5) do sistema de arquivos XFS, que oferece somas de verificação automáticas de metadados XFS, suporte a tipos de arquivo e um limite maior no número de listas de controle de acesso por arquivo. O novo formato se aplica automaticamente ao usar o YaST para criar os sistemas de arquivos XFS. Para criar um sistema de arquivos XFS no formato mais antigo por motivos de compatibilidade, use o comando mkfs.xfs sem a opção `-m crc=1`. 

1. Particione o disco **/dev/sdi** executando:

   ```sh
   fdisk /dev/sdi
   ```

1. Quando solicitado, digite, em sequência, `n`, `p`, `1` (seguido pela tecla **Enter** de cada vez) pressione a tecla **Enter** duas vezes e digite `w` para concluir a gravação.

1. Particione o disco **/dev/sdj** executando:

   ```sh
   fdisk /dev/sdj
   ```

1. Quando solicitado, digite, em sequência, `n`, `p`, `1` (seguido pela tecla **Enter** de cada vez) pressione a tecla **Enter** duas vezes e digite `w` para concluir a gravação.

1. Formate a partição recém-criada executando (digite `y` e pressione a tecla **Enter** quando for solicitada a confirmação):

   ```sh
   mkfs.xfs /dev/sdi -m crc=1 -f
   mkfs.xfs /dev/sdj -m crc=1 -f
   ```

1. Crie os diretórios que servirão como pontos de montagem executando:

   ```sh
   mkdir -p /hana/data
   mkdir -p /hana/log
   mkdir -p /hana/backup
   mkdir -p /hana/shared
   mkdir -p /usr/sap
   ```

1. Exiba as IDs dos volumes lógicos executando:

   ```sh
   blkid
   ```

   > **Observação**: Identifique os valores **UUID** associados aos grupos de volumes e partições recém-criados, incluindo **/dev/sdi** (para ser usado para **/hana/shared**) e **dev/sdj** (para ser usado para **/usr/sap**).


1. Abra o **/etc/fstab** no editor vi (você pode usar qualquer outro editor) executando:

   ```sh
   vi /etc/fstab
   ```

   > **Observação**: se você estiver usando o editor vi, pressione a **tecla i** para entrar no modo INSERT.

   ![indicador do modo INSERT no editor vi](../media/az120-lab01-vieditor-insert.png)

1. No editor, adicione as seguintes entradas ao **/etc/fstab** (em que `\<UUID of /dev/vg\_hana\_data-hana\_data\>`, , , , e `\<UUID of /dev/vg\_hana\_log-hana\_log\>`, `\<UUID of /dev/vg\_hana\_backup-hana\_backup\>``\<UUID of /dev/vg_hana_shared-hana_shared (/dev/sdi)\>``\<UUID of /dev/vg_usr_sap-usr_sap (/dev/sdj)\>`representam os ids que você identificou na etapa anterior):

   ```sh
   /dev/disk/by-uuid/<UUID of /dev/vg_hana_data-hana_data> /hana/data xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<UUID of /dev/vg_hana_log-hana_log> /hana/log xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<UUID of /dev/vg_hana_backup-hana_backup> /hana/backup xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<UUID of /dev/vg_hana_shared-hana_shared (/dev/sdi)> /hana/shared xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<UUID of /dev/vg_usr_sap-usr_sap (/dev/sdj)> /usr/sap xfs  defaults,nofail  0  2
   ```

1. Salve as alterações e feche o editor.

1. Monte os novos volumes executando:

   ```sh
   mount -a
   ```

1. Verifique se a montagem foi bem-sucedida executando:

   ```sh
   df -h
   ```

   ![saída df-h](../media/az120-lab01-df-output.png)
1. Saia do modo privilegiado executando:

   ```sh
   exit
   ```

1. Alterne para a sessão Bastion para **az12001a-vm1** e repita todas as etapas desta tarefa para configurar o armazenamento em **az12001a-vm1**.


### Tarefa 3: Habilitar o acesso SSH sem senha entre nós

1. Na sessão Bastion para **az12001a-vm1**, gere um par de chave SSH sem frase secreta executando:

      ```sh
   ssh-keygen -trsa
   ```

1. Quando solicitado, pressione a tecla **Enter** três vezes.

1. Copie a chave pública do par de chaves recém-gerado para **az12001a-vm0** executando:

      ```sh
   ssh-copy-id -i /home/student/.ssh/id_rsa.pub student@az12001a-vm0
   ```

1. Quando solicitado para confirmar se deseja continuar se conectando, digite **sim** e pressione a tecla **Enter**.

1. Quando solicitado a autenticar, insira a senha definida ao provisionar **az12001a-vm0** anteriormente neste laboratório.

1. Alterne para a sessão Bastion para a VM do Azure **az12001a-vm0**.

1. Na sessão Bastion para **az12001a-vm0**, gere um par de chave SSH sem frase executando:

      ```sh
   ssh-keygen -trsa
   ```

1. Quando solicitado, pressione a tecla **Enter** três vezes.

1. Copie a chave pública do par de chaves recém-gerado para **az12001a-vm1** executando:

      ```sh
   ssh-copy-id -i /home/student/.ssh/id_rsa.pub student@az12001a-vm1
   ```

1. Quando solicitado para confirmar se deseja continuar se conectando, digite **sim** e pressione a tecla **Enter**.

1. Quando solicitado a autenticar, insira a senha definida ao provisionar **az12001a-vm1** anteriormente neste laboratório.

1. Para verificar se a configuração foi bem-sucedida, na sessão Bastion para a VM do Azure **az12001a-vm0**, estabeleça uma sessão SSH como **estudante** para **az12001a-vm1** executando: 

   ```sh
   ssh student@az12001a-vm1
   ```

1. Verifique se você não foi solicitado a fornecer a senha.

1. Feche a sessão SSH de **az12001a-vm0** para **az12001a-vm1** executando: 

   ```sh
   exit
   ```

1. Alterne para a sessão Bastion para a VM do Azure **az12001a-vm1**.

1. Na sessão Bastion para **az12001a-vm1**, estabeleça uma sessão SSH como **estudante** para **az12001a-vm0** executando: 

   ```sh
   ssh student@az12001a-vm0
   ```

1. Verifique se você não foi solicitado a fornecer a senha.

1. Feche a sessão SSH de **az12001a-vm1** para **az12001a-vm0** executando: 

   ```sh
   exit
   ```

> **Resultado**: Depois de concluir este exercício, você terá configurado o sistema operacional de VMs do Azure que executam o Linux para dar suporte a uma instalação do SAP HANA altamente disponível


## Exercício 3: provisione os recursos de rede do Azure necessários para dar suporte a implantações SAP HANA altamente disponíveis

Duração: 30 minutos

Neste exercício, você implementará balanceadores de carga do Azure Load Balancer para acomodar instalações clusterizadas do SAP HANA.


### Tarefa 1: Configure as VMs do Azure para facilitar a instalação do balanceamento de carga.

1. No portal do Azure, navegue até a folha da VM do Azure **az12001a-vm0**.

1. Na folha **az12001a-vm0**, navegue até a folha **az12001a-vm0 \| Rede**. 

1. Na folha **az12001a-vm0 \| Rede**, selecione a entrada que representa a interface de rede do az12001a-vm0. 

1. Na folha da interface de rede da az12001a-vm0, navegue até a folha de configurações de IP e, a partir daí, exiba sua folha **ipconfig1**.

1. Na folha **ipconfig1**, defina a atribuição de endereço IP privado como **Estática** e salve a alteração.

1. No portal do Azure, navegue até a folha da VM do Azure **az12001a-vm1**.

1. Na folha **az12001a-vm1**, navegue até a folha **az12001a-vm1 \| Rede**. 

1. Na folha **az12001a-vm1 \| Rede**, navegue até a interface de rede da az12001a-vm1. 

1. Na folha da interface de rede da az12001a-vm1, navegue até a sua folha de configurações de IP e, a partir daí, exiba sua folha **ipconfig1**.

1. Na folha **ipconfig1**, defina a atribuição de endereço IP privado como **Estática** e salve a alteração.


### Tarefa 2: Criar e configurar a manipulação do tráfego de entrada de balanceadores de carga do Azure Load Balancer

1. No portal do Azure, use a caixa de texto **Pesquisar recursos, serviços e documentos** na parte superior da página do portal do Azure para procurar e navegar até a folha **Balanceadores de carga** e, na folha **Balanceadores de carga**, selecione **+ Criar**.

1. Na guia **Básico** da folha **Criar balanceador de carga**, especifique as seguintes configurações e selecione **Examinar + criar** (deixe as outras com os valores padrão):
    
   | Configuração | Valor |
   |   --    |  --   |
   | **Assinatura** | o nome da sua assinatura do Azure |
   | **Grupo de recursos** | **az12001a-RG** |
   | **Nome** | **az12001a-lb0** |
   | **Região** | a mesma região do Azure em que você implantou as VMs do Azure no primeiro exercício deste laboratório |
   | **SKU** | **Standard** |
   | **Tipo** | **Interna** |

1. Clique em **Avançar: Configuração do IP de front-end**. Na tela **Configuração de IP de front-end**, clique em **Adicionar uma configuração de IP de front-end** e, em seguida, clique em **Adicionar**.
    
   | Configuração | Valor |
   |   --    |  --   |
   | **Nome** | **frontend1** |
   | **Rede virtual** | **az12001a-RG-vnet** |
   | **Sub-rede** | **subnet-0** |
   | **Atribuição de endereço IP** | **Estático** |
   | **Endereço IP** | **192.168.0.240** |
   | **Zona de disponibilidade** | **Redundância de zona** |

1. Selecione **Examinar + Criar** e, em seguida, selecione **Criar**.

   > **Observação**: Aguarde até que o balanceador de carga seja provisionado. Isso deve levar menos de um minuto. 

1. No portal do Azure, navegue até a folha exibindo as propriedades do recém-provisionado balanceador de carga **az12001a-lb0**. 

1. Na folha **az12001a-lb0**, selecione **Pools de back-end**, **+ Adicionar** e, em **Adicionar pool de back-end**, especifique as seguintes configurações (deixe as outras com os valores padrão):
    
   | Configuração | Valor |
   |   --    |  --   |
   | **Nome** | **az12001a-lb0-bepool** |
   | **Rede virtual** | **az12001a-RG-vnet** |
   | **Configuração do pool de back-end** | **Endereço IP** |
   | **Endereço IP** | **192.168.0.4** Nome do recurso: **az12001a-vm0** |
   | **Endereço IP** | **192.168.0.5** Nome do recurso: **az12001a-vm1** |

1. Na folha **az12001a-lb0**, selecione **Investigações de integridade**, **+ Adicionar** e, na folha **Adicionar investigação de integridade**, especifique as seguintes configurações (deixe as outras com os valores padrão):
    
   | Configuração | Valor |
   |   --    |  --   |
   | **Nome** | **az12001a-lb0-hprobe** |
   | **Protocolo** | **TCP** |
   | **Porta** | **62500** |
   | **Intervalo** | **5** *segundos* |

1. Na folha **az12001a-lb0**, selecione **Regras de balanceamento de carga**, **+ Adicionar** e, na folha **Adicionar regras de balanceamento de carga**, especifique as seguintes configurações (deixe as outras com os valores padrão):
    
   | Configuração | Valor |
   |   --    |  --   |
   | **Nome** | **az12001a-lb0-lbruleAll** |
   | **Versão IP** | **IPv4** |
   | **Endereço IP de front-end** | **192.168.0.240 (LoadBalancerFrontEnd)** |
   | **Portas de HA** | Habilitado |
   | **Pool de back-end** | **az12001a-lb0-bepool (2 virtual machines)** |
   | **Investigação de integridade** | **az12001a-lb0-hprobe (TCP:62500)** |
   | **Persistência de sessão** | **Nenhuma** |
   | **Tempo limite de ociosidade (minutos)** | **4** |
   | **Redefinição de TCP** | desabilitado |
   | **IP flutuante (retorno de servidor direto)** | Habilitado |

### Tarefa 3: Criar e configurar balanceadores de carga do Azure Load Balancer que manipulam tráfego de saída

1. No portal do Azure, inicie uma sessão do Bash no Cloud Shell. 

1. No painel do Cloud Shell, execute o seguinte comando para definir o valor da variável `RESOURCE_GROUP_NAME` como o nome do grupo de recursos que contém os recursos provisionados no primeiro exercício neste laboratório:

   ```cli
   RESOURCE_GROUP_NAME='az12001a-RG'
   ```

1. No painel do Cloud Shell, execute o seguinte comando para criar o endereço IP público a ser usado pelo segundo balanceador de carga:

   ```cli
   LOCATION=$(az group list --query "[?name == '$RESOURCE_GROUP_NAME'].location" --output tsv)

   PIP_NAME='az12001a-lb1-pip'

   az network public-ip create --resource-group $RESOURCE_GROUP_NAME --name $PIP_NAME --sku Standard --location $LOCATION
   ```

1. No painel do Cloud Shell, execute o seguinte comando para criar o segundo balanceador de carga:

   ```cli
   LB_NAME='az12001a-lb1'

   LB_BE_POOL_NAME='az12001a-lb1-bepool'

   LB_FE_IP_NAME='az12001a-lb1-fe'

   az network lb create --resource-group $RESOURCE_GROUP_NAME --name $LB_NAME --sku Standard --backend-pool-name $LB_BE_POOL_NAME --frontend-ip-name $LB_FE_IP_NAME --location $LOCATION --public-ip-address $PIP_NAME
   ```

1. No painel do Cloud Shell, execute o seguinte comando para criar a regra de saída do segundo balanceador de carga:

   ```cli
   LB_RULE_OUTBOUND='az12001a-lb1-ruleoutbound'

   az network lb outbound-rule create --resource-group $RESOURCE_GROUP_NAME --lb-name $LB_NAME --name $LB_RULE_OUTBOUND --frontend-ip-configs $LB_FE_IP_NAME --protocol All --idle-timeout 4 --outbound-ports 1000 --address-pool $LB_BE_POOL_NAME
   ```

1. Feche o painel do Cloud Shell.

1. No portal do Azure, navegue até a folha que exibe as propriedades do Azure Load Balancer recém-criado **az12001a-lb1**.

1. Na folha **az12001a-lb1**, clique em **Pools de back-end**.

1. Na folha **az12001a-lb1 \| pools de back-end**, clique em **az12001a-lb1-bepool**.

1. Na folha **az12001a-lb1-bepool**, especifique as seguintes configurações e clique em **Salvar**:
   
   | Configuração | Valor |
   |   --    |  --   |
   | **Rede virtual** | **az12001a-rg-vnet (2 VM)** |
   | **Máquina virtual** | **az12001a-vm0**  Configuração IP: **ipconfig1 (192.168.0.4)** |
   | **Máquina virtual** | **az12001a-vm1**  Configuração IP: **ipconfig1 (192.168.0.5)** |

> **Resultado**: Depois de concluir este exercício, você terá provisionado os recursos de rede do Azure necessários para dar suporte a implantações do SAP HANA altamente disponíveis


## Exercício 4: remova recursos de laboratório

Duração: 10 minutos

Neste exercício, você removerá os recursos provisionados neste laboratório.

#### Tarefa 1: Listar grupos de recursos a serem excluídos

1. Na parte superior do portal, clique no ícone do **Cloud Shell** para abrir o painel do Cloud Shell e escolha o Bash como o shell.

1. No painel do Cloud Shell, execute o seguinte comando para definir o valor da variável `RESOURCE_GROUP_PREFIX` como o prefixo do nome do grupo de recursos que contém os recursos provisionados neste laboratório:

   ```cli
   RESOURCE_GROUP_PREFIX='az12001a-'
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
