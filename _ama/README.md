# Authentication Mechanism Assurance

_CertificateIssuanceOIDs.ps1.txt_ is a PowerShell script that imports a list of certification policies in Microsoft Active Directory. As a network administrator for a Federal Government agency, you can use this script to configure Microsoft Windows Active Directory (AD) Domain Service's (DS) Authentication Mechanism Assurance (AMA) on your Windows Server® 2008 R2 and later. 

The script has a list of policies to import grouped under different categories. You should only import the policies that are applicable to your agency.

You will have to rename the script file and remove the .txt extension in order to execute the script.

You may have to change the [powershell script execution policy](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-5.1&viewFallbackFrom=powershell-Microsoft.PowerShell.Core) to execute this script or sign the script to execute it after downloading.
