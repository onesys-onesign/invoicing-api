# OXINUS REST API Ηλεκτρονικής Τιμολόγησης 1.4

## Αριθμός έκδοσης

| Έκδοση  | Ημερομηνία | Περιγραφή                                                       |
|---------|------------|-----------------------------------------------------------------|
| 1.0     | 2023-10-01 | Αρχική έκδοση                                                   |
| 1.1     | 2023-11-15 | Προσθήκη crypto headers                                         |
| 1.2     | 2023-12-15 | Προσθήκη Ελληνικής έκδοσης ΑΑΔΕ/B2G                             | 
| 1.3     | 2023-12-22 | Προσθήκη endpoints, δείγματα payloads, response, postman collection |
| 1.4     | 2024-01-10 | Ενημέρωση των sample payloads with lineComments and itemCPV elements, HMAC οδηγός |

## Εισαγωγή

Καλώς ήρθατε στο e-invoicing API, ένα REST API σχεδιασμένο για τη διαχείριση λειτουργιών που σχετίζονται με την 
ηλεκτρονική τιμολόγηση της OXINUS και την υποβολή μέσω παρόχου προς την ΑΑΔΕ και στη συνέχεια προς το ΚΕΔ (ΓΓΠΣ) εφόσον αυτό απαιτείται. 
Αυτό το έγγραφο παρέχει λεπτομέρειες σχετικά με τον τρόπο αλληλεπίδρασης με το API, συμπεριλαμβανομένων των HTTP 
κεφαλίδων, δομών αιτήσεων και απαντήσεων, καθώς και της κρυπτογραφικής πιστοποίησης.

## Δοκιμαστικό Περιβάλλον (Sandbox)

Το δοκιμαστικό περιβάλλον, για όλες τις κλήσεις του API είναι:

```
https://api.invoicing.oxinus.net/
```

## Αυθεντικοποίηση (Authentication)

Για να υποβληθεί οποιαδήποτε κλήση προς το API, υποστηρίζονται οι παρακάτω δύο (2) μέθοδοι:
1) θα πρέπει να χρησιμοποιηθεί api key και Hash-based Message Authentication Code (HMAC)
2) Χρήση Json Web Token (JWT).

Στην περίπτωση Service 2 Service κλήσης, στις κεφαλίδες της κλήσης (HTTP Request-Header) που αφορούν την αυθεντικοποίηση 
του καλούντος θα πρέπει να υπάρχει το προσυμφωνημένο api-key

```
x-api-key:
x-signature:
```

## Οδηγός αυθεντικοποίησης με χρήση HMAC

### Σύνοψη

Το τμήμα αυτό περιέχει οδηγίες σχετικά με την χρήση authentication HMAC για την επικοινωνία server-to-server. 
Περιλαμβάνει την χρήση ενός ζεύγους κλειδιών API KEY κα μυστικού κωδικού SECRET. 
Το API KEY χρησιμοποιείται στην κεφαλίδα του μηνύματος (request header), και ο κωδικός Secret, χρησμοπειίται για να παραχθεί μία Υπογραφή HMAC του περιεχομένου του μηνύματος (request body).

### Απαιτήσεις

### API Key and Secret: 
Δύο ξεχωριστά δεδομένα θα απαιτηθούν, το API KEY και το SECRET.

**API Key:** Αποστέλλεται με τις κεφαλίδες (request header).

**Secret:** Χρησιμοποιείται για την παραγωγή της υπογραφής HMAC του payload.

**Κρυπτογράφηαση μηνύματος:**

Κρυπτογραφείτε το body του request χρησιμοποιώντας το secret.
Για περιεχόμενο XML, κρυπτογραφείτε το περιεχόμενο απαυθείας.
Για περιεχόμενο τύπου JSON, θα πρέπει να προηγηθεί πρώτα stringify του JSON πριν την κρυπτογράφηση.

**Κεφαλίδες μηνύματος (Request Headers):**

`x-api-key:` Το δοθέν API-KEY.

`x-signature:` Η υπογραφή HMAC του request body χρησιμοποιώντας το secret.

> **Σημείωση:**
> Σε περίπτωση χρήσης x-api-key, x-signature μεθόδου δεν θα πρέπει να στέλνεται μαζί το Authorization Bearer
>

Παράδειγμα, 

Δείγμα φευδοκώδικα σε Python:

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
    "x-api-key": api_key,
    "x-signature": signature
}

# send request with headers

