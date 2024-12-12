# ONESIGN REST API Electronic Invoicing 1.5.6

## Version History

| Version | Date       | Description                                                                                                                       |
|---------|------------|-----------------------------------------------------------------------------------------------------------------------------------|
| 1.0     | 2023-10-01 | Initial release                                                                                                                   |
| 1.1     | 2023-11-15 | Addition of crypto headers                                                                                                        |
| 1.2     | 2023-12-15 | Addition of Greek version AADE/B2G                                                                                                |
| 1.3     | 2023-12-22 | Added endpoints, sample payloads, responses, postman collection                                                                   |
| 1.4     | 2024-01-10 | Updated the sample payloads with lineComments and itemCPV elements, HMAC Guide                                                    |
| 1.4.1   | 2024-01-15 | Updates Sample B2G payload with PEPPOL required fields, counterpart.name, invoiceDetails.quantity, invoiceDetails.measurementUnit |
| 1.4.2   | 2024-01-16 | Updates HMAC headers                                                                                                              |
| 1.5.0   | 2024-02-22 | Added Transmission Failure 2, Status Request, Updated POSTMAN                                                                     |
| 1.5.1   | 2024-05-28 | Added self handling of Transmission Failure 2                                                                                     |
| 1.5.2   | 2024-09-11 | Added commands to extract raw keys, and some API updates, updated postman collection                                              |
| 1.5.3   | 2024-11-26 | Updated the AADE doc links, added sample B2B and B2C XML requests                                                                 |
| 1.5.4   | 2024-11-27 | Added the supported measurement units                                                                                             |
| 1.5.5   | 2024-11-27 | Added soft rejection resubmit header                                                                                              |
| 1.5.6   | 2024-12-12 | Added business-id and location-id headers                                                                                         |


## Introduction

Welcome to the e-invoicing API, a REST API designed for managing functions related to electronic invoicing by ONESIGN
and submission through a provider to AADE, and subsequently to KED (GGPS) if required.
This document provides details on how to interact with the API, including HTTP headers, request and response structures,
as well as cryptographic certification.

## Test Environment (Sandbox)

The test environment for all API calls is:

```
https://onesign-api.onesys.gr/
```

## Authentication

To submit any call to the API, the following two (2) methods are supported:
1) You should use an API key and Hash-based Message Authentication Code (HMAC).
2) Use Json Web Token (JWT).

In the case of a Service 2 Service call, in the headers of the call (HTTP Request-Header) related to the authentication
of the caller, the agreed-upon API key must be present.

```
client-api-key:
client-signature:
```

## HMAC Authentication Implementation Guide

### Overview

This section provides guidance on implementing HMAC authentication for server-to-server communication. It involves the
use of an API key and a secret. The API key is used in the request header, while the secret is used to generate an HMAC
signature from the request body.

### Requirements

### API Key and Secret:
Two distinct pieces of data will be provided:

**API Key:** To be sent in the request header.

**Secret:** To be used for generating the HMAC signature.

**Encryption of Request Body:**

Encrypt the body using the secret.
For XML content type, encrypt the body directly.
For JSON content type, stringify the JSON before encryption.

**Request Headers:**

`client-api-key:` Set this to the provided API key.

`client-signature:` Set this to the HMAC encrypted string of the request body using the secret.

> **Note:**
> Do not send Authorization Bearer in header for HMAC Authentication
>

Example

Here is a pseudocode example:

```{python}
import hmac
import hashlib

def encrypt_body(api_key, body, content_type):
    if content_type == 'application/json':
        body = json.stringify(body)
    signature = hmac.new(api_key.encode(), body.encode(), hashlib.sha256).hexdigest()
    return signature

api_key = "dsfdsf3434"
body = "<xml>...</xml>"  # or JSON object
content_type = "application/xml"  # or "application/json"

signature = encrypt_body(api_key, body, content_type)

headers = {
    "client-api-key": api_key,
    "client-signature": signature
}

# send request with headers

```

## 1. Invoice Submission
To submit an invoice, it is necessary for each call to be accompanied by a cryptographic header, which includes the
device signature, hashing of the essential elements of the invoice, details. Additionally, the payload  must comply with
the specifications of AADE for document submission.

### Key Generation and Device Certification in the Application
To sign each document, device certification in the application is required. The EdDSA algorithm, specifically Ed25519,
has been chosen as the primary encryption method.
More information about the algorithm can be found in the following link [EdDSA and Ed25519].

To certify a device in the application, the following steps should be followed:
1. Generate keys from the device.
2. Sign the keys from the application.

#### Key Generation
##### Linux/Unix/MacOS
From a Linux/Unix/MacOS system, keys can be generated using the following commands, utilizing OpenSSL through the
terminal/console.

```shell
$ openssl genpkey -algorithm Ed25519 -out private_key.pem
$ openssl pkey -in private_key.pem -pubout -out public_key.pem
```

For more information, you can refer to the following link [How To Generate ed25519 SSH Key].

##### Windows
From a Windows system (Windows 10+, Windows Server 2016+), keys can be generated using the following command,
using OpenSSL through Windows PowerShell.

```shell
ssh-keygen -t ed25519
```

Once the `private_key.pem` and `public_key.pem` files have been generated, we need to extract the
raw hexadecimal keys for use in your application. To do this, we can use the following commands:

```shell
$ openssl pkey -in private_key.pem -outform DER | tail -c 32 | xxd -p -c 32 > raw_private_key.hex
$ openssl pkey -in public_key.pem -pubin -outform DER | tail -c 32 | xxd -p -c 32 > raw_public_key.hex
```

For more information, refer to the following link: [Key-based authentication in OpenSSH for Windows].

#### Submission and Signing of Keys
After generating the keys, device certification in the application is required. To certify the device in the
application, the public key generated in the previous process should be signed, and both elements (public key and signature)
should be submitted using the "signing-devices" method in the application.
The application will in turn return a device specific signature that the device should use in each call.

Alternatively, it can be done using curl with the following command, replacing the necessary fields.

