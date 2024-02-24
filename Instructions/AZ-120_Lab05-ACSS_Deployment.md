---
lab:
  title: 05 - Automatizar a implantação usando o Centro do Azure para soluções SAP
  module: Design and implement an infrastructure to support SAP workloads on Azure
---

# Módulo AZ 120: Projetar e implementar uma infraestrutura para dar suporte a cargas de trabalho SAP no Azure
# Laboratório: Automatizar a implantação usando o Centro do Azure para soluções SAP

Tempo Estimado: 100 minutos

Todas as tarefas neste laboratório são executadas no portal do Azure

## Objetivos

Depois de realizar este laboratório, você será capaz de:

- Implementar pré-requisitos para a implantação de cargas de trabalho SAP no Azure usando o Centro do Azure para soluções SAP
- Implantar a infraestrutura que hospedará cargas de trabalho SAP no Azure usando o Centro do Azure para soluções SAP

## Exercício 1: Implementar pré-requisitos para a implantação de cargas de trabalho SAP no Azure usando o Centro do Azure para soluções SAP

Duração: 60 minutos

Neste exercício, você implementará os pré-requisitos para implantar cargas de trabalho SAP no Azure usando o Centro do Azure para soluções SAP. Isso incluirá as seguintes tarefas:
- Atender aos requisitos da vCPU na assinatura do Azure de destino
- Configurar atribuições de função de controle de acesso baseado em função (RBAC) do Azure para a conta de usuário do Microsoft Entra ID que será usada para executar a implantação
- Criar uma conta de armazenamento associada ao Centro do Azure para soluções SAP usada para a implantação
- Criar uma identidade gerenciada atribuída pelo usuário a ser usada pelo Centro do Azure para soluções SAP para autenticação e autorização da sua implantação automatizada
- Criar um grupo de segurança de rede (NSG) a ser usado em sub-redes da rede virtual que hospedará a implantação
- Criar tabelas de rotas a serem usadas em sub-redes da rede virtual que hospedará a implantação
- Criar e configurar a rede virtual que hospedará a implantação
- Implantar o Firewall do Azure na rede virtual que hospedará a implantação
- Implantar o Azure Bastion na rede virtual que hospedará a implantação

O exercício é composto pelas seguintes tarefas:

- Tarefa 1: Abordar os requisitos de vCPU na assinatura de destino do Azure
- Tarefa 2: Configurar atribuições de função de controle de acesso baseado em função (RBAC) do Azure para a conta de usuário do Microsoft Entra ID que será usada para executar a implantação
- Tarefa 3: Criar uma conta de armazenamento associada ao Centro do Azure para soluções SAP usada para a implantação
- Tarefa 4: Criar e configurar uma identidade gerenciada atribuída pelo usuário a ser usada pelo Centro do Azure para soluções SAP para autenticação e autorização de sua implantação automatizada
- Tarefa 5: Criar um grupo de segurança de rede (NSG) a ser usado nas sub-redes da rede virtual que hospedará a implantação
- Tarefa 6: Criar tabelas de rotas a serem usadas nas sub-redes da rede virtual que hospedará a implantação
- Tarefa 7: Criar e configurar a rede virtual que hospedará a implantação
- Tarefa 8: Implantar o Firewall do Azure na rede virtual que hospedará a implantação
- Tarefa 9: Implante o Azure Bastion na rede virtual que hospedará a implantação

### Tarefa 1: Abordar os requisitos de vCPU na assinatura de destino do Azure

