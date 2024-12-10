#ID-cert-request_v1-0.ps1 - 
**PS Script to generate a PKCS10 certificate signing requests (CSRs) for Domain Controller or computer certificates.** 
This automates the manual request generation steps in the DoD PKE Reference Guides for Enabling Smart Card Logon on Windows Servers.
The typical use of the CSR is for requesting certificates from an external PKI signing CA that is trusted in the Domain of the joined Computer
Generated CSRs include SID extension for Strong Certificate Mapping per Microsoft KB5014754 compatible with Microsoft MSCS and DoD NPE signing CAs

Run this script on a domain joined computer to generate a private key and Certificate Signing Request (CSR)
The CSR includes the domain joined Domain Controller or computer Microsoft proprietary SID extension for Strong Certificate Mapping, according to
KB5014754: Certificate-based authentication changes on Windows domain controllers
https://support.microsoft.com/en-us/topic/kb5014754-certificate-based-authentication-changes-on-windows-domain-controllers-ad2c23b0-15d8-4340-a468-4d4f3b188f16

Written by: Tim W. Baldridge
Date:     Nov 24, 2024
Time:     16:35 PM CDT
Version:  1.1 NIPRNet
Changelog: Updated AD Search to use domain of local computer, corrects issue with sys admin AD login from non-local domain
	
Date:     October 2, 2024
Time:     10:05 AM CDT
Version:  1.0 NIPRNet

#Requirements: Powershell 5.x, certreq.exe, domain joined computer or domain controller

##IMPORTANT: To generate a non-exportable private key run this script in UAC ADMIN Mode PowerShell locally on the target computer/DC endpoint. It is a critical violation of Zero Trust principals to create a CSR with a Microsoft AD account SID using an exportable private key. This script may be run in non-admin mode on a domain joined computer to create an .inf for a later private key and CSR generation.

Run this script before promoting a candidate computer to a Domain Controller. Promote the computer to a DC after installing the certificate.

Script execution with UAC admin rights generates a certificate signing request with a new private key
Script execution withOUT UAC admin rights generates an .inf file suitable for "certreq.exe -new" CSR generation in the current directory

This PowerShell script is a substantial rewrite of dc-cert-request_v1-0.ps1
First published by DISA PKE Engineering - dodpke@mail.mil
https://dl.cyber.mil/pki-pke/zip/dc-cert-request-v1-0.zip

Credit to Author: Carl S rqvist
https://blog.qdsecurity.se/author/carlsorqvist/
For: Manually injecting a SID in a certificate (KB5014754)
https://blog.qdsecurity.se/2022/05/27/manually-injecting-a-sid-in-a-certificate/

User Input:
		1. Set $ReqOU = <option> to inhibit script prompt for Organizational Unit (OU) to be used in the certificate request
               <option> = "none"
               <option> = "USA | USN | USAF | DISA | DHRA | etc -> OU=<input>,OU=PKI,OU=DoD,O=U.S. Government,C=US"
               <option> = <Example:> "DC=subdomain,DC=root"
               <option> = "" (Exit Script)
		2. Set $ReqType = " 0 | 1 | 2 " to inhibit script prompt if local machine is not a active domain controller.
              0 - Generate Computer certificate request
              1 - Generate Domain Controller Certificate Reqeust
              2 - Display all Domain Contoller information (on console and in .txt file)
    3. Set 	$ReqTemplate = "<CertificateTemplate>"
              Add [RequestAttributes] CertificateTemplate = "<CertificateTemplate>" to .inf
              Include this parameter when reqeusting certificate signing from Microsoft MSCS Enterprise CA
              $ReqTemplate = "ComputerKBR", e.g. Duplicate of the Computer template that permits CSR specfied identity
              $ReqTemplate = "KerberosAuthenticationKBR", e.g. Duplicate of the KerberosAuthentication template that permits CSR specified identity
              This parameter is not need for DoD NPE certificate signing.

Script Result: Output files (.req, .inf, .txt) are created in the current working directory where the script is executed from.
		1. If run as UAC Admin new private key generation and certificate request in \\cert:\LocalMachine\REQUESTS
    2. Output file (.req) containing PKCS10 certificate request and DC/Computer information (including SID and GUID)
    3. Output file (.txt) details of DC/Computer information (including SID and GUID)
    4. Output file (.inf) may be removed when private key and CSR are succesfully generated
         The files are named <common_name>_<date>.txt where <common_name> is obtained from the local computer 
         and <date> is the date/time the script ran in the format YYYY-MM-DD-hh-mm-ss.
		

#Script Execution: 
The script should be executed in a Powershell console by changing to the directory where the script is located and 
executing .\sid-cert-request_v1-0.ps1 or the script may be executed from a different directory using the full path to the script, 
such as c:\sid-cert-requests\sid-cert-request_v1-0.ps1.
 

#Troubleshooting: 
If desired for troubleshooting purposes set $DebugPreference = "Continue" (Default is: "SilentlyContinue") before executing the script. 
This causes the inf file used to generate the certificate request to not be deleted. 
The inf file will be named using the format <common_name>_<date>.inf.

Note: If the local computer is not domain joined or the script is executed as a local user instead of a domain user the script will error and exit while attempting to find the AD Computer object.

