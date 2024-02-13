---
lab:
  title: '06a: visão geral dos pré-requisitos para a implantação do Centro do Azure para soluções SAP (ACSS)'
  module: Design and implement an infrastructure to support SAP workloads on Azure
---

# Módulo AZ 1006: Projetar e implementar uma infraestrutura para dar suporte a cargas de trabalho SAP no Azure
# Laboratório de curso de 1 dia do AZ-1006: Visão geral dos pré-requisitos para a implantação de cargas de trabalho SAP com o Centro do Azure para soluções SAP (ACSS)

Tempo Estimado: 100 minutos

Todas as tarefas neste laboratório de curso de 1 dia do AZ-1006 são realizadas no portal do Azure

## Objetivos

Depois de realizar este laboratório, você será capaz de:

- Implementar pré-requisitos para a implantação de cargas de trabalho SAP no Azure usando o Centro do Azure para soluções SAP

## Instruções

### Exercício 1: Implementar pré-requisitos para a implantação de cargas de trabalho SAP no Azure usando o Centro do Azure para soluções SAP

Duração: 60 minutos

Neste exercício, examine e implemente os pré-requisitos para implantar cargas de trabalho SAP no Azure, usando o Centro do Azure para soluções SAP. Isso inclui as seguintes atividades:

- Criação de uma identidade gerenciada atribuída pelo usuário do Microsoft Entra a ser usada pelo Centro do Azure para soluções SAP para acesso ao Armazenamento do Microsoft Azure durante sua implantação.
- Criar a rede virtual do Azure que hospeda todas as Máquinas Virtuais do Azure incluídas na implantação.
- Criando um recurso do Azure Bastion para proteger a conectividade com as VMs do Azure a partir da Internet.
- Criar uma conta de Uso Geral v2 do Armazenamento do Microsoft Azure que esteja associada ao Centro do Azure para soluções SAP usado para a implantação
- Concedendo à identidade gerenciada atribuída pelo usuário do Microsoft Entra, que é usada para realizar a implantação, acesso à assinatura do Azure e à conta de Uso Geral v2 do Armazenamento do Microsoft Azure
- Criando uma conta de compartilhamento de arquivos Premium do Azure usada para implementar o Diretório de Transporte do SAP
- Criação e configuração de um grupo de segurança de rede (NSG) usado para restringir o acesso de saída de sub-redes da rede virtual que hospeda a implantação.
- Criar uma Máquina Virtual do Azure (VM) que será usada para a instalação do software SAP como parte de uma implantação do Centro do Azure para soluções SAP.
- Conectando-se à VM do Azure usando o Azure Bastion e configurando-o para a instalação do software SAP.
- Exclusão de todos os recursos do Azure provisionados neste laboratório.

Essas atividades correspondem às seguintes tarefas deste exercício:

- Tarefa 1: Criar uma identidade gerenciada atribuída pelo usuário do Microsoft Entra
- Tarefa 2: Criar a rede virtual do Azure
- Tarefa 3: Criar um recurso do Azure Bastion
- Tarefa 4: Criar uma conta de Uso Geral v2 de Armazenamento do Microsoft Azure
- Tarefa 5: Configurar a autorização da identidade gerenciada atribuída pelo usuário do Microsoft Entra
- Tarefa 6: Criar uma conta de compartilhamentos de arquivos Premium do Azure
- Tarefa 7: Criar e configurar um grupo de segurança de rede
- Tarefa 8: Criar uma Máquina Virtual do Azure
- Tarefa 9: Configurar a Máquina Virtual do Azure
- Tarefa 10: Remover recursos do Azure

#### Tarefa 1: Criar uma identidade gerenciada atribuída pelo usuário do Microsoft Entra

Nessa tarefa, crie uma identidade gerenciada atribuída pelo usuário do Microsoft Entra a ser usada pelo Centro do Azure para soluções SAP para acesso ao Armazenamento do Microsoft Azure durante sua implantação.

1. No computador do laboratório, inicie um navegador da Web, navegue até o portal do Azure em `https://portal.azure.com` e autentique-se usando uma conta da Microsoft ou uma conta do Microsoft Entra ID com a função Proprietário na assinatura do Azure que você usa nesse laboratório.
1. Na janela do navegador da Web que exibe o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Identidades Gerenciadas**.
1. Na página **Identidades gerenciadas**, selecione **+ Criar**.
1. Na guia **Básico** da página **Criar identidade gerenciada atribuída pelo usuário**, especifique as seguintes configurações e selecione **Examinar + criar**:

   |Configuração|Valor|
   |---|---|
   |Assinatura|O nome da assinatura do Azure que você usará neste laboratório|