```shell
curl --location 'https://onesign-api.onesys.gr/signing-devices' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer eyJhbGciOiJUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6Ijc2YjM1ZDdlLWVkODQtNGI0ZS1hODhiLWQ2Y2RlOWRiY2Q3YiIsImZpcnN0TmFtZSI6Ik94aW51cyIsImxhc3ROYW1lIjoiSG9sZGluZ3MiLCJlbWFpbCI6InNwQG94aW51cy5ob2xkaW5ncyIsImFjY2Vzc1R5cGUiOiJqd3QiLCJpYXQiOjE3MDI4Nzg3MTB9.ZLZGFse5RJMFkgziYs-nH8qYTveztOzmhApbXN0poPA' \
--data '{
    "deviceName": "device 1",
    "deviceType": "virtual",
    "deviceSerial": "98594583434",
    "devicePublicKey": "0e6c1b121a9ce7f593fb6f1a3794090885a93de2812fafaf6b6f5c0867477f4",
    "devicePublicKeySignature": "b926537302ebcacb367a2e4bf608a2c7751322fd835915a640b95ffa37208775d6b5d56d6c1f01b2df7f20b4fd31749cce8f6c950b10f202d09cc331664ef07",
    "authority": "AAD",
    "businessId": "<VAT_WITHOUT_EL>"
}'
```

#### Revoking a device
If the signing device needs to be revoked from use, a delete request can be sent to disable the device from signing anymore invoices.

```shell
curl --location --request DELETE 'https://onesign-api.onesys.gr/signing-devices/:businessId/:deviceId' \
--header 'Authorization: Bearer eyJhbGciiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImtqZG5nZTg0NTQwOTgiLCJmaXJzdE5hbWUiOiJSaWNreSIsImxhc3ROYW1lIjoiTWFydGluIiwiZW1haWwiOiJtLnNpZGRpcXVpQG94aW51cy5pbyIsImFwaUtleSI6ImhoaGpqamRkZGtrayIsInNlY3JldCI6InNlY3JldCIsImFjY2Vzc1R5cGUiOiJTMlMiLCJjcmVhdGVkQXQiOiIyMDIzLTEyLTE0VDA3OjA3OjE0LjgzMloiLCJ1cGRhdGVkQXQiOiIyMDIzLTEyLTE0VDA3OjA3OjE0LjgzMloiLCJpYXQiOjE3MDI1NDE3OTZ9.a3XfDBcXVJ5mZFkKR7u5Er_zT9L06SaIUzi9biYD6gU' \
--data ''
```

#### Retrieving an Invoice
A successfully submitted invoice can be retrieved with the Uid from the response of the successfully submitted invoice.

```shell
curl --location 'https://onesign-api.onesys.gr/invoice/public/qrcode/:uid'
```

### Request Headers:
Following is the list of supported headers that are analysed in detail.

| Name                                                   | Description (PEPPOL Identifier)                                                               | Example Value                                                                                                                      |
|--------------------------------------------------------|-----------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------|
| Settings-Authority-ID                                  | Identifier of the authority.                                                                  | "AAD"                                                                                                                              |                                                                                                               
| Settings-Is-Peppol-Required                            | Logical value indicating whether Peppol is required.                                          | true                                                                                                                               |                                                                                                                      
| Settings-Document-TemplateID                           | Identifier of the document template.                                                          | 1                                                                                                                                  |                                                                                                                           
| Fiscal-Header-Invoice-ID                               | Identifier of the invoice.                                                                    | "fd4bdc10-b0f3-4f3b-beb1-14fc4f42dc2b"                                                                                             |                                                                               
| Fiscal-Header-Issuer-TaxID                             | Tax identification number of the issuer (AccountingSupplierParty.PartyTaxScheme.CompanyID)    | "987654321"                                                                                                                        |                                                                                    
| Fiscal-Header-Recipient-TaxID                          | Tax identification number of the recipient (AccountingCustomerParty.PartyTaxScheme.CompanyID) | "123456789"                                                                                                                        |                                                                                       
| Fiscal-Header-TimeStamp-Epoch                          | Epoch timestamp (Invoice.IssueDate)                                                           | "1702140669"                                                                                                                       |                                                                                                            
| Fiscal-Header-Document-Type                            | Document type (Invoice.InvoiceTypeCode)                                                       | "380"                                                                                                                              |                                                                                                                 
| Fiscal-Header-Document-Value                           | Document value (Invoice.LegalMonetaryTotal.TaxInclusiveAmount)                                | 1240.00                                                                                                                            |                                                                                                                
| Fiscal-Header-Document-Tax-Value                       | Document tax value (Invoice.TaxTotal.TaxAmount)                                               | 240.00                                                                                                                             |                                                                                                                     
| Fiscal-Header-Currency                                 | Currency (Invoice.DocumentCurrencyCode)                                                       | "EUR"                                                                                                                              |                                                                                                            
| Fiscal-Header-Tax-Currency                             | Tax currency (Invoice.TaxCurrencyCode)                                                        | "EUR"                                                                                                                              |                                                                                                                
| Crypto-Header-Fiscal-Header-SHA256-Hash                | SHA256 hash of the fiscal header.                                                             | "ba2a7576612c69a3dd18da20b3c47c598afabe1c1aec9fa98a1f888daecf5a5d"                                                                 |                                                                
| Crypto-Header-Invoice-Payload-SHA256-Hash              | SHA256 Hash of the invoice content.                                                           | "31481f96c1677e8f0e1cd81d941561824f19b344476612c69a3dd18db1a47277"                                                                 |                                                                  
| Crypto-Header-Signature                                | Signature of (Fiscal Header and Invoice Payload Hash) with the Privatκό Κλειδί της Συσκευής   | "0D64FD1B9B95790E473489D6964B4D1A76811BC3A63407118B3CCCB3BC6F789A384F2EF80BD8A70347ACD89E0C342F1DA300C85A5830254011543A3B64EB9206" |            
| Crypto-Header-Previous-Invoice-Fiscal-Header           | SHA256 hash of the previous fiscal header.                                                    | "aa3a677e8f0e19a3dd1876612c69a3dd18d98afa76612c69a3dd18df888daea5a"                                                                |                                                                                     
| Crypto-Header-Public-Key-of-Signatory-Device           | Public Key of the Signing Device (Base 64)                                                    | "MCowBQYDK3VwAyEAvR97AJTKyGNAjOYROXGk+H367Ix1kOAMNKQwpTuvOfU="                                                                     |                     
| Crypto-Header-Signature-of-Signatory-Device-Public-Key | Signature of the Public Key of the Signing Device                                             | "E473489D6964B4D1A76811BC3A634070D64FD15830254011B9B95790118B3CCCB3BC6F789A384F2EFF1DA300C85A543A3B64EB920680BD8A70347ACD89E0C342" |
| Source-System                                          | Source System of the document                                                                 | "SRC-100"                                                                                                                          |
| Document-Recipient-Email                               | Email Address of Document Recipient                                                           | info@example.com                                                                                                                   |
| Soft-Rejection-Resubmit                                | Resubmission of B2G PEPPOL document after soft rejection                                      | true                                                                                                                               |
| Business-Id                                            | Business id in UUID format                                                                    | "4b0d382d-dc6a-4605-aadd-1313156915c4"                                                                                             |
| Location-Id                                            | Location id in UUID format                                                                    | "5d677662-7c3a-42ab-a757-aa777bfcd2c0"                                                                                             |