```

## 1. Υποβολή Τιμολογίου
Για την υποβολή ενός τιμολογίου, απαιτείται να υποβληθεί μαζί με την κλήση και το κρυπτογραφικό header το οποίο 
περιλαμβάνει την υπογραφή της συσκευής, το hashing των βασικών στοιχείων του τιμολογίου, τα στοιχεία καθώς και το 
payload το οποίο θα πρέπει να είναι συμβατό με τις προδιαγραφές της ΑΑΔΕ για την υποβολή παραστατικού.

### Εκδοση Κλειδιών και Πιστοποίηση συσκευής στην εφαρμογή.
Για την υπογραφή του εκάστοτε παραστατικού, απαιτείται η πιστοποίηση της συσκευής στην εφαρμογή.
Ως βασική μέθοδος κρυπτογράφησης, έχει επιλεγεί ο αλγόριθμος EdDSA και συγκεκριμένα ο Ed25519.
Περισσότερες πληροφορίες για τη χρήση του αλγορίθμου μπορείτε να βρείτε στον ακόλουθο σύνδεσμο [EdDSA and Ed25519]

Για την πιστοποίηση μίας συσκευής στην εφαρμογή θα πρέπει να ακολουθηθούν τα παρακάτω βήματα:
1. Να εκδοθούν κλειδιά από την συσκευή
2. Να υπογραφούν τα κλειδιά από την εφαρμογή.

#### Έκδοση κλειδιών
##### Linux/Unix/MacOS
Από ένα σύστημα Linux/Unix/MacOS μπορούν να εκδοθούν κλειδιά με τις παρακάτω εντολές, με τη χρήση του OpenSSL μέσω 
του terminal/console.

```shell
$ openssl genpkey -algorithm Ed25519 -out private_key.pem
$ openssl pkey -in private_key.pem -pubout -out public_key.pem
```

Για περισσότερες πληροφορίες μπορείτε να ανατρέξτε στον ακόλουθο σύνδεσμο [How To Generate ed25519 SSH Key]

##### Windows
Από ένα σύστημα Windows (Windows 10+, Windows Server 2016+) μπορούν να εκδοθούν κλειδιά με την παρακάτω εντολή, με τη 
χρήση του OpenSSL μέσω του Windows Powershell.

```shell
ssh-keygen -t ed25519
```

Για περισσότερες πληροφορίες ανατρέξτε στον ακόλουθο σύνδεσμο: [Key-based authentication in OpenSSH for Windows]

#### Υποβολή και Υπογραφή Κλειδιών
Μετά την παραγωγή των κλειδιών, θα πρέπει να γίνει πιστοποίηση της συσκευής στην εφαρμογή.
Για να γίνει η πιστοποίηση της συσκευής στην εφαρμογή, θα πρέπει να γίνει υπογραφή του δημόσιου κλειδιού που παράχθηκε 
από την προηγούμενη διαδικασία και υποβολή και των δύο (κλειδιού και υπογραφής του κλειδιού) με την κλήση της μεθόδου 
signing-devices στην εφαρμογή, από όπου θα ληφθεί η υπογραφή-signature της εφαρμογής την οποία θα πρέπει να 
χρησιμοποιεί η συσκευή σε κάθε κλήση.

Εναλλακτικά, μπορεί να γίνει μέσω curl χρησιμοποιώντας την παρακάτω εντολή, αντικαθιστώντας τα απαραίτητα πεδία.

```shell
curl --location 'https://api.invoicing.oxinus.net/signing-devices' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer eyJhbGciOiJUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6Ijc2YjM1ZDdlLWVkODQtNGI0ZS1hODhiLWQ2Y2RlOWRiY2Q3YiIsImZpcnN0TmFtZSI6Ik94aW51cyIsImxhc3ROYW1lIjoiSG9sZGluZ3MiLCJlbWFpbCI6InNwQG94aW51cy5ob2xkaW5ncyIsImFjY2Vzc1R5cGUiOiJqd3QiLCJpYXQiOjE3MDI4Nzg3MTB9.ZLZGFse5RJMFkgziYs-nH8qYTveztOzmhApbXN0poPA' \
--data '{
    "deviceName": "device 1",
    "deviceType": "virtual",
    "deviceSerial": "98594583434",
    "devicePublicKey": "0e6c1b121a9ce7f593fb6f1a3794090885a93de2812fafaf6b6f5c0867477f4",
    "devicePublicKeySignature": "b926537302ebcacb367a2e4bf608a2c7751322fd835915a640b95ffa37208775d6b5d56d6c1f01b2df7f20b4fd31749cce8f6c950b10f202d09cc331664ef07",
    "networkTxnId": "123243323",
    "country": "GR",
    "authority": "AAD",
    "entity": 1
}'
```

#### Λήψη ενός παραστατικού
Παραστατικό το οποίο έχει υποβληθεί επιτυχώς, μπορεί να γίνει λήφη, χρησιμοποιοώντας τον κωδικού UID ο οποίος περιλαμβάνεται στην απάντηση (response) κατά την υποβολή του.

```shell
curl --location 'https://api.invoicing.oxinus.net/invoice/DC9555A9F4786C0605698D7DA5EDED4EBCB87A73' \
--header 'Authorization: Bearer eyJhbGciOJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6ImtqZG5nZTg0NTQwOTgiLCJmaXJzdE5hbWUiOiJSaWNreSIsImxhc3ROYW1lIjoiTWFydGluIiwiZW1haWwiOiJtLnNpZGRpcXVpQG94aW51cy5pbyIsImFwaUtleSI6ImhoaGpqamRkZGtrayIsInNlY3JldCI6InNlY3JldCIsImFjY2Vzc1R5cGUiOiJTMlMiLCJjcmVhdGVkQXQiOiIyMDIzLTEyLTE0VDA3OjA3OjE0LjgzMloiLCJ1cGRhdGVkQXQiOiIyMDIzLTEyLTE0VDA3OjA3OjE0LjgzMloiLCJpYXQiOjE3MDI1NDE3OTZ9.a3XfDBcXVJ5mZFkKR7u5Er_zT9L06SaIUzi9biYD6gU'
```

### Κεφαλίδες Αίτησης:
Παρακάτω ακολουθεί η λίστα των headers του κάθε αιτήματος και στη συνέχεια αναλύονται διεξοδικά.

| Όνομα                                            | Περιγραφή (PEPPOL Identifier)                                   | Παράδειγμα Τιμής                                                  |
|--------------------------------------------------|-----------------------------------------------------------------|--------------------------------------------------------------------|
| Settings-Authority-ID                            | Αναγνωριστικό της αρχής.                                        | "AAD"                                                     |
| Settings-Is-Peppol-Required                      | Λογική τιμή που υποδηλώνει εάν απαιτείται το Peppol.            | true                                               |
| Settings-Document-TemplateID                     | Αναγνωριστικό του προτύπου έγγραφου.                            | 1                                             |
| Fiscal-Header-Invoice-ID                         | Αναγνωριστικό τιμολογίου.                                       | "fd4bdc10-b0f3-4f3b-beb1-14fc4f42dc2b"                                                    |
| Fiscal-Header-Issuer-TaxID                       | Κωδικός φορολογικού ταυτότητας του εκδότη (AccountingSupplierParty.PartyTaxScheme.CompanyID) | "987654321"                                             |
| Fiscal-Header-Recipient-TaxID                    | Κωδικός φορολογικού ταυτότητας του παραλήπτη (AccountingCustomerParty.PartyTaxScheme.CompanyID) | "123456789"                                       |
| Fiscal-Header-TimeStamp-Epoch                    | Χρονική σήμανση Epoch (Invoice.IssueDate)                       | "1702140669"                                                 |
| Fiscal-Header-Document-Type                      | Τύπος εγγράφου (Invoice.InvoiceTypeCode)                        | "380"                                                   |
| Fiscal-Header-Document-Value                     | Αξία εγγράφου (Invoice.LegalMonetaryTotal.TaxInclusiveAmount)   | 1240.00                                                  |
| Fiscal-Header-Document-Tax-Value                 | Φορολογική αξία εγγράφου (Invoice.TaxTotal.TaxAmount)           | 240.00                                              |
| Fiscal-Header-Currency                           | Νόμισμα (Invoice.DocumentCurrencyCode)                          | "EUR"                                                        |
| Fiscal-Header-Tax-Currency                       | Φορολογικό Νόμισμα (Invoice.TaxCurrencyCode)                    | "EUR"                                                    |
| Crypto-Header-Fiscal-Header-SHA256-Hash          | SHA256 hash της φορολογικής επικεφαλίδας                        | "ba2a7576612c69a3dd18da20b3c47c598afabe1c1aec9fa98a1f888daecf5a5d"                                       |
| Crypto-Header-Invoice-Payload-SHA256-Hash        | SHA256 Hash του περιεχομένου του τιμολογίου                     | "31481f96c1677e8f0e1cd81d941561824f19b344476612c69a3dd18db1a47277"                                     |
| Crypto-Header-Signature                          | Υπογραφή του (Fiscal Header και Invoice Payload Hash) με το Ιδιωτικό Κλειδί της Συσκευής | "0D64FD1B9B95790E473489D6964B4D1A76811BC3A63407118B3CCCB3BC6F789A384F2EF80BD8A70347ACD89E0C342F1DA300C85A5830254011543A3B64EB9206"                                          |
| Crypto-Header-Previous-Invoice-Fiscal-Header     | SHA256 hash της προηγούμενης φορολογικής επικεφαλίδας           | "aa3a677e8f0e19a3dd1876612c69a3dd18d98afa76612c69a3dd18df888daea5a"                 |
| Crypto-Header-Public-Key-of-Signatory-Device     | Δημόσιο Κλειδί της Συσκευής Υπογραφής (Base64)                  | "MCowBQYDK3VwAyEAvR97AJTKyGNAjOYROXGk+H367Ix1kOAMNKQwpTuvOfU="                                |
| Crypto-Header-Signature-of-Signatory-Device-Public-Key | Υπογραφή του Δημοσίου Κλειδιού της Συσκευής Υπογραφής     | "E473489D6964B4D1A76811BC3A634070D64FD15830254011B9B95790118B3CCCB3BC6F789A384F2EFF1DA300C85A543A3B64EB920680BD8A70347ACD89E0C342"                        |

Όπως φαίνεται στον παραπάνω πίνακα, οι κεφαλίδες που απαιτούνται για την ασφαλή και επιτυχημένη υποβολή ενός 
παραστατικού αποτελούνται από τρία ξεχωριστά μέρη:

- **Settings**: τμήμα το οποίο περιλαμβάνει τις ρυθμίσεις για το περιεχόμενο του API Payload.
- **Fiscal Header**: Οικονομικό μέρος του api payload.
- **Crypto Header**: Κρυπτογραφικό μέρος και υπογραφή του api payload.

#### SETTINGS - Ρυθμίσεις όδευσης παραστατικού

Το τμήμα settings, περιέχει τις παρακάτω τρεις (3) ρυθμίσεις που αφορούν το είδος του περιεχομένου που ανταλλάσσεται.

##### Authority ID
- Περιγραφή: Χαρακτηριστικό της αρχής στην οποία απευθύνεται το παραστατικό (πχ. ΑΑΔΕ)
- Ενδεικτική τιμή: "AAD"
- Προεπιλεγμένη τιμή: 
- Απαραίτητη τιμή: ΝΑΙ

##### Is Peppol Required
- Περιγραφή: Τιμή true/false η οποία υποδεικνύει αν το παραστατικό αφορά μόνο ΑΑΔΕ, ή και PEPPOL (B2G)
- Ενδεικτική Τιμή: false
- Προεπιλεγμένη τιμή: false
- Απαραίτητη τιμή: ΟΧΙ

##### Document TemplateID
- Περιγραφή: Προεπιλεγμένο πρότυπο (template) εμφάνισης/εκτύπωσης του παραστατικού. 
- Ενδεικτική Τιμή: 1
- Προεπιλεγμένη τιμή: 
- Απαραίτητη τιμή: ΟΧΙ

#### Fiscal Header - Οικονομικό τμήμα

To οικονομικό τμήμα του Header περιλαμβάνει τις παρακάτω τιμές οι οποίες είναι όλες απαραίτητες για την υποβολή
παραστατικού.

##### Invoice ID - Αναγνωριστικό Τιμολογίου
- Περιγραφή: Μοναδικό αναγνωριστικό του τιμολογίου σε μορφή UUID - (Προτεινόμενη έκδοση 4)
- Παράδειγμα Τιμής: "fd4bdc10-b0f3-4f3b-beb1-14fc4f42dc2b"

##### Issuer TaxID - Κωδικός ΑΦΜ Εκδότη
- Περιγραφή: Κωδικός Φορολογικής Ταυτότητας Εκδότη (AccountingSupplierParty.PartyTaxScheme.CompanyID)
- Παράδειγμα Τιμής: "987654321"

##### Recipient TaxID - Κωδικός ΑΦΜ Παραλήπτη
- Περιγραφή: Κωδικός Φορολογικής Ταυτότητας Παραλήπτη (AccountingCustomerParty.PartyTaxScheme.CompanyID)
- Παράδειγμα Τιμής: "123456789"

##### TimeStamp - Χρονική Σήμανση Epoch
- Περιγραφή: Χρονική Σήμανση Epoch (Invoice.IssueDate)
- Παράδειγμα Τιμής: "1702140669"

##### Document Type - Τύπος Εγγράφου
- Περιγραφή: Τύπος Εγγράφου, κατά PEPPOL (Invoice.InvoiceTypeCode)
- Παράδειγμα Τιμής: "380"

##### Document Value - Αξία Εγγράφου
- Περιγραφή: Αξία Εγγράφου (Invoice.LegalMonetaryTotal.TaxInclusiveAmount)
- Παράδειγμα Τιμής: 1240.00

##### Document Tax Value - Φορολογική Αξία Εγγράφου
- Περιγραφή: Φορολογική Αξία Εγγράφου (Invoice.TaxTotal.TaxAmount)
- Παράδειγμα Τιμής: 240.00

##### Currency - Νόμισμα
- Περιγραφή: Νόμισμα (Invoice.DocumentCurrencyCode)
- Παράδειγμα Τιμής: "EUR"

##### Tax Currency - Φορολογικό Νόμισμα
- Περιγραφή: Φορολογικό Νόμισμα (Invoice.TaxCurrencyCode)
- Παράδειγμα Τιμής: "EUR"

#### Crypto Header

Το κρυπτογραφικό τμήμα των headers, περιλαμβάνει τα απαραίτητα δεδομένα ασφαλείας και κρυπτογράφησης του παραστατικού 
ώστε να ελεγχθεί και επιβεβαιωθεί η εγκυρότητά του.

##### Fiscal Header SHA256 Hash
- Περιγραφή: Hash SHA256 όλων των κεφαλίδων του Οικονομικού τμήματος των headers (fiscal-header fields) ενωμένα σε ένα 
πεδίο διαχωρισμένο με τον χαρακτήρα "|" pipe.
π.χ: "fd4bdc10-b0f3-4f3b-beb1-14fc4f42dc2b|987654321|123456789|1702140669|380|1000.00|240.00|EUR|EUR"

Η σειρά εμφάνισης είναι η παρακάτω:
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

- Ενδεικτική τιμή παραδείγματος: "ba2a7576612c69a3dd18da20b3c47c598afabe1c1aec9fa98a1f888daecf5a5d"

##### Invoice Payload SHA256 Hash
- Περιγραφή: Hash SHA256 ολόκληρου του Payload της κλήσης.
- Ενδεικτική τιμή: "31481f96c1677e8f0e1cd81d941561824f19b34448026a2639e56a92b1a47277"

##### Signature
- Περιγραφή: Υπογραφή με το δημόσιο κλειδί της συσκευής (το οποίο έχει πιστοποιηθεί σε προγενέστερο βήμα) της ένωσης των
 δύο (2) προηγούμενων hashes (π.χ: sign((FiscalHeaderHash+InvoicePayloadHash), myPrivateKey)
- Ενδεικτική τιμή: "0D64FD1B9B95790E473489D6964B4D1A76811BC3A63407118B3CCCB3BC6F789A384F2EF80BD8A70347ACD89E0C342F1DA300C85A5830254011543A3B64EB9206"

##### Previous Invoice Fiscal Header SHA256 Hash
- Περιγραφή: Hash SHA256 όλων των κεφαλίδων του Οικονομικού τμήματος του *προηγούμενου παραστατικού* που εξέδωσε η συσκευή
- Ενδεικτική τιμή: "aa3a677e8f0e19a3dd1876612c69a3dd18d98afa76612c69a3dd18df888daea5a"

##### Public Key of Signatory Device
- Περιγραφή: Το δημόσιο κλειδί της συσκευής (Base 64)
- Ενδεικτική τιμή: "MCowBQYDK3VwAyEAvR97AJTKyGNAjOYROXGk+H367Ix1kOAMNKQwpTuvOfU="

##### Signature of Signatory Device Public Key
- Περιγραφή: Υπογραφή του δημόσιου κλειδιού της συσκευής (παρομοίως με την εγγραφή της συσκευής).
- Ενδεικτική τιμή: "E473489D6964B4D1A76811BC3A634070D64FD15830254011B9B95790118B3CCCB3BC6F789A384F2EFF1DA300C85A543A3B64EB920680BD8A70347ACD89E0C342"


#### Σώμα Αίτησης (Payload)

To σώμα του αιτήματος (request body) θα πρέπει να περιέχει τον μορφότυπο XML της ΑΑΔΕ, όπως αυτή έχει διαμορφωθεί με την 
τελευταία έκδοση των προδιαγραφών.

***Τρέχουσες προδιαγραφές 1.0.7***

Οι σχετικές προδιαγραφές βρίσκονται στον ακόλουθο σύνδεσμο (διασύνδεση μέσω Παρόχου).

[myDATA API Documentation_Providers_v1.0.7_official.pdf]

Οι σχετικές αναφορές που αναγράφονται στις προδιαγραφές του Παρόχου, παραπέμπουν στις προδιαγραφές ERP οι οποίες 
βρίσκονται στον ακόλουθο σύνδεσμο (διασύνδεση ERP).

[myDATA API Documentation_v1.0.7_official_ERP.pdf]

#### Τμήμα κλήσης για PEPPOL (B2G)
Σε περίπτωση που το τιμολόγιο αφορά παραστατικό B2G το οποίο απαιτείται να δρομολογηθεί μέσω δικτύου PEPPOL, θα πρέπει 
να συμπληρώνεται η ένδειξη Settings-Is-Peppol-Required, στον header της κλήσης, με την τιμή `true`.

Στην περίπτωση αυτή, το παραστατικό θα ελεγχθεί για την ύπαρξη των παρακάτω πεδίων σε επίπεδο τιμολογίου, τα οποία θα 
πρέπει να περιέχονται στο τελευταίο element <b2g-peppol> στο element invoice.

| Πεδίο              | Περιγραφή                              | Ενδεικτική Τιμή      | Υποχρεωτικό |PEPPOL BT |
|--------------------|----------------------------------------|----------------------|----------|-------------|
| projectReference   | Αριθμός αναφοράς για την προμήθεια B2G | {ProjectReference}   | ΝΑΙ      | BT-11 PROJECT_REFERENCE_ID |
| contractReference  | Αριθμός Σύμβασης (ΑΔΑΜ)                | {ContractReference}  | ΝΑΙ      | BT-12 CONTRACT_REFERENCE_ID |
| assignReference    | Αριθμός Αναθέτουσας αρχής              | 1000.E00961.0001     | ΝΑΙ      | BT-46 PARTY_ID |
| deliveryParty      | Περιοχή Παράδοσης Προϊόντων (αν αφορά προϊόντα) | ΓΕΝΙΚΗ ΔΙΕΥΘΥΝΣΗ | ΟΧΙ      | BT-70 PARTY_ID |

#### Project Reference
Το πεδίο projectReference (BT-11) συμπληρώνεται ως εξής σύμφωνα με τις οδηγίες της ΓΓΠΣ.
• «1| ΑΔΑ Ανάληψης» : (Αριθμός Διαδικτυακής Ανάρτησης) όταν η αναθέτουσα αρχή είναι φορέας της
Κεντρικής Διοίκησης σύμφωνα με το άρθρο 14 του νόμου 4270/2014 (Α.143) και οι δαπάνες βαρύνουν τον
τακτικό προϋπολογισμό. Δηλαδή καταχωρούνται: η ένδειξη «1», το διαχωριστικό «|» και ο Αριθμός
Διαδικτυακής Ανάρτησης (ΑΔΑ) της σχετικής απόφασης δέσμευσης πίστωσης.
• «2| ο κωδικοποιημένος Ενάριθμος» όταν οι δαπάνες βαρύνουν τον Προϋπολογισμό Δημοσίων
Επενδύσεων (ΠΔΕ), δηλαδή καταχωρούνται: η ένδειξη «2», το διαχωριστικό «|» και ο Ενάριθμος του
έργου ‘όπως αναφέρεται στην σχετική ΣΑ και μνημονεύεται υποχρεωτικά στο κείμενο της σύμβασης.
• «3|ΑΔΑ Ανάληψης» όταν οι δαπάνες δεν βαρύνουν τους ανωτέρω προϋπολογισμούς, δηλαδή
καταχωρούνται : η ένδειξη «3», το διαχωριστικό «|» και ο Αριθμός Διαδικτυακής Ανάρτησης της σχετικής
απόφασης δέσμευσης πίστωσης

#### Contract Reference
Στο πεδίο contractReference (BT-12) συμπληρώνεται ο Αριθμός Διαδικτυακής Ανάρτησης Μητρώου (ΑΔΑΜ) Σύμβασης.
Εάν δεν υπάρχει γραπτή σύμβαση (προμήθειες κάτω των 2.500€) μπαίνει ο ΑΔΑΜ της Απόφασης Ανάθεσης από το ΚΗΜΔΗΣ.
Καταχωρείται ή τιμή 0, εάν δεν υπάρχει Απόφαση Ανάθεσης ή είμαστε < 1000€.
Αν η Σύμβαση είναι κάτω των 30000€ θα καταχωρείται ο ΑΔΑΜ της Εντολής Αγοράς.

#### Assign Reference
Στο πεδίο assignReference (B5-46) συμπληρώνεται ο Αριθμός Αναθέτουσας αρχής, σύμφωνα με το μητρώο


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


Επίσης, εφόσον το παραστατικό αφορά B2G, θα πρέπει σε κάθε γραμμή, να περιλαμβάνεται ο κωδικός CPV του προϊόντος όπως 
αυτός είναι κωδικοποιημένος κατά Common Product Vocabulary σύμφωνα με τις οδηγίες EE.
Το παραστατικό θα ελεγχθεί για την ύπαρξη τιμής στο πεδίο itemCPV το οποίο θα πρέπει να είναι το τελευταίο πεδίο στο
element invoiceDetails.


| Πεδίο              | Περιγραφή                              | Ενδεικτική Τιμή      | Υποχρεωτικό | PEPPOL BT |
|--------------------|----------------------------------------|----------------------|-------------|-----------|
| itemCPV            | Κωδικός προϊόντος κατά Common Product Vocabulary (CPV) | 73000000-2 | ΝΑΙ   | BT-158 CLASSIFICATION_ID (CPV)


Παρακάτω ακολουθεί δείγμα του εν λόγω τμήματος ενσωματωμένο στον XML μορφότυπο της ΑΑΔΕ.

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

### Απάντηση Κλήσης

Σε περίπτωση επιτυχημένης υποβολής παραστατικού θα ληφθεί απάντηση που περιέχει τα παρακάτω πεδία:
- ΜΑΡΚ (Μοναδικός Αριθμός Καταχώρησης)
- UID (Υπογραφή HASH του παραστατικού από την ΑΑΔΕ)
- AUTH (Υπογραφή HASH του παραστατικού από τον πάροχο)
- QrCode (public url όπου μπορεί κάποιος να επιβεβαιώσει το παραστατικό)


### Μηνύματα Λάθους (B2G)

Εφόσον το παραστατικό αφορά B2G υπηρεσία, τότε γίνονται επιπρόσθετοι έλεγχοι και επιστρέφονται τα παρακάτω μηνύματα σε
περίπτωση λάθους

| HTTP Status | Error Code       | Error No | Πηγή  | Περιγραφή |                                                                                       
|-------------|------------------|----------|--------------|-------------------|
| 200 OK      | ValidationError  | 751      | Παραστατικό-B2G  | Παραστατικό B2G χωρίς κωδικό ΑΔΑΜ (Project Reference) |                                                
| 200 OK      | ValidationError  | 752      | Παραστατικό-B2G  | Παραστατικό B2G χωρίς κωδικό Σύμβασης (Contract Reference) |
| 200 OK      | ValidationError  | 753      | Παραστατικό-B2G  | Παραστατικό B2G χωρίς κωδικό Αναθέτουσας Αρχής (AssignReference) |
| 200 OK      | ValidationError  | 754      | Παραστατικό-B2G  | Παραστατικό B2G χωρίς κωδικό CPV στην γραμμή {lineNumber} (itemCPV) |                                        
| 200 OK      | ValidationError  | 755      | Παραστατικό-B2G  | Παραστατικό B2G χωρίς περιγραφή παράδοσης (DeliverParty) |                         
| 200 OK      | ValidationError  | 761      | Παραστατικό-B2G  | Παραστατικό B2G με λάθος κωδικό ΑΔΑΜ (Project Reference)  |     
| 200 OK      | ValidationError  | 752      | Παραστατικό-B2G  | Παραστατικό B2G με λάθος κωδικό Σύμβασης (Contract Reference) |
| 200 OK      | ValidationError  | 753      | Παραστατικό-B2G  | Παραστατικό B2G με λάθος κωδικό Αναθέτουσας Αρχής (AssignReference) |
| 200 OK      | ValidationError  | 754      | Παραστατικό-B2G  | Παραστατικό B2G με λάθος κωδικό CPV στην γραμμή {lineNumber} (itemCPV) |                                        


### Μηνύματα Λάθους (ΑΑΔΕ)

Σε περίπτωση αποτυχίας της υποβολής, επιστρέφονται τα παρακάτω λάθη από την εφαρμογή
### Λάθη Εφαρμογής

| HTTP Status | Error Code       | Error No | Πηγή  | Περιγραφή |                                                                                       
|-------------|------------------|----------|--------------|-------------------|
| 200 OK      | XMLSyntaxError   | 101      | Εφαρμογή  | XML Syntax Validation Error |                                                
| 200 OK      | ValidationError  | 102      | Εφαρμογή  | Vat number {vatNumber} does not belong to active corporation |                              
| 200 OK      | ValidationError  | 103      | Εφαρμογή  | Please pass mark in the request parameters |                                           
| 200 OK      | ValidationError  | 104      | Εφαρμογή  | Requested Invoice was not found |                         
| 200 OK      | ValidationError  | 105      | Εφαρμογή  | User VAT number {vatNumber} is not authorized to execute this method for VAT number {vatNumber} |           
    
### Λάθη Παραστατικού
| HTTP Status | Error Code       | Error No | Πηγή | Περιγραφή                                          |
|-------------|------------------|----------|--------------|------------------------------------------------------------|
| 200 OK      | ValidationError  | 201      | Παραστατικό  | User VAT number {vatNumber} is not authorized to execute this method |                     
| 200 OK      | ValidationError  | 202      | Παραστατικό  | Invalid Greek VAT number                                  |
| 200 OK      | ValidationError  | 203      | Παραστατικό  | Gross Value doesn't match with sum of net value plus taxes |
| 200 OK      | ValidationError  | 204      | Παραστατικό  | {Field} is mandatory for this invoice type                 |
| 200 OK      | ValidationError  | 205      | Παραστατικό  | {Field} is forbidden for this invoice type                 |
| 200 OK      | TechnicalError   | 206      | Παραστατικό  | Unexpected technical error for invoice line               |
| 200 OK      | ValidationError  | 207      | Παραστατικό  | The sum of net values of the invoice lines doesn't match with total net value of the invoice |
| 200 OK      | ValidationError  | 208      | Παραστατικό  | The sum of gross values of the invoice lines doesn't match with total gross value of the invoice |
| 200 OK      | ValidationError  | 209      | Παραστατικό  | The sum of vat amount of the invoice lines doesn't match with total vat amount of the invoice |
| 200 OK      | ValidationError  | 210      | Παραστατικό  | The sum of withheld amount of the invoice lines doesn't match with total withheld amount of the invoice |
| 200 OK      | ValidationError  | 211      | Παραστατικό  | Exchange Rate must be greater than 0 when the currency is not Euro                                                 |
| 200 OK      | ValidationError  | 212      | Παραστατικό  | AA element must be number with value between 0 and 18446744073709551615 for issuer from Greece                    |
| 200 OK      | ValidationError  | 213      | Παραστατικό  | {Field} must have value 0 for this invoice type                                                                    |
| 200 OK      | ValidationError  | 214      | Παραστατικό  | Element {Element} must be sent only if it is true                                                                 |
| 200 OK      | ValidationError  | 215      | Παραστατικό  | Vat category must have value {value} for this invoice type [Possible { value } values: {7, 8}                    |
| 200 OK      | ValidationError  | 216      | Παραστατικό  | Vat category must have value other than 8 for this invoice type                                                    |
| 200 OK      | ValidationError  | 217      | Παραστατικό  | When vatCategory has value 7, element vatExemptionCategory is mandatory                                            |
| 200 OK      | ValidationError  | 218      | Παραστατικό  | Vat Amount must have value 0 for this invoice type                                                                 |
| 200 OK      | ValidationError  | 219      | Παραστατικό  | Issuer Name is forbidden for Issuer from Greece                                                                    |
| 200 OK      | ValidationError  | 220      | Παραστατικό  | Counterpart Name is forbidden for Counterpart from Greece                                                          |
| 200 OK      | ValidationError  | 221      | Παραστατικό  | {Field} is forbidden for the lines that have invoiceDetailType = 2 for this invoice type                            |
| 200 OK      | ValidationError  | 222      | Παραστατικό  | {Field} must have value greater than 0 for this invoice type                                                        |
| 200 OK      | ValidationError  | 223      | Παραστατικό  | Unsupported invoice type                                                                                            |
| 200 OK      | ValidationError  | 224      | Παραστατικό  | Taxes are allowed either per invoice line or per invoice (not in both)                                             |
| 200 OK      | ValidationError  | 225      | Παραστατικό  | {Field} must exist (cannot be null) since the {Field} is not null (invoice line {lineNumber})                      |
| 200 OK      | ValidationError  | 226      | Παραστατικό  | The sum of {field} amount of the invoice {section} doesn't match with total {field} amount of the invoice          |
| 200 OK      | ValidationError  | 227      | Παραστατικό  | {Field1} cannot exist (must be null) since the {Field1} is null (invoice line : {lineNumber}) [ Possible {Field1, Field2} values: {‘feesAmount’, ‘feesPercentCategory’}, {‘stampDutyAmount, ‘stampDutyPercentCategory’}, {‘withheldAmount’, ‘withheldPercentCategory’}] |
| 200 OK      | ValidationError  | 228      | Παραστατικό  | {Field} is invalid [Possible {Field} values: {UID, InvoiceType}                               |
| 200 OK      | ValidationError  | 229      | Παραστατικό  | {Field1} is not correct according to the given: {Field2} (invoice line: {lineNumber}) [ Possible {Field1, Field2} values: {‘feesAmount’, ‘feesPercentCategory’}, {‘stampDutyAmount, ‘stampDutyPercentCategory’}, {‘withheldAmount’, ‘withheldPercentCategory’}] |
| 200 OK      | ValidationError  | 230      | Παραστατικό  | {Field} is mandatory for invoice detail (number} [Possible {Field} values: {E3 classifications, VAT classifications}        |
| 200 OK      | ValidationError  | 231      | Παραστατικό  | {Field} is forbidden for invoice detail (number} [Possible {Field} values: {E3 classifications, VAT classifications}       |
| 200 OK      | ValidationError  | 233      | Παραστατικό  | UID: " + {uid} + " has already been sent                                                   |
| 200 OK      | ValidationError  | 234      | Παραστατικό  | The values 7 or 8 are not allowed for Vat Category for this invoice type                                                  |
| 200 OK      | ValidationError  | 235      | Παραστατικό  | Issuer must be different from counterpart                                                                              |
| 200 OK      | ValidationError  | 236      | Παραστατικό  | The Sender (vatnumber): " + {afm} + " must be different from the issuer (vatnumber)                                    |
| 200 OK      | ValidationError  | 237      | Παραστατικό  | Underlying Value(s) of taxes cannot be greater than the total net value of invoice                                     |
| 200 OK      | ValidationError  | 239      | Παραστατικό  | Taxamount(s) of taxes cannot be greater than the total net value of invoice                                            |
| 200 OK      | ValidationError  | 240      | Παραστατικό  | Taxamount {Taxamount } of taxline: {A/A} cannot be greater than the corresponding underlying value                     |
| 200 OK      | ValidationError  | 241      | Παραστατικό  | "Field1} cannot be greater than the corresponding invoiceline net value (invoice line: + {linenumber} ) [ Possible {Field1} values: {‘feesAmount’, ‘otherTaxesPercentAmount’, ‘stampDutyAmount, ‘withheldAmount’}] |
| 200 OK      | ValidationError  | 242      | Παραστατικό  | {Field} 's country for this invoice type must be Greece [Possible {Field} values: {Issuer, Counterpart}                 |
| 200 OK      | ValidationError  | 243      | Παραστατικό  | {Field} 's country for this invoice type must be in Europe but not Greece [Possible {Field} values: {Issuer, Counterpart} |
| 200 OK      | ValidationError  | 244      | Παραστατικό  | {Field} 's country for this invoice type must not be in EU [Possible {Field} values: {Issuer, Counterpart}             |
| 200 OK      | ValidationError  | 245      | Παραστατικό  | Provider is not authorised to issue Invoices for: {vatNumber}                                                         |
| 200 OK      | ValidationError  | 246      | Παραστατικό  | Invoice of type 1.5 must have at least one line with detailtype = 1 and one with detail type=2                           |
| 200 OK      | ValidationError  | 247      | Παραστατικό  | Invoice line: {lineNumber}. {Field} is forbidden. [Possible {Field} values: {recType=1, recType=4, recType=5}           |
| 200 OK      | ValidationError  | 248      | Παραστατικό  | Invoice with MARK {mark} cannot be cancelled because was not posted by VAT number {vat}                                |
| 200 OK      | ValidationError  | 249      | Παραστατικό  | Invoice with MARK {mark} cannot be cancelled because of being posted by provider                                        |
| 200 OK      | ValidationError  | 250      | Παραστατικό  | Invoice with MARK {mark} cannot be cancelled because of being posted by myDATA Invoicing                                |
| 200 OK      | ValidationError  | 251      | Παραστατικό  | Invoice with MARK {mark} cannot be cancelled because of being already cancelled                                         |
| 200 OK      | ValidationError  | 252      | Παραστατικό  | Record with MARK {mark} is not a valid Invoice                                                                         |
| 200 OK      | ValidationError  | 254      | Παραστατικό  | TaxLine (TaxTotals) : + {taxlinenumber} . {field + fieldData} is forbidden                                            |
| 200 OK      | ValidationError  | 255      | Παραστατικό  | Invoice with MARK {mark} cannot be cancelled because it is connected with active invoice with MARK {mark1}              |
| 200 OK      | ValidationError  | 256      | Παραστατικό  | Invoice with MARK {mark} cannot be classified because of being already cancelled                                        |
| 200 OK      | ValidationError  | 257      | Παραστατικό  | Invoice with MARK {mark} cannot be cancelled because of being posted by TaxRegister                                     |
| 200 OK      | ValidationError  | 258      | Παραστατικό  | All invoice rows should have {Field} [Possible {Field} values: {recType=3}                                            |
| 200 OK      | ValidationError  | 259      | Παραστατικό  | Invoice cannot be posted because it replaces invoice with MARK {mark} having same UID and is still connected with active invoices |
| 200 OK      | ValidationError  | 260      | Παραστατικό  | IssueDate is invalid, it must be less or equal than current date                                                       |
| 200 OK      | ValidationError  | 261      | Παραστατικό  | Each Invoice row must have unique line number                                                                          |
| 200 OK      | ValidationError  | 262      | Παραστατικό  | {field} length must be less or equal than {number}                                                                     |
| 200 OK      | ValidationError  | 263      | Παραστατικό  | Invoice with invoiceVariationType {field} and IssueDate {issueDate} cannot be sent earlier than { date} [Possible {field} values: {invoiceVariationType=1,2,3,4} |
| 200 OK      | ValidationError  | 264      | Παραστατικό  | Invoice with invoiceVariationType {field} cannot be sent for issueDate's year earlier than 2021 [Possible {field} values: {invoiceVariationType=1,2,3,4}]                                      |
| 200 OK      | ValidationError  | 265      | Παραστατικό  | The maximum allowed number of invoices contained in one message is 5000                                                       |
| 200 OK      | ValidationError  | 266      | Παραστατικό  | {msg11} is forbidden {msg2} [Possible {msg1} values: {‘Invoice with SpecialInvoiceCategoryType field with value 4 (tax-free invoice)’} [Possible {msg2} values: {‘for invoices sent by provider channel’}] |
| 200 OK      | ValidationError  | 267      | Παραστατικό  | {msg11} is allowed {msg2} [Possible {msg1} values: {‘Invoice with SpecialInvoiceCategoryType field with value 4 (tax-free invoice)’} [Possible {msg2} values: {‘only for invoices sent by erp or timologio channel’}] |
| 200 OK      | ValidationError  | 268      | Παραστατικό  | In case of fuelInvoice, at least one line must have fuelCode different from 999                                                 |
| 200 OK      | ValidationError  | 269      | Παραστατικό  | In case of fuelInvoice, the net value of the invoice line with fuelCode 999 must be less or equal than the sum of net values of the other invoice lines                                   |
| 200 OK      | ValidationError  | 270      | Παραστατικό  | In case of fuelInvoice, only one line can have fuelCode equal to 999                                                            |
| 200 OK      | ValidationError  | 271      | Παραστατικό  | Invoice Line Number: {linenumber}. VatExemptionCategory field is used only in case of vatCategory = 7                         |
| 200 OK      | ValidationError  | 272      | Παραστατικό  | {field} is mandatory + {moreInfo} [{moreInfo} μπορεί να είναι null]                                                           |
| 200 OK      | ValidationError  | 273      | Παραστατικό  | {field} is not allowed + {moreInfo} [{moreInfo} μπορεί να είναι null]                                                          |
| 200 OK      | ValidationError  | 318      | Παραστατικό  | Element {Field} must have the same value as correlated's one                                                                  |
| 200 OK      | ValidationError  | 319      | Παραστατικό  | Net value of the correlated invoice has already exceeded by the sum of net values of invoices correlated to it                 |
| 200 OK      | ValidationError  | 320      | Παραστατικό  | Invalid correlated invoice type                                                                                             |
| 200 OK      | ValidationError  | 322      | Παραστατικό  | Unsupported correlated invoice type                                                                                         |

                                                                                                                          
## Λάθη Κατηγοριοποίησης                                                                                                  
 HTTP Status | Error Code       | Error No | Πηγή | Περιγραφή                                          |                  
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



[myDATA API Documentation_Providers_v1.0.7_official.pdf]: https://www.aade.gr/sites/default/files/2023-10/myDATA%20API%20Documentation_Providers_v1%200%207_official.pdf
[myDATA API Documentation_v1.0.7_official_ERP.pdf]: https://www.aade.gr/sites/default/files/2023-10/myDATA%20API%20Documentation%20v1.0.7_official_erp.pdf
[EdDSA and Ed25519]: https://cryptobook.nakov.com/digital-signatures/eddsa-and-ed25519
[Key-based authentication in OpenSSH for Windows]: https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_keymanagement
[How To Generate ed25519 SSH Key]: https://www.unixtutorial.org/how-to-generate-ed25519-ssh-key/
