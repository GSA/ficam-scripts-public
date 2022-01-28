import json
import platform
import os
from subprocess import call
from pathlib import Path

#-----------------------------------------------------------------------------------------------------------------------
# File Name: certLoader.py
# Script Version: 1.0
# Date: 14 June 2018
# Created by: GSA PA Support Team - For questions, contact fpki@gsa.gov.
# Description: This script will support USG trust store management activities on Windows and macOS clients.
# Goals:
# 1 - Read in a .json config file to determine desired PIV auth eligible subordinate CAs for installation
# 2 - Depending on OS...
#       On Windows - Create p7b file of desired CA certificates
#       On macOS - Create an (unsigned) Apple Configuration Profile containing desired CA Certificates
#=======================================================================================================================
# Assumptions
# A - This script was written for Python 3.x compatibility
# B - For Windows, this script assumes ability to use OpenSSL to generate p7b. Requires OpenSSL path variable to avoid
#     script from having to be aware of OpenSSL install directory. 
#=======================================================================================================================
# VARIABLE DEFS

OS_VER=platform.system()

# MODIFY FILE PATHS BELOW, AS NEEDED
#  - CONFIG_PATH - Path location for the json file containing CA installation targets
#  - CERT_PATH - Path location for the directory containing CA certificates
#  - OUTPUT_FILE - Path location for where output file will be generated

if OS_VER == 'Windows':
    FILEEXT = '.p7b'
elif OS_VER == 'Darwin':
    FILEEXT = '.mobileconfig'

HOME = str(Path.home())
CONFIG_PATH = os.path.join(HOME, 'Desktop', 'Trust_Store_Mangagement_Script_V1_1', 'targets.json')
CERT_PATH = os.path.join(HOME, 'Desktop', 'Trust_Store_Mangagement_Script_V1_1', 'id-fpki-common-auth')
OUTPUT_FILE = os.path.join(HOME, 'Desktop', 'tsms-output' + FILEEXT)

## THESE LISTS WILL BE USED TO PRUNE THE INSTALL LIST TO ONLY DESIRED CAS AND PREPARE INSTALL COMMANDS
INSTALL_LIST = []
CERT_VAR = []

########################################################################################################################

## LOAD JSON CONFIGURATION FILE
with open(CONFIG_PATH) as data_file:
    data = json.load(data_file)

## READ NUMBER OF CA ENTRIES IN THE JSON FILE
LENGTH = (len(data['CERTIFICATION_AUTHORITIES']))

## READ THE JSON FILE ELEMENTS. IF THE CA IS A TARGET FOR INSTALLATION, APPEND TO INSTALL LIST
for x in range(LENGTH):
    INSTALL = (data['CERTIFICATION_AUTHORITIES'][x]['INSTALL'])

    if INSTALL == 'TRUE':
        INSTALL_LIST.append(data['CERTIFICATION_AUTHORITIES'][x])

## IF THERE ARE NO INSTALL TARGETS, QUIT THE SCRIPT.
if not INSTALL_LIST:
    print('\nNo installation targets have been selected. Quitting')
    quit()

## DETERMINE # OF CAS FOR INSTALLATION
INSTALL_LENGTH=(len(INSTALL_LIST))

## PRESENT INSTALLATION TARGETS TO OPERATOR
print('\nYou have selected the following CAs for installation: \n')