As shown in the table above, the headers required for the secure and successful submission of a document consist of
three separate parts:

- **Settings**: Settings section includes configuration options for the API.
- **Fiscal Header**: Fiscal Header section contains information related to the fiscal header in the API.
- **Crypto Header**: Crypto Header section includes cryptographic information for the API.

#### SETTINGS - Document Configuration Settings

The settings section contains the following three (3) configurations related to the type of content being exchanged.

##### Authority ID
Description: Identifier of the authority to which the document is addressed (e.g., AADE).
- Example Value: "AAD"
- Default Value:
- Mandatory: YES

##### Is Peppol Required
Description: Boolean value indicating whether the document concerns only AADE or both AADE and PEPPOL (B2G).
- Example Value: false
- Default Value: false
- Mandatory: NO

##### Document Template ID
Description: Default template for displaying/printing the document.
- Example Value: 1
- Default Value:
- Mandatory: NO

#### Fiscal Header - Financial Section

The financial section of the header includes the following values, all of which are necessary for document submission.

##### Invoice ID - Document Identifier
Description: Unique identifier of the document in UUID format (Recommended version 4).
- Example Value: "fd4bdc10-b0f3-4f3b-beb1-14fc4f42dc2b"

##### Issuer Tax ID - Issuer's VAT Number
Description: VAT Identification Number of the issuer (AccountingSupplierParty.PartyTaxScheme.CompanyID).
- Example Value: "987654321"

##### Recipient Tax ID - Recipient's VAT Number
Description: VAT Identification Number of the recipient (AccountingCustomerParty.PartyTaxScheme.CompanyID).
- Example Value: "123456789"

##### TimeStamp - Epoch Timestamp
Description: Epoch timestamp (Invoice.IssueDate).
- Example Value: "1702140669"

##### Document Type - Document Type
Description: Document type according to PEPPOL (Invoice.InvoiceTypeCode).
- Example Value: "380"

##### Document Value - Document Value
Description: Document value (Invoice.LegalMonetaryTotal.TaxInclusiveAmount).
- Example Value: 1240.00

##### Document Tax Value - Document Tax Value
Description: Document tax value (Invoice.TaxTotal.TaxAmount).
- Example Value: 240.00

##### Currency - Currency
- Description: Currency (Invoice.DocumentCurrencyCode).
- Example Value: "EUR"

##### Tax Currency - Tax Currency
- Description: Tax currency (Invoice.TaxCurrencyCode).
- Example Value: "EUR"

#### Crypto Header

The cryptographic section of the headers includes the necessary security and encryption data of the document to verify
its validity.

##### Fiscal Header SHA256 Hash
- Description: SHA256 hash of all headers in the financial section (fiscal-header fields) concatenated into one field
- separated by the "|" pipe character.
  - e.g., "fd4bdc10-b0f3-4f3b-beb1-14fc4f42dc2b|987654321|123456789|1702140669|380|1000.00|240.00|EUR|EUR"

The order of appearance is the following:
- Fiscal-Header-Invoice-ID
- Fiscal-Header-Issuer-TaxID
- Fiscal-Header-Recipient-TaxID
- Fiscal-Header-TimeStamp-Epoch
- Fiscal-Header-Document-Type
- Fiscal-Header-Document-Value
- Fiscal-Header-Document-Tax-Value
- Fiscal-Header-Currency
- Fiscal-Header-Tax-Currency

- Example Value: "ba2a7576612c69a3dd18da20b3c47c598afabe1c1aec9fa98a1f888daecf5a5d"

##### Invoice Payload SHA256 Hash
- Description: SHA256 hash of the entire payload of the call.
- Example Value: "31481f96c1677e8f0e1cd81d941561824f19b34448026a2639e56a92b1a47277"

##### Signature
- Description: Signature with the device's public key (previously authenticated) of the concatenation of the two (2)
- previous hashes.
  e.g., sign((FiscalHeaderHash+ InvoicePayloadHash),myPrivateKey)
- Example Value: "0D64FD1B9B95790E473489D6964B4D1A76811BC3A63407118B3CCCB3BC6F789A384F2EF80BD8A70347ACD89E0C342F1DA300C85A5830254011543A3B64EB9206"

##### Previous Invoice Fiscal Header SHA256 Hash
- Description: SHA256 hash of all headers in the financial section of the *previous document* issued by the device.
- Example Value: "aa3a677e8f0e19a3dd1876612c69a3dd18d98afa76612c69a3dd18df888daea5a"

##### Public Key of Signatory Device
- Description: The public key of the device (Base 64)
- Example Value: "MCowBQYDK3VwAyEAvR97AJTKyGNAjOYROXGk+H367Ix1kOAMNKQwpTuvOfU="

