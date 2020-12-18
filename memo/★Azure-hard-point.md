1. query vm
    - a sample
    ```bash
    bender@u01:104-58$ az vm list \
    >  --resource-group $rg \
    >  --show-details \
    >  --query "[*].{Name:name, Provisioned:provisioningState, Power:powerState}"\
    >  --output table
    Name        Provisioned    Power
    ----------  -------------  ----------
    AppServer   Succeeded      VM running
    DataServer  Succeeded      VM running
    ```
    - azure help [Query Azure CLI command output](https://docs.microsoft.com/ja-jp/cli/azure/query-azure-cli?view=azure-cli-latest) 
    - [jmespath](https://jmespath.org/tutorial.html#basic-expressions)
2. AKS
   - https://docs.azure.cn/zh-cn/aks/
