# OneSys - ONESIGN API Documentation 1.5.9

## Version History

| Version | Date       | Description                                                                          |
|---------|------------|--------------------------------------------------------------------------------------|
| 1.0     | 2023-10-01 | Initial release                                                                      |
| 1.1     | 2023-11-15 | Added crypto headers                                                                 |
| 1.2     | 2023-12-16 | Added GR AADE integration                                                            |
| 1.3     | 2023-12-22 | Updates/Fixes, Postman, Sample Calls                                                 |
| 1.4     | 2024-01-10 | Updated Postman, XML Calls, added more details on S2S HMAC Authentication            |
| 1.4.1   | 2024-01-15 | Updated B2G sample with more fields, required by PEPPOL                              |
| 1.4.2   | 2024-01-16 | Updated HMAC Auth Headers                                                            |
| 1.5.0   | 2024-02-22 | Added Transmission Failure 2, Status Request, Updated POSTMAN                        |
| 1.5.1   | 2024-05-28 | Added self handling of Transmission Failure 2                                        |
| 1.5.2   | 2024-09-11 | Added commands to extract raw keys, and some API updates, updated postman collection |
| 1.5.3   | 2024-11-26 | Updated the AADE doc links, added sample B2B and B2C XML requests                    |
| 1.5.4   | 2024-11-27 | Added the supported measurement units                                                |  
| 1.5.5   | 2024-11-27 | Added soft rejection resubmit header                                                 |
| 1.5.6   | 2024-12-12 | Added business-id and location-id headers                                            |
| 1.5.7   | 2025-01-21 | Updated the postman collection with location-id and business-id headers              |
| 1.5.8   | 2025-01-22 | Corrected the "Fiscal Header SHA256 Hash" example                                    |
| 1.5.9   | 2025-03-21 | Corrected test environment URL                                                       |

## Supported Schemas

The ONESIGN Invoicing API supports the following schemas for invoice payloads:

1. **ONESIGN Schema**
    - Description: Schema for invoicing using ONESIGN envoicing solution
    - TBD

2. **GREECE AADE Schema**
    - Description: Schema for invoicing in Greece (AADE-mydata).
    - [Greek Version](readme.aad.en.md) - Language: English
    - [Ελληνική Έκδοση](readme.aad.gr.md) - Language: Ελληνικά

Feel free to refer to the appropriate schema documentation for detailed information on the structure and usage of each schema.