##### Signature of Signatory Device Public Key
- Description: Signature that was received on registering the device.
- Example Value: "E473489D6964B4D1A76811BC3A634070D64FD15830254011B9B95790118B3CCCB3BC6F789A384F2EFF1DA300C85A543A3B64EB920680BD8A70347ACD89E0C342"

##### Business-Id
- Description: Business Id of the business in UUID format
- Example Value: UUID format
- Mandatory: Yes

##### Location-Id
- Description: Location Id of the location in UUID format
- Example Value: UUID format
- Mandatory: Yes

##### Source System
- Description: Source System of the document
- Example Value: "SRC-100"
- Mandatory: NO

##### Document-Recipient-Email
- Description: Email Address of Document Recipient
- Example Value: "info@example.com"
- Mandatory: NO

##### Soft-Rejection-Resubmit
- Description: Resubmit the document after soft rejection
- Example Value: true
- Mandatory: NO


#### Request Body (Payload)

The request body should contain the XML format of AADE as it has been configured with the latest version of the
specifications.

***Current specifications 1.0.9***

The relevant specifications can be found in the following link (link through the Provider).

[myDATA API Documentation_Providers_v1.0.9_official.pdf]

The references mentioned in the Provider's specifications point to the ERP specifications, which are available at the
following link (link ERP).

[myDATA API Documentation_v1.0.9_official_ERP.pdf]


#### Call Section for PEPPOL (B2G)
In case the invoice pertains to a B2G document that needs to be routed through the PEPPOL network, the indication
`Settings-Is-Peppol-Required` should be filled in the call header with the value `true`.

In this case, the document will be checked for the existence of the following fields which should be included in the
last element <b2g-peppol> within <invoice>

| Field              | Description                             | Sample Value          | Required | Peppol Field |
|--------------------|-----------------------------------------|-----------------------|----------|----------|
| projectReference   | Reference for the project               | {ProjectReference}   | Yes      | BT-11 PROJECT_REFERENCE_ID |
| contractReference  | Reference for the contract              | {ContractReference}  | Yes      | BT-12 CONTRACT_REFERENCE_ID |
| assignReference    | ID of the party assigned the contract   | 1000.E00961.0001     | Yes      | BT-46 PARTY_ID |
| deliveryParty      | Delivery Location                       | General Management   | No       | BT-70 PARTY_ID |


#### Project Reference
The field projectReference (BT-11) is completed as follows according to the guidelines of the General Secretariat of
Information Systems (GSIS):

- "1| ΑΔΑ Ανάληψης": When the contracting authority is an entity of the Central Administration according to Article 14
  of Law 4270/2014 (A.143), and the expenses are charged to the regular budget. In this case, the entry includes:
- the indication "1," the separator "|," and the Internet Posting Number (ΑΔΑ) of the relevant credit commitment decision.

- "2| Encoded Reference Number": When the expenses are charged to the Public Investment Budget (ΠΔΕ). In this case,
  the entry includes: the indication "2," the separator "|," and the Encoded Reference Number of the project,
- as specified in the relevant Special Account and is mandatory in the contract text.

- "3| ΑΔΑ Ανάληψης": When the expenses do not burden the above budgets. In this case, the entry includes:
  the indication "3," the separator "|," and the Internet Posting Number (ΑΔΑ) of the relevant credit commitment decision.

#### Contract Reference
In the field contractReference (BT-12), the Internet Posting Number of the Contract Registry (ΑΔΑΜ) is filled in.
If there is no written contract (procurements under €2,500), the ΑΔΑΜ of the Assignment Decision from the Contracting
Authority is entered. The value is entered as 0 if there is no Assignment Decision or if the amount is less than €1,000.
If the Contract is under €30,000, the ΑΔΑΜ of the Purchase Order is entered.

#### Assign Reference
In the field assignReference (B5-46), the Assigning Authority Number is entered according to the registry.
Below is a sample of the mentioned section embedded in the XML format of AADE.

```xml
<invoice>
    ...
    <invoiceDetails>
        <lineNumber>...</lineNumber>
        ...
        <itemCPV>03.1234567</itemCPV>
    </invoiceDetails>
    <b2g-peppol>
        <projectReference>111/1123</projectReference>
        <contractReference>SSS/SSS</contractReference>
        <assignReference>1000.E00961.0001</assignReference>
        <deliveryParty>ΓΕΝΙΚΗ ΔΙΕΥΘΥΝΣΗ</deliveryParty>
    </b2g-peppol>
</invoice>
```

Furthermore, since the document concerns Business-to-Government (B2G), the CPV code of the product must be included
on each line, encoded according to the Common Product Vocabulary guidelines (CPV codes) as specified by the EE
(European Union). The document will be verified for the presence of a value in the itemCPV field, which should
be the last field in the invoiceDetails element.


| Πεδίο              | Περιγραφή                              | Ενδεικτική Τιμή      | Υποχρεωτικό | PEPPOL BT |
|--------------------|----------------------------------------|----------------------|-------------|-----------|
| itemCPV            | Common Product Vocabulary (CPV) | 73000000-2 | Yes   | BT-158 CLASSIFICATION_ID (CPV)


Below is a sample of the mentioned segment integrated into the XML format of AADE.

```xml
<invoice>
    ...
    <invoiceDetails>
        <lineNumber>...</lineNumber>
        ...
        <itemCPV>03.1234567</itemCPV>
    </invoiceDetails>
    ...
</invoice>
```

#### Measurement Units

Regarding B2G where measurement unit is a mandatory field the following Measurement Units are supported and below is the mapping of these (payload value, peppol mapped unit)

