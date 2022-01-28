# Trust Store Management Script

Last Updated: Dec 2018

The Trust Store Management Script (TSMS) was designed to help update operating system trust stores to facilitate smart card logon for all potential PIV issuers across the Federal Public Key Infrastructure (FPKI). The script was designed to run using Python 3.x on Windows or macOS. 

The execution of the script will vary dependent upon operating system. 
- On Windows, the script will assemble a .p7b file containing all desired PIV issuer certification authority (CA) signing certificates. 
- On macOS, the script will assemble an Apple Configuration Profile (.mobileconfig) containing all desired PIV issuer certification authority (CA) signing certificates. 

These script outputs can be used as inputs to standardize and streamline existing agency trust store management processes. For example, on Windows, the resulting .p7b file could be published to domain controllers via certutil or a GPO. On macOS, the resulting .mobileconfig file could be shared on an agency intranet site or distributed to devices in one of the methods described here.  

Please see the sections below for more information on the script:
- Download Location
- How it works
- Target Certification Authorities
- System Requirements
- Running the script
- FAQs

Download Location: 
Version 1 of the Trust Store Management Script can be downloaded from this link.

To ensure authenticity, please verify the package SHA-256 hash matches the below:
```
BACBCD9C805197E002FC6FFF289516E78EFE15ECF58B3A34E98E81E401BAE6B0
```

Verifying hash on Windows using Powershell...
```
> Get-FileHash [DOWNLOAD_LOCATION]\Trust_Store_Mangagement_Script_V1.zip | Format-List
```

Verifying hash on macOS using Terminal…
```
$ shasum -a 256 [DOWNLOAD_LOCATION]/Trust_Store_Mangagement_Script_V1.zip
```

## How it works

The TSMS installation package contains three artifacts:
1. certLoader.py - This script reads in the targets.json installation file and assembles the output file containing the desired CA certificates.
2. id-fpki-common-auth - This directory contains CA certificates for the eligible installation targets. 
3. targets.json - This JSON file contains attributes for the potential CA installation targets. More installation is presented below.

For each CA, the following attributes are included:
- ID: Contains a unique integer ID for the CA
- SUBJECT: Contains the CA’s subject name.
- ISSUER:  Contains the CA’s issuer.
- VALIDFROM:  Contains the validity start date for the certificate
- VALIDTO:  Contains the validity end date for the certificate
- SERIAL: Contains the certificate’s serial number
- FILENAME: Contains the certificate’s file name
- THUMBPRINT: Contains the certificate’s thumbprint (SHA-1 hash)
- INSTALL: Contains the installation flag (either TRUE or FALSE)

Sample below:

```
    "ID": "00001",
    "SUBJECT": "Betrusted Production SSP CA A1",
    "ISSUER": "Federal Common Policy CA",
    "VALIDFROM": "12/9/2010 19:55",
    "VALIDTO": "12/9/2020 19:49",
    "SERIAL": "19A",
    "FILENAME": "Betrusted_Production_SSP_CA_A1.cer",
    "THUMBPRINT": "0601bbdad5a28231bc9436750b4f3a484bab06c3",
    "INSTALL": "TRUE"
```

By default, all included CAs are marked for installation. Changing the installation flag from TRUE to FALSE will result in a given CA being omitted from the output file.

## Target CAs

The list of CAs included for installation in this script are presented below.

If you feel another CA should be included, let us know!


## System requirements:

Python v3.x is required to run the Trust Store Management Script. Additionally, on Windows, OpenSSL needs to be installed with an environment path variable set to support .p7b file generation.

Windows
Openssl
Show how to set path https://www.youtube.com/watch?v=z11siMaAfPA

Your feedback is valued to us. If you have any questions or recommended updates for this script, please contact us at FPKI@gsa.gov.


FAQs
Q: How often will it be updated?
A: We anticipate this script being updated quarterly. 
