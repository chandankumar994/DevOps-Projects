### Project.1 - How to Start/Stop VMs using AzureDevOps pipeline.

#### Prerequisites:
* Azure Subscription (You need to create an account on azure portal)
* App Service user that should have read write permission from IAM (This need to be confiured on azure portal itself)
* Azure Service connection (within azure devops for particular workspace).
  - You will be needed below details for service connection from azure portal.
    - Subscription ID:
    - Subscription Name:
    - Application Name:
    - Application (client) ID:
    - Tenant (directory) ID:
    - Secret Value:
* to create service connection go to `project settings` and select `service connections` option.

![image](https://github.com/chandankumar994/DevOps-Projects/assets/15160387/721ca8a4-7a2a-464e-b586-bc1ab515ab0b)

* Click on `New service connection` and then select `Azure Resource Manager` from the options and then click `next` button.
* Select `Service principal (manual)` and click `next` button.
* Provide all the information, and verify the connection.

  ![image](https://github.com/chandankumar994/DevOps-Projects/assets/15160387/0a95ec8f-1ea8-4602-a8d4-fcc23c53ad2a)

* Grant access to all pipeline then Click on `Save and Close`.
##### Step-1:
* Create a frsh repository in azure devops for example "azure-start-stop".

* Create 'start-azure-vms.yml' and 'stop-azure-vms.yml' files like below.

**start-azure-vms.yml**
```
trigger: none
pr: none

parameters:
  - name: vms
    type: object
    default: ["VM1","VM2"]

pool:
  vmImage: ubuntu-latest

jobs:
- job: startvmjob
  displayName: start VMs

  steps:
  - checkout: none

  - ${{ each vm in parameters.vms }}:
    - task: AzureCLI@2
      displayName: starting vm ${{vm}}
      inputs:
        azureSubscription: 'Service-Connection-Name'
        scriptType: 'pscore'
        scriptLocation: 'inlineScript'
        inlineScript: |
          $vmrequest = az vm list | ConvertFrom-Json | Where-Object {$_.Name -eq "${{vm}}"}
          az vm start -g $vmrequest.resourcegroup -n $vmrequest.name
```
**stop-azure-vms.yml**
```
trigger: none
pr: none

parameters:
  - name: vms
    type: object
    default: ["VM1","VM2"]

pool:
  vmImage: ubuntu-latest

jobs:
- job: stopvmjob
  displayName: stop VMs

  steps:
  - checkout: none

  - ${{ each vm in parameters.vms }}:
    - task: AzureCLI@2
      displayName: stopping vm ${{vm}}
      inputs:
        azureSubscription: 'Service-Connection-Name'
        scriptType: 'pscore'
        scriptLocation: 'inlineScript'
        inlineScript: |
          $vmrequest = az vm list | ConvertFrom-Json | Where-Object {$_.Name -eq "${{vm}}"}
          az vm deallocate -g $vmrequest.resourcegroup -n $vmrequest.name
```

##### Step-2:

* Now Create two seprate pipelines for both .yml files in azure devops.
![image](https://github.com/chandankumar994/DevOps-Projects/assets/15160387/fa0c7c52-7276-4c3b-b0f9-169ae1f08d0d)

##### Step-3:
* Prepare a schedule to run above pipelines according to your working hours. 
![image](https://github.com/chandankumar994/DevOps-Projects/assets/15160387/15ecf38c-c18a-4961-b294-9fb61ad3b4ed)