| Payload Value | Peppol Mapped Unit | Peppol Unit Description |
|---------------|--------------------|-------------------------|
| 1             | "H87"              |                         |
| 2             | "KGM"              |                         |
| 3             | "LTR"              |                         |
| 4             | "MTR"              |                         |
| 5             | "MTK"              |                         |
| 6             | "MTQ"              |                         |
| 7             | "H87"              |                         |
| 101           | "C62"              | Unit                    |
| 102           | "KWT"              | Kilowatt                |
| 103           | "KWH"              | Kilowatt hour           |
| 104           | "EA"               | Each                    |
| 105           | "HUR"              | Hour                    |
| 106           | "XBX"              | Box                     |
| 107           | "KGM"              | Kilogram                |
| 108           | "LTR"              | Litre                   |
| 109           | "MTR"              | Metre                   |
| 110           | "MTQ"              | Cubic metre             |
| 111           | "MAW"              | Mega watt               |
| 112           | "MWH"              | Mega watt hour          |
| 113           | "PR"               | Pair                    |
| 114           | "SET"              | Set                     |
| 115           | "H87"              | Piece                   |
| 116           | "HUR"              | Hours                   |
| 117           | "TNE"              | Tonne                   |
| 118           | "M4"               | Monetary value          |


### Response to Request

In the case of a successful document submission, you will receive a response containing the following fields:
- **MARK (Unique Registration Number)**
- **UID (HASH Signature of the document from AADE)**
- **AUTH (HASH Signature of the document from the provider)**
- **QrCode (public URL where someone can verify the document)**

### Transmission Failure 2 (AADE Down Scenario)

Depending on the business settings, ONESIGN handles this scenario differently.

#### Scenario 1 - ONESIGN re-submits the document to AADE

In this scenario, in the case of an unsuccessful document submission from the Provider to AADE, provided that the invoice is valid, you
will receive similar success response containing the same fields as the successful response (empty), with the addition
of the following field:
- **extRefId (Unique ONESIGN Reference Id)**

Additionaly, the statusCode will be "Transmission Failure"

The following is a sample response

```xml
<?xml version=`"1.0" encoding="UTF-8" standalone="yes"?>
<ResponseDoc xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <response>
    <index>1</index>
    <invoiceUid/>
    <invoiceMark/>
    <authenticationCode/>
    <statusCode>Transmission Failure</statusCode>
    <qrUrl>https://onesign.onesys.gr/invoices/ca06f967-cc01-49ca-aa64-a7dacfc21813</qrUrl>
    <extRefId>3965ab8e-4b59-499d-a102-32b3eeef3268</extRefId>
  </response>
</ResponseDoc>
```

Using the returned external reference, extRefId, you can query the status of the document, whether is was submitted at a
later stage. ONESIGN will retry indefinitely the submission to AADE and eventually the document will receive the required
values (MARK, AUTH, UID).

#### Scenario 2 - Customer re-submits the document to ONESIGN

In this scenario, on un-successful document submission from the Provider to AADE, ONESIGN will not re-submit the document to AADE. ONESIGN
will return a response as below, and the customer is responsible for re-submitting the document.

The following is a sample response

```xml
<?xml version=`"1.0" encoding="UTF-8" standalone="yes"?>
<ResponseDoc xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <response>
    <index>1</index>
    <invoiceUid/>
    <invoiceMark/>
    <authenticationCode/>
    <statusCode>Transmission Failure</statusCode>
    <qrUrl/>
  </response>
</ResponseDoc>
```


#### Document Status Request
To query the status of the transmission failed document, you can use the following endpoint :
```http
GET https://onesign-api.onesys.gr/pending-documents/:extRefId/status
```

You will receive the following response:

```XML
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<ResponseDoc>
    <response>
        <index>1</index>
        <invoiceUid>398DF85AF27196AF7F617DEA648C26D49EA85064</invoiceUid>
        <invoiceMark>400001924527016</invoiceMark>
        <authenticationCode>B94D9C24035C10383913CE7FDE249F9875326A99</authenticationCode>
        <statusCode>Success</statusCode>
        <qrUrl>https://onesign.onesys.gr/invoices/ca06f967-cc01-49ca-aa64-a7dacfc21813</qrUrl>
    </response>
</ResponseDoc>
```


### Error Messages (B2G)

If the document pertains to a B2G service, additional checks are performed, and the following error messages are returned in case of errors:

| HTTP Status | Error Code       | Error No | Source         | Description                                                                                          |
|-------------|------------------|----------|----------------|------------------------------------------------------------------------------------------------------|
| 200 OK      | ValidationError  | 751      | Document-B2G   | B2G Document without ADAM code (Project Reference)                                                 |
| 200 OK      | ValidationError  | 752      | Document-B2G   | B2G Document without contract code (Contract Reference)                                            |
| 200 OK      | ValidationError  | 753      | Document-B2G   | B2G Document without assigning authority code (AssignReference)                                    |
| 200 OK      | ValidationError  | 754      | Document-B2G   | B2G Document without CPV code on line {lineNumber} (itemCPV)                                       |
| 200 OK      | ValidationError  | 755      | Document-B2G   | B2G Document without delivery description (DeliverParty)                                           |
| 200 OK      | ValidationError  | 761      | Document-B2G   | B2G Document with incorrect ADAM code (Project Reference)                                           |
| 200 OK      | ValidationError  | 762      | Document-B2G   | B2G Document with incorrect contract code (Contract Reference)                                      |
| 200 OK      | ValidationError  | 763      | Document-B2G   | B2G Document with incorrect assigning authority code (AssignReference)                              |
| 200 OK      | ValidationError  | 764      | Document-B2G   | B2G Document with incorrect CPV code on line {lineNumber} (itemCPV)                                 |

In case of submission failure, the following errors are returned by the application

###Application Errors

| HTTP Status | Error Code       | Error No | Source  | Description |                                                                                       
|-------------|------------------|----------|--------------|-------------------|
| 200 OK      | XMLSyntaxError   | 101      | App  | XML Syntax Validation Error |                                                
| 200 OK      | ValidationError  | 102      | App  | Vat number {vatNumber} does not belong to active corporation |                              
| 200 OK      | ValidationError  | 103      | App  | Please pass mark in the request parameters |                                           
| 200 OK      | ValidationError  | 104      | App  | Requested Invoice was not found |                         
| 200 OK      | ValidationError  | 105      | App  | User VAT number {vatNumber} is not authorized to execute this method for VAT number {vatNumber} |           