>**Observação**: Não é necessário concluir esta tarefa se você já tiver implementado todos os [pré-requisitos do laboratório AZ-120](https://github.com/MicrosoftLearning/AZ-120-Planning-and-Administering-Microsoft-Azure-for-SAP-Workloads/blob/master/Instructions/AZ-120_Lab00_Prerequisites.md).

>**Observação**: Para concluir esse laboratório (conforme descrito), você precisará de uma assinatura do Microsoft Azure com as cotas de vCPU que acomodam a implantação das seguintes VMs:

- 2 x VMs Standard_E4ds_v4 (4 vCPUs e 32 GiB de memória cada) ou 2 X VMs Standard_D4ds_v4 (4 vCPUs e 16 GiB de memória cada) para a camada do ASCS
- 2 x VMs Standard_E4ds_v4 (4 vCPUs e 32 GiB de memória cada) ou 2 X VMs Standard_D4ds_v4 (4 vCPUs e 16 GiB de memória cada) para a camada de aplicativo 
- 2 x VMs Standard_M64ms (64 vCPUs e 1750 GiB de memória cada) para a camada de banco de dados

>**Observação**: Para minimizar os requisitos de vCPU e memória para as VMs de banco de dados, você pode alterar o SKU da VM para Standard_M32ts (32 vCPUs e 192 GiB de memória cada).

1. No computador do laboratório, inicie um navegador da Web e navegue até o portal do Azure em `https://portal.azure.com`.
1. No portal do Azure, selecione o ícone do **Cloud Shell** e inicie uma sessão do PowerShell no Cloud Shell. 

    > **Observação**: Se esta for a primeira vez que você está iniciando o Cloud Shell na assinatura do Azure que será usada neste laboratório, será solicitado que você crie um compartilhamento de arquivo do Azure para persistir arquivos do Cloud Shell. Em caso afirmativo, aceite os padrões, o que resultará na criação de uma conta de armazenamento em um grupo de recursos gerado automaticamente.

1. No portal do Microsoft Azure, no painel **Cloud Shell**, no prompt do PowerShell, execute o seguinte (se necessário, substitua `eastus` pelo nome da região do Azure na qual você pretende implantar recursos neste laboratório):

    > **Observação**: Para identificar os nomes das regiões do Azure, no **Cloud Shell**, no prompt do Bash, execute `(Get-AzLocation).Location`
     
    ```powershell
    Set-Variable -Name "Azure_region" -Value ('eastus') -Option constant -Scope global -Description "All processes" -PassThru

    Get-AzVMUsage -Location $Azure_region | Where-Object {$_.Name.Value -eq 'standardEDSv4Family'}
    
    Get-AzVMUsage -Location $Azure_region | Where-Object {$_.Name.Value -eq 'standardDSv4Family'}

    Get-AzVMUsage -Location $Azure_region | Where-Object {$_.Name.Value -eq 'standardMSFamily'}

    Get-AzVMUsage -Location $Azure_region | Where-Object {$_.Name.Value -eq 'cores'}
    ```

1. Examine a saída para identificar o uso atual da vCPU e o limite da vCPU. Verifique se a diferença entre eles é suficiente para acomodar as vCPUs das VMs do Azure que você implantará neste laboratório. Leve em conta os números de vCPU regionais específicos da família da VM e do total. 
1. Se o número de vCPUs não for suficiente, feche o painel Cloud Shell, no portal do Azure, na caixa de texto **Search**, pesquise e selecione **Quotas**.
1. Na página **Quotas**, selecione **Compute**.
1. Na página **Quotas \| Compute**, use o filtro **Region** para selecionar a região do Azure na qual você pretende implantar recursos neste laboratório.
1. Na coluna **Nome da cota**, localize e selecione o nome do SKU da VM que requer um aumento de cota. 
1. Na mesma linha, verifique a entrada na coluna **Ajustável**. A próxima etapa depende de se a coluna contém a entrada **Sim** ou **Não**.

   - Se a entrada estiver definida como **Sim**, selecione o ícone **Solicitar ajuste**, na **Nova solicitação de cota**, na caixa de texto **Novo limite**, insira o novo limite de cota e selecione **Enviar**.
   - Se a entrada estiver definida como **Não**, selecione o ícone **Solicitar acesso ou obter recomendações** e, no painel **Recomendações de cota**, selecione a opção **Entrar em contato com o suporte** e selecione **Avançar**. 
1. Na guia **Descrição do problema** da página **Nova solicitação de suporte**, especifique as seguintes configurações e selecione **Avançar**:

    |Configuração|Valor|
    |---|---|
    |A que o seu problema está relacionado?|**Serviços do Azure**|
    |Tipo de problema|**Limites de serviço e assinatura (cotas)**|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Tipo de cota|**Aumentos de limite de assinatura de computação/VM (núcleos/vCPUs)**|

1. Na guia **Detalhes adicionais**, selecione **Inserir detalhes**.
1. Na guia **Detalhes da cota**; na lista suspensa **Modelo de implantação**, selecione **Gerenciador de recursos**; na lista suspensa **Locais**, selecione a região do Azure de destino; na lista suspensa **Cotas**, selecione a série de VMs do Azure para a qual você precisa aumentar os limites de cota; e na caixa de texto **Novo limite**, insira o novo limite de cota e selecione **Salvar e continuar**.
1. De volta à guia **Detalhes adicionais**, na guia **Informações avançadas de diagnóstico**, selecione **Sim (recomendado)**.
1. Na seção **Método de suporte**, selecione **Email** ou **Telefone** como o seu método de contato preferencial e selecione **Avançar**.
1. Na guia **Revisar + criar**, selecione **Criar**.

    > **Observação**: Aguarde até que a solicitação para aumentar os limites de cota seja concluída com êxito antes de prosseguir para a próxima tarefa.

### Tarefa 2: Configurar atribuições de função de controle de acesso baseado em função (RBAC) do Azure para a conta de usuário do Microsoft Entra ID que será usada para executar a implantação

1. No computador do laboratório, inicie o Microsoft Edge e navegue até o portal do Azure em `https://portal.azure.com`.
1. Quando solicitado a autenticar-se, entre usando as credenciais do Microsoft Entra ID com a função Proprietário na assinatura do Azure que você usará para este laboratório. 
1. No portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Assinaturas**.
1. Na página **Assinaturas**, selecione a entrada que representa a assinatura do Azure que você usará para este laboratório. 
1. Na página que exibe as propriedades da assinatura do Azure, selecione **Controle de acesso (IAM)**.
1. Na página **Controle de acesso (IAM)**, selecione **+ Adicionar** e, no menu suspenso, selecione **Adicionar atribuição de função**.
1. Na guia **Função** da página **Adicionar atribuição de função**, na listagem de **Funções de trabalho**, pesquise e selecione a entrada **Administrador do Centro do Azure para soluções SAP** e selecione **Avançar**.
1. Na guia **Membros** da página **Adicionar atribuição de função**, clique em **+ Selecionar membros**. 
1. No painel **Selecionar membros**, na caixa de texto **Selecionar**, insira o nome da conta de usuário do Microsoft Entra ID usada para acessar a assinatura do Azure que você está usando para este laboratório, selecione-a na lista de resultados correspondentes à sua entrada e clique em **Selecionar**.
1. De volta à guia **Membros**, selecione **Examinar + atribuir**.
1. Na guia **Revisão + atribuição**, selecione **Examinar + atribuir**.
1. Repita as seis etapas anteriores para atribuir a função **Operador de identidade gerenciada** à conta de usuário que você está usando para este laboratório.

### Tarefa 3: Criar uma conta de armazenamento associada ao Centro do Azure para soluções SAP usada para a implantação

1. No computador do laboratório, na janela do Microsoft Edge que exibe o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Contas de armazenamento**.
1. Na página **Contas de Armazenamento**, selecione **+ Criar**.
1. Na guia **Básico** da página **Criar uma conta de armazenamento**, especifique as seguintes configurações e selecione **Avançar: Avançado >**.

    |Configuração|Valor|
    |---|---|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|o nome de um **novo** grupo de recursos **ACSS-DEMO**|
    |Nome da conta de armazenamento|qualquer nome globalmente exclusivo de 3 a 24 caracteres composto por letras e dígitos|
    |Region|o nome da região do Azure na qual você tem cotas de vCPU suficientes para executar esse laboratório|
    |Desempenho|**Standard**|
    |Redundância|**Armazenamento com redundância geográfica (GRS)**|
    |Disponibilizar o acesso de leitura aos dados em caso de disponibilidade regional|Desabilitado|

1. Na guia **Avançado**, examine as opções disponíveis, aceite os padrões e selecione **Avançar: Rede >**.
1. Na guia **Rede**, examine as opções disponíveis, certifique-se de que a opção **Habilitar o acesso público de todas as redes** esteja habilitada e selecione **Revisar**.
1. Na guia **Revisar**, aguarde a conclusão do processo de validação e selecione **Criar**.

    >**Observação**: Não aguarde a conclusão do provisionamento da conta de Armazenamento do Microsoft Azure. Em vez disso, prossiga para a próxima tarefa. O provisionamento pode levar cerca de 2 minutos.

### Tarefa 4: Criar e configurar uma identidade gerenciada atribuída pelo usuário a ser usada pelo Centro do Azure para soluções SAP para autenticação e autorização de sua implantação automatizada

1. No computador do laboratório, na janela do Microsoft Edge exibindo o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Identidades gerenciadas**.
1. Na página **Identidades gerenciadas**, selecione **+ Criar**.
1. Na guia **Noções básicas** da página **Criar Identidade Gerenciada Atribuída ao Usuário**, especifique as seguintes configurações e selecione **Examinar + Criar**:

    |Configuração|Valor|
    |---|---|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|**ACSS-DEMO**|
    |Region|o nome da região do Azure na qual você provisionou a conta de armazenamento anteriormente neste laboratório|
    |Nome|**Contoso-MSI**|

1. Na guia **Revisar**, aguarde a conclusão do processo de validação e selecione **Criar**.

    >**Observação**: Aguarde a conclusão do provisionamento da identidade gerenciada atribuída pelo usuário. Isso deve levar apenas alguns segundos.

1. No portal do Azure, navegue até a página **Identidades gerenciadas** e selecione a entrada **Contoso-MSI**.
1. Na página **Contoso-MSI**, selecione **Atribuições de função do Azure**.
1. Na página **Atribuições de função do Azure**, selecione **+ Adicionar atribuição de função (Versão prévia)**.
1. No painel **+ Adicionar atribuição de função (Versão prévia)**, especifique as seguintes configurações e selecione **Salvar**:

    |Configuração|Valor|
    |---|---|
    |Escopo|**Assinatura**|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Função|**Função de serviço do Centro do Azure para soluções SAP**|

2. De volta à página **Atribuições de função do Azure**, selecione **+ Adicionar atribuição de função (Versão prévia)**.
3. No painel **+ Adicionar atribuição de função (Versão prévia)**, especifique as seguintes configurações e selecione **Salvar**:

    |Configuração|Valor|
    |---|---|
    |Escopo|**Storage**|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Recurso|O nome da conta de Armazenamento do Microsoft Azure que você criou na tarefa anterior|
    |Função|**Acesso a Dados e Leitor**|

### Tarefa 5: Criar um grupo de segurança de rede (NSG) a ser usado nas sub-redes da rede virtual que hospedará a implantação

1. No computador do laboratório, na janela do Microsoft Edge exibindo o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Grupos de segurança de rede**.
1. Na página **Grupos de segurança de rede**, selecione **+ Criar**.
1. Na guia **Básico** da página **Criar grupo** de segurança de rede, especifique as seguintes configurações e selecione **Examinar + criar**:

    |Configuração|Valor|
    |---|---|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|O nome de um **novo** grupo de recursos **CONTOSO-VNET-RG**|
    |Nome|**ACSS-DEMO-NSG**|
    |Region|o nome da região do Azure na qual você provisionou a conta de armazenamento anteriormente neste laboratório|

1. Na guia **Revisar + criar**, aguarde a conclusão do processo de validação e selecione **Criar**.

    >**Observação**: Por padrão, as regras internas de grupos de segurança de rede permitem todo o tráfego de saída, todo o tráfego dentro da mesma rede virtual, bem como todo o tráfego entre redes virtuais emparelhadas. Isso é suficiente para concluir o laboratório com êxito. Dependendo dos seus requisitos de segurança, você pode considerar bloquear parte desse tráfego. Nesse caso, confira as diretrizes incluídas na documentação do Microsoft Learn [Preparar a rede para implantação de infraestrutura](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network).

### Tarefa 6: Criar tabelas de rotas a serem usadas nas sub-redes da rede virtual que hospedará a implantação

1. No computador do laboratório, na janela do Microsoft Edge que exibe o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Tabelas de roteamento**.
1. Na página **Tabelas de rotas**, selecione **+ Criar**.
1. Na guia **Básico** da página **Criar tabela de rotas**, especifique as seguintes configurações e selecione **Examinar + criar**:

    |Configuração|Valor|
    |---|---|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|**CONTOSO-VNET-RG**|
    |Region|o nome da região do Azure na qual você provisionou recursos anteriormente neste laboratório|
    |Nome|**ACSS-ROUTE**|
    |Propagar rotas de gateway|**Não**|

1. Na guia **Revisar + criar**, aguarde a conclusão do processo de validação e selecione **Criar**.

### Tarefa 7: Criar e configurar a rede virtual que hospedará a implantação

1. No computador do laboratório, na janela do Microsoft Edge que exibe o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Redes virtuais**. 
1. Na página **Redes virtuais**, selecione **+ Criar**.
1. Na guia **Noções básicas** da página **Criar rede virtual**, especifique as seguintes configurações e selecione **Avançar**:

    |Configuração|Valor|
    |---|---|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|**CONTOSO-VNET-RG**|
    |Nome da rede virtual|**CONTOSO-VNET**|
    |Region|o nome da região do Azure na qual você provisionou recursos anteriormente neste laboratório|

1. Na guia **Segurança**, aceite as configurações padrão e selecione **Avançar**.

    >**Observação**: Neste momento, você pode provisionar o Azure Bastion e o Firewall do Azure; no entanto, você os provisionará separadamente depois que a rede virtual for criada.

1. Na guia **Endereços IP**, especifique as seguintes configurações e selecione **Examinar + criar**:

    |Configuração|Valor|
    |---|---|
    |Espaço de endereços IP|**10.5.0.0/16 (65.536 endereços)**|

    >**Observação**: Exclua todas as entradas de sub-rede pré-criadas. Você adicionará sub-redes depois que a rede virtual for criada.

1. Na guia **Revisar + criar**, aguarde a conclusão do processo de validação e selecione **Criar**.
1. Navegue de volta para a página **Redes virtuais**, selecione a entrada **CONTOSO-VNET**. 
1. Na página **CONTOSO-VNET**, na barra de menus vertical no lado esquerdo da página, selecione **Sub-redes**.
1. Na página **CONTOSO-VNET \| Sub-redes**, selecione **+ Sub-rede**. 
1. No painel **Adicionar sub-rede**, especifique as seguintes configurações e selecione **Salvar**:

    |Configuração|Valor|
    |---|---|
    |Nome|**app**|
    |Intervalo de endereços da sub-rede|**10.5.0.0/24**|
    |Grupo de segurança de rede|**ACSS-DEMO-NSG**|
    |Tabela de rotas|**ACSS-ROUTE**|

1. De volta à **CONTOSO-VNET \| Sub-redes**, selecione **+ Sub-rede**. 
1. No painel **Adicionar sub-rede**, especifique as seguintes configurações e selecione **Salvar**:

    |Configuração|Valor|
    |---|---|
    |Nome|**AzureBastionSubnet**|
    |Intervalo de endereços da sub-rede|**10.5.1.0/26**|

1. De volta à **CONTOSO-VNET \| Sub-redes**, selecione **+ Sub-rede**. 
1. No painel **Adicionar sub-rede**, especifique as seguintes configurações e selecione **Salvar**:

    |Configuração|Valor|
    |---|---|
    |Nome|**db**|
    |Intervalo de endereços da sub-rede|**10.5.2.0/24**|
    |Grupo de segurança de rede|**ACSS-DEMO-NSG**|
    |Tabela de rotas|**ACSS-ROUTE**|

1. De volta à **CONTOSO-VNET \| Sub-redes**, selecione **+ Sub-rede**. 
1. No painel **Adicionar sub-rede**, especifique as seguintes configurações e selecione **Salvar**:

    |Configuração|Valor|
    |---|---|
    |Nome|**AzureFirewallSubnet**|
    |Intervalo de endereços da sub-rede|**10.5.3.0/24**|

### Tarefa 8: Implantar o Firewall do Azure na rede virtual que hospedará a implantação

>**Observação**: Antes de implantar uma instância do Firewall do Azure, primeiro você criará uma política de firewall e um endereço IP público a ser usado pela instância.

1. No computador do laboratório, na janela do Microsoft Edge exibindo o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Políticas de Firewall**.
1. Na página **Políticas de Firewall**, selecione **+ Criar**.
1. Na guia **Básico** da página **Criar uma política do Firewall do Azure**, especifique as seguintes configurações e selecione **Avançar: Configurações de DNS >**:

    |Configuração|Valor|
    |---|---|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|**CONTOSO-VNET-RG**|
    |Nome|**FirewallPolicy_contoso-firewall**|
    |Region|o nome da região do Azure na qual você provisionou recursos anteriormente neste laboratório|
    |Nível de política|**Standard**|
    |Política pai|**Nenhuma**|

1. Na guia **Configurações de DNS**, aceite a opção padrão **Desabilitado** e selecione **Avançar: Inspeção do TLS >**.
1. Na guia **Inspeção do TLS**, selecione **Avançar: Regras >**.
1. Na guia **Regras**, selecione **+ Adicionar uma coleção de regras**.
1. No painel **Adicionar uma coleção de regras**, especifique as seguintes configurações:

    |Configuração|Valor|
    |---|---|
    |Nome|**AllowOutbound**|
    |Tipo de coleção de regras|**Rede**|
    |Prioridade|**101**|
    |Ação da coleção de regras|**Permitir**|
    |Grupo de coleções de regras|**DefaultNetworkRuleCollectionGroup**|

1. No painel **Adicionar uma coleção de regras**, na seção **Regras**, adicione uma regra com as seguintes configurações:

    |Configuração|Valor|
    |---|---|
    |Nome|**RHEL**|
    |Tipo de origem|**Endereço IP**|
    |Origem|*|
    |Protocolo|**Qualquer**|
    |Portas de Destino|*|
    |Tipo de destino|**Endereço IP**|
    |Destino|**13.91.47.76,40.85.190.91,52.187.75.218,52.174.163.213,52.237.203.198**|

    >**Observação**: Para identificar os endereços IP a serem usados para RHEL, confira [Preparar a rede para implantação de infraestrutura](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network)

    |Configuração|Valor|
    |---|---|
    |Nome|**ServiceTags**|
    |Tipo de origem|**Endereço IP**|
    |Origem|*|
    |Protocolo|**Qualquer**|
    |Portas de Destino|*|
    |Tipo de destino|**Marca de serviço**|
    |Destino|**AzureActiveDirectory, AzureKeyVault,Armazenamento**|

    >**Observação**: Se preferir, é possível usar marcas de serviço com escopos regionais. 

    |Configuração|Valor|
    |---|---|
    |Nome|**SUSE**|
    |Tipo de origem|**Endereço IP**|
    |Origem|*|
    |Protocolo|**Qualquer**|
    |Portas de Destino|*|
    |Tipo de destino|**Endereço IP**|
    |Destino|**52.188.224.179,52.186.168.210,52.188.81.163,40.121.202.140**|

    >**Observação**: Para identificar os endereços IP a serem usados para o SUSE, confira [Preparar a rede para implantação de infraestrutura](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network)

    |Configuração|Valor|
    |---|---|
    |Nome|**AllowOutbound**|
    |Tipo de origem|**Endereço IP**|
    |Origem|*|
    |Protocolo|**TCP,UDP,ICMP,Qualquer**|
    |Portas de Destino|*|
    |Tipo de destino|**Endereço IP**|
    |Destino|*|

1. Selecione o botão **Adicionar** para salvar todas as regras.
1. De volta à guia **Regras**, selecione **Avançar: IDPS >**.
1. Na guia **IDPS**, selecione **Avançar: Inteligência contra ameaças >**.

    >**Observação**: A funcionalidade IDPS requer o SKU Premium.

1. Na guia **Inteligência contra ameaças**, examine as configurações disponíveis sem fazer alterações e selecione **Examinar + criar**.
1. Na guia **Revisar + criar**, aguarde a conclusão do processo de validação e selecione **Criar**.

    >**Observação**: Aguarde a conclusão do provisionamento da política de firewall. O provisionamento deve levar cerca de 1 minuto.

1. No computador do laboratório, na janela do Microsoft Edge exibindo o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Endereços IP públicos**.
1. Na página **Endereços IP públicos**, selecione **+ Criar**.
1. Na guia **Básico** da página **Criar um endereço IP público**, especifique as seguintes configurações e selecione **Examinar + criar**:

    |Configuração|Valor|
    |---|---|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|**CONTOSO-VNET-RG**|
    |Region|o nome da região do Azure na qual você provisionou recursos anteriormente neste laboratório|
    |Nome|**contoso-firewal-pip**|
    |Versão IP|**IPv4**|
    |SKU|**Standard**|
    |Zona de disponibilidade|**Nenhuma zona**|
    |Camada|**Regional**|
    |Preferência de roteamento|**Rede da Microsoft**|
    |Tempo limite de ociosidade (minutos)|**4**|
    |Rótulo do nome DNS|não definido|

1. Na guia **Revisar + criar**, aguarde a conclusão do processo de validação e selecione **Criar**.

    >**Observação**: Aguarde a conclusão do provisionamento do endereço IP público. O provisionamento deve levar alguns segundos.

1. No computador do laboratório, na janela do Microsoft Edge que exibe o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Firewalls**.
1. Na página **Firewalls**, selecione **+ Criar**.
1. Na guia **Básico** da página **Criar um firewall**, especifique as seguintes configurações e selecione **Examinar + criar**:

    |Configuração|Valor|
    |---|---|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|**CONTOSO-VNET-RG**|
    |Nome|**contoso-firewall**|
    |Region|o nome da região do Azure na qual você provisionou recursos anteriormente neste laboratório|
    |Zona de disponibilidade|**Nenhuma**|
    |SKU do Firewall|**Standard**|
    |Gerenciamento do firewall|**Usar uma política de firewall para gerenciar este firewall**|
    |Política de firewall|**FirewallPolicy_contoso-firewall**|
    |Escolher uma rede virtual|**Usar existente**|
    |Rede virtual|**CONTOSO-VNET**|
    |Endereço IP público|**contoso-firewall-pip**|
    |Túnel forçado|**Desabilitado**|

    >**Observação**: Aguarde a conclusão do provisionamento do Firewall do Azure. O provisionamento pode levar cerca de 3 minutos.

1. No portal do Azure, navegue de volta para a página **Firewalls**.
1. Na página **Firewalls**, selecione a entrada **contoso-firewall**.
1. Na página **contoso-firewall**, observe a entrada **IP privado** definida como **10.5.3.4** representando o endereço IP privado da instância do Firewall do Azure.

    >**Observação**: Para que o tráfego de rede seja roteado por meio do Firewall do Azure, você precisa adicionar rotas definidas pelo usuário às tabelas de rotas associadas ao aplicativo e sub-redes de banco de dados da rede virtual que hospedará a implantação do SAP.

1. No portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Tabelas de rotas**.
1. Na página **Tabelas de rotas**, selecione a entrada **ACSS-ROUTE**.
1. Na página **ACSS-ROUTE**, selecione **Rotas**.
1. Na página **ACSS-ROUTE \| Rotas**, selecione **+ Adicionar**.
1. No painel **Adicionar rota**, especifique as seguintes configurações e selecione **Adicionar**:

    |Configuração|Valor|
    |---|---|
    |Nome da rota|**Firewall**|
    |Tipo de destino|**Endereços IP**|
    |Endereços IP de destino/intervalos CIDR|**0.0.0.0/0**|
    |Tipo do próximo salto|**Solução de virtualização**|
    |Endereço do próximo salto|**10.5.3.4**|

### Tarefa 9: Implante o Azure Bastion na rede virtual que hospedará a implantação

1. No computador do laboratório, na janela do Microsoft Edge que exibe o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Bastions**. 
1. Na página **Bastions**, selecione **+ Criar**.
1. Na guia **Básico** da página **Bastions**, especifique as seguintes configurações e selecione **Avançar: Marcas >**:

    |Configuração|Valor|
    |---|---|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|**CONTOSO-VNET-RG**|
    |Nome|**ACSS-BASTION**|
    |Region|o nome da região do Azure na qual você provisionou recursos anteriormente neste laboratório|
    |Camada|**Basic**|
    |Contagem de instâncias|**2**|
    |Rede virtual|**CONTOSO-VNET**|
    |Sub-rede|**AzureBastionSubnet**|
    |Endereço IP público|**Criar novo**|
    |Nome do endereço IP público|**ACSS-BASTION-PIP**|

1. Na guia **Marcas**, selecione **Avançar: Avançado >**
1. Na guia **Avançado**, examine as configurações disponíveis sem fazer alterações e selecione **Avançar: Examinar + criar >**
1. Na guia **Revisar + criar**, aguarde a conclusão do processo de validação e selecione **Criar**.

    >**Observação**: Não aguarde a conclusão do provisionamento do Bastion host. Em vez disso, prossiga para a próxima tarefa. O provisionamento pode levar cerca de 15 minutos.

## Exercício 2: Implantar a infraestrutura que hospedará cargas de trabalho SAP no Azure usando o Centro do Azure para soluções SAP

Duração: 40 minutos

Neste exercício, você usará as soluções do Centro do Azure para soluções SAP para implantar a infraestrutura que hospedará cargas de trabalho SAP na assinatura do Azure usada no exercício anterior. Após a implantação bem-sucedida, você pode prosseguir com a instalação do software SAP usando o Centro do Azure para soluções SAP ou excluir os recursos do Azure provisionados neste laboratório.

>**Observação**: Para obter informações sobre a instalação do software SAP usando o Centro do Azure para soluções SAP, consulte a documentação do Microsoft Learn que descreve como [Obter a mídia de instalação do SAP](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/get-sap-installation-media) e [Instalar o software SAP](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/install-software). As instruções para exclusão dos recursos do Azure provisionados neste laboratório estão incluídas na segunda tarefa deste exercício.

O exercício consiste na seguinte tarefa:

- Tarefa 1: Criar Instância Virtual para soluções SAP
- Tarefa 2: Excluir os recursos do Azure provisionados neste laboratório

### Tarefa 1: Criar Instância Virtual para soluções SAP

1. No computador do laboratório, na janela do Microsoft Edge exibindo o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Centro do Azure para soluções SAP**. 
1. Na página **Visão geral \| do Centro do Azure para Soluções SAP**, selecione** Criar um novo sistema SAP**.
1. Na guia **Básico** da página **Criar Instância Virtual para soluções SAP**, especifique as seguintes configurações e selecione **Avançar: Máquinas virtuais**

    |Configuração|Valor|
    |---|---|
    |Assinatura|O nome da assinatura do Azure que você está usando neste laboratório|
    |Grupo de recursos|O nome de um **novo** grupo de recursos **Contoso-SAP-C1S**|
    |Nome (SID)|**C1S**|
    |Region|o nome da região do Azure na qual você provisionou recursos anteriormente neste laboratório|
    |Tipo de ambiente|**Produção**|
    |Produto SAP|**S/4HANA**|
    |Backup de banco de dados|**HANA**|
    |Método de escala HANA|**Escalar verticalmente (recomendado)**|
    |Tipo de implantação|**Distribuída com Alta Disponibilidade (HA)**|
    |Disponibilidade de computação|**99,95 (conjunto de disponibilidade)**|
    |Rede virtual|**CONTOSO-VNET**|
    |Sub-rede do aplicativo|**aplicativo (10.5.0.0/24)**|
    |Sub-rede do banco de dados|**banco de dados (10.5.2.0/24)**|
    |Imagem do sistema operacional do aplicativo|**Red Hat Enterprise Linux 8.2 para aplicativos SAP – x64 Gen2 mais recente**|
    |Imagem do sistema operacional do banco de dados|**Red Hat Enterprise Linux 8.2 para aplicativos SAP - x64 Gen2 mais recente**|
    |Opção de transporte do SAP|**Criar um novo diretório de transporte SAP**|
    |Grupo de recursos de transporte|**ACSS-DEMO**|
    |Nome da conta de armazenamento|nenhuma entrada|
    |Tipo de autenticação|**SSH público**|
    |Nome de Usuário|**contososapadmin**|
    |Origem de chave pública SSH|**Gerar novo par de chaves**|
    |Nome do par de chaves|**contosoc1skey**|
    |SQP FQDN|**sap.contoso.com**|
    |Fonte de Identidade gerenciada|**Usar a identidade gerenciada atribuída ao usuário existente**|
    |Nome da identidade gerenciada|**Contoso-MSI**|

1. Na guia **Máquinas virtuais**, especifique as seguintes configurações:

    |Configuração|Valor|
    |---|---|
    |Gerar Recomendação baseada em|**Padrão de Desempenho do Aplicativo SAP (SAPS): selecione essa opção para fornecer o SAPS para a camada de aplicativo e o tamanho da memória do banco de dados e clique em Gerar Recomendações**|
    |SAPS para camada de aplicativo|**10000**|
    |Tamanho da memória do banco de dados (GiB)|**1024**|

1. Selecione **Gerar recomendação**.
1. Examine o tamanho e o número de VMs para as máquinas virtuais ASCS, aplicativo e banco de dados. 

    >**Observação**: Se necessário, ajuste os tamanhos recomendados selecionando o link **Ver todos os tamanhos** para cada conjunto de máquinas virtuais e escolhendo um tamanho alternativo. Por padrão, o tipo de implantação distribuída com alta disponibilidade, bem como a camada de aplicativo SAPS e o tamanho da memória do banco de dados especificados acima resultam nas seguintes recomendações de SKU de VM:
    - 2 x VMs Standard_E4ds_v4 para o ASCS (4 vCPUs e 32 GiB de memória cada)
    - 2 x VMs Standard_E4ds_v4 para as VMs do aplicativo (4 vCPUs e 32 GiB de memória cada)
    - 2 x VMs Standard_M64ms para as VMs de banco de dados (64 vCPUs e 1750 GiB de memória cada)

    >**Observação**: Para minimizar os requisitos de vCPU e memória para as VMs de banco de dados, considere alterar o SKU da VM para Standard_M32ts (32 vCPUs e 192 GiB de memória cada).

    >**Observação**: Se necessário, você pode solicitar aumento de cota selecionando o link **Cota de Solicitação** para um SKU específico de máquinas virtuais e enviando uma solicitação de aumento de cota. O processamento de uma solicitação normalmente leva alguns minutos.

    >**Observação**: o Centro do Azure para soluções SAP impõem o uso de SKUs de VM com suporte do SAP durante a implantação.

1. Na guia **Máquinas virtuais**, na seção **Discos de dados**, selecione o link **Exibir e personalizar a configuração**.
1. Na página de **Configuração do disco de banco de dados**, examine a configuração recomendada sem fazer alterações e selecione **Fechar**.
1. De volta à guia **Máquinas virtuais**, selecione **Avançar: Visualizar arquitetura**.
1. Na guia **Visualizar Arquitetura**, examine o diagrama que ilustra a arquitetura recomendada e selecione **Examinar + criar**.
1. Na guia **Revisar + criar**, aguarde a conclusão do processo de validação, marque a caixa de seleção para confirmar se você tem uma cota ampla disponível na região de implantação para evitar executar o erro "Cota Insuficiente" e selecione **Criar**.
1. Na janela pop-up **Gerar novo par de chaves**, selecione **Baixar chave privada e criar recurso**.

    >**Observação**: A chave privada necessária para se conectar às VMs do Azure incluídas na implantação será baixada para o computador do qual você está executando este laboratório.

    >**Observação**: aguarde até que a implantação seja concluída. Isso pode levar cerca de 25 minutos.

>**Observação**: Após a implantação, prossiga com a instalação do software SAP usando o Centro do Azure para soluções SAP ou exclua os recursos do laboratório seguindo as instruções da próxima tarefa.

### Tarefa 2: Excluir os recursos do Azure provisionados neste laboratório

>**Importante**: O custo dos recursos que você implantou é significativo, portanto, certifique-se de desprovisionar o laboratório se não pretender usá-lo além desse ponto. A exclusão da instância virtual para soluções SAP não excluirá os recursos de infraestrutura subjacentes. Para excluir os recursos, você deve usar o procedimento descrito nesta tarefa, que tem como destino os recursos em três grupos de recursos:

- **Contoso-SAP-C1S**
- **CONTOSO-VNET-RG**
- **ACSS-DEMO**

1. No computador do laboratório, na janela do Microsoft Edge que exibe o portal do Azure, selecione o ícone **Cloud Shell** e inicie uma sessão do PowerShell no Cloud Shell. 

    > **Observação**: Se esta for a primeira vez que você está iniciando o Cloud Shell na assinatura do Azure que será usada neste laboratório, será solicitado que você crie um compartilhamento de arquivo do Azure para persistir arquivos do Cloud Shell. Em caso afirmativo, aceite os padrões, o que resultará na criação de uma conta de armazenamento em um grupo de recursos gerado automaticamente.

1. No portal do Microsoft Azure, no painel **Cloud Shell**, no prompt do PowerShell, execute os seguintes comandos para parar e desalocar todas as VMs do Azure implantadas neste laboratório:

    ```powershell
    $resourceGroupName = 'Contoso-SAP-C1S'
    $vms = Get-AzVM -ResourceGroupName $resourceGroupName
    foreach ($vm in $vms) {
       Stop-AzVM -ResourceGroupName $resourceGroupName -Name $vm.Name -Force 
    }
    ```

1. No prompt do PowerShell, execute os seguintes comandos para desanexar todos os discos de dados de todas as VMs do Azure implantadas neste laboratório:

    ```powershell
    foreach ($vm in $vms) {  
       $vmDisks = $vm.StorageProfile.DataDisks
       foreach ($vmDisk in $vmDisks) {
          Remove-AzVMDataDisk -VM $vm -Name $vmDisk.Name
       }
       Update-AzVM -ResourceGroupName $resourceGroupName -VM $vm -ErrorAction SilentlyContinue
    }
    ```

1. No prompt do PowerShell, execute os seguintes comandos para habilitar a opção de exclusão de interfaces de rede e discos anexados a todas as VMs do Azure implantadas neste laboratório:

    ```powershell
    foreach ($vm in $vms) {
       $vmConfig = Get-AzVM -ResourceGroupName $resourceGroupName -Name $vm.Name
       $vmConfig.StorageProfile.OsDisk.DeleteOption = 'Delete'
       $vmConfig.StorageProfile.DataDisks | ForEach-Object { $_.DeleteOption = 'Delete' }
       $vmConfig.NetworkProfile.NetworkInterfaces | ForEach-Object { $_.DeleteOption = 'Delete' }
       $vmConfig | Update-AzVM
    }   
    ```

1. No prompt do PowerShell, execute os seguintes comandos para excluir todas as VMs do Azure implantadas neste laboratório:

    ```powershell
    foreach ($vm in $vms) {
       Remove-AzVm -ResourceGroupName $resourceGroupName -Name $vm.Name -ForceDeletion $true -Force
    }
    ```

1. No prompt do PowerShell, execute os seguintes comandos para excluir o grupo de recursos **Contoso-SAP-C1S** e todos os seus recursos restantes:

    ```powershell
    Remove-AzResourceGroup -Name 'Contoso-SAP-C1S' -Force -AsJob
    ```

1. No prompt do PowerShell, execute os seguintes comandos para excluir o grupo de recursos **CONTOSO-VNET-RG** e todos os seus recursos restantes:

    ```powershell
    Remove-AzResourceGroup -Name 'CONTOSO-VNET-RG' -Force -AsJob
    ```

1. No prompt do PowerShell, execute os seguintes comandos para excluir o grupo de recursos **ACSS-DEMO** e todos os seus recursos restantes:

    ```powershell
    Remove-AzResourceGroup -Name 'ACSS-DEMO' -Force -AsJob
    ```

    >**Observação**: Os três últimos comandos são executados de forma assíncrona (conforme determinado pelo parâmetro -AsJob), portanto, embora você possa executar o próximo comando do PowerShell imediatamente depois na mesma sessão do PowerShell, levará alguns minutos até que os grupos de recursos e seus recursos sejam realmente removidos.
