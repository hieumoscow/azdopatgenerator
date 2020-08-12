# Personal Access Token (PAT) generator for Azure DevOps 

## Getting Started

These instructions will cover usage information and for the docker container 

### Prerequisites

1. Create an Active Directory application (API Permission granted fo Azure DevOps user_impersonation - https://app.vssps.visualstudio.com/user_impersonation)
2. Service account must be mail enabled
3. Must be granted access to your Azure DevOps Organization
4. Must be enabled for 2FA bypass
5. If Azure Key Vault (AKV) is specified, the AAD app in 1. need to be granted access into this AKV for Secret's Set operation.

Install Docker:
* [Windows](https://docs.docker.com/windows/started)
* [OS X](https://docs.docker.com/mac/started/)
* [Linux](https://docs.docker.com/linux/started/)



### Usage

#### Environment Variables

| Name | Type  | Description |
| -- | -- | -- |
| AZDOPATGEN_ACCOUNT | string | (Required) Your Azure DevOps account name (can be found in url: https://accountname.visualstudio.com/). |
| AZDOPATGEN_TENANTID | string | (Required) tenandId, e.g.: yourtenant.onmicrosoft.com. |
| AZDOPATGEN_CLIENTID | string | (Required) clientId, client ID of Active Directory application. |
| AZDOPATGEN_CLIENTSECRET | string| (Required) clientSecret, client secret of Active Directory application|
| AZDOPATGEN_SCOPE | list | (Required) list of Azure Devops scopes, https://aka.ms/vstspatgen e.g.: <br>  vso.build,vso.build_execute,vso.profile,vso.agentpools_manag. |
| AZDOPATGEN_USERNAME | string | (Required) user / service_account email, e.g: patserviceaccount@yourtenant.onmicrosoft.com. |
| AZDOPATGEN_PASSWORD | string| (Required) user / service_account password |
| AZDOPATGEN_EXPIRY | datetime | (Required - only for PAT Creation) expiration date of PAT token e.g.:2020-06-18T10:41:31Z |
| AZDOPATGEN_AKV_NAME | string | (Optional - only for PAT Creation) if provided, will SetSecret in AKV on behalf of Client ID, otherwise the tool will output the PAT instead. |
| AZDOPATGEN_AKV_SECRET_NAME | string | (Optional - only for PAT Creation) Name of AKV Secret, if not provided, will auto generate: <br> $"{username} {DateTime.Now.ToString("dd/MM/yyyy-hh:mm")}". |

#### Create
```shell
docker run --env-file ./env.list -it hieunhu/azdopatgenerator 
```
Example env.list:
```shell
AZDOPATGEN_ACCOUNT=accountname
AZDOPATGEN_TENANTID= yourtenant.onmicrosoft.com
AZDOPATGEN_CLIENTID=00000000-0000-0000-0000-00000000000
AZDOPATGEN_CLIENTSECRET=0000000000000000000000000000000
AZDOPATGEN_SCOPE=vso.build,vso.build_execute,vso.profile,vso.agentpools_manage,vso.variablegroups_manage
AZDOPATGEN_USERNAME=patserviceaccount@yourtenant.onmicrosoft.com
AZDOPATGEN_PASSWORD=password
AZDOPATGEN_EXPIRY=2020-06-18T10:41:31Z
AZDOPATGEN_AKV_NAME=akv_name
AZDOPATGEN_AKV_SECRET_NAME=akv_secret_name
```

Output, AKV:
```shell
{
  "akvSecretName":"{AZDOPATGEN_AKV_SECRET_NAME}",
  "akvSecretId":"https://{akv_name}.vault.azure.net:443/secrets/{GUID}"
}
```

Output, (without AKV):
```shell
{
  "id":"{PAT_ID}",
  "name":"{AZDOPATGEN_AKV_SECRET_NAME}", 
  "pat":"{PAT_TOKEN}"}
```

#### List
```shell
docker run --env-file ./env.list -it hieunhu/azdopatgenerator -list
```
Output:
```shell
{
  "pats": [
    {
      "ClientId": "00000000-0000-0000-0000-000000000000",
      "AccessId": "{GUID}",
      "AuthorizationId": "{PAT_ID}",
      "HostAuthorizationId": "00000000-0000-0000-0000-000000000000",
      "UserId": "{GUID}",
      "ValidFrom": "{DateTime}",
      "ValidTo": "{DateTime}",
      "DisplayName": "{AZDOPATGEN_AKV_SECRET_NAME}",
      "Scope": "vso.build,vso.build_execute,vso.profile,vso.agentpools_manage,vso.variablegroups_manage",
      "TargetAccounts": [
        "{GUID}"
      ],
      "Token": null,
      "AlternateToken": null,
      "IsValid": true,
      "IsPublic": false,
      "PublicData": null,
      "Source": null,
      "Claims": null
    }
  ],
  "count": 1
```

#### Revoke
```shell
docker run --env-file ./env.list -it hieunhu/azdopatgenerator -revoke:PAT_ID
```
Output:
{"id":"PAT_ID"} 

#### Revoke All
```shell
docker run --env-file ./env.list -it hieunhu/azdopatgenerator -revokeall
```

#### Command Line arguments (stdin) alternatives
```shell
docker run -it hieunhu/azdopatgenerator -h
``` 

## Versioning

hieunhu/azdopatgenerator:$(date +"%g%m.%d%H%M")

## Acknowledgements

This is an extension of PatGenerator tool of Microsoft AD tenant (https://aka.ms/vstspatgen). 
This extension is built to support additional features:
* Linux container
* All AD tenants
* Environment variables
* Azure Key Vault
