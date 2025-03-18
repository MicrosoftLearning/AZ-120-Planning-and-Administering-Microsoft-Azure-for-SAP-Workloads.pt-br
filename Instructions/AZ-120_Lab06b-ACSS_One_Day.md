---
lab:
  title: 06b – Visão geral da implantação e manutenção das soluções do Centro do Azure para soluções SAP (ACSS)
  module: Design and implement an infrastructure to support SAP workloads on Azure
---

# Módulo AZ 1006: Elaborar e implementar uma infraestrutura para dar suporte a cargas de trabalho SAP no Azure
# Laboratório de curso de um dia do AZ-1006: Visão geral da implantação e manutenção do Centro do Azure para soluções SAP (ACSS)

>**Importante**: **No momento, este laboratório não tem suporte** (24 de fevereiro).
    - Para obter uma implantação de pré-requisito manual detalhada, confira [Lab 6a](https://github.com/MicrosoftLearning/AZ-120-Planning-and-Administering-Microsoft-Azure-for-SAP-Workloads/blob/master/Instructions/AZ-120_Lab06a-ACSS_One_Day.md)
    - Para a implantação da infraestrutura de demonstração do ACSS, confira [Lab 6c(https://github.com/MicrosoftLearning/AZ-120-Planning-and-Administering-Microsoft-Azure-for-SAP-Workloads/blob/master/Instructions/AZ-120_Lab06c-ACSS_One_Day.md)].

Tempo Estimado: 100 minutos

Todas as tarefas neste laboratório de curso de 1 dia do AZ-1006 são realizadas no portal do Azure

## Objetivos

Depois de realizar este laboratório, você será capaz de:

- implementar os pré-requisitos mínimos para avaliar a implantação de cargas de trabalho SAP no Azure usando o Centro do Azure para soluções SAP
- implantar a infraestrutura que hospedará cargas de trabalho SAP no Azure usando o Centro do Azure para soluções SAP
- Manter as cargas de trabalho SAP no Azure juntamente com o uso de soluções do Centro do Azure para soluções SAP

## Instruções

### Exercício 1: Implementar os pré-requisitos mínimos para avaliar a implantação de cargas de trabalho SAP no Azure usando o Centro do Azure para soluções SAP

Duração: 30 minutos

Nesse exercício, você implementa os pré-requisitos mínimos para avaliar a implantação de cargas de trabalho SAP no Azure usando o Centro do Azure para soluções SAP. Isso inclui as seguintes atividades:

>**Importante**: os pré-requisitos implementados neste exercício *não* se destinam a representar as práticas recomendadas para implantar cargas de trabalho SAP no Azure usando o Centro do Azure para soluções SAP. Sua finalidade é minimizar o tempo, o custo e os recursos necessários para avaliar a mecânica de implantação de cargas de trabalho SAP no Azure usando o Centro do Azure para soluções SAP e executando tarefas de gerenciamento e manutenção pós-implantação. A implementação dos pré-requisitos inclui as seguintes atividades:

- Criar uma identidade gerenciada atribuída pelo usuário do Microsoft Entra a ser usada pelo Centro do Azure para soluções SAP para acesso ao Armazenamento do Azure durante sua implantação.
- Conceder a identidade gerenciada atribuída ao usuário do Microsoft Entra que é usada para realizar o acesso de implantação à assinatura do Azure e à conta V2 de Uso Geral do Armazenamento do Azure
- Criar a rede virtual do Azure que hospeda todas as máquinas virtuais do Azure incluídas na implantação.
- Criar e configurar um grupo de segurança de rede (NSG) usado para restringir o acesso de saída de sub-redes da rede virtual que hospeda a implantação.
- Criar e configurar um gateway da NAT que permitirá a conectividade de saída de VMs do Azure que fazem parte da implantação.

Essas atividades correspondem às seguintes tarefas desse exercício:

- Tarefa 1: criar uma identidade gerenciada atribuída pelo usuário do Microsoft Entra
- Tarefa 2: configurar atribuições de função controle de acesso baseado em função (RBAC) do Azure para a identidade gerenciada atribuída pelo usuário do Microsoft Entra ID
- Tarefa 3: Criar a rede virtual do Azure
- Tarefa 4: Criar e configurar um grupo de segurança de rede
- Tarefa 5: Criar e configurar um gateway da NAT

#### Tarefa 1: Criar uma identidade gerenciada atribuída pelo usuário do Microsoft Entra

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

   >**Observação**: Não espere que o processo de provisionamento seja concluído, mas prossiga para a próxima tarefa. O provisionamento deve levar apenas alguns segundos.

   >**Observação**: Em uma das próximas tarefas, você autorizará o acesso da identidade gerenciada à conta de armazenamento que hospeda a mídia de instalação do SAP para acomodar a instalação de software SAP por meio do Centro do Azure para soluções SAP.

#### Tarefa 2: Configure as atribuições de função do controle de acesso baseado em função (RBAC) do Azure para a conta de usuário do Microsoft Entra ID usada para realizar a implantação.

1. No computador do laboratório, na janela do navegador da Web que exibe o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Assinaturas**.
1. Na página **Assinaturas**, selecione a entrada que representa a assinatura do Azure que você usará para esse laboratório. 
1. Na página que exibe as propriedades da assinatura do Azure, selecione **Controle de acesso (IAM)**.
1. Na página **Controle de acesso (IAM),** selecione **+ Adicionar** e, no menu suspenso, selecione **Adicionar atribuição de função**.
1. Na guia **Função** da página **Adicionar atribuição de função**, na listagem **Funções de função de trabalho**, pesquise e selecione a entrada **função de serviço** do Centro do Azure para soluções SAP e selecione **Avançar**.
1. Na guia **Membros** da página **Adicionar atribuição de função**, para** atribuir acesso**, selecione **Identidade Gerenciada** e clique em + **Selecionar membros**.
1. No painel **Selecionar identidades gerenciadas**, especifique as seguintes configurações:

   |Configuração|Valor|
   |---|---|
   |Assinatura|O nome da assinatura do Azure usada nesse laboratório|
   |Identidade gerenciada|**Identidade gerenciada atribuída pelo usuário**|
   |Selecionar|**acss-infra-MI**|
1. Na lista de identidades gerenciadas, selecione a entrada **acss-infra-MI** e clique em **Selecionar**.
1. De volta à guia **Membros**, selecione **Examinar + atribuir**.  
1. Na guia **Revisão + atribuição**, selecione **Examinar + atribuir**.
   
   ##### Adicionar: "Identidade do administrador de soluções do Centro do Azure para soluções SAP"
   
1. Na folha **Controle de acesso (IAM),** selecione **+ Adicionar** e, no menu suspenso, selecione **Adicionar atribuição de função**.
1. Na guia **Função** da página **Adicionar atribuição de função**, na listagem **Funções de função de trabalho**, pesquise e selecione a entrada **Administrador do Centro do Azure para soluções SAP** e selecione **Avançar**.
1. Na guia **Membros**
   - em **Atribuir acesso a**, selecione **Usuário, Grupo ou Entidade de Serviço**
   - clique **em + Selecionar membros**.
1. No painel **Selecionar membros**, na caixa **Selecionar texto**, digite o nome da conta de usuário do Microsoft Entra ID usada para acessar a assinatura do Azure que você está usando para este laboratório, selecione-a na lista de resultados correspondentes à sua entrada e clique em **Selecionar**.
1. De volta à guia **Membros**, selecione **Examinar + atribuir**.
1. Na guia **Revisão + atribuição**, selecione **Examinar + atribuir**.
   
   ##### Adicionar: "Operador de Identidade Gerenciada"
   
1. Na folha **Controle de acesso (IAM),** selecione **+ Adicionar** e, no menu suspenso, selecione **Adicionar atribuição de função**.
1. Na guia **Função** da página **Adicionar atribuição de função**, na listagem **Funções de função de trabalho**, pesquise e selecione a entrada **Operador de Identidade Gerenciada** e selecione **Avançar**.
1. Na guia **Membros**
   - em **Atribuir acesso a**, selecione **Usuário, Grupo ou Entidade de Serviço**
   - clique **em + Selecionar membros**.
1. No painel **Selecionar membros**, na caixa **Selecionar texto**, digite o nome da conta de usuário do Microsoft Entra ID usada para acessar a assinatura do Azure que você está usando para este laboratório, selecione-a na lista de resultados correspondentes à sua entrada e clique em **Selecionar**.
1. De volta à guia **Membros**, selecione **Examinar + atribuir**.
1. Na guia **Revisão + atribuição**, selecione **Examinar + atribuir**.

#### Tarefa 3: Criar a rede virtual

Nessa tarefa, você cria a rede virtual do Azure que hospeda todas as máquinas virtuais do Azure incluídas na implantação. Além disso, na rede virtual, você cria as seguintes sub-redes:

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

1. Na guia **Segurança**, aceite as configurações padrão e selecione **Avançar**.

   >**Observação**: Neste momento, você pode provisionar o Azure Bastion e o Firewall do Azure; no entanto, você os provisionará separadamente depois que a rede virtual for criada.

1. Na guia **Endereços IP**, especifique as seguintes configurações e selecione **Examinar + criar**:

   |Configuração|Valor|
   |---|---|
   |Espaço de endereços IP|**10.0.0.0/16 (65,536 addresses)**|

1. Na lista de sub-redes, selecione o ícone da lixeira para excluir a sub-rede **padrão**.
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

   >**Observação**: Não espere que o processo de provisionamento seja concluído, mas prossiga para a próxima tarefa. O provisionamento deve levar apenas alguns segundos.

#### Tarefa 4: Criar e configurar um grupo de segurança de rede

Nessa tarefa, você cria e configura um grupo de segurança de rede (NSG) usado para restringir o acesso de saída de sub-redes da rede virtual que hospeda a implantação. Você pode fazer isso bloqueando a conectividade com a Internet, mas permitindo explicitamente conexões com os seguintes serviços:

- Pontos de extremidade de infraestrutura de atualização do SUSE ou Red Hat
- Armazenamento do Azure
- Cofre de Chave do Azure
- Microsoft Entra ID
- Azure Resource Manager

>**Observação**: Em geral, você deve considerar o uso do Firewall do Azure em vez de NSGs para proteger a conectividade de rede para sua implantação do SAP. 

1. No computador do laboratório, na janela do navegador da Web que exibe o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Grupos de segurança de rede**.
1. Na página **Grupos de segurança de rede**, selecione **+ Criar**.
1. Na guia **Básico** da página **Criar grupo** de segurança de rede, especifique as seguintes configurações e selecione **Examinar + criar**:

   |Configuração|Valor|
   |---|---|
   |Assinatura|O nome da assinatura do Azure usada nesse laboratório|
   |Grupo de recursos|**acss-infra-RG**|
   |Nome|**acss-infra-NSG**|
   |Region|o nome da mesma região do Azure que você usou anteriormente neste exercício|

1. Na guia **Revisar + criar**, aguarde a conclusão do processo de validação e selecione **Criar**.

   >**Observação**: Aguarde o processo de provisionamento ser concluído. O provisionamento deve levar menos de 1 minuto.

1. No painel **A implantação foi concluída**, selecione **Ir para o recurso**.

   >**Observação**: Por padrão, as regras internas de grupos de segurança de rede permitem todo o tráfego de saída, todo o tráfego dentro da mesma rede virtual, bem como todo o tráfego entre redes virtuais emparelhadas. Do ponto de vista da segurança, você deve considerar restringir esse comportamento padrão. A configuração proposta restringe a conectividade de saída para a Internet e o Azure. Você também pode usar regras NSG para restringir a conectividade em uma rede virtual.

1. Na página **acss-infra-NSG**, no menu de navegação, na seção **Configurações**, selecione **Regras de segurança de saída**.
1. Na página **Regras de segurança de saída \|acss-infra-NSG**, selecione **+ Adicionar**.
1. No painel **Adicionar regra de segurança de saída**, especifique as seguintes configurações e selecione **Adicionar**:

   >**Observação**: A regra a seguir deve ser adicionada para permitir explicitamente a conectividade com os pontos de extremidade de infraestrutura de atualização do Red Hat.

   >**Observação**: Para identificar os endereços IP a serem usados para RHEL, confira [Preparar a rede para implantação de infraestrutura](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network#allowlist-suse-or-red-hat-endpoints)

   |Configuração|Valor|
   |---|---|
   |Fonte|**Qualquer**|
   |Intervalos de portas de origem|*|
   |Destino|**Endereços IP**|
   |Intervalos de CIDR /endereço IP de destino|**13.91.47.76,40.85.190.91,52.187.75.218,52.174.163.213,52.237.203.198,52.136.197.163,20.225.226.182,52.142.4.99,20.248.180.252,20.24.186.80**|
   |Serviço|**Personalizado**|
   |Intervalos de portas de destino|*|
   |Protocolo|**Qualquer**|
   |Ação|**Permitir**|
   |Prioridade|**300**|
   |Nome|**AllowAnyRHELOutbound**|
   |Descrição|**Permitir conectividade de saída com pontos de extremidade de infraestrutura de atualização do RHEL**|

1. Na página **Regras de segurança de saída \|acss-infra-NSG**, selecione **+ Adicionar**.
1. No painel **Adicionar regra de segurança de saída**, especifique as seguintes configurações e selecione **Adicionar**:

   >**Observação**: A regra a seguir deve ser adicionada para permitir explicitamente a conectividade com pontos de extremidade de infraestrutura de atualização do SUSE.

   >**Observação**: Para identificar os endereços IP a serem usados para SUSE, confira [Preparar a rede para implantação de infraestrutura](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network#allowlist-suse-or-red-hat-endpoints)

   |Configuração|Valor|
   |---|---|
   |Fonte|**Qualquer**|
   |Intervalos de portas de origem|*|
   |Destino|**Endereços IP**|
   |Intervalos de CIDR /endereço IP de destino|**52.188.224.179,52.186.168.210,52.188.81.163,40.121.202.140**|
   |Serviço|**Personalizado**|
   |Intervalos de portas de destino|*|
   |Protocolo|**Qualquer**|
   |Ação|**Permitir**|
   |Prioridade|**305**|
   |Nome|**AllowAnySUSEOutbound**|
   |Descrição|**Permitir conectividade de saída com pontos de extremidade de infraestrutura de atualização do SUSE**|

   >**Observação**: A regra a seguir deve ser adicionada para permitir explicitamente a conectividade com o Armazenamento do Microsoft Azure.

1. Na página **Regras de segurança de saída \|acss-infra-NSG**, selecione **+ Adicionar**.
1. No painel **Adicionar regra de segurança de saída**, especifique as seguintes configurações e selecione **Adicionar**:

   |Configuração|Valor|
   |---|---|
   |Fonte|**Qualquer**|
   |Intervalos de portas de origem|*|
   |Destino|**Marca de serviço**|
   |Marca de serviço de destino|**Storage**|
   |Serviço|**Personalizado**|
   |Intervalos de portas de destino|*|
   |Protocolo|**Qualquer**|
   |Ação|**Permitir**|
   |Prioridade|**400**|
   |Nome|**AllowAnyCustomStorageOutbound**|
   |Descrição|**Permitir conectividade de saída com o Armazenamento do Microsoft Azure**|

   >**Observação**: Você pode substituir a marca de serviço **Armazenamento** por uma que seja específica da região, como **Storage.EastUS**.

   >**Observação**: A regra a seguir deve ser adicionada para permitir explicitamente a conectividade com o Azure Key Vault.

1. Na página **Regras de segurança de saída \|acss-infra-NSG**, selecione **+ Adicionar**.
1. No painel **Adicionar regra de segurança de saída**, especifique as seguintes configurações e selecione **Adicionar**:

   |Configuração|Valor|
   |---|---|
   |Fonte|**Qualquer**|
   |Intervalos de portas de origem|*|
   |Destino|**Marca de serviço**|
   |Marca de serviço de destino|**AzureKeyVault**|
   |Serviço|**Personalizado**|
   |Intervalos de portas de destino|*|
   |Protocolo|**Qualquer**|
   |Ação|**Permitir**|
   |Prioridade|**500**|
   |Nome|**AllowAnyCustomKeyVaultOutbound**|
   |Descrição|**Permitir conectividade de saída com o Azure Key Vault**|

   >**Observação**: A regra a seguir deve ser adicionada para permitir explicitamente a conectividade com o Microsoft Entra ID.

1. Na página **Regras de segurança de saída \|acss-infra-NSG**, selecione **+ Adicionar**.
1. No painel **Adicionar regra de segurança de saída**, especifique as seguintes configurações e selecione **Adicionar**:

   |Configuração|Valor|
   |---|---|
   |Fonte|**Qualquer**|
   |Intervalos de portas de origem|*|
   |Destino|**Marca de serviço**|
   |Marca de serviço de destino|**AzureActiveDirectory**|
   |Serviço|**Personalizado**|
   |Intervalos de portas de destino|*|
   |Protocolo|**Qualquer**|
   |Ação|**Permitir**|
   |Prioridade|**600**|
   |Nome|**AllowAnyCustomEntraIDOutbound**|
   |Descrição|**Permitir conectividade de saída com a ID do Microsoft Entra**|

   >**Observação**: A regra a seguir deve ser adicionada para permitir explicitamente a conectividade com o Azure Resource Manager.

1. Na página **Regras de segurança de saída \|acss-infra-NSG**, selecione **+ Adicionar**.
1. No painel **Adicionar regra de segurança de saída**, especifique as seguintes configurações e selecione **Adicionar**:

   |Configuração|Valor|
   |---|---|
   |Fonte|**Qualquer**|
   |Intervalos de portas de origem|*|
   |Destino|**Marca de serviço**|
   |Marca de serviço de destino|**AzureResourceManager**|
   |Serviço|**Personalizado**|
   |Intervalos de portas de destino|*|
   |Protocolo|**Qualquer**|
   |Ação|**Permitir**|
   |Prioridade|**700**|
   |Nome|**AllowAnyCustomARMOutbound**|
   |Descrição|**Permitir conectividade de saída com o Azure Resource Manager**|

   >**Observação**: A última regra deve ser adicionada para bloquear explicitamente a conectividade com a Internet.

1. Na página **Regras de segurança de saída \|acss-infra-NSG**, selecione **+ Adicionar**.
1. No painel **Adicionar regra de segurança de saída**, especifique as seguintes configurações e selecione **Adicionar**:

   |Configuração|Valor|
   |---|---|
   |Fonte|**Qualquer**|
   |Intervalos de portas de origem|*|
   |Destino|**Marca de serviço**|
   |Marca de serviço de destino|**Internet**|
   |Serviço|**Personalizado**|
   |Intervalos de portas de destino|*|
   |Protocolo|**Qualquer**|
   |Ação|**Deny**|
   |Prioridade|**1000**|
   |Nome|**DenyAnyCustomInternetOutbound**|
   |Descrição|**Negar a conectividade de saída com a Internet**|

   >**Observação**: Por fim, você precisa atribuir o NSG às sub-redes relevantes da rede virtual que hospedará a implantação do SAP.

1. No painel **Adicionar regra de segurança de saída**, no menu de navegação, na seção **Configurações**, selecione **Sub-redes**.
1. Na página **Sub-redes \| acss-infra-NSG**, selecione **+ Associar**.
1. No painel **Associar sub-rede**, na lista suspensa **Rede virtual**, selecione **acss-intra-VNET (acss-infra-RG)**. Na lista suspensa **Sub-rede**, selecione **aplicativo** e selecione **OK**.
1. No painel **Associar sub-rede**, na lista suspensa **Rede virtual**, selecione **acss-intra-VNET (acss-infra-RG)**. Na lista suspensa **Sub-rede**, selecione **db** e selecione **OK**.

#### Tarefa 5: Criar e configurar um gateway da NAT

Nessa tarefa, você irá criar e configurar um gateway da conversão de endereços de rede (NAT) que permitirá que as VMs do Azure incluídas na implantação alcancem serviços externos, como pontos de extremidade SUSE e RHEL.

1. No computador do laboratório, na janela do navegador da Web que exibe o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Gateways da NAT**.
1. Na página de **Gateways da NAT**, selecione **+ Criar**.
1. Na guia **Noções básicas** da página **Criar conversão de endereço de rede (NAT)**, especifique as seguintes configurações e selecione **Avançar: IP de saída >**:

   |Configuração|Valor|
   |---|---|
   |Assinatura|O nome da assinatura do Azure usada nesse laboratório|
   |Grupo de recursos|**acss-infra-RG**|
   |Nome do gateway da NAT|**acss-infra-NATGW**|
   |Region|o nome da mesma região do Azure que você usou anteriormente nesse exercício|
   |Zona de disponibilidade|**Nenhuma Zona**|
   |Tempo limite ocioso do TCP (minutos)|**4**|

1. Na guia **IP de saída**, abaixo da lista suspensa **Endereços IP públicos**, selecione o link **Criar um novo endereço IP público**, no painel **Adicionar um endereço IP público**, na caixa de texto **Nome**, digite **acss-infra-NATWG-PIP** e selecione**OK **.
1. De volta à guia **IP de saída**, selecione **Avançar: Sub-rede >**.
1. Na guia **Sub-rede**, na lista suspensa **Rede virtual**, selecione **acss-infra-VNET**. Na lista de sub-redes, selecione a caixa de seleção ao lado das entradas **aplicativo** e **banco de dados** e selecione **Examinar + criar**:
1. Na guia **Revisar + criar**, aguarde a conclusão do processo de validação e selecione **Criar**.

   >**Observação**: Aguarde o processo de provisionamento ser concluído. O provisionamento deve levar cerca de um minuto.

### Exercício 2: Implantar a infraestrutura que hospedará cargas de trabalho SAP no Azure usando o Centro do Azure para soluções SAP

Duração: 40 minutos

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
   |Grupo de recursos|O nome de um **novo** grupo de recursos **acss-vi-RG**|
   |Nome (SID)|**VI1**|
   |Region|o nome da região do Azure que hospeda a implantação SAP registrada pelo ACSS ou outra região na mesma região geográfica|
   |Tipo de ambiente|**Não produção**|
   |Produto SAP|**S/4HANA**|
   |Backup de banco de dados|**HANA**|
   |Método de escala HANA|**Escalar verticalmente (recomendado)**|
   |Tipo de implantação|**Distribuída com Alta Disponibilidade (HA)**|
   |Disponibilidade de computação|**99,95 (conjunto de disponibilidade)**|
   |Rede virtual|**acss-infra-VNET**|
   |Sub-rede do aplicativo|**app (10.0.2.0/24)**|
   |Sub-rede do banco de dados|**db (10.0.3.0/24)**|
   |Opções de imagem do sistema operacional do aplicativo|**Usar uma imagem do marketplace**|
   |Imagem do sistema operacional do aplicativo|**Red Hat Enterprise Linux 8.4 para Aplicativos SAP — x64 Gen2 mais recente**|
   |Opções de imagem do sistema operacional de banco de dados|**Usar uma imagem do marketplace**|
   |Imagem do sistema operacional do banco de dados|**Red Hat Enterprise Linux 8.4 para Aplicativos SAP — x64 Gen2 mais recente**|
   |Opções de transporte SAP|**Não incluir o diretório de transporte SAP**|
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
   - 2 x Standard_D4ds_v5 para as VMs do ASCS (4 vCPUs e 16 GiB de memória cada)
   - 2 x Standard_D4ds_v5 para as VMs do aplicativo (4 vCPUs e 16 GiB de memória cada)
   - 2 x Standard_E16ds_v5 para as VMs de banco de dados (16 vCPUs e 128 GiB de memória cada)

   >**Observação**: Se necessário, você pode solicitar aumento de cota selecionando o link **Cota de Solicitação** para um SKU específico de máquinas virtuais e enviando uma solicitação de aumento de cota. O processamento de uma solicitação normalmente leva alguns minutos.

   >**Observação**: o Centro do Azure para soluções SAP impõem o uso de SKUs de VM com suporte do SAP durante a implantação.

1. Na guia **Máquinas virtuais**, na seção **Discos de dados**, selecione o link **Exibir e personalizar a configuração**.
1. Na página de **Configuração do disco de banco de dados**, examine a configuração recomendada sem fazer alterações e selecione **Fechar**.
1. De volta à guia **Máquinas virtuais**, selecione **Avançar: Visualizar arquitetura**.
1. Na guia **Visualizar Arquitetura**, examine o diagrama que ilustra a arquitetura recomendada e selecione **Examinar + criar**.
1. Na guia **Revisar + criar**, aguarde a conclusão do processo de validação, marque a caixa de seleção para confirmar se você tem uma cota ampla disponível na região de implantação para evitar executar o erro "Cota Insuficiente" e selecione **Criar**.
1. Na janela pop-up **Gerar novo par de chaves**, selecione **Baixar chave privada e criar recurso**.

   >**Observação**: A chave privada necessária para se conectar às VMs do Azure incluídas na implantação será baixada para o computador do qual você está executando este laboratório.

   >**Observação**: aguarde até que a implantação seja concluída. Isso pode levar cerca de 25 minutos.

   >**Observação**: Após a implantação, você pode continuar a instalar o software SAP usando as soluções do Centro do Azure para soluções SAP. Nesse laboratório, você irá explorar os recursos do Centro do Azure para soluções SAP sem instalar o software SAP.

### Exercício 3: Manter cargas de trabalho SAP no Azure usando o Centro do Azure para soluções SAP

Duração: 60 minutos

Nesse exercício, você examinará o gerenciamento pós-implantação e o monitoramento de cargas de trabalho SAP usando o Centro do Azure para soluções SAP. Isso inclui as seguintes atividades:

- Implementar pré-requisitos para backup de cargas de trabalho SAP gerenciadas pelo Centro do Azure para soluções SAP 
- Implementação pré-requisitos para recuperação de desastre de cargas de trabalho SAP gerenciadas pelo Centro do Azure para soluções SAP 
- Revisar as opções de monitoramento disponíveis para cargas de trabalho SAP gerenciadas pelo Centro do Azure para soluções SAP 
- Excluir todos os recursos do Azure provisionados neste laboratório.

Essas atividades correspondem às seguintes tarefas desse exercício:

- Tarefa 1: Implementar pré-requisitos para backup de cargas de trabalho SAP gerenciadas pelo Centro do Azure para soluções SAP 
- Tarefa 2: Implementar pré-requisitos para recuperação de desastre de cargas de trabalho SAP gerenciadas pelo Centro do Azure para soluções SAP 
- Tarefa 3: Examinar as opções de monitoramento para cargas de trabalho SAP gerenciadas pelo Centro do Azure para soluções SAP 
- Tarefa 4: Excluir os recursos do Azure provisionados nesse laboratório

#### Tarefa 1: Implementar pré-requisitos para backup de cargas de trabalho SAP gerenciadas pelo Centro do Azure para soluções SAP

>**Observação**: Ao configurar o Backup do Azure no nível de recurso vis nas soluções do Centro do Azure para soluções SAP, você pode, em uma etapa, habilitar o backup para VMs do Azure que hospedam o banco de dados, os servidores de aplicativos e a instância do Serviços do Centro do SAP e para o BD HANA. Para o backup do BD HANA, o Centro do Azure para soluções SAP executa automaticamente o script de pré-registro de backup.

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
1. Na guia **Revisar + criar**, aguarde a conclusão do processo de validação e selecione **Criar**.

   >**Observação**: Aguarde o processo de provisionamento ser concluído. O provisionamento pode levar cerca de dois minutos.

1. No painel **A implantação foi concluída**, selecione **Ir para o recurso**.
1. Na página **acss-backup-RSV**, no menu de navegação vertical no lado esquerdo, na seção **Gerenciar**, selecione **Políticas de Backup**.
1. Na página **Políticas de Backup\| do acss-backup-RSV**, selecione **+ Adicionar**.
1. Na página **Selecionar tipo de política**, examine os tipos de política disponíveis.

   >**Observação**: Você começará configurando a política de backup para VMs do Azure que hospedam o banco de dados, os servidores de aplicativos e a instância dos Serviços Centrais do SAP.

1. Na página **Selecionar tipo de política**, selecione **Máquina Virtual do Azure**.
1. Na página **Criar política**, examine as diferenças entre os subtipos de política **Standard** e **Avançado** e selecione **Avançado**.

   >**Observação**: Você deve basear sua escolha em suas necessidades de resiliência. Da mesma forma, você deve ajustar os valores a seguir listados de acordo com suas necessidades.

1. Na caixa de texto **Nome da política**, digite**acss-vm-enhanced-backup-policy**.
1. Defina a frequência de agendamento de backup como **Por hora**, hora de início para **18h**, agendamento para **A cada 6 horas**, duração de **12 horas** e fuso horário para o fuso horário local.
1. Defina **Reter os instantâneos de recuperação da instância para o valor ** como **5** dias.
1. Defina **Retenção de pontos de backup diários** como **60** dias.
1. Defina o período de retenção de pontos de backup semanais, mensais e anuais com base em suas preferências.

   >**Observação**: Para habilitar o nivelamento, você precisa habilitar pontos de backup mensais ou anuais.

   >**Observação**: você também tem a opção de designar a convenção de nomenclatura do grupo de recursos gerado automaticamente pelo Backup do Azure.

1. Selecione **Criar**.
1. No painel **A implantação foi concluída**, selecione **Ir para o recurso**.
1. Na página **acss-backup-RSV**, no menu de navegação vertical no lado esquerdo, na seção **Gerenciar**, selecione **Políticas de Backup**.
1. Na página **Políticas de Backup\| do acss-backup-RSV**, selecione **+ Adicionar**.

   >**Observação**: Em seguida, você configurará as políticas de backup para o BD do HANA. A configuração apresentada aqui segue as diretrizes descritas no artigo do MS Learn [Fazer backup de instantâneos de instância de banco de dados SAP HANA em VMs do Azure](https://learn.microsoft.com/en-us/azure/backup/sap-hana-database-instances-backup).

1. Na página **Selecionar tipo de política**, selecione ** SAP HANA na VM do Azure (Banco de Dados via Backint)**
1. Na página **Criar política**, na caixa de texto **Nome da política**, digite **acss-hanadb-backint-backup-policy**.
1. Aceite as configurações padrão para os backups completos e de log. Mantenha os backups diferenciais e incrementais desabilitados.

   >**Observação**: Você tem a opção de habilitar a compactação de backup do HANA e mover pontos de recuperação qualificados para o arquivo morto do cofre. 

1. Selecione **Criar**.

1. No painel **A implantação foi concluída**, selecione **Ir para o recurso**.
1. Na página **acss-backup-RSV**, no menu de navegação vertical no lado esquerdo, na seção **Gerenciar**, selecione **Políticas de Backup**.
1. Na página **Políticas de Backup\| do acss-backup-RSV**, selecione **+ Adicionar**.
1. Na página **Selecionar tipo de política**, selecione **SAP HANA na VM do Azure (Instância de BD por meio de instantâneo)**
1. Na página **Criar política**, na caixa de texto **Nome da política**, digite **acss-hanadb-snapshot-backup-policy**.
1. Defina a frequência do backup de instantâneo como 13h30 do fuso horário atual.
1. Configure para manter o instantâneo de recuperação instantânea por **Cinco** dias.
1. Mantenha a seleção padrão do grupo de recursos de instantâneo, que deve ser definido como **acss-mgmt-RG**.
1. Na seção **Identidade Gerenciada**, selecione **Criar Identidade Gerenciada**. Isso abre automaticamente outra guia na janela do navegador da Web, exibindo a página **Identidade Gerenciada** no portal do Azure.
1. Na página **Identidade Gerenciada**, selecione **+ Criar**.
1. Na guia **Noções básicas** da página **Criar Identidade Gerenciada Atribuída ao Usuário**, especifique as seguintes configurações e selecione **Examinar + Criar**:

   |Configuração|Valor|
   |---|---|
   |Assinatura|O nome da assinatura do Azure usada nesse laboratório|
   |Grupo de recursos|**acss-mgmt-RG**|
   |Region|o nome da região do Azure que hospeda a implantação do ACSS|
   |Nome|**acss-mgmt-MI**|

1. Na guia **Revisão**, aguarde a conclusão do processo de validação e selecione **Criar**.

   >**Observação**: Aguarde o processo de provisionamento ser concluído. O provisionamento deve levar apenas alguns segundos.

1. Feche a guia atual do navegador e volte para aquela que exibe a página **Criar política**.
1. Na seção **Identidade Gerenciada**, na lista suspensa **Grupo de recursos**, selecione **acss-mgmt-RG** e, na lista suspensa **Identidade gerenciada**, selecione **acss-mgmt-MI**.
1. Selecione **Criar**.

   >**Observação**: ao configurar o backup no nível do VIS na interface do Centro do Azure para soluções SAP, você poderá aproveitar o cofre existente e suas políticas.

   >**Observação**: Depois que o backup for configurado no nível do VIS, você poderá monitorar o status dos trabalhos de backup das VMs do Azure e do BD HANA da interface do VIS no portal do Azure.

#### Tarefa 2: Implementar pré-requisitos para recuperação de desastre de cargas de trabalho SAP gerenciadas pelo Centro do Azure para soluções SAP 

>**Observação**: Embora o serviço de soluções do Centro do Azure para soluções SAP seja um serviço com redundância de zona, não há failover iniciado pela Microsoft no caso de uma interrupção de região. Para corrigir esse cenário, você deve configurar a recuperação de desastre para sistemas SAP implantados usando o Centro do Azure para soluções SAP seguindo as diretrizes descritas na [Visão geral de recuperação de desastres e nas diretrizes de infraestrutura para a carga de trabalho SAP](https://learn.microsoft.com/en-us/azure/sap/workloads/disaster-recovery-overview-guide), que envolve o uso do Azure Site Recovery (ASR). Nessa tarefa, você percorrerá o processo de implementação de uma solução de recuperação de desastre baseada em ASR que depende dessas diretrizes.

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

   >**Observação**: Você também tem a opção de configurar a reserva de capacidade.

   >**Importante**: Observe que os espaços de endereço IP diferem entre a rede virtual nas regiões primária e secundária. Isso é intencional, pois permitirá conectar as duas redes virtuais juntas, o que é necessário para configurar a replicação entre servidores de banco de dados hospedados nas duas regiões. Essa conexão pode ser estabelecida usando o emparelhamento de rede virtual.

1. De volta à guia **Configurações de Replicação** da página **Habilitar replicação**, selecione **Avançar**.
1. Na guia **Gerenciar** da página **Habilitar replicação**, execute as seguintes ações:

   1. Abaixo da lista suspensa **Política de replicação**, selecione **Criar novo**.
   1. No painel **Criar política de replicação**, na caixa de texto **Nome**, digite**2-day-retention-policy**, na caixa de texto **Período de Retenção (em dias)**, digite**2**e selecione **OK**.
   1. Você tem a opção de criar grupos de replicação. Essa opção não é aplicável ao nosso caso de uso.
   1. Deixe a opção **Atualizar configurações** das **Configurações de extensão ** configuradas para **Permitir que o ASR gerencie**.
   1. Aceite o nome padrão atribuído automaticamente da **Conta de Automação**.

1. Na guia **Gerenciar** da página **Habilitar replicação**, selecione **Avançar**.
1. Na guia **Revisão** da página **Habilitar replicação**, selecione **Habilitar replicação**.

   >**Observação**: Agora você percorrerá a habilitação da replicação para as duas VMs restantes do Azure que hospedam os servidores de banco de dados.

1. De volta à página **acss-dr-RSV\| Site Recovery**, na seção **Máquinas virtuais do Azure**, selecione **1. Habilitar a replicação**. 
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

   >**Observação**: Você precisa configurar a replicação para as duas VMs restantes do Azure separadamente, pois elas estão conectadas a uma sub-rede diferente.

1. Na guia **Configurações de Replicação**, execute as seguintes ações:

   1. Se necessário, na lista de seleção **Local de destino**, selecione a região do Azure na qual você criou o cofre dos Serviços de Recuperação **acss-dr-RSV**.
   1. Verifique se o nome da assinatura do Azure usada neste laboratório aparece na lista seleção de **Assinatura de destino**.
   1. Na lista suspensa **Grupo de recursos de destino**, selecione **acss-dr-RG**.
   1. Na lista suspensa **Rede virtual de failover**, selecione **CONTOSO-VNET**.
   1. Na lista suspensa **Sub-rede de failover**, selecione **aplicativo (10.10.0.0/24)**.
   1. Na seção **Armazenamento**, selecione o link **Exibir/editar configuração de armazenamento**.
   1. Na página **Personalizar configurações de destino**, examine a configuração resultante, mas não faça alterações e selecione **Cancelar**.
   1. Na seção **Opções de disponibilidade**, selecione o link **Exibir/editar opções de disponibilidade**.
   1. Na página **Opções de disponibilidade**, você tem a opção de implementar grupos de posicionamento por proximidade para recursos de destino, mas não faça nenhuma alteração e selecione **Cancelar**.

   >**Observação**: Você também tem a opção de configurar a reserva de capacidade.

1. De volta à guia **Configurações de Replicação** da página **Habilitar replicação**, selecione **Avançar**.
1. Na guia **Gerenciar** da página **Habilitar replicação**, execute as seguintes ações:

   1. Abaixo da lista suspensa **Política de replicação**, selecione **Criar novo**.
   1. No painel **Criar política de replicação**, na caixa de texto **Nome**, digite**2-day-retention-policy**, na caixa de texto **Período de Retenção (em dias)**, digite**2**e selecione **OK**.
   1. Você tem a opção de criar grupos de replicação. Essa opção não é aplicável ao nosso caso de uso.
   1. Deixe a opção **Atualizar configurações** das **Configurações de extensão ** configuradas para **Permitir que o ASR gerencie**.
   1. Aceite o nome padrão atribuído automaticamente da **Conta de Automação**.

1. Na guia **Gerenciar** da página **Habilitar replicação**, selecione **Avançar**.
1. Na guia **Revisão** da página **Habilitar replicação**, selecione **Habilitar replicação**.

   >**Observação**: a replicação inicial pode levar um tempo considerável para ser concluída. Considerando o tempo limitado alocado para este laboratório, consulte o instrutor sobre as etapas extras a serem executadas como parte dessa tarefa. Na ausência de orientações específicas, prossiga diretamente para a próxima tarefa.

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