`   |Grupo de recursos| Escolha ou crie um novo **acss-infra-RG**|
   |Region|o nome da região do Azure, que você usa para a implantação do ACSS|
   |Nome|**acss-infra-MI**|

1. Na guia **Revisar**, aguarde a conclusão do processo de validação e selecione **Criar**.

   >**Observação**: Não espere que o processo de provisionamento seja concluído, mas prossiga para a próxima tarefa. O provisionamento deve levar apenas alguns segundos.

   >**Observação**: Em uma das próximas tarefas, você autorizará o acesso da identidade gerenciada à conta de armazenamento que hospeda a mídia de instalação do SAP para acomodar a instalação de software SAP por meio do Centro do Azure para soluções SAP.

#### Tarefa 2: Criar a rede virtual

Nesta tarefa, crie a rede virtual do Azure que hospeda todas as Máquinas Virtuais do Azure incluídas na implantação. Além disso, dentro da rede virtual, crie as seguintes sub-redes:

- AzureFirewallSubnet: destinado à implantação do Firewall do Azure
- AzureBastionSubnet: destinado à implantação do Azure Bastion
- dmz: destinado à implantação da VM do Azure usada para implantar o software SAP
- aplicativo: destinado a hospedar o aplicativo SAP e os servidores de instância do SAP Central Services
- db: destinado à hospedagem da camada de banco de dados SAP

1. No computador do laboratório, na janela do navegador da Web que exibe o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Redes virtuais**. 
1. Na página **Redes virtuais**, selecione **+ Criar**.
1. Na guia **Básico** da página **Criar rede virtual**, especifique as seguintes configurações e selecione **Avançar**:

   |Configuração|Valor|
   |---|---|
   |Assinatura|O nome da assinatura do Azure que você usará neste laboratório|
   |Grupo de recursos|**acss-infra-RG**|
   |Nome da rede virtual|**acss-infra-VNET**|
   |Region|o nome da mesma região do Azure que você usou na tarefa anterior desse exercício|

1. Na guia **Segurança**, aceite as configurações padrão e selecione **Avançar**.

   >**Observação**: Neste momento, você pode provisionar o Azure Bastion e o Firewall do Azure; no entanto, você os provisionará separadamente depois que a rede virtual for criada.

1. Na guia **Endereços IP**, especifique as seguintes configurações de sub-rede e selecione **Examinar + criar**:

   |Configuração|Valor|
   |---|---|
   |Espaço de endereços IP|**10.0.0.0/16 (65.536 endereços)**|

1. Na lista de sub-redes, selecione o ícone da lixeira para excluir a sub-rede **padrão**.
1. Selecione **+Adicionar uma sub-rede**.
1. No painel **Adicionar uma sub-rede**, especifique as seguintes configurações e selecione **Adicionar** (deixe as demais com seus valores padrão):

   |Configuração|Valor|
   |---|---|
   |Finalidade da sub-rede|**Firewall do Azure**|
   |Endereço inicial|**10.0.0.0**|

   >**Observação**: Isso atribuirá automaticamente à sub-rede o nome **AzureFirewallSubnet** e definirá seu tamanho como **/26 (64 endereços)**.

1. Selecione **+Adicionar uma sub-rede**.
1. No painel **Adicionar uma sub-rede**, especifique as seguintes configurações e selecione **Adicionar** (deixe as demais com seus valores padrão):

   |Configuração|Valor|
   |---|---|
   |Nome|**dmz**|
   |Endereço inicial|**10.0.0.128**|
   |Tamanho|**/26 (64 endereços)**|

   >**Observação**: Essa sub-rede será usada para hospedar a VM do Azure que você usará para instalar o software SAP por meio do Centro do Azure para soluções SAP.

1. Selecione **+Adicionar uma sub-rede**.
1. No painel **Adicionar uma sub-rede**, especifique as seguintes configurações e selecione **Adicionar** (deixe as demais com seus valores padrão):

   |Configuração|Valor|
   |---|---|
   |Finalidade da sub-rede|**Azure Bastion**|
   |Endereço inicial|**10.0.1.0**|
   |Tamanho|**/24 (256 endereços)**|

   >**Observação**: Isso atribuirá automaticamente à sub-rede o nome **AzureBastionSubnet**.

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

   >**Observação**: Não espere que o processo de provisionamento seja concluído, mas prossiga para a próxima tarefa. O provisionamento deve levar apenas alguns segundos.

#### Tarefa 3: Criar um recurso do Azure Bastion

Nesta tarefa, crie um recurso do Azure Bastion para proteger a conectividade com as VMs do Azure a partir da Internet.

1. No computador do laboratório, na janela do navegador da Web que exibe o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Bastions**. 
1. Na página **Bastions**, selecione **+ Criar**.
1. Na guia **Básico** da página **Bastions**, especifique as seguintes configurações e selecione **Avançar: Avançado >**:

   |Configuração|Valor|
   |---|---|
   |Assinatura|O nome da assinatura do Azure que você usará neste laboratório|
   |Grupo de recursos|**acss-infra-RG**|
   |Nome|**acss-infra-BASTION**|
   |Region|o nome da mesma região do Azure que você usou anteriormente neste exercício|
   |Camada|**Basic**|
   |Contagem de instâncias|**2**|
   |Rede virtual|**acss-infra-VNET**|
   |Sub-rede|**AzureBastionSubnet (10.0.1.0/24)**|
   |Endereço IP público|**Criar novo**|
   |Nome do endereço IP público|**acss-bastion-PIP**|

1. Na guia **Avançado**, revise as opções disponíveis sem fazer nenhuma alteração e selecione **Examinar + criar**.
1. Na guia **Examinar + criar**, aguarde a conclusão do processo de validação e selecione **Criar**.

   >**Observação**: Não espere que o provisionamento seja concluído, mas prossiga para a próxima tarefa. O provisionamento pode levar cerca de 5 minutos.

#### Tarefa 4: Criar uma conta de Uso Geral v2 de Armazenamento do Microsoft Azure

Nesta tarefa, crie uma conta de Uso Geral v2 do Armazenamento do Microsoft Azure que esteja associada ao Centro do Azure para soluções SAP usado para a implantação. Essa conta de armazenamento é usada para hospedar a mídia de instalação do SAP para acomodar a instalação do software SAP por meio do Centro do Azure para soluções SAP.

1. No computador do laboratório, na janela do navegador da Web que exibe o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Contas de Armazenamento**.
1. Na página **Contas de Armazenamento**, selecione **+ Criar**.
1. Na guia **Básico** da página **Criar uma conta de armazenamento**, especifique as seguintes configurações e selecione **Avançar: Avançado >**.

   |Configuração|Valor|
   |---|---|
   |Assinatura|O nome da assinatura do Azure que você usará neste laboratório|
   |Grupo de recursos|**acss-infra-RG**|
   |Nome da conta de armazenamento|qualquer nome globalmente exclusivo de 3 a 24 caracteres composto por letras e dígitos|
   |Region|o nome da mesma região do Azure que você usou anteriormente neste exercício|
   |Desempenho|**Standard**|
   |Redundância|**Armazenamento com redundância geográfica (GRS)**|
   |Disponibilizar o acesso de leitura aos dados em caso de disponibilidade regional|Desabilitado|

1. Na guia **Avançado**, examine as opções disponíveis, aceite os padrões e selecione **Avançar: Rede >**.
1. Na guia **Rede**, execute as seguintes ações e selecione **Examinar**.

   1. Selecione **Habilitar acesso público a partir de redes virtuais e endereços IP selecionados**.
   1. Na seção **Redes virtuais**, certifique-se de que a lista suspensa **Assinatura da rede virtua** exiba o nome da assinatura do Azure usada neste laboratório.
   1. Na seção **Redes virtuais**, na lista suspensa **Rede virtual**, selecione **acss-infra-VNET**.
   1. Na lista suspensa **Sub-redes**, selecione as sub-redes **aplicativo**, **db** e **dmz**.

1. Na guia **Revisar**, aguarde a conclusão do processo de validação e selecione **Criar**.

   >**Observação**: Aguarde o processo de provisionamento ser concluído. O provisionamento deve levar menos de 1 minuto.

1. No painel **A implantação foi concluída**, selecione **Ir para o recurso**.
1. Na página da conta de armazenamento, no menu de navegação vertical do lado esquerdo, na seção **Armazenamento de dados**, selecione **Contêineres**.
1. Selecionar **+ Contêiner**.
1. No painel **Novo contêiner**, na caixa de texto **Nome**, insira **sapbits** e selecione **Criar**.

   >**Observação**: O contêiner **sapbits** hospedará a mídia de instalação do SAP.

#### Tarefa 5: Configurar a autorização da identidade gerenciada atribuída pelo usuário do Microsoft Entra

Nesta tarefa, use uma atribuição de função de controle de acesso baseado em função (RBAC) do Azure para conceder a identidade gerenciada atribuída pelo usuário do Microsoft Entra. A identidade gerenciada é usada para realizar o acesso para implantação da assinatura do Azure e da conta de Uso Geral v2 do Armazenamento do Microsoft Azure criada na tarefa anterior.

1. No portal do Azure, na janela do navegador da Web que exibe o portal do Azure, na caixa de texto **Pesquisar**, procure e selecione **Identidades Gerenciadas**.
1. Na página Identidades Gerenciadas e, selecione a entrada **acss-infra-MI**.
1. Na página **acss-infra-MI**, no menu de navegação vertical do lado esquerdo, selecione **Atribuições de função do Azure**.
1. Na página **Atribuições de função do Azure**, selecione **+ Adicionar atribuição de função (Versão prévia)**.
1. No painel **+ Adicionar atribuição de função (Versão prévia)**, especifique as seguintes configurações e selecione **Salvar**:

   |Configuração|Valor|
   |---|---|
   |Escopo|**Assinatura**|
   |Assinatura|O nome da assinatura do Azure que você usará neste laboratório|
   |Função|**Função de serviço do Centro do Azure para soluções SAP**|

1. De volta à página **Atribuições de função do Azure**, selecione **+ Adicionar atribuição de função (Versão prévia)**.
1. No painel **+ Adicionar atribuição de função (Versão prévia)**, especifique as seguintes configurações e selecione **Salvar**:

   |Configuração|Valor|
   |---|---|
   |Escopo|**Storage**|
   |Assinatura|O nome da assinatura do Azure que você usará neste laboratório|
   |Recurso|O nome da conta de Armazenamento do Microsoft Azure que você criou na tarefa anterior|
   |Função|**Acesso a Dados e Leitor**|

#### Tarefa 6: Criar uma conta de compartilhamentos de arquivos Premium do Azure

Nesta tarefa, crie uma conta de compartilhamento de arquivos Premium do Azure usada para implementar o Diretório de Transporte do SAP.

1. No computador do laboratório, na janela do navegador da Web que exibe o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Contas de Armazenamento**.
1. Na página **Contas de Armazenamento**, selecione **+ Criar**.
1. Na guia **Básico** da página **Criar uma conta de armazenamento**, especifique as seguintes configurações e selecione **Avançar: Avançado >**.

   |Configuração|Valor|
   |---|---|
   |Assinatura|O nome da assinatura do Azure usada neste laboratório|
   |Grupo de recursos|**acss-infra-RG**|
   |Nome da conta de armazenamento|qualquer nome globalmente exclusivo de 3 a 24 caracteres composto por letras e dígitos|
   |Region|o nome da mesma região do Azure que você usou anteriormente neste exercício|
   |Desempenho|**Premium**|
   |tipo de conta Premium|**Compartilhamentos de arquivo**|
   |Redundância|**ZRS (armazenamento com redundância de zona)**|

1. Na guia **Avançado**, desabilite a configuração **Exigir transferência segura para operações da API REST** e selecione **Avançar: Rede >**.

   >**Observação**: O protocolo NFS não dá suporte à criptografia e, em vez disso, depende da segurança em nível de rede. Essa configuração deve ser desativada para que o NFS funcione.

1. Na guia **Rede**, execute as seguintes ações e selecione **Examinar**.

   1. Selecione **Habilitar acesso público a partir de redes virtuais e endereços IP selecionados**.
   1. Na seção **Redes virtuais**, certifique-se de que a lista suspensa **Assinatura de rede virtual** exiba o nome da assinatura do Azure usada neste laboratório.
   1. Na seção **Redes virtuais**, na lista suspensa **Rede virtual**, selecione **acss-infra-VNET**.
   1. Na lista suspensa **Sub-redes**, selecione as sub-redes **aplicativo**, **db** e **dmz**.

   >**Observação**: Em geral, evite permitir o acesso aos seus recursos internos a partir de sub-redes de perímetro. Nesse caso, o único motivo para fazer isso é permitir a validação desse acesso posteriormente neste laboratório.

1. Na guia **Revisar**, aguarde a conclusão do processo de validação e selecione **Criar**.

   >**Observação**: Aguarde o processo de provisionamento ser concluído. O provisionamento deve levar menos de 1 minuto.

1. No painel **A implantação foi concluída**, selecione **Ir para o recurso**.
1. Na página da conta de armazenamento, no menu de navegação vertical do lado esquerdo, na seção **Armazenamento de dados**, selecione **Compartilhamentos de arquivos** e, em seguida, **+Compartilhamento de arquivos**.
1. Na guia **Básico** da página **Novo compartilhamento de arquivo**, especifique as seguintes configurações e selecione **Examinar + criar**:

   |Configuração|Valor|
   |---|---|
   |Nome|**trans**|
   |Capacidade provisionada|**128**|
   |Protocolo|**NFS**|
   |Squash raiz|*Nenhuma Squash Raiz**|

1. Na guia **Examinar + criar**, aguarde a conclusão do processo de validação e selecione **Criar**.

   >**Observação**: Aguarde a conclusão do provisionamento do compartilhamento de arquivos. O provisionamento deve levar apenas alguns segundos.

1. Na página **Conectar-se a este compartilhamento NFS a partir do Linux**, na lista suspensa **Selecione sua distribuição Linux**, selecione **SUSE** na lista suspensa Distribuição Linux e examine os comandos de exemplo para montar esse compartilhamento NFS.

#### Tarefa 7: Criar e configurar um grupo de segurança de rede

Nesta tarefa, crie e configure um grupo de segurança de rede (NSG) usado para restringir o acesso de saída de sub-redes da rede virtual que hospeda a implantação. Você pode fazer isso bloqueando a conectividade com a Internet, mas permitindo explicitamente conexões com os seguintes serviços:

- Pontos de extremidade de infraestrutura de atualização do SUSE ou Red Hat
- Armazenamento do Azure
- Cofre de Chave do Azure
- Microsoft Entra ID
- Azure Resource Manager

>**Observação**: Em geral, você deve considerar o uso do Firewall do Azure em vez de NSGs para proteger a conectividade de rede para sua implantação do SAP. Este laboratório aborda as duas opções.

1. No computador do laboratório, na janela do navegador da Web que exibe o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Grupos de Segurança de Rede**.
1. Na página **Grupos de segurança de rede**, selecione **+ Criar**.
1. Na guia **Básico** da página **Criar grupo** de segurança de rede, especifique as seguintes configurações e selecione **Examinar + criar**:

   |Configuração|Valor|
   |---|---|
   |Assinatura|O nome da assinatura do Azure usada neste laboratório|
   |Grupo de recursos|**acss-infra-RG**|
   |Nome|**acss-infra-NSG**|
   |Region|o nome da mesma região do Azure que você usou anteriormente neste exercício|

1. Na guia **Revisar + criar**, aguarde a conclusão do processo de validação e selecione **Criar**.

   >**Observação**: Aguarde o processo de provisionamento ser concluído. O provisionamento deve levar menos de 1 minuto.

1. No painel **A implantação foi concluída**, selecione **Ir para o recurso**.

   >**Observação**: Por padrão, as regras internas de grupos de segurança de rede permitem todo o tráfego de saída, todo o tráfego dentro da mesma rede virtual, bem como todo o tráfego entre redes virtuais emparelhadas. Do ponto de vista da segurança, você deve considerar restringir esse comportamento padrão. A configuração proposta restringe a conectividade de saída para a Internet e o Azure. Você também pode usar as regras do NSG para restringir a conectividade em uma rede virtual.

1. Na página **acss-infra-NSG**, no menu de navegação vertical do lado esquerdo, na seção **Configurações**, selecione **Regras de segurança de saída**.
1. Na página **Regras de segurança de saída \|acss-infra-NSG**, selecione **+ Adicionar**.
1. No painel **Adicionar regra de segurança de saída**, especifique as seguintes configurações e selecione **Adicionar**:

   >**Observação**: A regra a seguir deve ser adicionada para permitir explicitamente a conectividade com os pontos de extremidade de infraestrutura de atualização do Red Hat.

   >**Observação**: Para identificar os endereços IP a serem usados para RHEL, confira [Preparar a rede para implantação de infraestrutura](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network#allowlist-suse-or-red-hat-endpoints)

   |Configuração|Valor|
   |---|---|
   |Fonte|**Qualquer**|
   |Intervalos de portas de origem|*|
   |Destino|**Endereços IP**|
   |Intervalos de CIDR /endereço IP de destino|**13.91.47.76,40.85.190.91,52.187.75.218,52.174.163.213,52.237.203.198**|
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

   >**Observação**: Por fim, você precisa atribuir o NSG às sub-redes relevantes da rede virtual que hospedará a implantação do SAP.

1. No painel **Adicionar regra de segurança de saída**, no menu de navegação vertical do lado esquerdo, na seção **Configurações**, selecione **Sub-redes**.
1. Na página **acss-infra-NSG \| Sub-redes**, selecione **+ Associar**.
1. No painel **Associar sub-rede**, na lista suspensa **Rede virtual**, selecione **acss-intra-VNET (acss-infra-RG)**. Na lista suspensa **Sub-rede**, selecione **aplicativo** e selecione **OK**.
1. No painel **Associar sub-rede**, na lista suspensa **Rede virtual**, selecione **acss-intra-VNET (acss-infra-RG)**. Na lista suspensa **Sub-rede**, selecione **db** e selecione **OK**.

#### Tarefa 8: Criar uma Máquina Virtual do Azure

Nesta tarefa, crie uma Máquina Virtual (VM) do Azure usada para a instalação do software SAP como parte de uma implantação do Centro do Azure para soluções SAP.

1. No computador do laboratório, na janela do navegador da Web que exibe o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Máquinas Virtuais**.
1. Na página **Máquinas virtuais**, selecione **+ Criar** e, no menu suspenso, selecione **Máquina Virtual do Azure**.
1. Na guia **Básico** da página **Criar uma máquina virtual**, especifique as seguintes configurações e selecione **Avançar: Discos >** (deixe todas as outras configurações com seu valor padrão):

   |Configuração|Valor|
   |---|---|
   |Assinatura|O nome da assinatura do Azure usada neste laboratório|
   |Grupo de recursos|**acss-infra-RG**|
   |Nome da máquina virtual|**acss-infra-vm0**|
   |Region|o nome da mesma região do Azure que você usou anteriormente neste exercício|
   |Opções de disponibilidade|**Nenhuma redundância de infraestrutura necessária**|
   |Tipo de segurança|**Máquina virtual de inicialização confiável**|
   |Imagem|**Ubuntu Server 20.04 LTS - x64 Gen2**|
   |Arquitetura de VMs;|**x64**|
   |Executar com o Desconto de Spot do Azure|desabilitado|
   |Tamanho|**Standard_B2ms**|
   |Tipo de autenticação|**Senha**|
   |Nome de Usuário|qualquer nome de usuário válido|
   |Senha|qualquer senha complexa de sua escolha|
   |Porta de entrada públicas|**Nenhuma**|
   
    > **Observação**: Certifique-se de lembrar o nome de usuário e a senha que especificou. Você precisará disso em uma etapa posterior deste laboratório.

1. Na guia **Discos**, aceite os valores padrão e selecione **Avançar: Rede >**.
1. Na guia **Rede**, especifique as seguintes configurações e selecione **Avançar: Gerenciamento >** (deixe todas as outras configurações com seu valor padrão):

   |Configuração|Valor |
   |---|---|
   |Rede virtual|**acss-infra-VNET**|
   |Sub-rede|**dmz**|
   |Endereço IP público|**Nenhuma**|
   |Grupo de segurança de rede da NIC|**Nenhuma**|
   |Excluir o adaptador de rede quando a VM é excluída|Habilitado|
   |Opções de balanceamento de carga|**Nenhuma**|

1. Na guia **Gerenciamento**, deixe todas as configurações com o valor padrão e selecione **Avançar: Monitoramento >**
1. Na guia **Monitoramento**, defina **Diagnóstico de Inicialização** como **Desabilitar** e selecione **Avançar: Avançado >** (deixe todas as outras configurações com seu valor padrão)
1. Na guia **Avançado**, selecione **Examinar + criar** (deixe todas as configurações com seu valor padrão).
1. Na guia **Examinar + criar** do menu **Criar uma máquina virtual**, selecione **Criar**.

   > **Observação**: Aguarde a conclusão do provisionamento. O provisionamento pode levar cerca de 3 minutos.

#### Tarefa 9: Configurar a VM do Azure

Nesta tarefa, conecte-se à VM do Azure usando o Azure Bastion e configure-a para a instalação do software SAP. 

> **Observação**: Antes de iniciar essa tarefa, verifique se o provisionamento do Azure Bastion foi concluído.

> **Observação**: Verifique se o seu navegador da Web não bloqueia janelas pop-up e, se for o caso, desative a funcionalidade Bloqueador de pop-up.

1. No computador do laboratório, na janela do navegador da Web que exibe o portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Máquinas Virtuais**. 
1. Na página **Máquinas virtuais**, selecione a entrada **acss-infra-vm0**.
1. Na página **acss-infra-vm0**, na barra de ferramentas, selecione **Conectar** e, no menu suspenso, selecione **Conectar via Bastion**.
1. Na página **acss-infra-vm0 \| Bastion**, certifique-se de que **Tipo de Autenticação** esteja definido como **Senha de VM**, nas caixas de texto **Nome de usuário** e **Senha**, insira o nome de usuário e a senha que você definiu ao provisionar a VM do Azure, certifique-se de que a caixa de seleção **Abrir em uma nova guia do navegador** esteja habilitada e selecione **Conectar**.

   > **Observação**: Isso deve abrir outra guia da janela do navegador da Web exibindo a sessão do shell em execução na VM do Azure.

   > **Observação**: Para preparar o servidor Ubuntu para o carregamento da mídia de instalação do SAP, você instalará a CLI do Azure.

1. Na guia do navegador recém-aberta, dentro da sessão do shell, execute o seguinte comando para instalar a CLI do Azure:

   ```bash
   curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
   ```

1. Na sessão do shell, execute o seguinte comando para instalar o PIP3:

   ```bash
   sudo apt install python3-pip
   ```

1. Na sessão do shell, execute o seguinte comando para instalar o Ansible 2.13.19:

   ```bash
   sudo pip3 install ansible-core==2.13.9
   ```

1. Na sessão do shell, execute o seguinte comando para instalar os módulos de coleção de galáxias do Ansible:

   ```bash
   sudo ansible-galaxy collection install ansible.netcommon:==5.0.0 -p /opt/ansible/collections
   sudo ansible-galaxy collection install ansible.posix:==1.5.1 -p /opt/ansible/collections
   sudo ansible-galaxy collection install ansible.utils:==2.9.0 -p /opt/ansible/collections
   sudo ansible-galaxy collection install ansible.windows:==1.13.0 -p /opt/ansible/collections
   sudo ansible-galaxy collection install community.general:==6.4.0 -p /opt/ansible/collections
   ```

1. Na sessão do shell, execute o seguinte comando para clonar o repositório de amostras de automação SAP do GitHub:

   ```bash
   git clone https://github.com/Azure/SAP-automation-samples.git
   ```

1. Na sessão do shell, execute o seguinte comando para clonar o repositório de automação do SAP a partir do GitHub:

   ```bash
   git clone https://github.com/Azure/sap-automation.git
   ```

1. Na sessão do shell, execute o seguinte comando para encerrar a sessão:

   ```bash
   logout
   ```

1. Quando solicitado, selecione **Fechar**.

#### Tarefa 10: Remover recursos do Azure

Nesta tarefa, remova todos os recursos do Azure provisionados neste laboratório.

1. No computador do laboratório, na janela do navegador da Web que exibe o portal do Microsoft Azure, selecione o ícone **Cloud Shell** para abrir o painel do Cloud Shell. Se necessário, selecione **Bash** para iniciar uma sessão do shell Bash. 

   > **Observação**: Se esta for a primeira vez que você está iniciando o Cloud Shell na assinatura do Azure que estava usando neste laboratório, será solicitado que você crie um compartilhamento de arquivos do Azure para manter os arquivos do Cloud Shell. Em caso afirmativo, aceite os padrões, o que resultará na criação de uma conta de armazenamento em um grupo de recursos gerado automaticamente.

1. No painel Cloud Shell, execute o seguinte comando para excluir o grupo de recursos **acss-infra-RG** e todos os seus recursos.

   ```cli
   az group delete --name 'acss-infra-RG' --no-wait --yes
   ```

   > **Observação**: O comando é executado de forma assíncrona (conforme determinado pelo parâmetro `--nowait`), portanto, embora o prompt do shell seja exibido imediatamente após a invocação, alguns minutos se passarão antes que o grupo de recursos e seus recursos sejam realmente removidos.

1. Feche o painel do Cloud Shell.