for x in range(INSTALL_LENGTH):
    ID = (INSTALL_LIST[x]['ID'])
    SUBJECT = (INSTALL_LIST[x]['SUBJECT'])
    ISSUER = (INSTALL_LIST[x]['ISSUER'])
    VALIDFROM = (INSTALL_LIST[x]['VALIDFROM'])
    VALIDTO = (INSTALL_LIST[x]['VALIDTO'])
    SERIAL = (INSTALL_LIST[x]['SERIAL'])
    FILENAME = (INSTALL_LIST[x]['FILENAME'])
    THUMBPRINT = (INSTALL_LIST[x]['THUMBPRINT'])
    CATEGORY = (INSTALL_LIST[x]['CATEGORY'])

    ### PRETTY PRINT CERT DATA FOR OPERATOR CONVENIENCE
    print('ID: ' + ID)
    print('SUBJECT: ' + SUBJECT)
    print('ISSUER: ' + ISSUER)
    print('VALID FROM: ' + VALIDFROM)
    print('VALID TO: ' + VALIDTO)
    print('SERIAL: ' + SERIAL)
    print('THUMBPRINT: ' + THUMBPRINT)
    print('CATEGORY: ' + CATEGORY)
    print('\n')

    ### DEPENDING ON OS, PREPARE INSTALLATION COMMAND ELEMENTS
    if OS_VER == 'Windows':
        #### ITERATE INSTALLATION TARGET FILENAMES TO ASSEMBLE OPENSSL COMMAND
        CERT_VAR.append('-certfile ' + CERT_PATH + '\\' + FILENAME + ' ')
        output = ''.join(CERT_VAR)

    elif OS_VER == 'Darwin':
        #### ITERATE INSTALLATION TARGET ATTRIBUTES TO ASSEMBLE REQUIRED CONFIG PROFILE XML ELEMENTS
        UNIQUESTRING = 'FEDERAL_PKI_CA_TSMS_' + (data['CERTIFICATION_AUTHORITIES'][x]['ID'])

        BLOB_PATH = os.path.join(CERT_PATH, FILENAME)
        with open(BLOB_PATH) as b:
            BLOB = b.read()
            #### STRIP HEADER/FOOTER (REQUIRED BY APPLE CONFIG)
            BLOB = BLOB.replace('-----BEGIN CERTIFICATE-----' + '\n' , '')
            BLOB = BLOB.replace('-----END CERTIFICATE-----' + '\n' , '')

        #### ASSEMBLE CA SPECIFIC XML ELEMENTS
        CERT_VAR.append('<dict>')
        CERT_VAR.append('<key>PayloadCertificateFileName</key>')
        CERT_VAR.append('<string>' + FILENAME + '</string>')
        CERT_VAR.append('<key>PayloadContent</key>')
        CERT_VAR.append('<data>')
        CERT_VAR.append(BLOB)
        CERT_VAR.append('</data>')
        CERT_VAR.append('<key>PayloadDescription</key>')
        CERT_VAR.append('<string>Adds a PKCS#1-formatted certificate</string>')
        CERT_VAR.append('<key>PayloadDisplayName</key>')
        CERT_VAR.append('<string>' + SUBJECT + '</string>')
        CERT_VAR.append('<key>PayloadIdentifier</key>')
        CERT_VAR.append('<string>com.apple.security.pkcs1.'+ UNIQUESTRING + '</string>')
        CERT_VAR.append('<key>PayloadType</key>')
        CERT_VAR.append('<string>com.apple.security.pkcs1</string>')
        CERT_VAR.append('<key>PayloadUUID</key>')
        CERT_VAR.append('<string>' + UNIQUESTRING + '</string>')
        CERT_VAR.append('<key>PayloadVersion</key>')
        CERT_VAR.append('<integer>1</integer>')
        CERT_VAR.append('</dict>')
        output = '\n'.join(CERT_VAR)

## ON WINDOWS: GENERATE P7B OF DESIRED INSTALLATION TARGETS USING OPENSSL
if OS_VER == 'Windows':
    ## PASS REQUIRED ASSEMBLY COMMAND TO CMD LINE CALL
    CMD = 'openssl crl2pkcs7 -nocrl ' + output + ' -out ' + OUTPUT_FILE
    GENERATE_P7B = call(CMD, shell=True)

## ON MACOS: GENERATE APPLE MOBILE CONFIG PROFILE CONTAINING DESIRED INSTALLATION TARGETS
elif OS_VER == 'Darwin':
    ## NOTE: DESIGN DECISION TO WRITE REQUIRED PROFILE "FOREMATTER" TO OUTPUT FILE RATHER THAN INCLUDING ANOTHER FILE
    ## IN THE INSTALLATION PACKAGE. PURPOSEFULLY DID NOT COMBINE LINES TO FACILITATE SCRIPT REVIEW BY OPERATORS.

    with open(OUTPUT_FILE, 'w') as f:
        xml_var = []
        xml_var.append('<?xml version=\'1.0\' encoding=\'UTF-8\'?>')
        xml_var.append('<!DOCTYPE plist PUBLIC \'-//Apple//DTD PLIST 1.0//EN\' \'http://www.apple.com/DTDs/PropertyList-1.0.dtd\'>')
        xml_var.append('<plist version=\'1.0\'>')
        xml_var.append('<dict>')
        xml_var.append('<key>PayloadContent</key>')
        xml_var.append('<array>')
        ### INJECT TARGET CERTIFICATE DATA BELOW
        xml_var.append(output)
        xml_var.append('</array>')
        xml_var.append('<key>PayloadDisplayName</key>')
        xml_var.append('<string>Federal PKI CA Trust Store Management Tool</string>')
        xml_var.append('<key>PayloadIdentifier</key>')
        xml_var.append('<string>FEDERAL_PKI_CA_TSMS</string>')
        xml_var.append('<key>PayloadRemovalDisallowed</key>')
        xml_var.append('<false/>')
        xml_var.append('<key>PayloadType</key>')
        xml_var.append('<string>Configuration</string>')
        xml_var.append('<key>PayloadUUID</key>')
        xml_var.append('<string>AAD17D9A-DA41-4197-9F0F-3C3C6B4512F9</string>')
        xml_var.append('<key>PayloadVersion</key>')
        xml_var.append('<integer>1</integer>')
        xml_var.append('</dict>')
        xml_var.append('</plist>')
        config_output = '\n'.join(xml_var)
        f.write(config_output)

print('An installation file has been created at: ' + OUTPUT_FILE)
