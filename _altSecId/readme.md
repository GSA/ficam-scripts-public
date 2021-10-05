# Autopopulate altSecurityIdentities

mapByCert-directoty.ps1.txt is a PowerShell script that imports a PIV authentication certificate information into Microsoft Active Directory. As a network administrator for a Federal Government agency, you can use this script to automate Microsoft Windows Active Directory (AD) user information to enable altSecurityIdentity account linking as described in https://playbooks.idmanagement.gov/piv/network/account/

This script was developed through collaboration with a number of U.S. Federal Agencies. If you have questions about the script or developing it further, please contact icam at gsa.gov.

The script has a list of parameters to add or remove subject information into Active Directory.

You will have to rename the script file and remove the .txt extension in order to execute the script.

You may have to change the powershell script execution policy to execute this script or sign the script to execute it after downloading.