### Invoice Errors
| HTTP Status | Error Code       | Error No | Source | Description                                          |
|-------------|------------------|----------|--------------|------------------------------------------------------------|
| 200 OK      | ValidationError  | 201      | Invoice  | User VAT number {vatNumber} is not authorized to execute this method |                     
| 200 OK      | ValidationError  | 202      | Invoice  | Invalid Greek VAT number                                  |
| 200 OK      | ValidationError  | 203      | Invoice  | Gross Value doesn't match with sum of net value plus taxes |
| 200 OK      | ValidationError  | 204      | Invoice  | {Field} is mandatory for this invoice type                 |
| 200 OK      | ValidationError  | 205      | Invoice  | {Field} is forbidden for this invoice type                 |
| 200 OK      | TechnicalError   | 206      | Invoice  | Unexpected technical error for invoice line               |
| 200 OK      | ValidationError  | 207      | Invoice  | The sum of net values of the invoice lines doesn't match with total net value of the invoice |
| 200 OK      | ValidationError  | 208      | Invoice  | The sum of gross values of the invoice lines doesn't match with total gross value of the invoice |
| 200 OK      | ValidationError  | 209      | Invoice  | The sum of vat amount of the invoice lines doesn't match with total vat amount of the invoice |
| 200 OK      | ValidationError  | 210      | Invoice  | The sum of withheld amount of the invoice lines doesn't match with total withheld amount of the invoice |
| 200 OK      | ValidationError  | 211      | Invoice  | Exchange Rate must be greater than 0 when the currency is not Euro                                                 |
| 200 OK      | ValidationError  | 212      | Invoice  | AA element must be number with value between 0 and 18446744073709551615 for issuer from Greece                    |
| 200 OK      | ValidationError  | 213      | Invoice  | {Field} must have value 0 for this invoice type                                                                    |
| 200 OK      | ValidationError  | 214      | Invoice  | Element {Element} must be sent only if it is true                                                                 |
| 200 OK      | ValidationError  | 215      | Invoice  | Vat category must have value {value} for this invoice type [Possible { value } values: {7, 8}                    |
| 200 OK      | ValidationError  | 216      | Invoice  | Vat category must have value other than 8 for this invoice type                                                    |
| 200 OK      | ValidationError  | 217      | Invoice  | When vatCategory has value 7, element vatExemptionCategory is mandatory                                            |
| 200 OK      | ValidationError  | 218      | Invoice  | Vat Amount must have value 0 for this invoice type                                                                 |
| 200 OK      | ValidationError  | 219      | Invoice  | Issuer Name is forbidden for Issuer from Greece                                                                    |
| 200 OK      | ValidationError  | 220      | Invoice  | Counterpart Name is forbidden for Counterpart from Greece                                                          |
| 200 OK      | ValidationError  | 221      | Invoice  | {Field} is forbidden for the lines that have invoiceDetailType = 2 for this invoice type                            |
| 200 OK      | ValidationError  | 222      | Invoice  | {Field} must have value greater than 0 for this invoice type                                                        |
| 200 OK      | ValidationError  | 223      | Invoice  | Unsupported invoice type                                                                                            |
| 200 OK      | ValidationError  | 224      | Invoice  | Taxes are allowed either per invoice line or per invoice (not in both)                                             |
| 200 OK      | ValidationError  | 225      | Invoice  | {Field} must exist (cannot be null) since the {Field} is not null (invoice line {lineNumber})                      |
| 200 OK      | ValidationError  | 226      | Invoice  | The sum of {field} amount of the invoice {section} doesn't match with total {field} amount of the invoice          |
| 200 OK      | ValidationError  | 227      | Invoice  | {Field1} cannot exist (must be null) since the {Field1} is null (invoice line : {lineNumber}) [ Possible {Field1, Field2} values: {‘feesAmount’, ‘feesPercentCategory’}, {‘stampDutyAmount, ‘stampDutyPercentCategory’}, {‘withheldAmount’, ‘withheldPercentCategory’}] |
| 200 OK      | ValidationError  | 228      | Invoice  | {Field} is invalid [Possible {Field} values: {UID, InvoiceType}                               |
| 200 OK      | ValidationError  | 229      | Invoice  | {Field1} is not correct according to the given: {Field2} (invoice line: {lineNumber}) [ Possible {Field1, Field2} values: {‘feesAmount’, ‘feesPercentCategory’}, {‘stampDutyAmount, ‘stampDutyPercentCategory’}, {‘withheldAmount’, ‘withheldPercentCategory’}] |
| 200 OK      | ValidationError  | 230      | Invoice  | {Field} is mandatory for invoice detail (number} [Possible {Field} values: {E3 classifications, VAT classifications}        |
| 200 OK      | ValidationError  | 231      | Invoice  | {Field} is forbidden for invoice detail (number} [Possible {Field} values: {E3 classifications, VAT classifications}       |
| 200 OK      | ValidationError  | 233      | Invoice  | UID: " + {uid} + " has already been sent                                                   |
| 200 OK      | ValidationError  | 234      | Invoice  | The values 7 or 8 are not allowed for Vat Category for this invoice type                                                  |
| 200 OK      | ValidationError  | 235      | Invoice  | Issuer must be different from counterpart                                                                              |
| 200 OK      | ValidationError  | 236      | Invoice  | The Sender (vatnumber): " + {afm} + " must be different from the issuer (vatnumber)                                    |
| 200 OK      | ValidationError  | 237      | Invoice  | Underlying Value(s) of taxes cannot be greater than the total net value of invoice                                     |
| 200 OK      | ValidationError  | 239      | Invoice  | Taxamount(s) of taxes cannot be greater than the total net value of invoice                                            |
| 200 OK      | ValidationError  | 240      | Invoice  | Taxamount {Taxamount } of taxline: {A/A} cannot be greater than the corresponding underlying value                     |
| 200 OK      | ValidationError  | 241      | Invoice  | "Field1} cannot be greater than the corresponding invoiceline net value (invoice line: + {linenumber} ) [ Possible {Field1} values: {‘feesAmount’, ‘otherTaxesPercentAmount’, ‘stampDutyAmount, ‘withheldAmount’}] |
| 200 OK      | ValidationError  | 242      | Invoice  | {Field} 's country for this invoice type must be Greece [Possible {Field} values: {Issuer, Counterpart}                 |
| 200 OK      | ValidationError  | 243      | Invoice  | {Field} 's country for this invoice type must be in Europe but not Greece [Possible {Field} values: {Issuer, Counterpart} |
| 200 OK      | ValidationError  | 244      | Invoice  | {Field} 's country for this invoice type must not be in EU [Possible {Field} values: {Issuer, Counterpart}             |
| 200 OK      | ValidationError  | 245      | Invoice  | Provider is not authorised to issue Invoices for: {vatNumber}                                                         |
| 200 OK      | ValidationError  | 246      | Invoice  | Invoice of type 1.5 must have at least one line with detailtype = 1 and one with detail type=2                           |
| 200 OK      | ValidationError  | 247      | Invoice  | Invoice line: {lineNumber}. {Field} is forbidden. [Possible {Field} values: {recType=1, recType=4, recType=5}           |
| 200 OK      | ValidationError  | 248      | Invoice  | Invoice with MARK {mark} cannot be cancelled because was not posted by VAT number {vat}                                |
| 200 OK      | ValidationError  | 249      | Invoice  | Invoice with MARK {mark} cannot be cancelled because of being posted by provider                                        |
| 200 OK      | ValidationError  | 250      | Invoice  | Invoice with MARK {mark} cannot be cancelled because of being posted by myDATA Invoicing                                |
| 200 OK      | ValidationError  | 251      | Invoice  | Invoice with MARK {mark} cannot be cancelled because of being already cancelled                                         |
| 200 OK      | ValidationError  | 252      | Invoice  | Record with MARK {mark} is not a valid Invoice                                                                         |
| 200 OK      | ValidationError  | 254      | Invoice  | TaxLine (TaxTotals) : + {taxlinenumber} . {field + fieldData} is forbidden                                            |
| 200 OK      | ValidationError  | 255      | Invoice  | Invoice with MARK {mark} cannot be cancelled because it is connected with active invoice with MARK {mark1}              |
| 200 OK      | ValidationError  | 256      | Invoice  | Invoice with MARK {mark} cannot be classified because of being already cancelled                                        |
| 200 OK      | ValidationError  | 257      | Invoice  | Invoice with MARK {mark} cannot be cancelled because of being posted by TaxRegister                                     |
| 200 OK      | ValidationError  | 258      | Invoice  | All invoice rows should have {Field} [Possible {Field} values: {recType=3}                                            |
| 200 OK      | ValidationError  | 259      | Invoice  | Invoice cannot be posted because it replaces invoice with MARK {mark} having same UID and is still connected with active invoices |
| 200 OK      | ValidationError  | 260      | Invoice  | IssueDate is invalid, it must be less or equal than current date                                                       |
| 200 OK      | ValidationError  | 261      | Invoice  | Each Invoice row must have unique line number                                                                          |
| 200 OK      | ValidationError  | 262      | Invoice  | {field} length must be less or equal than {number}                                                                     |
| 200 OK      | ValidationError  | 263      | Invoice  | Invoice with invoiceVariationType {field} and IssueDate {issueDate} cannot be sent earlier than { date} [Possible {field} values: {invoiceVariationType=1,2,3,4} |
| 200 OK      | ValidationError  | 264      | Invoice  | Invoice with invoiceVariationType {field} cannot be sent for issueDate's year earlier than 2021 [Possible {field} values: {invoiceVariationType=1,2,3,4}]                                      |
| 200 OK      | ValidationError  | 265      | Invoice  | The maximum allowed number of invoices contained in one message is 5000                                                       |
| 200 OK      | ValidationError  | 266      | Invoice  | {msg11} is forbidden {msg2} [Possible {msg1} values: {‘Invoice with SpecialInvoiceCategoryType field with value 4 (tax-free invoice)’} [Possible {msg2} values: {‘for invoices sent by provider channel’}] |
| 200 OK      | ValidationError  | 267      | Invoice  | {msg11} is allowed {msg2} [Possible {msg1} values: {‘Invoice with SpecialInvoiceCategoryType field with value 4 (tax-free invoice)’} [Possible {msg2} values: {‘only for invoices sent by erp or timologio channel’}] |
| 200 OK      | ValidationError  | 268      | Invoice  | In case of fuelInvoice, at least one line must have fuelCode different from 999                                                 |
| 200 OK      | ValidationError  | 269      | Invoice  | In case of fuelInvoice, the net value of the invoice line with fuelCode 999 must be less or equal than the sum of net values of the other invoice lines                                   |
| 200 OK      | ValidationError  | 270      | Invoice  | In case of fuelInvoice, only one line can have fuelCode equal to 999                                                            |
| 200 OK      | ValidationError  | 271      | Invoice  | Invoice Line Number: {linenumber}. VatExemptionCategory field is used only in case of vatCategory = 7                         |
| 200 OK      | ValidationError  | 272      | Invoice  | {field} is mandatory + {moreInfo} [{moreInfo} μπορεί να είναι null]                                                           |
| 200 OK      | ValidationError  | 273      | Invoice  | {field} is not allowed + {moreInfo} [{moreInfo} μπορεί να είναι null]                                                          |
| 200 OK      | ValidationError  | 318      | Invoice  | Element {Field} must have the same value as correlated's one                                                                  |
| 200 OK      | ValidationError  | 319      | Invoice  | Net value of the correlated invoice has already exceeded by the sum of net values of invoices correlated to it                 |
| 200 OK      | ValidationError  | 320      | Invoice  | Invalid correlated invoice type                                                                                             |
| 200 OK      | ValidationError  | 322      | Invoice  | Unsupported correlated invoice type                                                                                         |


## Classification Errors
HTTP Status | Error Code       | Error No | Source | Description                                          |                  
-------------|------------------|----------|--------------|------------------------------------------------------------|  
| 200 OK      | ValidationError  | 301     | Classification  | Invoices with ΜΑΡΚ {mark} requested not found                                                                                 |
| 200 OK      | ValidationError  | 302     | Classification  | Duplicate classification line number {lineNumber}                                                                           |
| 200 OK      | ValidationError  | 303     | Classification  | Line number {lineNumber} not found in the invoice with MARK {mark}                                                            |
| 200 OK      | ValidationError  | 304     | Classification  | All invoice rows or none should have classifications included                                                                |
| 200 OK      | ValidationError  | 305     | Classification  | Invoice line: {lineNumber}. Duplicate classification type {classificationType} and category {classificationCategory}           |
| 200 OK      | ValidationError  | 306     | Classification  | Invoice line: {lineNumber}. Sum of classifications is not equal to the line's net value                                      |
| 200 OK      | ValidationError  | 307     | Classification  | Classification type {classificationType} is forbidden for Classification category {classificationCategory} combined with invoice type {invoiceType}                                   |
| 200 OK      | ValidationError  | 308     | Classification  | Classification category {classificationCategory} is forbidden for Invoice type {classificationType}                          |
| 200 OK      | ValidationError  | 309     | Classification  | {ClassificationMode} classifications are forbidden for the invoice with MARK {mark} on behalf of VAT number {vatNumber} for the invoice row with detail type {detailType}                    |
| 200 OK      | TechnicalError   | 310     | Classification  | All classifications of the invoice or none should have category value {classificationCategory}                               |
| 200 OK      | ValidationError  | 311     | Classification  | Classification with type {classificationType} and category " {classificationCategory} not found in the invoice summary          |
| 200 OK      | ValidationError  | 312     | Classification  | Sum of classifications with type {classificationType} and category {classificationCategory} not matching with the related total in the invoice summary                                    |
| 200 OK      | ValidationError  | 313     | Classification  | Classification type {classificationType} is forbidden for Classification category {classificationCategory} combined with invoice type {invoiceType}                                   |
| 200 OK      | ValidationError  | 314     | Classification  | All invoices should contain either income or expenses classifications section, not both or none                               |
| 200 OK      | ValidationError  | 315     | Classification  | VAT classifications have no category                                                                                        |
| 200 OK      | ValidationError  | 316     | Classification  | VAT classifications are not allowed in case of VAT exemption                                                                 |
| 200 OK      | ValidationError  | 317     | Classification  | Invoice detail {lineNumber}: VAT classification must be of type 366 in case vatExemptionCategory = 16                       |
| 200 OK      | ValidationError  | 321     | Classification  | Classifications are not allowed only in the invoice summary                                                                  |
| 200 OK      | ValidationError  | 323     | Classification  | User cannot use this service directly due to annual gross income limits                                                       |
| 200 OK      | ValidationError  | 324     | Classification  | Classification rejection or deviation is not allowed to be sent with this method                                             |
| 200 OK      | TechnicalError   | 330     | Classification  | Unexpected technical error for classification line                                                                          |
| 200 OK      | TechnicalError   | 331     | Classification  | Could not load/find valid validation doc for classification with category {classification_category} and type {classification_type}  |
| 200 OK      | TechnicalError   | 332     | Classification  | In case of VAT exemption, the sum of E3 classifications' amounts and the sum of VAT classifications' vat amounts must be equal to the invoice total net value plus taxes to be classified plus excluded vat amount |
| 200 OK      | TechnicalError   | 333     | Classification  | The sum of E3 classifications' amounts must be equal to the invoice total net value plus taxes to be classified                 |
| 200 OK      | TechnicalError   | 334     | Classification  | In case of VAT exemption, a combination of vatCategory {field1}, vatExemptionCategory {field2}: In case of without VAT obligation, a classification with similar VAT exemption category, VAT category of 24% or 17%, and classification type 366 is required [Possible {field1} values: {‘null’, [1-8]}] [Possible {field2} values: {‘null’, [εύρος τιμών των αιτιών εξαίρεσης από το σχετικό παράρτημα]}] |
| 200 OK      | TechnicalError   | 335     | Classification  | The SendExpensesClassificationPerInvoice method is not supported for fuel invoices or invoices of type 1.5                 |
| 200 OK      | TechnicalError   | 336     | Classification  | In case of VAT exemption, a percentage of vat amount must be referred to in E3 classification with category 2.5               |
| 200 OK      | TechnicalError   | 337     | Classification  | In case of VAT exemption, a combination of vatCategory {field1}, vatExemptionCategory {field2}: Fields vatAmount, vatCategory, and vatExemptionCategory must be null when classifications are posted per line [Possible {field1} values: {‘null’, [1-8]}] [Possible {field2} values: {‘null’, [εύρος τιμών των αιτιών εξαίρεσης ΦΠΑ από το σχετικό πίνακα του παραρτήματος]}] |
| 200 OK      | TechnicalError   | 338     | Classification  | Classification is forbidden for invoice row {lineNumber} because of its recType or its detailType Value                       |
| 200 OK      | TechnicalError   | 339     | Classification  | {msg} classification is forbidden for invoice row {lineNumber} because of its detailType value [Possible {msg} values: {‘incomeClassification’, ‘expensesClassification’}]               |
| 200 OK      | TechnicalError   | 340     | Classification  | ClassifiacationPostMode field must not be contained in xml                                                                 |




[myDATA API Documentation_Providers_v1.0.9_official.pdf]: https://www.aade.gr/sites/default/files/2024-10/ENG_myDATA%20API%20Providers_v1%200%209_official.pdf
[myDATA API Documentation_v1.0.9_official_ERP.pdf]: https://www.aade.gr/sites/default/files/2024-10/ENG_myDATA%20API%20Documentation%20v1.0.9_official_erp.pdf
[EdDSA and Ed25519]: https://cryptobook.nakov.com/digital-signatures/eddsa-and-ed25519
[Key-based authentication in OpenSSH for Windows]: https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_keymanagement
[How To Generate ed25519 SSH Key]: https://www.unixtutorial.org/how-to-generate-ed25519-ssh-key/
