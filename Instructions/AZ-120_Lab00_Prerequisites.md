---
lab:
  title: 00 - Pré-requisitos do laboratório
  module: Module 00 - Lab prerequisites
---

# AZ 120: Pré-requisitos do laboratório

## Principais requisitos da vCPU

> **Importante**: Os principais requisitos da vCPU dependem dos laboratórios que você pretende implementar.

- Para concluir o laboratório 04b - Implementar arquitetura SAP em VMs do Azure que executam o Windows, você precisará de uma assinatura do Microsoft Azure com pelo menos 28 vCPU disponíveis na região do Azure que ofereça suporte a zonas de disponibilidade onde as VMs do Azure implantadas neste laboratório residirão.

    - 4 x Standard_DS1_v2 (1 vCPUs cada) = 4
    - 6 x Standard_D4s_v3 (4 vCPUs cada) = 24

    > **Observação**: Considere o uso das regiões **Leste dos EUA** ou **Leste dos EUA 2** para a implantação de seus recursos.

    > **Observação**: Para identificar as regiões do Azure que dão suporte a zonas de disponibilidade, consulte <https://docs.microsoft.com/en-us/azure/availability-zones/az-overview>

- Para concluir o laboratório 05 - Automatizar a implantação usando o Centro do Azure para soluções SAP, você precisará de uma assinatura do Microsoft Azure com a seguinte disponibilidade de vCPU na região do Azure onde as VMs do Azure implantadas neste laboratório residirão.

    - 4 x Standard_E4ds_v4 (4 vCPUs cada) ou 4 X Standard_D4ds_v4 (4 vCPUs cada) = 8
    - 2 x Standard_M64ms (64 vCPUs cada) = 128

>**Observação**: Para minimizar os requisitos de vCPU e memória, você pode usar Standard_M32ts (32 vCPUs e 192 GiB de memória cada) ao invés de Standard_M64m.

>**Observação**: Embora os requisitos de vCPU para os três primeiros laboratórios deste curso sejam menores, recomendamos que você solicite aumento de cotas para atender aos requisitos de todos os laboratórios, já que o processo de aumento de cotas pode levar um tempo (mesmo que as solicitações de aumento de cota normalmente sejam concluídas no mesmo dia útil).

## Antes do laboratório prático

Período: 30 minutos

### Tarefa 1: Validar um número suficiente de núcleos de vCPU para o laboratório Implementar a arquitetura SAP em VMs do Azure que executam o Windows

1. No computador do laboratório, inicie um navegador da Web e navegue até o portal do Azure em `https://portal.azure.com`.
1. No portal do Azure, inicie uma sessão do PowerShell no Cloud Shell. 

    > **Observação**: Se esta for a primeira vez que você está iniciando o Cloud Shell na assinatura atual do Azure, você será solicitado a criar um compartilhamento de arquivos do Azure para persistir arquivos do Cloud Shell. Nesse caso, aceite os padrões, o que resultará na criação de uma conta de armazenamento em um grupo de recursos gerado automaticamente.

1. No portal do Azure, no painel do **Cloud Shell**, no prompt do PowerShell, execute o seguinte: em que `<Azure_region>` designa a região do Azure de destino que você pretende usar para este laboratório (por exemplo, `eastus`):

    ```powershell
    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}

    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'StandardDSv2Family'}
    ``` 

    > **Observação**: Para identificar os nomes das regiões do Azure, no **Cloud Shell**, no prompt do Bash, execute `(Get-AzLocation).Location`
   
1. Revise o valor atual e as entradas de limite na saída dos comandos executados na etapa anterior e verifique se você tem um número suficiente de vCPUs na região do Azure de destino.
1. Se o número de vCPUs não for suficiente, no portal do Azure, navegue de volta para a folha de assinatura e clique em **Uso + cotas**. 
1. No painel **Uso + cotas** da assinatura, clique em **Solicitar Aumento**.
1. No painel **Descrição do problema**, especifique o seguinte:

    -   Tipo de problema: **Limites de serviço e assinatura (cotas)**
    -   Assinatura: o nome da assinatura do Azure que você usará neste laboratório
    -   Tipo de cota: **Aumentos de limite de assinatura de computação/VM (núcleos/vCPUs)**

1. Clique em **Gerenciar Cota**.
1. Use o menu suspenso do local para filtrar os resultados para a região do Azure que você planeja usar. É recomendável usar **Leste dos EUA** ou **Leste dos EUA2**.
1. Localize o tipo de cota de **VCPUs da Família DSv3 Standard** e selecione o lápis de edição.
1. No campo Novo Limite, especifique **40** e clique em **Salvar e Continuar**.
1. Localize o tipo de cota **Total de vCPUs regionais** e selecione o lápis de edição.
1. No campo Novo Limite, especifique **40** e clique em **Salvar e Continuar**.

   > **Observação**: A solicitação de aumento de cota deve ser aprovada automaticamente. Se a solicitação for negada, prossiga para abrir um tíquete de suporte para solicitar o aumento da cota.

### Tarefa 2: Validar um número suficiente de núcleos vCPU para o laboratório Automatizar a implantação usando a Centro do Azure para soluções SAP

> **Observação**: Para concluir este laboratório (conforme descrito), você precisará de uma assinatura do Microsoft Azure com as cotas de vCPU que acomodam a implantação das seguintes VMs:

- 2 x VMs Standard_E4ds_v4 (4 vCPUs e 32 GiB de memória cada) ou 2 X VMs Standard_D4ds_v4 (4 vCPUs e 16 GiB de memória cada) para a camada do ASCS
- 2 x VMs Standard_E4ds_v4 (4 vCPUs e 32 GiB de memória cada) ou 2 X VMs Standard_D4ds_v4 (4 vCPUs e 16 GiB de memória cada) para a camada de aplicativo 
- 2 x VMs Standard_M64ms (64 vCPUs e 1750 GiB de memória cada) para a camada de banco de dados

1. No computador do laboratório, inicie um navegador da Web e navegue até o portal do Azure em `https://portal.azure.com`.
1. No portal do Azure, selecione o ícone do **Cloud Shell** e inicie uma sessão do PowerShell no Cloud Shell. 
1. No portal do Azure, no painel do **Cloud Shell**, no prompt do PowerShell, execute o seguinte: em que `<Azure_region>` designa a região do Azure para a qual você pretende implantar recursos neste laboratório (por exemplo, `eastus`):

    ```powershell
    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'standardEDSv4Family'}

    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'standardDSv4Family'}

    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'standardMSFamily'}

    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'cores'}
    ```

    > **Observação**: Para identificar os nomes das regiões do Azure, no **Cloud Shell**, no prompt do Bash, execute `(Get-AzLocation).Location`

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

    > **Observação**: Aguarde até que a solicitação para aumentar os limites de cota seja concluída com êxito antes de iniciar a implantação do Automate de laboratório usando o Centro do Azure para soluções SAP.
