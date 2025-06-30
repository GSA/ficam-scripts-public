# Autopopulate altSecurityIdentities (IssuerSubject)

mapByCert-directoty.ps1.txt is a PowerShell script that imports a PIV authentication certificate information into Microsoft Active Directory, specifically the Issuer/Subject DN mapping. As a network administrator for a Federal Government agency, you can use this script to automate Microsoft Windows Active Directory (AD) user information to enable altSecurityIdentity account linking as described in [https://idmanagement.gov/implement/scl-windows/](https://www.idmanagement.gov/implement/scl-windows/)

This script was developed through collaboration with a number of U.S. Federal Agencies. If you have questions about the script or developing it further, please contact icam at gsa.gov.

## Script Functionality
The script provides for several modes and parameters to add or remove subject information into Active Directory.  AD admins will need to select one of 3 modes of operation while running the script:
1. -add - adds the concatenated *issuer/subject* altsecid attribute to a matched user AD acocunt from source certificates
2. -remove - cleares altsecids for matched users accounts
3. -check - queries and returns AD accounts missing the altsecid attribute

If in the mode to add an identifier, the script conducts the follwoing actions: 
1. it loads the x.509 certificates from a given directory
2. it extracts the UPN from the x.509 certificate
3. it matches the UPN from the x.509 certificate that to a user AD account
4. it matched, it will then add a new (issuer/subject) altsecid attribute extracted from other attributes in the x.509 certificate to the user AD account

The script can also support a "schedule" mode where it will delete the original x.509 certificate file after processing.

## Additonal Details
You will have to rename the script file and remove the .txt extension in order to execute the script.

You may have to change the powershell script execution policy to execute this script or sign the script to execute it after downloading.

Note that the Issuer/Subject mapping in AD after September 2024 will require a policy tuple registry edit to be supported.  This policy tuple prevents certificate name spoofing vulnerabilities by defining acceptable issuing CAs, acceptable certificate policy OIDs, and defining the IssuerSubject mapping as acceptable.  It is recommended to use stronger identifier for account mapping like Subject Key Identifier (x509:"<SKI>").
