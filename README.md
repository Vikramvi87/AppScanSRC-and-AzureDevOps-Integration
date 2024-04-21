# AppScan Source (SAST) and AzureDevOps Integration
</br>
It will help to Integrate AppScan Source on AzureDevOps. It will enable AzureDevOps to start scan, generate report, publish results to AppScan Source Database and AppScan Enterprise and check for Security Gate.<br>
<br>
Requirements:<br>
1 - AppScan Source installed in a Windows Server.<br>
2 - AppScan Source for Automation License. It will enable run scan through command line.<br>
3 - Install Powershell 7.<br>
4 - Add AppScan Source bin folder to Windows PATH (System) Environment Variable.<br>
5 - Install a Self-hosted agent of Azure Pipeline in same Windows Server that has AppScan Source. https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/windows-agent?view=azure-devops<br>
5.1 - Connection (Firewall rules) between Self-hosted agent (AppScan Source Server) and  AzureDevOps.<br>
5.2 - Add Self-hosted agent as a Service.<br>
5.3 - Change User Service to same User that has access in AppScan Enterprise (Service User).<br>
6 - Create a AppScan Source token <install_dir>\bin\ounceautod.exe -u domain\user -p password --persist.<br>
7 - The AppScan Enterprise URL to be used in the AzureDevOps Pipeline as a variable.<br>
8 - Generate a key pair of AppScan Enterprise Rest API to be used in the AzureDevOps Pipeline as a variable. Open Appscan Enterprise, click in Rest API in main menu, click in the Account endpoint, click in POST /account/key, click in Execute Request button and copy the key pair.<br>
<br>

```yaml
variables:
  aseApiKeyId: xxxxxxxxxxxxxxxx
  aseApiKeySecret: xxxxxxxxxxxxxxxx
  compiledArtifactFolder: none
  scanConfig: Normal scan
  aseAppName: $(Build.Repository.Name)
  aseHostname: xxxxxxxxxxxxxxxx
  aseToken: C:\ProgramData\HCL\AppScanSource\config\ounceautod.token
  sevSecGw: highIssues
  maxIssuesAllowed: 100
  WorkingDirectory: $(System.DefaultWorkingDirectory)
  BuildNumber: $(Build.BuildNumber)

trigger: none

pool: default

steps:
- pwsh: |
    Invoke-WebRequest -Uri https://raw.githubusercontent.com/jrocia/AppScanSRC-and-AzureDevOps-Integration/main/scripts/appscanase_create_application_ase.ps1 -OutFile appscanase_create_application_ase.ps1
    .\appscanase_create_application_ase.ps1
  displayName: 'Checking Application ID in ASE'
- pwsh: |
    Invoke-WebRequest -Uri https://raw.githubusercontent.com/jrocia/AppScanSRC-and-AzureDevOps-Integration/main/scripts/appscansrc_create_config_scan_folder.ps1 -OutFile appscansrc_create_config_scan_folder.ps1
    .\appscansrc_create_config_scan_folder.ps1
  displayName: 'Creating AppScan Source Config file'
- pwsh: |
    Invoke-WebRequest -Uri https://raw.githubusercontent.com/jrocia/AppScanSRC-and-AzureDevOps-Integration/main/scripts/appscansrc_scan.ps1 -OutFile appscansrc_scan.ps1
    .\appscansrc_scan.ps1
  displayName: 'Running SAST scan'
- pwsh: |
    Invoke-WebRequest -Uri https://raw.githubusercontent.com/jrocia/AppScanSRC-and-AzureDevOps-Integration/main/scripts/appscansrc_publish_assessment_to_enterprise.ps1 -OutFile appscansrc_publish_assessment_to_enterprise.ps1
    .\appscansrc_publish_assessment_to_enterprise.ps1
  displayName: 'Publishing Result Scan into ASE'
- pwsh: |
    Invoke-WebRequest -Uri https://raw.githubusercontent.com/jrocia/AppScanSRC-and-AzureDevOps-Integration/main/scripts/appscansrc_check_security_gate.ps1 -OutFile appscansrc_check_security_gate.ps1
    .\appscansrc_check_security_gate.ps1
  displayName: 'Checking Security Gate'
- publish: $(aseAppName)-$(BuildNumber).pdf
  artifact: $(aseAppName)-$(BuildNumber).pdf
  continueOnError: On
  displayName: 'Uploading PDF to Azure Artifacts'
```
