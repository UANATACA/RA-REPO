
# What it is

<div style="text-align: justify">
The Registration Authority (RA) solution to manage the entire digital certificates life cycle. 
<br></br>
The service facilitates the automation of processes through integration via API, allowing you to incorporate the process of issuing digital certificates into your business flows or procedures.
<br></br>
The issuance of digital certificates generates 100% paperless. All information related with the registration and contract issuance is electronically signed and saved in the system to avoid storing physical documentation for years.
</div>

# How it works

Uanataca RA API allows the generation and distribution of digital certificates, either by integrating the issuance and management into a business application.

The Registration Authority is composed of people and processes, with the main function of identifying applicants of digital certificates and sending verified data for the issuance of the certificate.

Digital certificates are created and used as part of a life-cycle that includes three fundamental stages:

- Registration including documentation
- Data validation
- Certificate issuance

The complete flow is explained in the next section <a href="#section/Flow-chart">Flow chart</a> 

As part of the paperless procedure, every Request has its own digital contract that has to be electronically signed by the RA Operator (in charge of approving the Request) and the certificate subscriber, each one with their respective digital certificates.
<br></br>

<img style="width: 80%; display: block; margin-left: auto; margin-right: auto" src="https://github.com/UANATACA/RA-REPO/raw/main/img/ra-hiw.png"/> 

# Glossary

List of entities and names used to describe UANATACA's services

>Certification Authority (CA)

The CA is the issuer of the certificates requested by the Registration Authority.

>Registration Authority (RA)

The RA manages the entire life-cycle of digital identities, from the certificate issuance to suspension, reactivation, renewal and revocation of the PKI credentials. Reuests are generated and verified by a Registration Authority Officer.

>Registration Authority Officer (RAO)

The RAO follows strict guidelines and policies defined to ensure the trust of the CA. RAO is responsible for managing the requests for digital certificates and verifying the content of the requests as well as identifying people requesting them.

>API User

The Account having access to the APIs provided by the system. It is generally used for a server to server interaction.

>Certificate Request (Request)

It is a request to issue a new certificate. A request can be associated with only one RA and has a status attribute to monitor the progress of the application:

**CREATED:** The request has been created and associated to an RA, but the content of the request has not been validated yet. In this state, data can also be inconsistent, the system will not throw an error. The content of the request can be edited at any moment to make it valid.

**ENROLLREADY**: The certificates are ready to be issued. The request arrives at this stage, if it has been approved and signed by a RAO, who is part of the RA in charge of the request.

**ISSUED:** The certificate is issued by the user's self-service page on the platform. The user must first set a PIN code or a password regarding the secure element.

**RENEWED:** The certificate is renewed by the user's self-service page on the platform.

**CANCELLED:** The request is cancelled and the digital certificate can not be issued.


>Scratchcard

It is a virtual scratch card containing the secret codes of the user.

The card contains:

- A serial number: it uniquely identifies the user
- An enrollment code: secret code, that is sent to the user by email

It is important to notice that a **scratchcard** can be used only once. Every **request** must be associated with a different **scratchcard**.


# Classic Workflow

<div style="text-align: justify">
The following image summarizes the common digital certificate request and issue flow:
</div>
<br></br>

![img](https://github.com/UANATACA/RA-REPO/raw/main/img/ra-flc2.png)

</br>

1. An end user requests a digital certificate to the Registration Authority (RA)
2. A Registration Authority Officer (RAO) identifies the user and requests the required documentation
3. The user sends the required documentation according to the certificate profile
4. The RAO creates a digital certificate request
5. The response returns a request ID
6. The RAO uploads the required documentation according to the request
7. The RAO verifies all request data and documentation
8. The RAO checks declaration and signs service contract.
9. The RAO approves the digital certificate request to allow the certificate issue
10. The user receives an email with a link to start the certificate generation process
11. The user access to the online digital certificate generation process
12. During the process, the service contract is shown to the user
13. An OTP code is sent to the user via sms
14. The user inserts the OTP code and creates a custom PIN
15. The certificate is generated and the service contract is signed by the user
16. Finally, the user receives the signed contract and an email with the certificate credentials and instructions

</br>

The common digital certificate generation process involves the following steps:

**1) CREATION OF A REQUEST**

**2) UPLOAD DOCUMENTS**

**3) REQUEST APPROVAL**

**4) CLOUD/SOFTWARE ENROLLMENT**


</br>

> **STEP 1: CREATION OF A REQUEST**

</br>

**API reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests/post">Create Request</a>

This call must include enough information to identify the requester user. The full description of the arguments accepted by this endpoint can be found in the API call detailed documentation.

    curl -i -X POST 'https://api.uanataca.com/api/v1/requests/' \
    -H 'Content-Type: application/json' \
    --cert 'cer.pem' --key 'key.pem'
    -d '{
      "profile": "PFnubeAFCiudadano",
      "scratchcard": "5053311",
      "secure_element": "2",
      "registration_authority": "116",
      "country_name": "ES",
      "serial_number": "12345678A",
      "id_document_country": "ES",
      "id_document_type": "IDC",
      "given_name": "Name",
      "surname_1": "Surname1",
      "surname_2" "Surname2"
      "email": "mail@domain.com",
      "mobile_phone_number": "+34611223344",
      "paperless_mode": 1
    }'


The response is a JSON containing info from the created request. One of the most important parameters from this JSON is the `pk` which represents the request unique identifier and is used for every operation related to this request.

    {
      "pk": 11223,
      "given_name": "Name",
      "surname_1": "Surname1",
      "surname_2": "Surname2",
      "sex": null,
      "id_document_type": "IDC",
      "id_document_country": "ES",
      "serial_number": "12345678A",
      "country_name": "ES",
      "citizenship": null,
      "residence": null,
      "organization_email": null,
      "email": "mail@domain.com",
      "title": null,
      "organization_name": null,
      "organizational_unit_1": null,
      ...
    }

</br>

> **STEP 2: UPLOAD DOCUMENTS**

</br>

**API reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1pl_upload_document/post">Upload Document</a>

The created request needs documents, so we can query with an HTTP POST request to upload the files.

The required documents for every request are:<br>
`document_front` : The photo of the front side of the requester ID card<br>
`document_rear` : The photo of the rear side of the requester ID card<br>
`extra_document` : If necessary, it is possibile to upload extra documents that represents additional requester information


Additionally a selfie of the requester showing the ID card under the chin can be uploaded as an evidence under the type `document_owner`.

Note that this endpoint has to be queried for every document type that the Request needs.

    curl -i -X POST 'https://api.uanataca.com/api/v1/requests/11223/pl_upload_document/' \
    --cert 'cer.pem' --key 'key.pem'
      -H 'Content-Type: multipart/form-data' \
      -F document=@/idc_front.jpg \
      -F type=document_front

The response contains the uploaded document unique identifier associated to the request.

    {
      "pk": 11314,
      "type": "document_front"
    }

</br>

> **STEP 3: REQUEST APPROVAL**

</br>

If all information is correct, the RAO will approve the request by signing the receipt and contract with his or her own cloud certificate. These calls are shown below:

</br>

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1generates_tbs_receipt/post">Generate RAO Declaration</a>

    curl -i -X POST https://api.uanataca.com/api/v1/requests/25139/generates_tbs_receipt/ \
    -H 'Content-Type: application/json' \
    -d '{
      "rao": "1400",
      "type": "APPROVE"
    }'

The following JSON object contains the receipt:

    {
      "serial_number": "3ef3696d2939241d",
      "receipt": "El operador RAO_Name RAO_Surname1 con número de identificación 12345678P\r\nactuando en calidad de operador autorizado de registro del prestador de servicios\r\n
      de confianza UANATACA, S.A. con NIF A66721499, (UANATACA en lo sucesivo)\r\n\r\nDECLARA\r\n\r\nQue previa verificación de acuerdo a la Declaración de Prácticas de
      UANATACA\r\npublicadas en www.uanataca.com, la información detallada a continuación es\r\ncorrecta y será incluida (donde aplicable) en la solicitud de 
      certificados\r\ncualificados:\r\n\r\n- Datos de Identificación de la solicitud de certificados: 36893\r\n- Nombre y Apellidos del Firmante: Name Surname1 Surname2\r\n- DNI/
      NIE/PASAPORTE del Firmante: 11111111B\r\n- Dirección de correo electrónico del Firmante: mail@domain.com\r\n\r\n\r\n18/03/
      2021\r\n\r\n\r\n\r\n--------------------------------------------------------------------\r\nFdo. User Admin\r\nOperador autorizado de registro"
    }

</br>

Similarly, it is necessary to retrieve the service contract and present it to the RAO before approval.

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1pl_get_document/post">Generate Contract</a> (`type`: **contract**)

    curl -i -X POST https://api.uanataca.com/api/v1/requests/25139/pl_get_document/ \
    -H 'Content-Type: application/json' \
    -d '{
      "type": "contract"
      "rao_id": "1400"    
    }'


The response consists in a JSON structure containing the contract in Base64 format.

    [
      {
        "document": "JVBERi0xLjQKJZOMi54gUmVwb3J0TGFiIEdlbmVyYXRlZCBQREYgZG9jdW1lbnQgaHR0cDovL3d3\ndy5yZXBvcnRsYWIuY29tCjEgMCBvYmoKPDwKL0YxIDIgMCBSCj4 (...)\n",
        "type": "contract"
      }
    ]

</br>

**API reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1pl_approve/post">Approve Request</a>

A Registration Authority Officer must first validate the request data and documentation. If the information is correct, the RAO will approve the request by signing the receipt and contract with his or her own cloud certificate.

In order to approve a Request, this must be in the status of CREATED and must have at least the required documents (document_front and document_rear).

    curl -i -X POST 'https://api.uanataca.com/api/v1/requests/' \
    -H 'Content-Type: application/json' \
    --cert 'cer.pem' --key 'key.pem'
    -d '{
      "username": "1000279",
      "password": "3DPTm:N4",
      "pin": "23bYQq9a",
      "rao_id": 123
    }'

</br>

> **STEP 4: CLOUD/SOFTWARE ENROLLMENT**

</br>

In this step, the service contract must be presented to the signer before enrollment.

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1pl_get_document/post">Generate Contract</a> (Body `type`: **contract**)

There are different endpoints to enroll a request depending on the secure element chosen. The next action involves sending an otp code to the requester using the calls shown below. Software and cloud certificates use the same call to send the otp code, as cloud-qscd certificates use a different one.

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1generate_otp/post">Generate OTP (Cloud or Software)</a>

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1generate_otp_for_qs/post">Generate OTP (Cloud or QSCD)</a>

</br>

**SOFTWARE**

</br>

API reference: <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1pl_p12_enroll/post">Software Enroll</a>

For the Software enrollemnt the parameters required are the secret OTP code send to the requester and the p12password set by the requester to import the generated p12:

    {
      "secret": "000000",
      "p12password": "password12"
    }

At the end of the enrollment the server replies with the P12 generated in PEM format.

</br>

**CLOUD**

</br>

API reference: <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1pl_cloud_enroll/post">Cloud Enroll</a>

For the cloud enrollemnt the parameters required are the secret OTP code send to the requester and the PIN code set by the requester to use the generated certificate:

    {
      "secret": "000000",
      "pin": "pincode12"
    }

At the end of the enrollment the server replies with a JSON containing all requesta data.

</br>

**CLOUD-QSCD**

</br>

API reference: <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1plq_cloud_enroll/post">Cloud-QSCD Enroll</a>

For the cloud enrollemnt the parameters required are the secret OTP code send to the requester and the PIN code set by the requester to use the generated certificate:

    {
      "secret": "000000",
      "pin": "pincode12"
    }

At the end of enrollment the server replies with a JSON containing all request data.

</br>

> **PROCESS COMPLETION**</br>


For correct process completion, the following information must be delivered to the requester:

- The certificate in .p12 format (Software Enroll)

- The certificate set of credentials (Cloud Enroll)

- The contract signed by both parties. Available when executing the <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1pl_get_document/post">Get Signed Contract</a> call (Body `type`: **signed_contract**)

</br>

> **OPTIONAL**

</br>

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}/get">Get Request</a>

</html>


# Video ID Workflows




## eIDAS VideoID


In eIDAS VideoID mode, a request approval it is required. For this reason, executing the validation step is not required.(Duda)

![img](https://raw.githubusercontent.com/UANATACA/RA-REPO/main/img/flc-1step.png)

</br>

This certificate generation process involves the following steps:

</br>

**1) CREATION OF A REQUEST**

**2) VIDEO IDENTIFICATION**

**3) VALIDATION OF VIDEO IDENTIFICATION**

**4) UPDATE OF REQUEST (OPTIONAL)**

**5) DOCUMENTS UPLOAD (OPTIONAL)**

**6) REQUEST APPROVAL**

</br>

> **STEP 1: CREATION OF A REQUEST**

</br>

**API Reference:** <a href="#tag/Scratchcards/paths/~1api~1v1~1scratchcards~1get_first_unused/get">Get First Unused Scratchcard</a>

This call simply requires a Registration Authority (RA) id number. Scratchcards must be available for this RA for successful response.

    curl -i -X GET https://api.uanataca.com/api/v1/scratchcards/get_first_unused/ \
    -H 'Content-Type: application/json' \
    --cert 'cer.pem' --key 'key.pem'
    -d '{
      "ra": "121"
    }'

The response is a JSON object containing the single-use Scratchcard associated data. The scratchcard number `sn` must be added to the <a href="#tag/Requests/paths/~1api~1v1~1requests/post">Create Request</a> call. 

    {
      "pk": 1193,
      "sn": "1256948",
      "secrets": "{\"erc\": \"6292998123\", \"enrollment_code\": \"_,463vt:\", \"pin\": \"08695572\", \"puk\": \"52351291\"}",
      "registration_authority": 121
    }

</br>

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests/post">Create Request</a>

This call must include this parameters to success. The full description of the arguments accepted by this endpoint can be found in the call detailed documentation.

    curl -i -X POST 'https://api.uanataca.com/api/v1/requests/' \
    -H 'Content-Type: application/json' \
    --cert 'cer.pem' --key 'key.pem'
    -d '{
      "profile": "PFnubeAFCiudadano",
      "scratchcard": "5053311",
      "secure_element": "2",
      "registration_authority": "116",
      "email": "mail@domain.com",
      "mobile_phone_number": "+34611223344",
      "videoid_mode": 1
    }'

The response is the a JSON containing info from the created request in **VIDEOPENDING** status. One of the most important parameters from this JSON is the `pk` which represents the request unique identifier and is used for every operation related to this request.

    {
      "pk": 25139,
      "given_name": "Name",
      "surname_1": "Surname1",
      "surname_2": "Surname2",
      "sex": null,
      "id_document_type": "IDC",
      "id_document_country": "ES",
      "serial_number": "A9999999E",
      "country_name": "ES",
      "citizenship": null,
      "residence": null,
      "organization_email": null,
      "email": "mail@domain.com",
      "title": null,
      "organization_name": null,
      "organizational_unit_1": null,
      (...)
    }


At this point, the workflow progress will depend on the video-identification process taken place on client side. Its successful completion will change request status from to **VIDEOREVIEW**. </br>

<blockquote style="background-color: #faf3ac; border-color: #5a5a5a; color: #3b3b3b;">⚠ In case the process is not totally completed or has failed for any reason, the request will change to <b>VIDEOINCOMPLETE</b> or <b>VIDEOERROR</b> respectively.</blockquote></br>

To inform business app and validation RAO about this change at the time it takes place, we recommend the implementation of a **Webhook**. Check our documentation for <a href='#section/Configuration/Webhooks'>Webhook Configuration</a>.</br>

If request data needs to be modified, use the <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}/put">Update Request</a> call. Check API Reference.</br> (Move to step 4)

If request data needs to be retrieved, use the <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}/get">Get Request</a> call. Check API Reference.

If request needs to be cancelled, use the <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1cancel/delete">Cancel Request</a> call. Check API Reference.

</br>


> **STEP 2: VIDEO IDENTIFICATION**

</br>

Through Video Identification process, evidences from the certificate subscriber such as name,surname or data related to subscriber ID card are obtained.

</br>


> **STEP 3: VALIDATION OF VIDEO IDENTIFICATION**

</br>

With the information gathered in the above mentioned process, a UANATACA Certified Operator will be in charge of validating every evidence.

In case of Natural Person profiles, the request gets approved in this step, meaning that it is completed.

</br>



</br>

> **STEP 4: UPDATE OF THE REQUEST**

</br>

RAO entity compliments every field that does not belong to subscriber identification.
Be aware that the fields filled in with information obtained from the video identification process cannot be modified in any way.

This step is required for all profiles except Natural Person.

</br>

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}/put">Update Request</a>

    curl -i -X PUT https://api.uanataca.com/api/v1/requests/{ID}/ \
    -H 'Content-Type: application/json' \
    -d '{
      "given_name": "Name",
      "surname_1": "Surname1",
      "scratchcard": "1234567890",
      "country_name": "ES",
      "email": "mail@domain.com",
      "profile": "PFnubeAFCiudadano",
      "registration_authority": 64
    }'

</br>

> **STEP 5: DOCUMENTS UPLOAD**

</br>

RAO entity uploads required documentation subject to the type of profile being issued.

This step is required for all profiles except Natural Person.

</br>

**API Reference:** Upload documents

    curl -i -X PUT https://api.uanataca.com/api/v1/requests/{request}/upload_videoid_evidence/ \
    -H 'Content-Type: multipart/form-data' \
    -F document=@sample_folder/sample.pdf \
    -F label=test label

</br>

> **STEP 6: REQUEST APPROVAL**

</br>

To finalize a request, you must make sure to validate both additional documents and additional fields.
Before approval, you will be shown a preview of RAO declaration.

</br>

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1generates_tbs_receipt/post">Generate RAO Declaration</a>

    curl -i -X POST https://api.uanataca.com/api/v1/requests/25139/generates_tbs_receipt/ \
    -H 'Content-Type: application/json' \
    -d '{
      "rao": "1400",
      "type": "APPROVE"
    }'

The following JSON object contains the receipt:

    {
      "serial_number": "3ef3696d2939241d",
      "receipt": "El operador RAO_Name RAO_Surname1 con número de identificación 12345678P\r\nactuando en calidad de operador autorizado de registro del prestador de servicios\r\n
      de confianza UANATACA, S.A. con NIF A66721499, (UANATACA en lo sucesivo)\r\n\r\nDECLARA\r\n\r\nQue previa verificación de acuerdo a la Declaración de Prácticas de
      UANATACA\r\npublicadas en www.uanataca.com, la información detallada a continuación es\r\ncorrecta y será incluida (donde aplicable) en la solicitud de 
      certificados\r\ncualificados:\r\n\r\n- Datos de Identificación de la solicitud de certificados: 36893\r\n- Nombre y Apellidos del Firmante: Name Surname1 Surname2\r\n- DNI/
      NIE/PASAPORTE del Firmante: 11111111B\r\n- Dirección de correo electrónico del Firmante: mail@domain.com\r\n\r\n\r\n18/03/
      2021\r\n\r\n\r\n\r\n--------------------------------------------------------------------\r\nFdo. User Admin\r\nOperador autorizado de registro"
    }

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1pl_approve/post">Approve Request</a>

This call makes the request ready for enrollment. Its status changes to **ENROLLREADY** after executing this call. In 1-step mode, both validation and approval occur when executing this call.

    curl -i -X POST 'https://api.uanataca.com/api/v1/requests/' \
    -H 'Content-Type: application/json' \
    --cert 'cer.pem' --key 'key.pem'
    -d '{
      "username": "1000279",
      "password": "3DPTm:N4",
      "pin": "23bYQq9a",
      "rao_id": "1400",
      "lang": "ES"
    }'

The response is a JSON object with added request approval information. 

    {
      "secrets": {
        "puk": "38812452",
        "enrollment_code": ".R4P9qgA",
        "pin": "31945152",
        "erc": "3417062505"
      },
      "request": {
        "pk": 25139,
        "given_name": "Name",
        "surname_1": "Surname1",
        "surname_2": "Surname2",
        "sex": null,
        "id_document_type": "IDC",
        "id_document_country": "ES",
        "serial_number": "A9999999E",
        (...)
        "approving_rao": {
          "pk": 1400,
          "given_name": "RAO_Name",
          "surname_1": "RAO_Surname1",
          "surname_2": "RAO_Surname2",
          (...)
        }
      }
    }

</br>

**PROCESS COMPLETION**

For correct process completion, the following information must be delivered to the requester:

- The certificate in .p12 format (Software)

- The certificate set of credentials (Cloud)

- The contract signed by both parties. Available when executing the <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1pl_get_document/post">Get Signed Contract</a> call (`type`: **signed_contract** in body)

</br>

> **OPTIONAL**

</br>

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}/get">Get Request</a>

**API Reference:** <a href="#tag/Video-ID/paths/~1api~1v1~1download~1video~1{video_identifier}/get">Download video</a>

</html>








## 2-Step Validation

In 2-Step mode Video ID, request validations and approvals are performed in different stages, by the same or different operators.

![img](https://raw.githubusercontent.com/UANATACA/RA-REPO/main/img/flc-2step.png)

</br>

This certificate generation process involves the following steps:

</br>

**1) CREATION OF A REQUEST**

**2) REQUEST VALIDATION**

**3) REQUEST APPROVAL**

**4) CLOUD/SOFTWARE ENROLLMENT**

</br>

> **STEP 1: CREATION OF A REQUEST**

</br>

**API Reference:** <a href="#tag/Scratchcards/paths/~1api~1v1~1scratchcards~1get_first_unused/get">Get First Unused Scratchcard</a>

This call simply requires a Registration Authority (RA) id number. Scratchcards must be available for this RA for successful response.

    curl -i -X GET https://api.uanataca.com/api/v1/scratchcards/get_first_unused/ \
    -H 'Content-Type: application/json' \
    --cert 'cer.pem' --key 'key.pem'
    -d '{
      "ra": "121"
    }'

The response is a JSON object containing the single-use Scratchcard associated data. The scratchcard number `sn` must be added to the <a href="#tag/Requests/paths/~1api~1v1~1requests/post">Create Request</a> call. 

    {
      "pk": 1193,
      "sn": "1256948",
      "secrets": "{\"erc\": \"6292998123\", \"enrollment_code\": \"_,463vt:\", \"pin\": \"08695572\", \"puk\": \"52351291\"}",
      "registration_authority": 121
    }

</br>

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests/post">Create Request</a>

This call must include enough information to identify the end user. The full description of the arguments accepted by this endpoint can be found in the call detailed documentation.

    curl -i -X POST 'https://api.uanataca.com/api/v1/requests/' \
    -H 'Content-Type: application/json' \
    --cert 'cer.pem' --key 'key.pem'
    -d '{
      "profile": "PFnubeAFCiudadano",
      "scratchcard": "5053311",
      "secure_element": "2",
      "registration_authority": "116",
      "country_name": "ES",
      "serial_number": "12345678A",
      "id_document_country": "ES",
      "id_document_type": "IDC",
      "given_name": "Name",
      "surname_1": "Surname1",
      "surname_2" "Surname2"
      "email": "mail@domain.com",
      "mobile_phone_number": "+34611223344",
      "videoid_mode": 1,
      "webhook_url": "my-webhook-url.com"
    }'

The response is the a JSON containing info from the created request in **VIDEOPENDING** status. One of the most important parameters from this JSON is the `pk` which represents the request unique identifier and is used for every operation related to this request.

    {
      "pk": 25139,
      "given_name": "Name",
      "surname_1": "Surname1",
      "surname_2": "Surname2",
      "sex": null,
      "id_document_type": "IDC",
      "id_document_country": "ES",
      "serial_number": "A9999999E",
      "country_name": "ES",
      "citizenship": null,
      "residence": null,
      "organization_email": null,
      "email": "mail@domain.com",
      "title": null,
      "organization_name": null,
      "organizational_unit_1": null,
      (...)
    }


At this point, the workflow progress will depend on the video-identification process taken place on client side. Its successful completion will change request status to **VIDEOREVIEW**. </br>

<blockquote style="background-color: #faf3ac; border-color: #5a5a5a; color: #3b3b3b;">⚠ In case the process is not totally completed or has failed for any reason, the request will change to <b>VIDEOINCOMPLETE</b> or <b>VIDEOERROR</b> respectively.</blockquote>

If request data needs to be modified, use the <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}/put">Update Request</a> call. Check API Reference.</br>

If request data needs to be retrieved, use the <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}/get">Get Request</a> call. Check API Reference.

</br>

> **STEP 2: REQUEST VALIDATION** `2-step mode only`

</br>

**API Reference:** <a href="#tag/Video-ID/paths/~1api~1v1~1requests~1{id_request}~1validate_videoid/post">Validate Video ID Request</a>

A Registration Authority Officer must validate the request data and evidences before approval. This call is used only for 2-step mode.  

    curl -i -X POST https://api.uanataca.com/api/v1/requests/25139/validate_videoid \
    -H 'Content-Type: application/json' \
    --cert 'cer.pem' --key 'key.pem'
    -d '{
      "username": "5012345",
      "password": "Gy6F37xK",
      "pin": "belorado74",
      "rao_id": "1400"
    }'

The validation successful response changes the request to **CREATED** status as a JSON object containing full request information is returned.

    {
      "secrets": {
        "puk": "38812452",
        "enrollment_code": ".R4P9qgA",
        "pin": "31945152",
        "erc": "3417062505"
      },
      "request": {
        "pk": 25139,
        "given_name": "Name",
        "surname_1": "Surname1",
        "surname_2": "Surname2",
        "sex": null,
        "id_document_type": "IDC",
        "id_document_country": "ES",
        "serial_number": "A9999999E",
        (...)
      }
    }

</br>

For unsuccessful validations leading to a request refusal, the corresponding call is  <a href="#tag/Video-ID/paths/~1api~1v1~1requests~1{id_request}~1refuse_videoid/post">Refuse Request</a>. Check API Reference.

</br>

> **STEP 3: REQUEST APPROVAL**

</br>

If all information is correct, the RAO will approve the request by signing the receipt and contract with his or her own cloud certificate. These calls are shown below:

</br>

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1generates_tbs_receipt/post">Generate RAO Declaration</a>

    curl -i -X POST https://api.uanataca.com/api/v1/requests/25139/generates_tbs_receipt/ \
    -H 'Content-Type: application/json' \
    -d '{
      "rao": "1400",
      "type": "APPROVE"
    }'

The following JSON object contains the receipt:

    {
      "serial_number": "3ef3696d2939241d",
      "receipt": "El operador RAO_Name RAO_Surname1 con número de identificación 12345678P\r\nactuando en calidad de operador autorizado de registro del prestador de servicios\r\n
      de confianza UANATACA, S.A. con NIF A66721499, (UANATACA en lo sucesivo)\r\n\r\nDECLARA\r\n\r\nQue previa verificación de acuerdo a la Declaración de Prácticas de
      UANATACA\r\npublicadas en www.uanataca.com, la información detallada a continuación es\r\ncorrecta y será incluida (donde aplicable) en la solicitud de 
      certificados\r\ncualificados:\r\n\r\n- Datos de Identificación de la solicitud de certificados: 36893\r\n- Nombre y Apellidos del Firmante: Name Surname1 Surname2\r\n- DNI/
      NIE/PASAPORTE del Firmante: 11111111B\r\n- Dirección de correo electrónico del Firmante: mail@domain.com\r\n\r\n\r\n18/03/
      2021\r\n\r\n\r\n\r\n--------------------------------------------------------------------\r\nFdo. User Admin\r\nOperador autorizado de registro"
    }

</br>

Similarly, it is necessary to retrieve the service contract and present it to the RAO before approval.

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1pl_get_document/post">Generate Contract</a> (use `type`: **contract** in body)

    curl -i -X POST https://api.uanataca.com/api/v1/requests/25139/pl_get_document/ \
    -H 'Content-Type: application/json' \
    -d '{
      "type": "contract"
      "rao_id": "1400"    
    }'


The response consists in a JSON structure containing the contract in Base64 format.

    [
      {
        "document": "JVBERi0xLjQKJZOMi54gUmVwb3J0TGFiIEdlbmVyYXRlZCBQREYgZG9jdW1lbnQgaHR0cDovL3d3\ndy5yZXBvcnRsYWIuY29tCjEgMCBvYmoKPDwKL0YxIDIgMCBSCj4 (...)\n",
        "type": "contract"
      }
    ]

</br>

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1pl_approve/post">Approve Request</a>

This call makes the request ready for enrollment. Its status changes to **ENROLLREADY**. In 1-step mode, both validation and approval occur when executing this call.

    curl -i -X POST 'https://api.uanataca.com/api/v1/requests/' \
    -H 'Content-Type: application/json' \
    --cert 'cer.pem' --key 'key.pem'
    -d '{
      "username": "1000279",
      "password": "3DPTm:N4",
      "pin": "23bYQq9a",
      "rao_id": 123,
      "lang": "ES"
    }'

The response is a JSON object with added request approval information. 

    {
      "secrets": {
        "puk": "38812452",
        "enrollment_code": ".R4P9qgA",
        "pin": "31945152",
        "erc": "3417062505"
      },
      "request": {
        "pk": 25139,
        "given_name": "Name",
        "surname_1": "Surname1",
        "surname_2": "Surname2",
        "sex": null,
        "id_document_type": "IDC",
        "id_document_country": "ES",
        "serial_number": "A9999999E",
        (...)
        "approving_rao": {
          "pk": 1400,
          "given_name": "RAO_Name",
          "surname_1": "RAO_Surname1",
          "surname_2": "RAO_Surname2",
          (...)
        }
      }
    }

</br>

In case of not approving a request for any reason, the call <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1cancel/delete">Cancel Request</a> must be executed. Check API Reference.

</br>

> **STEP 4: CLOUD/SOFTWARE ENROLLMENT**

</br>

In this step, the service contract must be presented to the signer before enrollment.

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1pl_get_document/post">Generate Contract</a> (use `type`: **contract** in body)

There are different endpoints to enroll a request depending on the secure element chosen. The next action involves sending an otp code to the requester using the calls shown below. Software and cloud certificates use the same call to send the otp code, as cloud-qscd certificates use a different one.

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1generate_otp/post">Generate OTP (Cloud or Software)</a>

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1generate_otp_for_qs/post">Generate OTP (Cloud or QSCD)</a>

</br>

**Software**

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1pl_p12_enroll/post">Software Enroll</a>

For the Software enrollemnt the parameters required are the secret OTP code send to the requester and the p12password set by the requester to import the generated p12:

    {
      "secret": "000000",
      "p12password": "password12"
    }

At the end of the enrollment the server replies with the P12 generated in PEM format.

</br>

**Cloud**

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1pl_cloud_enroll/post">Cloud Enroll</a>

For the cloud enrollemnt the parameters required are the secret OTP code send to the requester and the PIN code set by the requester to use the generated certificate:

    {
      "secret": "000000",
      "pin": "pincode12"
    }

At the end of the enrollment the server replies with a JSON containing all requesta data.

</br>

**Cloud-QSCD**

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1plq_cloud_enroll/post">Cloud-QSCD Enroll</a>

For the cloud enrollemnt the parameters required are the secret OTP code send to the requester and the PIN code set by the requester to use the generated certificate:

    {
      "secret": "000000",
      "pin": "pincode12"
    }

After this call, the server replies with a JSON object containing all request data.

</br>

**PROCESS COMPLETION**

For correct process completion, the following information must be delivered to the requester:

- The certificate in .p12 format (Software Enroll)

- The certificate set of credentials (Cloud Enroll)

- The contract signed by both parties. Available when executing the <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1pl_get_document/post">Get Signed Contract</a> call (use `type`: **signed_contract** in body)

</br>

> **OPTIONAL**

</br>

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}/get">Get Request</a>

**API Reference:** <a href="#tag/Video-ID/paths/~1api~1v1~1download~1video~1{video_identifier}/get">Download video</a>

</html>




## External Mode


![img](https://raw.githubusercontent.com/UANATACA/RA-REPO/main/img/flc-ext.png)

</br>

The External-Mode Video ID certificate generation process involves the following steps:

</br>

**1) CREATION OF A REQUEST**

**2) UPLOAD EVIDENCES**

**3) REQUEST VALIDATION**

**4) REQUEST APPROVAL**

**5) CLOUD/SOFTWARE ENROLLMENT**

</br>

> **STEP 1: CREATION OF A REQUEST**

</br>

**API Reference:** <a href="#tag/Scratchcards/paths/~1api~1v1~1scratchcards~1get_first_unused/get">Get First Unused Scratchcard</a>

This call simply requires a Registration Authority (RA) id number. Scratchcards must be available for this RA for successful response.

    curl -i -X GET https://api.uanataca.com/api/v1/scratchcards/get_first_unused/ \
    -H 'Content-Type: application/json' \
    --cert 'cer.pem' --key 'key.pem'
    -d '{
      "ra": "121"
    }'

The response is a JSON object containing the single-use Scratchcard associated data. The scratchcard number `sn` must be added to the <a href="#tag/Requests/paths/~1api~1v1~1requests/post">Create Request</a> call. 

    {
      "pk": 1193,
      "sn": "1256948",
      "secrets": "{\"erc\": \"6292998123\", \"enrollment_code\": \"_,463vt:\", \"pin\": \"08695572\", \"puk\": \"52351291\"}",
      "registration_authority": 121
    }

</br>

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests/post">Create Request</a>

This call must include enough information to identify the end user. The full description of the arguments accepted by this endpoint can be found in the call detailed documentation.

    curl -i -X POST 'https://api.uanataca.com/api/v1/requests/' \
    -H 'Content-Type: application/json' \
    --cert 'cer.pem' --key 'key.pem'
    -d '{
      "profile": "PFnubeAFCiudadano",
      "scratchcard": "5053311",
      "secure_element": "2",
      "registration_authority": "116",
      "country_name": "ES",
      "serial_number": "12345678A",
      "id_document_country": "ES",
      "id_document_type": "IDC",
      "given_name": "Name",
      "surname_1": "Surname1",
      "surname_2" "Surname2"
      "email": "mail@domain.com",
      "mobile_phone_number": "+34611223344",
      "videoid_mode": 1
    }'

The response is the a JSON containing info from the created request in **VIDEOPENDING** status. One of the most important parameters from this JSON is the `pk` which represents the request unique identifier and is used for every operation related to this request.

    {
      "pk": 25139,
      "given_name": "Name",
      "surname_1": "Surname1",
      "surname_2": "Surname2",
      "sex": null,
      "id_document_type": "IDC",
      "id_document_country": "ES",
      "serial_number": "A9999999E",
      "country_name": "ES",
      "citizenship": null,
      "residence": null,
      "organization_email": null,
      "email": "mail@domain.com",
      "title": null,
      "organization_name": null,
      "organizational_unit_1": null,
      (...)
    }


If request data needs to be modified, use the <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}/put">Update Request</a> call. Check API Reference.

If request data needs to be retrieved, use the <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}/get">Get Request</a> call. Check API Reference.

</br>

> **STEP 2: UPLOAD EVIDENCES**

</br>

A previously created Video ID Request needs a set of information defined as evidences. The succesful upload of **ALL** this information will change the request status to **VIDEOREVIEW**.

Data and images are uploaded by using the following call:

**API Reference:** <a href="#tag/Video-ID/paths/~1api~1v1~1upload~1data~1{video_identifier}/post">Upload Data Evidence</a>

</br>

**Data objects in detail:**

`acceptance` : Client acceptance parameters (e.g. Terms & Conditions,  Privacy Policy). This is a customizable JSON object.<br>
`data` : Set of pictures associated to the client's ID document plus a selfie of him/her. <br>
`ocr_data` : Text information extracted from the client's ID document via Optical Character Recognition (OCR). <br>
`security_checks` : Set of validation fields associated to the client's identity (underaging, matching info, liveliness, etc) <br>
`similarity_level` : Similarity between the client's selfie and the picture is shown on his/her ID document.  <br>

    curl -i -X PUT https://api.uanataca.com/api/v1/videoid/45836/evidences \
        -H 'Content-Type: application/json' \
        -d '{
            "acceptance": {
                "description": "User Accepted Terms and Conditions and Privacy Policy",
                "url-doc-privacypolicy": "https://www.uanataca.com/public/pki/privacidad-PSC/",
                "ip": "186.0.91.53",
                "url-web-videoid": "https://cms.access.bit4id.org:13035/lcmpl/videoid/46b92251-4ba8-4930-a5aa-8631ec4666b6",
                "user-agent": "Mozilla/5.0 (Linux; Android 11; AC2003)",
                "date": 1622823879708,
                "url-doc-termsconditions": "https://www.uanataca.com/public/pki/terminos-VID/"
            },
            "videoid_data": {
                "images": {
                    "document_front": "/9j/4AAQSkZJRgABAQAAAQABAAD/2wBDAAIBAQEBAQIBAQECAgICAgQDAgICAgUEBAM (...)",
                    "document_rear": "/I7ye60+aOKS0mVGVSD9RVfyXukjmnS3cAEbpMVm6M1ncWqS3FszptO1lPRRDJ+orI8b (...)",
                    "document_photo": "AkjOOwFfHFrrNlpXxcbU9QuIIIkvR56yddgHpX3GEj1PmanmdS/xV1ySVlv/AIbXLPO (...)",
                    "document_owner": "SSVnovgCZ4Lhk+R3lJPUDJr5t/Z/wBV1DWfjRbeI75B5iQytcykc7yMEAV2/iwC0T34 (...)"
                },
                "ocr_data": {
                    "given_name": "Name",
                    "surname_1": "Surname 1",
                    "surname_2": "Surname 2",
                    "mobile_phone_number": "+34999999999",
                    "email": "mail@domain",
                    "serial_number": "A9999999E",
                    "id_document_type": "IDC",
                    "id_document_country": ES
                },
                "security_checks": {
                    "otp_validation": true,
                    "documents_match": true,
                    "data_integrity": true,
                    "document_notcopy": true,
                    "document_notexpired": true,
                    "document_notunderage": true,
                    "liveliness": true
                },
                "similarity_level": "high"
            }
        }

Successful response status

    {
      "status": "200 OK"
    }

</br>

In the same way, MP4-format Video evidence is uploaded by using the following call:

**API Reference:** <a href="#tag/Video-ID/paths/~1api~1v1~1upload~1video~1{video_identifier}/post">Upload Video</a>

    curl -i -X POST https://api.uanataca.com/v1/upload/video/30e57b02819a430d8386fd85be9f499f/ \
    -H 'Content-Type: multipart/form-data' \
    -F video=@sample_folder/sample_video.mp4 

Successful response status

    {
      "status": "200 OK"
    }

If the uploaded video needs to be retrieved, use <a href="#tag/Video-ID/paths/~1api~1v1~1download~1video~1{video_identifier}/get">Download Video</a> call.

</br>

> **STEP 3: REQUEST VALIDATION** `2-step mode only`

</br>

**API Reference:** <a href="#tag/Video-ID/paths/~1api~1v1~1requests~1{id_request}~1validate_videoid/post">Validate Video ID Request</a>

A Registration Authority Officer must validate the request data and evidences before approval. This call is used only for 2-step mode.  


    curl -i -X POST https://api.uanataca.com/api/v1/requests/25139/validate_videoid \
    -H 'Content-Type: application/json' \
    --cert 'cer.pem' --key 'key.pem'
    -d '{
      "username": "5012345",
      "password": "Gy6F37xK",
      "pin": "belorado74",
      "rao_id": "1400"
    }'

The validation successful response changes the request to **CREATED** status as a JSON object containing full request information is returned.

    {
      "secrets": {
        "puk": "38812452",
        "enrollment_code": ".R4P9qgA",
        "pin": "31945152",
        "erc": "3417062505"
      },
      "request": {
        "pk": 25139,
        "given_name": "Name",
        "surname_1": "Surname1",
        "surname_2": "Surname2",
        "sex": null,
        "id_document_type": "IDC",
        "id_document_country": "ES",
        "serial_number": "A9999999E",
        (...)
      }
    }

</br>

For unsuccessful validations leading to a request refusal, the corresponding call is  <a href="#tag/Video-ID/paths/~1api~1v1~1requests~1{id_request}~1refuse_videoid/post">Refuse Request</a>. Check API Reference.

</br>

> **STEP 4: REQUEST APPROVAL**

</br>

If all information is correct, the RAO will approve the request by signing the receipt and contract with his or her own cloud certificate. These calls are shown below:

</br>

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1generates_tbs_receipt/post">Generate RAO Declaration</a>

    curl -i -X POST https://api.uanataca.com/api/v1/requests/25139/generates_tbs_receipt/ \
    -H 'Content-Type: application/json' \
    -d '{
      "rao": "1400",
      "type": "APPROVE"
    }'

The following JSON object contains the receipt:

    {
      "serial_number": "3ef3696d2939241d",
      "receipt": "El operador RAO_Name RAO_Surname1 con número de identificación 12345678P\r\nactuando en calidad de operador autorizado de registro del prestador de servicios\r\n
      de confianza UANATACA, S.A. con NIF A66721499, (UANATACA en lo sucesivo)\r\n\r\nDECLARA\r\n\r\nQue previa verificación de acuerdo a la Declaración de Prácticas de
      UANATACA\r\npublicadas en www.uanataca.com, la información detallada a continuación es\r\ncorrecta y será incluida (donde aplicable) en la solicitud de 
      certificados\r\ncualificados:\r\n\r\n- Datos de Identificación de la solicitud de certificados: 36893\r\n- Nombre y Apellidos del Firmante: Name Surname1 Surname2\r\n- DNI/
      NIE/PASAPORTE del Firmante: 11111111B\r\n- Dirección de correo electrónico del Firmante: mail@domain.com\r\n\r\n\r\n18/03/
      2021\r\n\r\n\r\n\r\n--------------------------------------------------------------------\r\nFdo. User Admin\r\nOperador autorizado de registro"
    }

</br>

Similarly, it is necessary to retrieve the service contract and present it to the RAO before approval.

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1pl_get_document/post">Generate Contract</a> (use `type`: **contract** in body)

    curl -i -X POST https://api.uanataca.com/api/v1/requests/25139/pl_get_document/ \
    -H 'Content-Type: application/json' \
    -d '{
      "type": "contract"
      "rao_id": "1400"    
    }'


The response consists in a JSON structure containing the contract in Base64 format.

    [
      {
        "document": "JVBERi0xLjQKJZOMi54gUmVwb3J0TGFiIEdlbmVyYXRlZCBQREYgZG9jdW1lbnQgaHR0cDovL3d3\ndy5yZXBvcnRsYWIuY29tCjEgMCBvYmoKPDwKL0YxIDIgMCBSCj4 (...)\n",
        "type": "contract"
      }
    ]

</br>

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1pl_approve/post">Approve Request</a>

This call makes the request ready for enrollment. Its status changes to **ENROLLREADY**. In 1-step mode, both validation and approval occur when executing this call.

    curl -i -X POST 'https://api.uanataca.com/api/v1/requests/' \
    -H 'Content-Type: application/json' \
    --cert 'cer.pem' --key 'key.pem'
    -d '{
      "username": "1000279",
      "password": "3DPTm:N4",
      "pin": "23bYQq9a",
      "rao_id": 123,
      "lang": "ES"
    }'

The response is a JSON object with added request approval information. 

    {
      "secrets": {
        "puk": "38812452",
        "enrollment_code": ".R4P9qgA",
        "pin": "31945152",
        "erc": "3417062505"
      },
      "request": {
        "pk": 25139,
        "given_name": "Name",
        "surname_1": "Surname1",
        "surname_2": "Surname2",
        "sex": null,
        "id_document_type": "IDC",
        "id_document_country": "ES",
        "serial_number": "A9999999E",
        (...)
        "approving_rao": {
          "pk": 1400,
          "given_name": "RAO_Name",
          "surname_1": "RAO_Surname1",
          "surname_2": "RAO_Surname2",
          (...)
        }
      }
    }

</br>

In case of not approving a request for any reason, the call <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1cancel/delete">Cancel Request</a> must be executed. Check API Reference.

</br>

> **STEP 4: CLOUD/SOFTWARE ENROLLMENT**

</br>

In this step, the service contract must be presented to the signer before enrollment.

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1pl_get_document/post">Generate Contract</a> (use `type`: **contract** in body)

There are different endpoints to enroll a request depending on the secure element chosen. The next action involves sending an otp code to the requester using the calls shown below. Software and cloud certificates use the same call to send the otp code, as cloud-qscd certificates use a different one.

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1generate_otp/post">Generate OTP (Cloud or Software)</a>

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1generate_otp_for_qs/post">Generate OTP (Cloud or QSCD)</a>

</br>

**Software**

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1pl_p12_enroll/post">Software Enroll</a>

For the Software enrollemnt the parameters required are the secret OTP code send to the requester and the p12password set by the requester to import the generated p12:

    {
      "secret": "000000",
      "p12password": "password12"
    }

At the end of the enrollment the server replies with the P12 generated in PEM format.

</br>

**Cloud**

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1pl_cloud_enroll/post">Cloud Enroll</a>

For the cloud enrollemnt the parameters required are the secret OTP code send to the requester and the PIN code set by the requester to use the generated certificate:

    {
      "secret": "000000",
      "pin": "pincode12"
    }

At the end of the enrollment the server replies with a JSON containing all requesta data.

</br>

**Cloud-QSCD**

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1plq_cloud_enroll/post">Cloud-QSCD Enroll</a>

For the cloud enrollemnt the parameters required are the secret OTP code send to the requester and the PIN code set by the requester to use the generated certificate:

    {
      "secret": "000000",
      "pin": "pincode12"
    }

After this call, the server replies with a JSON object containing all request data.

</br>

**PROCESS COMPLETION**

For correct process completion, the following information must be delivered to the requester:

- The certificate in .p12 format (Software Enroll)

- The certificate set of credentials (Cloud Enroll)

- The contract signed by both parties. Available when executing the <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}~1pl_get_document/post">Get Signed Contract</a> call (use `type`: **signed_contract** in body)

</br>

> **OPTIONAL**

</br>

**API Reference:** <a href="#tag/Requests/paths/~1api~1v1~1requests~1{id}/get">Get Request</a>

**API Reference:** <a href="#tag/Video-ID/paths/~1api~1v1~1download~1video~1{video_identifier}/get">Download video</a>

</html>





# Webhook Configuration


One-Shot API requires a Webhook implemented on customer business side to manage our service callbacks. Every request status change will trigger a simple event-notification via HTTP POST, consisting on a JSON object to an URL that must be explicitly included as a **required parameter** in the <a href='#tag/Video-ID/paths/~1api~1v1~1videoid/post'>Create Video ID Request</a> call when using Uanataca 1-step or 2-step mode. 

The following is a sample view of the JSON object that is sent as a callback at every status change:

	{
		"status": "VIDEOINCOMPLETE", 
		"date": "2021-07-20T08:08:21.132394", 
		"previous_status": "VIDEOPENDING", 
		"request": 46760, 
		"registration_authority": 455
	}

Where:

**Status** is the most recent status, this is, the status that triggered the notification.</br>
**Date** is the date of the request status change in datetime format.</br>
**Previous_status** is the status inmediately previous to last change.</br>
**Request** is the request unique id.</br>
**Registration_authority** is the Registration Authority id number the request is associated.</br>

</br>

> **Sample code**

In this sample, every JSON object is stored in a file named 'videoid'.

The webhook parameter used in the <a href='#tag/Video-ID/paths/~1api~1v1~1videoid/post'>Create Video ID Request</a> call is defined as:

	{host}/videoid

where {host} is the IP or domain from the server exposing the webhook.

</br>

*Python*

	import web
	import datetime
	
	urls = (
	        '/videoid, 'videoid',
	        )
	
	app = web.application(urls, globals())
	app = app.wsgifunc()
	
	class video:
		def POST(self):
			data = web.data()
			f = open("status.json",'a+')
			f.write(data)
			f.close()
			return ''

	if __name__ == "__main__":
	    app.run()


*PHP*

	<?php
	
	//videoid.json

	$post = file_get_contents('php://input',true);
	$file_handle = fopen('/videoid/status.json', 'w');
	fwrite($file_handle, $post);
	fclose($file_handle);
	
	?>




# Endpoint URLs

Uanataca expose its API on urls composed as follows:

	https://{uanatacahost}/api/{version}/{resource}/

<blockquote style="background-color: #faf3ac; border-color: #5a5a5a; color: #3b3b3b;">⚠ Make sure the URL always ends with a forward slash ("/")</blockquote>

> Uanatacahost
 
The host changes according to the environment:

- **access.bit4id.org:13035** for test environment
- **api.uanataca.com** for production environment

<blockquote style="background-color: #faf3ac; border-color: #5a5a5a; color: #3b3b3b;">⚠ In test environment you need to trust the certificate <a href="https://cdn.bit4id.com/es/uanataca/public/ra/Bit4idCA.crt">Bit4idCA.crt</a></blockquote>

> Version
 
It is the api version (currently **v1**)

> Resource
 
It is the name of the resource of our interest.

Each resource can also have path parameters and sub-resources that are defined in the <a href="#APIReference">API Reference</a> below:

This is an example of endpoint exposed by Uanataca:

	https://api.uanataca.com/api/v1/requests/123/cloud_enroll/


# Authentication

The API authentication is perfomed providing to the server the certificate and the key of an enabled API User.

This is an example HTTP POST request perfomed with cURL:

	1 | curl --key key.pem --cert cert.pem -H "Content-Type: application/json" -d @params.json -X POST https://api.uanataca.com/api/v1/requests/

and a Python with requests package example:

	1 | import requests
	2 | requests.get(
	3 |     'https://api.uanataca.com/api/v1/scratchcards/',
	4 |     cert = ('/path/to/cert.pem', '/path/to/key.pem')
	5 | )


# Responses

Every response body returned by Uanataca is JSON object.

If the response is successful, the content of the JSON depends on the API queried.

Instead, the error response is always composed of these keys:

<html>
<table>
  <tr>
    <th>Key</th><th>Description</th>
  </tr>
  <tr>
    <td><b>error</b></td><td>A string that describe the error occured</td>
  </tr>
  <tr>
    <td><b>code</b></td><td>The HTTP response code related. See <a href="#section/Responses/HTTP-Status-Codes">table descriptions</a></td>
  </tr>
  <tr>
    <td><b>id</b></td><td>The unique identifier of the error generated by Uanataca</td>
  </tr>
</table> 
</br> 
</html>

In the <a href="#APIReference">API Reference</a> are described the response structures for each API call.

A successful response:

	1 | [
	2 |     {
	3 |         "data": "MIIHyTCCBbGgAwIBAgIIcO...",
	4 |         "profile": "PFnubeAF",
	5 |         "subject": "CN=RAO COFTenerife API, 2.5.4.5=TINIT-TSTAPI74S23C129Y, 2.5.4.42=RAO, 2.5.4.4=API, C=ES",
	6 |         "issuer": "2.5.4.97=VATES-A66721499, CN=UANATACA CA1 2016, OU=AC-UANATACA, O=UANATACA S.A., L=Barcelona (see current address at www.uanataca.com/address), C=ES",
	7 |         "valid_from": "2018-10-16T16:41:00",
	8 |         "valid_to": "2020-10-15T16:41:00",
	9 |         "serial_number": "70e07489bfccd478",
	10|         "status": 0,
	11|         "pk": 1980,
	12|         "revokation_reason": null,
	13|         "type": "FIRSTISSUE"
	14|     }
	15| ]

An error response:

	1 | {
	2 |     "code": "500",
	3 |     "id": "8e782cdcdb600a90",
	4 |     "error": "Invalid ScratchCard"
	5 | }


## HTTP Status Codes

<html>
<table>
  <tr>
    <th>Code</th><th>Description</th>
  </tr>
  <tr>
    <td>200</td><td>Everything went <b>OK</b>. The server elaborated correctly the request and returned the response to the client.</td>
  </tr>
  <tr>
    <td>201</td><td>The object is successfully <b>Created</b> with the parameters sent by the client.</td>
  </tr>
  <tr>
    <td>202</td><td>The request sent by the client has been <b>Accepted</b> and is under process.</td>
  </tr>
  <tr>
    <td>204</td><td><b>No Content.</b> The operation was successful but no content is provided in the response body.</td>
  </tr>
  <tr>
    <td>400</td><td><b>Bad Request.</b> The parameters sent, are not well formatted or are missing.</td>
  </tr>
  <tr>
    <td>401</td><td><b>Unauthorized.</b> The user used for making the request is not authorized to consume that resource.</td>
  </tr>
  <tr>
    <td>403</td><td><b>Forbidden.</b>The user used for making the request has no permissions to do it.</td>
  </tr>  
  <tr>
    <td>404</td><td>The resource requested is <b>Not Found.</b></td>
  </tr>
  <tr>
    <td>405</td><td><b>Method not allowed.</b> The endpoint called has not the method specified.</td>
  </tr>
  <tr>
    <td>412</td><td><b>Precondition Failed.</b> The operation has some requirements that are not satisfied. For example if a Request is in a wrong state for the operation requested.</td>
  </tr>
  <tr>
    <td>429</td><td><b>Too Many Requests.</b></td>
  </tr>
  <tr>
    <td>500</td><td><b>Internal Server Error.</b> An error occured during the elaboration of the request.</td>
  </tr>
  <tr>
    <td>502</td><td><b>Bad Gateway.</b></td>
  </tr>
  <tr>
    <td>503</td><td><b>Service Unavailable.</b></td>
  </tr>
</table> 
</br> 
</html>

## Pagination

Some endpoints works with the mechanism of pagination.

This means that when an API returns the content requested, the JSON is composed with the keys:

<html>
<table>
  <tr>
    <th>Key</th><th>Description</th>
  </tr>
  <tr>
    <td><b>count</b></td><td>Represents the number of object found</td>
  </tr>
  <tr>
    <td><b>next</b></td><td>Represents the url of the next page (it is null if there are no more pages)</td>
  </tr>
  <tr>
    <td><b>previous</b></td><td>Represents the url of the previous page (it is null if there are no more pages)</td>
  </tr>
  <tr>
    <td><b>result</b></td><td>Contains a list of objects found</td>
  </tr>
</table> 
</br> 
</html>

Every page contains at most **10 objects**.

**Examples**

Here are display a couple of JSON returned by Uanataca.

A list of Scratchcards:

	1 | {
	2 |     "count": 40,
	3 |     "next": "https://api.uanataca.com/api/v1/scratchcards/?page=2®istration_authority=8",
	4 |     "previous": null,
	5 |     "results": [
	6 |         {
	7 |             "pk": 801,
	8 |             "sn": "2000100",
	9 |             ...
	10|         },
	11|         {
	12|             "pk": 800,
	13|             "sn": "2000099",
	14|             ...
	15|         },
	16|         ...
	17|     ]
	18| }

A list of Requests:

	1 | {
	2 |     "count": 643,
	3 |     "next": "https://api.uanataca.com/api/v1/requests/?page=2",
	4 |     "previous": null,
	5 |     "results": [
	6 |         {
	7 |             "pk": 788,
	8 |             ...
	9 |         },
	10|         {
	11|             "pk": 789,
	12|             ...
	13|         },
	14|         ....
	15|     ]
	16| }


# Certificate Profiles

Uanataca provides different certificate profiles for different purpose.

Each profile has their set of fields and each field can be mandatory or not.

## Europe (eIDAS)

<html>
<table>
  <tr>
    <th>Profile</th><th>Description</th><th>Element</th>
  </tr>
  <tr>
    <td><a href="#PFSoftAFCiudadano">PFSoftAFCiudadano</a></td><td>Natural person</td><td>Software</td>
  </tr>
  <tr>
    <td><a href="#PFqscdCiudadano">PFqscdCiudadano</a></td></td><td>Natural person</td><td>Smartcard/Token</td>
  </tr>
  <tr>
    <td><a href="#PFnubeAFCiudadano">PFnubeAFCiudadano</a></td></td><td>Natural person</td><td>Cloud</td>
  </tr>
  <tr>
    <td><a href="#PFnubeQAFCiudadano">PFnubeQAFCiudadano</a></td><td>Natural person</td><td>Cloud-QSCD</td>
  </tr>
  <tr>
    <td><a href="#PFSoftAFEmpresa">PFSoftAFEmpresa</a></td><td>Natural person belonging to an organization</td><td>Software</td>
  </tr>
  <tr>
    <td><a href="#PFqscdEmpresa">PFqscdEmpresa</a></td><td>Natural person belonging to an organization</td><td>Smartcard/Token</td>
  </tr>
  <tr>
    <td><a href="#PFnubeAFEmpresa">PFnubeAFEmpresa</a></td><td>Natural person belonging to an organization</td><td>Cloud</td>
  </tr>
  <tr>
    <td><a href="#PFnubeQAFEmpresa">PFnubeQAFEmpresa</a></td><td>Natural person belonging to an organization</td><td>Cloud-QSCD</td>
  </tr>
  <tr>
    <td><a href="#PFSoftAFColegiado">PFSoftAFColegiado</a></td><td>Natural person belonging to a professional association</td><td>Software</td>
  </tr>
  <tr>
    <td><a href="#PFqscdColegiado">PFqscdColegiado</a></td><td>Natural person belonging to a professional association</td><td>Smartcard/Token</td>
  </tr>
  <tr>
    <td><a href="#PFnubeAFColegiado">PFnubeAFColegiado</a></td><td>Natural person belonging to a professional association</td><td>Cloud</td>
  </tr>
  <tr>
    <td><a href="#PFnubeQAFColegiado">PFnubeQAFColegiado</a></td><td>Natural person belonging to a professional association</td><td>Cloud-QSCD</td>
  </tr>
  <tr>
    <td><a href="#REPsoft">REPsoft</a></td><td>Natural person representative</td><td>Software</td>
  </tr>
  <tr>
    <td><a href="#REPqscd">REPqscd</a></td><td>Natural person representative (signature only)</td><td>Smartcard/Token</td>
  </tr>
  <tr>
    <td><a href="#REPnubeQ">REPnubeQ</a></td><td>Natural person representative (signature only)</td><td>Cloud-QSCD</td>
  </tr>
  <tr>
    <td><a href="#REPPJsoft">REPPJsoft</a></td><td>Natural person representative of legal person with the administration</td><td>Software</td>
  </tr>
  <tr>
    <td><a href="#REPPJnube">REPPJnube</a></td><td>Natural person representative of legal person with the administration</td><td>Cloud</td>
  </tr>
  <tr>
    <td><a href="#REPPJqscd">REPPJqscd</a></td><td>Natural person representative of legal person with the administration</td><td>Smartcard/Token</td>
  </tr>
  <tr>
    <td><a href="#REPPJnubeQ">REPPJnubeQ</a></td><td>Natural person representative of legal person with the administration</td><td>Cloud-QSCD</td>
  </tr>
  <tr>
    <td><a href="#EMPUBsoft">EMPUBsoft</a></td><td>Public employee - Medium level</td><td>Cloud/Software</td>
  </tr>
  <tr>
    <td><a href="#EMPUBqscd">EMPUBqscd</a></td><td>Public employee signature - High level</td><td>Smartcard/Token</td>
  </tr>
  <tr>
    <td><a href="#EMPUBnubeQ">EMPUBnubeQ</a></td><td>Public employee signature - High level</td><td>Cloud-QSCD</td>
  </tr>
  <tr>
    <td><a href="#REPESPJsoft">REPESPJsoft</a></td><td>Natural person representative of entity without legal personality with the administrations</td><td>Software</td>
  </tr>
  <tr>
    <td><a href="#REPESPJnube">REPESPJnube</a></td><td>Natural person representative of entity without legal personality with the administrations</td><td>Cloud</td>
  </tr>
  <tr>
    <td><a href="#REPESPJqscd">REPESPJqscd</a></td><td>Natural person representative of entity without legal personality with the administrations</td><td>Smartcard/Token</td>
  </tr>
  <tr>
    <td><a href="#REPESPJnubeQ">REPESPJnubeQ</a></td><td>Natural person representative of entity without legal personality with the administrations</td><td>Cloud-QSCD</td>
  </tr>
  <tr>
    <td><a href="#SELLOPJnube">SELLOPJnube</a></td><td>Electronic seal</td><td>Cloud</td>
  </tr>
  <tr>
    <td><a href="#SELLOPJsoft">SELLOPJsoft</a></td><td>Electronic seal</td><td>Software</td>
  </tr>
  <tr>
    <td><a href="#SELLOPJnubeQ">SELLOPJnubeQ</a></td><td>Electronic seal</td><td>Cloud-QSCD</td>
  </tr>
  <tr>
    <td><a href="#SELLOPJqscd">SELLOPJqscd</a></td><td>Electronic seal</td><td>Smartcard/Token</td>
  </tr>
  <tr>
    <td><a href="#SELLOMedio">SELLOMedio</a></td><td>Electronic seal – Medium Level APE</td><td>Cloud/Software</td>
  </tr>
  <tr>
    <td><a href="#SELLOAlto">SELLOAlto</a></td><td>Electronic seal – High Level APE</td><td>Smartcard/Token</td>
  </tr>
  <tr>
    <td><a href="#SelloOrganoAltoNubeQ">SelloOrganoAltoNubeQ</a></td><td>Electronic seal – High Level APE</td><td>Cloud-QSCD</td>
  </tr>
</table> 
</br> 
</html>


<div id="PFSoftAFCiudadano" style="padding-top: 60px;"><h2>PFSoftAFCiudadano<h2></div>

Certificate of a natural person issued on a cryptographic container in P12 format and intended for authentication and electronic signature.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>0</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the software element.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>PFSoftAFCiudadano</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_description</td>
<td></td>
<td>The cardholder document description</td>
<td>No</td>
</tr>
<tr>
<td>id_document_issuer</td>
<td></td>
<td>The cardholder document issuer</td>
<td>No</td>
</tr>
<tr>
<td>id_document_number</td>
<td></td>
<td>The cardholder document number</td>
<td>No</td>
</tr>
<tr>
<td>residence_address</td>
<td></td>
<td>The cardholder address of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_city</td>
<td></td>
<td>The cardholder city of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_province</td>
<td></td>
<td>The cardholder province of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence</td>
<td></td>
<td>The cardholder country of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_postal_code</td>
<td></td>
<td>The cardholder postal code of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_state</td>
<td></td>
<td>The cardholder state of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_district</td>
<td></td>
<td>The cardholder district of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_canton</td>
<td></td>
<td>The cardholder canton code of residence</td>
<td>No</td>
</tr>
</table>


<div id="PFqscdCiudadano" style="padding-top: 60px;"><h2>PFqscdCiudadano<h2></div>

Certificate of a natural person issued on a smartcard or a cryptographic token and intended for authentication and electronic signature.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>1</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the smartcard element.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>PFqscdCiudadano</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>smartcard_sn</td>
<td></td>
<td>The smartcard serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_description</td>
<td></td>
<td>The cardholder document description</td>
<td>No</td>
</tr>
<tr>
<td>id_document_issuer</td>
<td></td>
<td>The cardholder document issuer</td>
<td>No</td>
</tr>
<tr>
<td>id_document_number</td>
<td></td>
<td>The cardholder document number</td>
<td>No</td>
</tr>
<tr>
<td>residence_address</td>
<td></td>
<td>The cardholder address of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_city</td>
<td></td>
<td>The cardholder city of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_province</td>
<td></td>
<td>The cardholder province of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence</td>
<td></td>
<td>The cardholder country of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_postal_code</td>
<td></td>
<td>The cardholder postal code of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_state</td>
<td></td>
<td>The cardholder state of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_district</td>
<td></td>
<td>The cardholder district of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_canton</td>
<td></td>
<td>The cardholder canton code of residence</td>
<td>No</td>
</tr>
</table>


<div id="PFnubeAFCiudadano" style="padding-top: 60px;"><h2>PFnubeAFCiudadano<h2></div>

Certificate of a natural person issued in the centralized custody system of Uanataca certificates and intended for authentication and electronic signature.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>2</td>
<td>
    Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
    This profile only allows the cloud element.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>PFnubeAFCiudadano</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_description</td>
<td></td>
<td>The cardholder document description</td>
<td>No</td>
</tr>
<tr>
<td>id_document_issuer</td>
<td></td>
<td>The cardholder document issuer</td>
<td>No</td>
</tr>
<tr>
<td>id_document_number</td>
<td></td>
<td>The cardholder document number</td>
<td>No</td>
</tr>
<tr>
<td>residence_address</td>
<td></td>
<td>The cardholder address of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_city</td>
<td></td>
<td>The cardholder city of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_province</td>
<td></td>
<td>The cardholder province of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence</td>
<td></td>
<td>The cardholder country of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_postal_code</td>
<td></td>
<td>The cardholder postal code of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_state</td>
<td></td>
<td>The cardholder state of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_district</td>
<td></td>
<td>The cardholder district of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_canton</td>
<td></td>
<td>The cardholder canton code of residence</td>
<td>No</td>
</table>


<div id="PFnubeQAFCiudadano" style="padding-top: 60px;"><h2>PFnubeQAFCiudadano<h2></div>

Certificate of a natural person issued in the centralized custody system of Uanataca certificates and intended for authentication and qualified electronic signature.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>2</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the cloud element.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>PFnubeQAFCiudadano</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_description</td>
<td></td>
<td>The cardholder document description</td>
<td>No</td>
</tr>
<tr>
<td>id_document_issuer</td>
<td></td>
<td>The cardholder document issuer</td>
<td>No</td>
</tr>
<tr>
<td>id_document_number</td>
<td></td>
<td>The cardholder document number</td>
<td>No</td>
</tr>
<tr>
<td>residence_address</td>
<td></td>
<td>The cardholder address of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_city</td>
<td></td>
<td>The cardholder city of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_province</td>
<td></td>
<td>The cardholder province of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence</td>
<td></td>
<td>The cardholder country of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_postal_code</td>
<td></td>
<td>The cardholder postal code of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_state</td>
<td></td>
<td>The cardholder state of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_district</td>
<td></td>
<td>The cardholder district of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_canton</td>
<td></td>
<td>The cardholder canton code of residence</td>
<td>No</td>
</tr>
</table>

<div id="PFSoftAFEmpresa" style="padding-top: 60px;"><h2>PFSoftAFEmpresa<h2></div>

Certificate of a natural person belonging to an organization or company issued on cryptographic token in P12 format and intended for authentication and electronic signature.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>0</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the software element.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>PFSoftAFEmpresa</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>title</td>
<td></td>
<td>The cardholder professional title</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td></td>
<td>The cardholder first organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>organization_rol</td>
<td></td>
<td></td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>responsible_serial</td>
<td></td>
<td>The responsible serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_tax_number</td>
<td></td>
<td>The organization tax number</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_description</td>
<td></td>
<td>The cardholder document description</td>
<td>No</td>
</tr>
<tr>
<td>id_document_issuer</td>
<td></td>
<td>The cardholder document issuer</td>
<td>No</td>
</tr>
<tr>
<td>residence_address</td>
<td></td>
<td>The cardholder address of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_city</td>
<td></td>
<td>The cardholder city of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_province</td>
<td></td>
<td>The cardholder province of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence</td>
<td></td>
<td>The cardholder country of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_postal_code</td>
<td></td>
<td>The cardholder postal code of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_state</td>
<td></td>
<td>The cardholder state of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_district</td>
<td></td>
<td>The cardholder district of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_canton</td>
<td></td>
<td>The cardholder canton code of residence</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
<tr>
<td>organization_state</td>
<td></td>
<td>The organization state</td>
<td>No</td>
</tr>
<tr>
<td>organization_url</td>
<td></td>
<td>The organization web url</td>
<td>No</td>
</tr>
</table>


<div id="PFqscdEmpresa" style="padding-top: 60px;"><h2>PFqscdEmpresa<h2></div>

Certificate of a natural person belonging to an organization or company issued on a smartcard or cryptographic token and intended for authentication and eltronic signature.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>1</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the smartcard element.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>PFqscdEmpresa</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>smartcard_sn</td>
<td></td>
<td>The smartcard serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>title</td>
<td></td>
<td>The cardholder professional title</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td></td>
<td>The cardholder first organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>organization_rol</td>
<td></td>
<td></td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>responsible_serial</td>
<td></td>
<td>The responsible serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_tax_number</td>
<td></td>
<td>The organization tax number</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_description</td>
<td></td>
<td>The cardholder document description</td>
<td>No</td>
</tr>
<tr>
<td>id_document_issuer</td>
<td></td>
<td>The cardholder document issuer</td>
<td>No</td>
</tr>
<tr>
<td>residence_address</td>
<td></td>
<td>The cardholder address of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_city</td>
<td></td>
<td>The cardholder city of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_province</td>
<td></td>
<td>The cardholder province of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence</td>
<td></td>
<td>The cardholder country of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_postal_code</td>
<td></td>
<td>The cardholder postal code of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_state</td>
<td></td>
<td>The cardholder state of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_district</td>
<td></td>
<td>The cardholder district of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_canton</td>
<td></td>
<td>The cardholder canton code of residence</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
<tr>
<td>organization_state</td>
<td></td>
<td>The organization state</td>
<td>No</td>
</tr>
<tr>
<td>organization_url</td>
<td></td>
<td>The organization web url</td>
<td>No</td>
</tr>
</table>

<div id="PFnubeAFEmpresa" style="padding-top: 60px;"><h2>PFnubeAFEmpresa<h2></div>

Certificate of a natural person belonging to an organization or company issued in the centralized custody system of Uanataca certificates and intended for authentication and electronic signature.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>2</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the cloud element.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>PFnubeAFEmpresa</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>title</td>
<td></td>
<td>The cardholder professional title</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td></td>
<td>The cardholder first organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>organization_rol</td>
<td></td>
<td></td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>responsible_serial</td>
<td></td>
<td>The responsible serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_tax_number</td>
<td></td>
<td>The organization tax number</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_description</td>
<td></td>
<td>The cardholder document description</td>
<td>No</td>
</tr>
<tr>
<td>id_document_issuer</td>
<td></td>
<td>The cardholder document issuer</td>
<td>No</td>
</tr>
<tr>
<td>residence_address</td>
<td></td>
<td>The cardholder address of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_city</td>
<td></td>
<td>The cardholder city of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_province</td>
<td></td>
<td>The cardholder province of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence</td>
<td></td>
<td>The cardholder country of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_postal_code</td>
<td></td>
<td>The cardholder postal code of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_state</td>
<td></td>
<td>The cardholder state of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_district</td>
<td></td>
<td>The cardholder district of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_canton</td>
<td></td>
<td>The cardholder canton code of residence</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
<tr>
<td>organization_state</td>
<td></td>
<td>The organization state</td>
<td>No</td>
</tr>
<tr>
<td>organization_url</td>
<td></td>
<td>The organization web url</td>
<td>No</td>
</tr>
</table>

<div id="PFnubeQAFEmpresa" style="padding-top: 60px;"><h2>PFnubeQAFEmpresa<h2></div>

Certificate of a natural person belonging to an organization or company issued in the centralized custody system of Uanataca certificates and intended for authentication and qualified eltronic signature.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>2</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the cloud element.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>PFnubeQAFEmpresa</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>title</td>
<td></td>
<td>The cardholder professional title</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td></td>
<td>The cardholder first organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>organization_rol</td>
<td></td>
<td></td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>responsible_serial</td>
<td></td>
<td>The responsible serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_tax_number</td>
<td></td>
<td>The organization tax number</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_description</td>
<td></td>
<td>The cardholder document description</td>
<td>No</td>
</tr>
<tr>
<td>id_document_issuer</td>
<td></td>
<td>The cardholder document issuer</td>
<td>No</td>
</tr>
<tr>
<td>residence_address</td>
<td></td>
<td>The cardholder address of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_city</td>
<td></td>
<td>The cardholder city of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_province</td>
<td></td>
<td>The cardholder province of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence</td>
<td></td>
<td>The cardholder country of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_postal_code</td>
<td></td>
<td>The cardholder postal code of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_state</td>
<td></td>
<td>The cardholder state of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_district</td>
<td></td>
<td>The cardholder district of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_canton</td>
<td></td>
<td>The cardholder canton code of residence</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
<tr>
<td>organization_state</td>
<td></td>
<td>The organization state</td>
<td>No</td>
</tr>
<tr>
<td>organization_url</td>
<td></td>
<td>The organization web url</td>
<td>No</td>
</tr>
</table>

<div id="PFSoftAFColegiado" style="padding-top: 60px;"><h2>PFSoftAFColegiado<h2></div>

Certificate of a registered individual, for authentication and electronic signature issued in cryptographic container format P12.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>0</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the software element.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>PFSoftAFColegiado</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>title</td>
<td></td>
<td>The cardholder professional title</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td></td>
<td>The cardholder first organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>professional_id_number</td>
<td></td>
<td></td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_tax_number</td>
<td></td>
<td>The organization tax number</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_serial</td>
<td></td>
<td>The responsible serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_description</td>
<td></td>
<td>The cardholder document description</td>
<td>No</td>
</tr>
<tr>
<td>id_document_issuer</td>
<td></td>
<td>The cardholder document issuer</td>
<td>No</td>
</tr>
<tr>
<td>residence_address</td>
<td></td>
<td>The cardholder address of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_city</td>
<td></td>
<td>The cardholder city of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_province</td>
<td></td>
<td>The cardholder province of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence</td>
<td></td>
<td>The cardholder country of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_postal_code</td>
<td></td>
<td>The cardholder postal code of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_state</td>
<td></td>
<td>The cardholder state of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_district</td>
<td></td>
<td>The cardholder district of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_canton</td>
<td></td>
<td>The cardholder canton code of residence</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
<tr>
<td>organization_state</td>
<td></td>
<td>The organization state</td>
<td>No</td>
</tr>
<tr>
<td>organization_url</td>
<td></td>
<td>The organization web url</td>
<td>No</td>
</tr>
</table>

<div id="PFqscdColegiado" style="padding-top: 60px;"><h2>PFqscdColegiado<h2></div>

Certificate of a registered individual, for authentication and electronic signature issued on a cryptographic card or token.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>1</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the smartcard element.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>PFqscdColegiado</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>smartcard_sn</td>
<td></td>
<td>The smartcard serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>title</td>
<td></td>
<td>The cardholder professional title</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td></td>
<td>The cardholder first organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>professional_id_number</td>
<td></td>
<td></td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_tax_number</td>
<td></td>
<td>The organization tax number</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_serial</td>
<td></td>
<td>The responsible serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_description</td>
<td></td>
<td>The cardholder document description</td>
<td>No</td>
</tr>
<tr>
<td>id_document_issuer</td>
<td></td>
<td>The cardholder document issuer</td>
<td>No</td>
</tr>
<tr>
<td>residence_address</td>
<td></td>
<td>The cardholder address of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_city</td>
<td></td>
<td>The cardholder city of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_province</td>
<td></td>
<td>The cardholder province of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence</td>
<td></td>
<td>The cardholder country of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_postal_code</td>
<td></td>
<td>The cardholder postal code of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_state</td>
<td></td>
<td>The cardholder state of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_district</td>
<td></td>
<td>The cardholder district of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_canton</td>
<td></td>
<td>The cardholder canton code of residence</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
<tr>
<td>organization_state</td>
<td></td>
<td>The organization state</td>
<td>No</td>
</tr>
<tr>
<td>organization_url</td>
<td></td>
<td>The organization web url</td>
<td>No</td>
</tr>
</table>

<div id="PFnubeAFColegiado" style="padding-top: 60px;"><h2>PFnubeAFColegiado<h2></div>

Certificate of a registered individual, for authentication and electronic signature issued in the centralized custody system of Uanataca certificates.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>2</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the cloud element.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>PFnubeAFColegiado</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>title</td>
<td></td>
<td>The cardholder professional title</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td></td>
<td>The cardholder first organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>professional_id_number</td>
<td></td>
<td></td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_tax_number</td>
<td></td>
<td>The organization tax number</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_serial</td>
<td></td>
<td>The responsible serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_description</td>
<td></td>
<td>The cardholder document description</td>
<td>No</td>
</tr>
<tr>
<td>id_document_issuer</td>
<td></td>
<td>The cardholder document issuer</td>
<td>No</td>
</tr>
<tr>
<td>residence_address</td>
<td></td>
<td>The cardholder address of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_city</td>
<td></td>
<td>The cardholder city of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_province</td>
<td></td>
<td>The cardholder province of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence</td>
<td></td>
<td>The cardholder country of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_postal_code</td>
<td></td>
<td>The cardholder postal code of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_state</td>
<td></td>
<td>The cardholder state of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_district</td>
<td></td>
<td>The cardholder district of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_canton</td>
<td></td>
<td>The cardholder canton code of residence</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
<tr>
<td>organization_state</td>
<td></td>
<td>The organization state</td>
<td>No</td>
</tr>
<tr>
<td>organization_url</td>
<td></td>
<td>The organization web url</td>
<td>No</td>
</tr>
</table>

<div id="PFnubeQAFColegiado" style="padding-top: 60px;"><h2>PFnubeQAFColegiado<h2></div>

Certificate of a registered individual, for authentication and qualified electronic signature issued in the centralized custody system of Uanataca certificates.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>2</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the cloud element.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>PFnubeQAFColegiado</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>title</td>
<td></td>
<td>The cardholder professional title</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td></td>
<td>The cardholder first organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>professional_id_number</td>
<td></td>
<td></td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_tax_number</td>
<td></td>
<td>The organization tax number</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_serial</td>
<td></td>
<td>The responsible serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_description</td>
<td></td>
<td>The cardholder document description</td>
<td>No</td>
</tr>
<tr>
<td>id_document_issuer</td>
<td></td>
<td>The cardholder document issuer</td>
<td>No</td>
</tr>
<tr>
<td>residence_address</td>
<td></td>
<td>The cardholder address of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_city</td>
<td></td>
<td>The cardholder city of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_province</td>
<td></td>
<td>The cardholder province of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence</td>
<td></td>
<td>The cardholder country of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_postal_code</td>
<td></td>
<td>The cardholder postal code of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_state</td>
<td></td>
<td>The cardholder state of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_district</td>
<td></td>
<td>The cardholder district of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_canton</td>
<td></td>
<td>The cardholder canton code of residence</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
<tr>
<td>organization_state</td>
<td></td>
<td>The organization state</td>
<td>No</td>
</tr>
<tr>
<td>organization_url</td>
<td></td>
<td>The organization web url</td>
<td>No</td>
</tr>
</table>

<div id="REPsoft" style="padding-top: 60px;"><h2>REPsoft<h2></div>

Certificate of legal entity representative, suitable for the relationship between Spanish or European companies, for authentication and electronic signature issued in cryptographic container format P12.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_identifier</td>
<td></td>
<td>The organization identifier</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>subscriber_responsible_serial</td>
<td></td>
<td>The organization representative document number</td>
<td>No</td>
</tr>
<tr>
<td>empowerment</td>
<td></td>
<td>The cardholder legal representation level</td>
<td>No</td>
</tr>
<tr>
<td>representation</td>
<td></td>
<td>The cardholder legal representation document</td>
<td>No</td>
</tr>
<tr>
<td>circumstances</td>
<td></td>
<td>The cardholder legal conditions</td>
<td>No</td>
</tr>
<tr>
<td>limit</td>
<td></td>
<td>The cardholder ristrict of representation</td>
<td>No</td>
</tr>
<tr>
<td>registration</td>
<td></td>
<td>The cardholder representation registry data</td>
<td>No</td>
</tr>
<tr>
<td>description</td>
<td></td>
<td>A description</td>
<td>No</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>REPsoft</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>secure_element</td>
<td>[0, 2]</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>citizen_tax_number</td>
<td></td>
<td>The citizen tax number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>sex</td>
<td></td>
<td>The cardholder sex</td>
<td>No</td>
</tr>
<tr>
<td>birth_city</td>
<td></td>
<td>The cardholder city of residence</td>
<td>No</td>
</tr>
<tr>
<td>birth_province</td>
<td></td>
<td>The cardholder province of residence</td>
<td>No</td>
</tr>
<tr>
<td>birth_state</td>
<td></td>
<td>The cardholder state of residence</td>
<td>No</td>
</tr>
<tr>
<td>birth_district</td>
<td></td>
<td>The cardholder district of residence</td>
<td>No</td>
</tr>
<tr>
<td>birth_canton</td>
<td></td>
<td>The cardholder canton code of residence</td>
<td>No</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_description</td>
<td></td>
<td>The cardholder document description</td>
<td>No</td>
</tr>
<tr>
<td>id_document_issuer</td>
<td></td>
<td>The cardholder document issuer</td>
<td>No</td>
</tr>
<tr>
<td>title</td>
<td></td>
<td>The cardholder professional title</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td></td>
<td>The cardholder first organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>organizational_unit_2</td>
<td></td>
<td>The cardholder second organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>fix_phone_number</td>
<td></td>
<td></td>
<td>No</td>
</tr>
<tr>
<td>residence_address</td>
<td></td>
<td>The cardholder address of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_city</td>
<td></td>
<td>The cardholder city of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_province</td>
<td></td>
<td>The cardholder province of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence</td>
<td></td>
<td>The cardholder country of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_postal_code</td>
<td></td>
<td>The cardholder postal code of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_state</td>
<td></td>
<td>The cardholder state of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_district</td>
<td></td>
<td>The cardholder district of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_canton</td>
<td></td>
<td>The cardholder canton code of residence</td>
<td>No</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
<tr>
<td>organization_state</td>
<td></td>
<td>The organization state</td>
<td>No</td>
</tr>
<tr>
<td>organization_url</td>
<td></td>
<td>The organization web url</td>
<td>No</td>
</tr>
</table>

<div id="REPqscd" style="padding-top: 60px;"><h2>REPqscd<h2></div>

Certificate of legal entity representative, suitable for the relationship between Spanish or European companies, for the electronic signature issued on a cryptographic card or token.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_identifier</td>
<td></td>
<td>The organization identifier</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>subscriber_responsible_serial</td>
<td></td>
<td>The organization representative document number</td>
<td>No</td>
</tr>
<tr>
<td>empowerment</td>
<td></td>
<td>The cardholder legal representation level</td>
<td>No</td>
</tr>
<tr>
<td>representation</td>
<td></td>
<td>The cardholder legal representation document</td>
<td>No</td>
</tr>
<tr>
<td>circumstances</td>
<td></td>
<td>The cardholder legal conditions</td>
<td>No</td>
</tr>
<tr>
<td>limit</td>
<td></td>
<td>The cardholder ristrict of representation</td>
<td>No</td>
</tr>
<tr>
<td>registration</td>
<td></td>
<td>The cardholder representation registry data</td>
<td>No</td>
</tr>
<tr>
<td>description</td>
<td></td>
<td>A description</td>
<td>No</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>REPqscd</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>secure_element</td>
<td>1</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the smartcard element.
</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>citizen_tax_number</td>
<td></td>
<td>The citizen tax number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>sex</td>
<td></td>
<td>The cardholder sex</td>
<td>No</td>
</tr>
<tr>
<td>birth_city</td>
<td></td>
<td>The cardholder city of residence</td>
<td>No</td>
</tr>
<tr>
<td>birth_province</td>
<td></td>
<td>The cardholder province of residence</td>
<td>No</td>
</tr>
<tr>
<td>birth_state</td>
<td></td>
<td>The cardholder state of residence</td>
<td>No</td>
</tr>
<tr>
<td>birth_district</td>
<td></td>
<td>The cardholder district of residence</td>
<td>No</td>
</tr>
<tr>
<td>birth_canton</td>
<td></td>
<td>The cardholder canton code of residence</td>
<td>No</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_description</td>
<td></td>
<td>The cardholder document description</td>
<td>No</td>
</tr>
<tr>
<td>id_document_issuer</td>
<td></td>
<td>The cardholder document issuer</td>
<td>No</td>
</tr>
<tr>
<td>title</td>
<td></td>
<td>The cardholder professional title</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td></td>
<td>The cardholder first organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>organizational_unit_2</td>
<td></td>
<td>The cardholder second organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>fix_phone_number</td>
<td></td>
<td></td>
<td>No</td>
</tr>
<tr>
<td>residence_address</td>
<td></td>
<td>The cardholder address of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_city</td>
<td></td>
<td>The cardholder city of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_province</td>
<td></td>
<td>The cardholder province of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence</td>
<td></td>
<td>The cardholder country of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_postal_code</td>
<td></td>
<td>The cardholder postal code of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_state</td>
<td></td>
<td>The cardholder state of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_district</td>
<td></td>
<td>The cardholder district of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_canton</td>
<td></td>
<td>The cardholder canton code of residence</td>
<td>No</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
<tr>
<td>organization_state</td>
<td></td>
<td>The organization state</td>
<td>No</td>
</tr>
<tr>
<td>organization_url</td>
<td></td>
<td>The organization web url</td>
<td>No</td>
</tr>
</table>

<div id="REPnubeQ" style="padding-top: 60px;"><h2>REPnubeQ<h2></div>

Certificate of representative of legal entity, suitable for the relationship between Spanish or European companies, for the qualified electronic signature, and issued in the centralized custody system of Uanataca certificates.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_identifier</td>
<td></td>
<td>The organization identifier</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>subscriber_responsible_serial</td>
<td></td>
<td>The organization representative document number</td>
<td>No</td>
</tr>
<tr>
<td>empowerment</td>
<td></td>
<td>The cardholder legal representation level</td>
<td>No</td>
</tr>
<tr>
<td>representation</td>
<td></td>
<td>The cardholder legal representation document</td>
<td>No</td>
</tr>
<tr>
<td>circumstances</td>
<td></td>
<td>The cardholder legal conditions</td>
<td>No</td>
</tr>
<tr>
<td>limit</td>
<td></td>
<td>The cardholder ristrict of representation</td>
<td>No</td>
</tr>
<tr>
<td>registration</td>
<td></td>
<td>The cardholder representation registry data</td>
<td>No</td>
</tr>
<tr>
<td>description</td>
<td></td>
<td>A description</td>
<td>No</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>REPnubeQ</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>secure_element</td>
<td>2</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the cloud element.
</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>citizen_tax_number</td>
<td></td>
<td>The citizen tax number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>sex</td>
<td></td>
<td>The cardholder sex</td>
<td>No</td>
</tr>
<tr>
<td>birth_city</td>
<td></td>
<td>The cardholder city of residence</td>
<td>No</td>
</tr>
<tr>
<td>birth_province</td>
<td></td>
<td>The cardholder province of residence</td>
<td>No</td>
</tr>
<tr>
<td>birth_state</td>
<td></td>
<td>The cardholder state of residence</td>
<td>No</td>
</tr>
<tr>
<td>birth_district</td>
<td></td>
<td>The cardholder district of residence</td>
<td>No</td>
</tr>
<tr>
<td>birth_canton</td>
<td></td>
<td>The cardholder canton code of residence</td>
<td>No</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_description</td>
<td></td>
<td>The cardholder document description</td>
<td>No</td>
</tr>
<tr>
<td>id_document_issuer</td>
<td></td>
<td>The cardholder document issuer</td>
<td>No</td>
</tr>
<tr>
<td>title</td>
<td></td>
<td>The cardholder professional title</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td></td>
<td>The cardholder first organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>organizational_unit_2</td>
<td></td>
<td>The cardholder second organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>fix_phone_number</td>
<td></td>
<td></td>
<td>No</td>
</tr>
<tr>
<td>residence_address</td>
<td></td>
<td>The cardholder address of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_city</td>
<td></td>
<td>The cardholder city of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_province</td>
<td></td>
<td>The cardholder province of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence</td>
<td></td>
<td>The cardholder country of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_postal_code</td>
<td></td>
<td>The cardholder postal code of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_state</td>
<td></td>
<td>The cardholder state of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_district</td>
<td></td>
<td>The cardholder district of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_canton</td>
<td></td>
<td>The cardholder canton code of residence</td>
<td>No</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
<tr>
<td>organization_state</td>
<td></td>
<td>The organization state</td>
<td>No</td>
</tr>
<tr>
<td>organization_url</td>
<td></td>
<td>The organization web url</td>
<td>No</td>
</tr>
</table>

<div id="REPPJsoft" style="padding-top: 60px;"><h2>REPPJsoft<h2></div>

Certificate of the legal entity representative issued on a cryptographic container in P12 format and intended for authentication and electronic signature.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>0</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>REPPJsoft</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>title</td>
<td></td>
<td>The cardholder professional title</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td></td>
<td>The cardholder first organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_identifier</td>
<td></td>
<td>The organization identifier</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>description</td>
<td></td>
<td>One of the following options:<br>
- Registro Mercantil: Reg: XXX /Hoja: XXX /Tomo:XXX /Sección:XXX /Libro:XXX /Folio:XXX /Fecha: dd-mm-aaaa /Inscripción: XXX.<br>
- Poder Notarial: Notario: Nombre Apellido1 Apellido2 /Núm Protocolo: XXX /Fecha Otorgamiento: dd-mm-aaaa.<br>
- Boletines Oficiales: Boletín: XXX /Fecha: dd-mm-aaaa /Numero resolución: XXX.<br>
- Otros: Documento: XXX /Fecha: dd-mm-aaaa
</td>
<td>Yes</td>
</tr>
<tr>
<td>empowerment</td>
<td></td>
<td>The cardholder legal representation level</td>
<td>Yes</td>
</tr>
<tr>
<td>representation</td>
<td></td>
<td>The cardholder legal representation document</td>
<td>Yes</td>
</tr>
<tr>
<td>subscriber_responsible_serial</td>
<td></td>
<td>The organization representative document number</td>
<td>No</td>
</tr>
<tr>
<td>circumstances</td>
<td></td>
<td>The cardholder legal conditions</td>
<td>No</td>
</tr>
<tr>
<td>limit</td>
<td></td>
<td>The cardholder ristrict of representation</td>
<td>No</td>
</tr>
<tr>
<td>registration</td>
<td></td>
<td>The cardholder representation registry data</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
<tr>
<td>organization_url</td>
<td></td>
<td>The organization web url</td>
<td>No</td>
</tr>
</table>

<div id="REPPJnube" style="padding-top: 60px;"><h2>REPPJnube<h2></div>

Certificate of the legal entity representative issued in the centralized custody system of Uanataca certificates and intended for authentication and electronic signature.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>2</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the cloud element.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>REPPJnube</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>title</td>
<td></td>
<td>The cardholder professional title</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td></td>
<td>The cardholder first organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_identifier</td>
<td></td>
<td>The organization identifier</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>description</td>
<td></td>
<td>One of the following options:<br>
- Registro Mercantil: Reg: XXX /Hoja: XXX /Tomo:XXX /Sección:XXX /Libro:XXX /Folio:XXX /Fecha: dd-mm-aaaa /Inscripción: XXX.<br>
- Poder Notarial: Notario: Nombre Apellido1 Apellido2 /Núm Protocolo: XXX /Fecha Otorgamiento: dd-mm-aaaa.<br>
- Boletines Oficiales: Boletín: XXX /Fecha: dd-mm-aaaa /Numero resolución: XXX.<br>
- Otros: Documento: XXX /Fecha: dd-mm-aaaa
</td>
<td>Yes</td>
</tr>
<tr>
<td>empowerment</td>
<td></td>
<td>The cardholder legal representation level</td>
<td>Yes</td>
</tr>
<tr>
<td>representation</td>
<td></td>
<td>The cardholder legal representation document</td>
<td>Yes</td>
</tr>
<tr>
<td>subscriber_responsible_serial</td>
<td></td>
<td>The organization representative document number</td>
<td>No</td>
</tr>
<tr>
<td>circumstances</td>
<td></td>
<td>The cardholder legal conditions</td>
<td>No</td>
</tr>
<tr>
<td>limit</td>
<td></td>
<td>The cardholder ristrict of representation</td>
<td>No</td>
</tr>
<tr>
<td>registration</td>
<td></td>
<td>The cardholder representation registry data</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
<tr>
<td>organization_url</td>
<td></td>
<td>The organization web url</td>
<td>No</td>
</tr>
</table>

<div id="REPPJqscd" style="padding-top: 60px;"><h2>REPPJqscd<h2></div>

Certificate of the legal entity representative issued on a smartcard or on a cryptographic token and intended for authentication and electronic signature.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>1</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the smartcard element.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>REPPJqscd</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>smartcard_sn</td>
<td></td>
<td>The smartcard serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>title</td>
<td></td>
<td>The cardholder professional title</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td></td>
<td>The cardholder first organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_identifier</td>
<td></td>
<td>The organization identifier</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>description</td>
<td></td>
<td>One of the following options:<br>
- Registro Mercantil: Reg: XXX /Hoja: XXX /Tomo:XXX /Sección:XXX /Libro:XXX /Folio:XXX /Fecha: dd-mm-aaaa /Inscripción: XXX.<br>
- Poder Notarial: Notario: Nombre Apellido1 Apellido2 /Núm Protocolo: XXX /Fecha Otorgamiento: dd-mm-aaaa.<br>
- Boletines Oficiales: Boletín: XXX /Fecha: dd-mm-aaaa /Numero resolución: XXX.<br>
- Otros: Documento: XXX /Fecha: dd-mm-aaaa
</td>
<td>Yes</td>
</tr>
<tr>
<td>empowerment</td>
<td></td>
<td>The cardholder legal representation level</td>
<td>Yes</td>
</tr>
<tr>
<td>representation</td>
<td></td>
<td>The cardholder legal representation document</td>
<td>Yes</td>
</tr>
<tr>
<td>subscriber_responsible_serial</td>
<td></td>
<td>The organization representative document number</td>
<td>No</td>
</tr>
<tr>
<td>circumstances</td>
<td></td>
<td>The cardholder legal conditions</td>
<td>No</td>
</tr>
<tr>
<td>limit</td>
<td></td>
<td>The cardholder ristrict of representation</td>
<td>No</td>
</tr>
<tr>
<td>registration</td>
<td></td>
<td>The cardholder representation registry data</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
<tr>
<td>organization_url</td>
<td></td>
<td>The organization web url</td>
<td>No</td>
</tr>
</table>

<div id="REPPJnubeQ" style="padding-top: 60px;"><h2>REPPJnubeQ<h2></div>

Certificate of representative of legal entity, suitable to interact with Spanish Public Administrations, issued in the centralized custody system of Uanataca certificates and intended for authentication and qualified electronic signature.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>2</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the cloud element.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>REPPJnubeQ</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>title</td>
<td></td>
<td>The cardholder professional title</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td></td>
<td>The cardholder first organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_identifier</td>
<td></td>
<td>The organization identifier</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>description</td>
<td></td>
<td>One of the following options:<br>
- Registro Mercantil: Reg: XXX /Hoja: XXX /Tomo:XXX /Sección:XXX /Libro:XXX /Folio:XXX /Fecha: dd-mm-aaaa /Inscripción: XXX.<br>
- Poder Notarial: Notario: Nombre Apellido1 Apellido2 /Núm Protocolo: XXX /Fecha Otorgamiento: dd-mm-aaaa.<br>
- Boletines Oficiales: Boletín: XXX /Fecha: dd-mm-aaaa /Numero resolución: XXX.<br>
- Otros: Documento: XXX /Fecha: dd-mm-aaaa
</td>
<td>Yes</td>
</tr>
<tr>
<td>empowerment</td>
<td></td>
<td>The cardholder legal representation level</td>
<td>Yes</td>
</tr>
<tr>
<td>representation</td>
<td></td>
<td>The cardholder legal representation document</td>
<td>Yes</td>
</tr>
<tr>
<td>subscriber_responsible_serial</td>
<td></td>
<td>The organization representative document number</td>
<td>No</td>
</tr>
<tr>
<td>circumstances</td>
<td></td>
<td>The cardholder legal conditions</td>
<td>No</td>
</tr>
<tr>
<td>limit</td>
<td></td>
<td>The cardholder ristrict of representation</td>
<td>No</td>
</tr>
<tr>
<td>registration</td>
<td></td>
<td>The cardholder representation registry data</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
<tr>
<td>organization_url</td>
<td></td>
<td>The organization web url</td>
<td>No</td>
</tr>
</table>

<div id="EMPUBsoft" style="padding-top: 60px;"><h2>EMPUBsoft<h2></div>

Certificate of public employee of a Spanish Public Administration, issued on a cryptographic container in P12 format and intended for authentication and electronic signature.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>[0, 2]</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>EMPUBsoft</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td>CERTIFICADO ELECTRONICO DE EMPLEADO PUBLICO</td>
<td>The cardholder first organizational unit</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_2</td>
<td></td>
<td>The cardholder second organizational unit</td>
<td>Yes</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_tax_number</td>
<td></td>
<td>The organization tax number</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_serial</td>
<td></td>
<td>The responsible serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_position</td>
<td></td>
<td>The responsible position</td>
<td>No</td>
</tr>
<tr>
<td>organizational_unit_3</td>
<td></td>
<td>The cardholder third organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>organizational_unit_4</td>
<td></td>
<td>The cardholder fourth organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
<tr>
<td>organization_url</td>
<td></td>
<td>The organization web url</td>
<td>No</td>
</tr>
</table>

<div id="EMPUBqscd" style="padding-top: 60px;"><h2>EMPUBqscd<h2></div>

Certificate of public employee of a Spanish Public Administration, issued on a smartcard or a cryptographic token and intended for electronic signature.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>1</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the smartcard element.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>EMPUBqscd</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>smartcard_sn</td>
<td></td>
<td>The smartcard serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td>CERTIFICADO ELECTRONICO DE EMPLEADO PUBLICO</td>
<td>The cardholder first organizational unit</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_2</td>
<td></td>
<td>The cardholder second organizational unit</td>
<td>Yes</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_tax_number</td>
<td></td>
<td>The organization tax number</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_serial</td>
<td></td>
<td>The responsible serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_position</td>
<td></td>
<td>The responsible position</td>
<td>No</td>
</tr>
<tr>
<td>organizational_unit_3</td>
<td></td>
<td>The cardholder third organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>organizational_unit_4</td>
<td></td>
<td>The cardholder fourth organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
<tr>
<td>organization_url</td>
<td></td>
<td>The organization web url</td>
<td>No</td>
</tr>
</table>

<div id="EMPUBnubeQ" style="padding-top: 60px;"><h2>EMPUBnubeQ<h2></div>

Certificate of public employee of a Spanish Public Administration, issued in the centralized custody system of Uanataca certificates and intended for the qualified electronic signature.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>2</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the cloud element.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>EMPUBnubeQ</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td>CERTIFICADO ELECTRONICO DE EMPLEADO PUBLICO</td>
<td>The cardholder first organizational unit</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_2</td>
<td></td>
<td>The cardholder second organizational unit</td>
<td>Yes</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_tax_number</td>
<td></td>
<td>The organization tax number</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_serial</td>
<td></td>
<td>The responsible serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_position</td>
<td></td>
<td>The responsible position</td>
<td>No</td>
</tr>
<tr>
<td>organizational_unit_3</td>
<td></td>
<td>The cardholder third organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>organizational_unit_4</td>
<td></td>
<td>The cardholder fourth organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
<tr>
<td>organization_url</td>
<td></td>
<td>The organization web url</td>
<td>No</td>
</tr>
</table>

<div id="REPESPJsoft" style="padding-top: 60px;"><h2>REPESPJsoft<h2></div>

Certificate of representative of entity without legal license issued on a cryptographic container in P12 format and intended for authentication and electronic signature.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>0</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>REPESPJsoft</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>title</td>
<td></td>
<td>The cardholder professional title</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td></td>
<td>The cardholder first organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_identifier</td>
<td></td>
<td>The organization identifier</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>description</td>
<td></td>
<td>One of the following options:<br>
- Registro Mercantil: Reg: XXX /Hoja: XXX /Tomo:XXX /Sección:XXX /Libro:XXX /Folio:XXX /Fecha: dd-mm-aaaa /Inscripción: XXX.<br>
- Poder Notarial: Notario: Nombre Apellido1 Apellido2 /Núm Protocolo: XXX /Fecha Otorgamiento: dd-mm-aaaa.<br>
- Boletines Oficiales: Boletín: XXX /Fecha: dd-mm-aaaa /Numero resolución: XXX.<br>
- Otros: Documento: XXX /Fecha: dd-mm-aaaa
</td>
<td>Yes</td>
</tr>
<tr>
<td>empowerment</td>
<td></td>
<td>The cardholder legal representation level</td>
<td>Yes</td>
</tr>
<tr>
<td>representation</td>
<td></td>
<td>The cardholder legal representation document</td>
<td>Yes</td>
</tr>
<tr>
<td>subscriber_responsible_serial</td>
<td></td>
<td>The organization representative document number</td>
<td>No</td>
</tr>
<tr>
<td>circumstances</td>
<td></td>
<td>The cardholder legal conditions</td>
<td>No</td>
</tr>
<tr>
<td>limit</td>
<td></td>
<td>The cardholder ristrict of representation</td>
<td>No</td>
</tr>
<tr>
<td>registration</td>
<td></td>
<td>The cardholder representation registry data</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
<tr>
<td>organization_url</td>
<td></td>
<td>The organization web url</td>
<td>No</td>
</tr>
</table>

<div id="REPESPJnube" style="padding-top: 60px;"><h2>REPESPJnube<h2></div>

Certificate of representative of entity without legal license issued in the centralized custody system of Uanataca certificates and intended for authentication and electronic signature.</p>

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>                
<tr>
<td>secure_element</td>
<td>2</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the cloud element.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>REPESPJnube</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>title</td>
<td></td>
<td>The cardholder professional title</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td></td>
<td>The cardholder first organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_identifier</td>
<td></td>
<td>The organization identifier</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>description</td>
<td></td>
<td>One of the following options:<br>
- Registro Mercantil: Reg: XXX /Hoja: XXX /Tomo:XXX /Sección:XXX /Libro:XXX /Folio:XXX /Fecha: dd-mm-aaaa /Inscripción: XXX.<br>
- Poder Notarial: Notario: Nombre Apellido1 Apellido2 /Núm Protocolo: XXX /Fecha Otorgamiento: dd-mm-aaaa.<br>
- Boletines Oficiales: Boletín: XXX /Fecha: dd-mm-aaaa /Numero resolución: XXX.<br>
- Otros: Documento: XXX /Fecha: dd-mm-aaaa
</td>
<td>Yes</td>
</tr>
<tr>
<td>empowerment</td>
<td></td>
<td>The cardholder legal representation level</td>
<td>Yes</td>
</tr>
<tr>
<td>representation</td>
<td></td>
<td>The cardholder legal representation document</td>
<td>Yes</td>
</tr>
<tr>
<td>subscriber_responsible_serial</td>
<td></td>
<td>The organization representative document number</td>
<td>No</td>
</tr>
<tr>
<td>circumstances</td>
<td></td>
<td>The cardholder legal conditions</td>
<td>No</td>
</tr>
<tr>
<td>limit</td>
<td></td>
<td>The cardholder ristrict of representation</td>
<td>No</td>
</tr>
<tr>
<td>registration</td>
<td></td>
<td>The cardholder representation registry data</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
<tr>
<td>organization_url</td>
<td></td>
<td>The organization web url</td>
<td>No</td>
</tr>
</table>

<div id="REPESPJqscd" style="padding-top: 60px;"><h2>REPESPJqscd<h2></div>

Certificate of representative of entity without legal license issued on a smartcard or on a cryptographic token and intended for authentication and electronic signature.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>1</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the smartcard element.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>REPESPJqscd</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>smartcard_sn</td>
<td></td>
<td>The smartcard serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>title</td>
<td></td>
<td>The cardholder professional title</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td></td>
<td>The cardholder first organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_identifier</td>
<td></td>
<td>The organization identifier</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>description</td>
<td></td>
<td>One of the following options:<br>
- Registro Mercantil: Reg: XXX /Hoja: XXX /Tomo:XXX /Sección:XXX /Libro:XXX /Folio:XXX /Fecha: dd-mm-aaaa /Inscripción: XXX.<br>
- Poder Notarial: Notario: Nombre Apellido1 Apellido2 /Núm Protocolo: XXX /Fecha Otorgamiento: dd-mm-aaaa.<br>
- Boletines Oficiales: Boletín: XXX /Fecha: dd-mm-aaaa /Numero resolución: XXX.<br>
- Otros: Documento: XXX /Fecha: dd-mm-aaaa
</td>
<td>Yes</td>
</tr>
<tr>
<td>empowerment</td>
<td></td>
<td>The cardholder legal representation level</td>
<td>Yes</td>
</tr>
<tr>
<td>representation</td>
<td></td>
<td>The cardholder legal representation document</td>
<td>Yes</td>
</tr>
<tr>
<td>subscriber_responsible_serial</td>
<td></td>
<td>The organization representative document number</td>
<td>No</td>
</tr>
<tr>
<td>circumstances</td>
<td></td>
<td>The cardholder legal conditions</td>
<td>No</td>
</tr>
<tr>
<td>limit</td>
<td></td>
<td>The cardholder ristrict of representation</td>
<td>No</td>
</tr>
<tr>
<td>registration</td>
<td></td>
<td>The cardholder representation registry data</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
<tr>
<td>organization_url</td>
<td></td>
<td>The organization web url</td>
<td>No</td>
</tr>
</table>

<div id="REPESPJnubeQ" style="padding-top: 60px;"><h2>REPESPJnubeQ<h2></div>

Certificate of representative of entity without legal license, suitable to relate with the Spanish Public Administrations, issued in the centralized custody system of Uanataca certificates and intended for authentication and qualified electronic signature.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>2</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the cloud element.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>REPESPJnubeQ</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>title</td>
<td></td>
<td>The cardholder professional title</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td></td>
<td>The cardholder first organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_identifier</td>
<td></td>
<td>The organization identifier</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>description</td>
<td></td>
<td>One of the following options:<br>
- Registro Mercantil: Reg: XXX /Hoja: XXX /Tomo:XXX /Sección:XXX /Libro:XXX /Folio:XXX /Fecha: dd-mm-aaaa /Inscripción: XXX.<br>
- Poder Notarial: Notario: Nombre Apellido1 Apellido2 /Núm Protocolo: XXX /Fecha Otorgamiento: dd-mm-aaaa.<br>
- Boletines Oficiales: Boletín: XXX /Fecha: dd-mm-aaaa /Numero resolución: XXX.<br>
- Otros: Documento: XXX /Fecha: dd-mm-aaaa
</td>
<td>Yes</td>
</tr>
<tr>
<td>empowerment</td>
<td></td>
<td>The cardholder legal representation level</td>
<td>Yes</td>
</tr>
<tr>
<td>representation</td>
<td></td>
<td>The cardholder legal representation document</td>
<td>Yes</td>
</tr>
<tr>
<td>subscriber_responsible_serial</td>
<td></td>
<td>The organization representative document number</td>
<td>No</td>
</tr>
<tr>
<td>circumstances</td>
<td></td>
<td>The cardholder legal conditions</td>
<td>No</td>
</tr>
<tr>
<td>limit</td>
<td></td>
<td>The cardholder ristrict of representation</td>
<td>No</td>
</tr>
<tr>
<td>registration</td>
<td></td>
<td>The cardholder representation registry data</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
<tr>
<td>organization_url</td>
<td></td>
<td>The organization web url</td>
<td>No</td>
</tr>
</table>

<div id="SELLOPJnube" style="padding-top: 60px;"><h2>SELLOPJnube<h2></div>

Electronic seal certificate issued in the centralized custody system of Uanataca certificates and intended for electronic signature, usually in unassisted processes.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>2</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the cloud element.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>SELLOPJnube</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_identifier</td>
<td></td>
<td>The organization identifier</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td></td>
<td>The cardholder first organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>responsible_serial</td>
<td></td>
<td>The responsible serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_email</td>
<td></td>
<td>The responsible email</td>
<td>No</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>process_application</td>
<td></td>
<td>The application name that will use the certificate</td>
<td>Yes</td>
</tr>
<tr>
<td>description</td>
<td></td>
<td>A description</td>
<td>Yes</td>
</tr>
<tr>
<td>representation</td>
<td></td>
<td>The cardholder legal representation document</td>
<td>Yes</td>
</tr>
<tr>
<td>empowerment</td>
<td></td>
<td>The cardholder legal representation level</td>
<td>Yes</td>
</tr>
<tr>
<td>circumstances</td>
<td></td>
<td>The cardholder legal conditions</td>
<td>No</td>
</tr>
<tr>
<td>limit</td>
<td></td>
<td>The cardholder ristrict of representation</td>
<td>No</td>
</tr>
<tr>
<td>registration</td>
<td></td>
<td>The cardholder representation registry data</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
</table>

<div id="SELLOPJsoft" style="padding-top: 60px;"><h2>SELLOPJsoft<h2></div>

Electronic seal certificate issued in a cryptographic container in P12 format and intended for electronic signature, usually in unassisted processes.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>0</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>SELLOPJsoft</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_identifier</td>
<td></td>
<td>The organization identifier</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td></td>
<td>The cardholder first organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>responsible_serial</td>
<td></td>
<td>The responsible serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_email</td>
<td></td>
<td>The responsible email</td>
<td>No</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>process_application</td>
<td></td>
<td></td>
<td>Yes</td>
</tr>
<tr>
<td>description</td>
<td></td>
<td>A description</td>
<td>No</td>
</tr>
<tr>
<td>representation</td>
<td></td>
<td>The cardholder legal representation document</td>
<td>Yes</td>
</tr>
<tr>
<td>empowerment</td>
<td></td>
<td>The cardholder legal representation level</td>
<td>Yes</td>
</tr>
<tr>
<td>circumstances</td>
<td></td>
<td>The cardholder legal conditions</td>
<td>No</td>
</tr>
<tr>
<td>limit</td>
<td></td>
<td>The cardholder ristrict of representation</td>
<td>No</td>
</tr>
<tr>
<td>registration</td>
<td></td>
<td>The cardholder representation registry data</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
</table>

<div id="SELLOPJnubeQ" style="padding-top: 60px;"><h2>SELLOPJnubeQ<h2></div>

Electronic seal certificate issued in the centralized custody system of Uanataca certificates and intended for the qualified electronic signature, usually in unassisted processes.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>2</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the cloud one.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>SELLOPJnubeQ</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_identifier</td>
<td></td>
<td>The organization identifier</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td></td>
<td>The cardholder first organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>responsible_serial</td>
<td></td>
<td>The responsible serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_email</td>
<td></td>
<td>The responsible email</td>
<td>No</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>process_application</td>
<td></td>
<td></td>
<td>Yes</td>
</tr>
<tr>
<td>description</td>
<td></td>
<td>A description</td>
<td>No</td>
</tr>
<tr>
<td>representation</td>
<td></td>
<td>The cardholder legal representation document</td>
<td>Yes</td>
</tr>
<tr>
<td>empowerment</td>
<td></td>
<td>The cardholder legal representation level</td>
<td>Yes</td>
</tr>
<tr>
<td>circumstances</td>
<td></td>
<td>The cardholder legal conditions</td>
<td>No</td>
</tr>
<tr>
<td>limit</td>
<td></td>
<td>The cardholder ristrict of representation</td>
<td>No</td>
</tr>
<tr>
<td>registration</td>
<td></td>
<td>The cardholder representation registry data</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
</table>

<div id="SELLOPJqscd" style="padding-top: 60px;"><h2>SELLOPJqscd<h2></div>

Electronic seal certificate, issued on a smartcard or a cryptographic token and intended for the electronic signature.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>1</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the smartcard one.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>SELLOPJqscd</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>smartcard_sn</td>
<td></td>
<td>The smartcard serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_identifier</td>
<td></td>
<td>The organization identifier</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td></td>
<td>The cardholder first organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>responsible_serial</td>
<td></td>
<td>The responsible serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_email</td>
<td></td>
<td>The responsible email</td>
<td>No</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>process_application</td>
<td></td>
<td>The application name that will use the certificate</td>
<td>Yes</td>
</tr>
<tr>
<td>description</td>
<td></td>
<td>A description</td>
<td>No</td>
</tr>
<tr>
<td>representation</td>
<td></td>
<td>The cardholder legal representation document</td>
<td>Yes</td>
</tr>
<tr>
<td>empowerment</td>
<td></td>
<td>The cardholder legal representation level</td>
<td>Yes</td>
</tr>
<tr>
<td>circumstances</td>
<td></td>
<td>The cardholder legal conditions</td>
<td>No</td>
</tr>
<tr>
<td>limit</td>
<td></td>
<td>The cardholder ristrict of representation</td>
<td>No</td>
</tr>
<tr>
<td>registration</td>
<td></td>
<td>The cardholder representation registry data</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
</table>

<div id="SELLOMedio" style="padding-top: 60px;"><h2>SELLOMedio<h2></div>

Electronic seal certificate for Spanish Public Administrations, intended for advanced electronic signature, generally in unattended processes, and issued issued in cryptographic container format P12.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>[0, 2]</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the software element.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>SELLOMedio</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_identifier</td>
<td></td>
<td>The organization identifier</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td>SELLO ELECTRONICO</td>
<td>The cardholder first organizational unit</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_2</td>
<td></td>
<td>The cardholder second organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>organizational_unit_3</td>
<td></td>
<td>The cardholder third organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>responsible_serial</td>
<td></td>
<td>The responsible serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_email</td>
<td></td>
<td>The responsible email</td>
<td>No</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>process_application</td>
<td></td>
<td>The application name that will use the certificate</td>
<td>Yes</td>
</tr>
<tr>
<td>description</td>
<td></td>
<td>A description</td>
<td>No</td>
</tr>
<tr>
<td>representation</td>
<td></td>
<td>The cardholder legal representation document</td>
<td>Yes</td>
</tr>
<tr>
<td>empowerment</td>
<td></td>
<td>The cardholder legal representation level</td>
<td>Yes</td>
</tr>
<tr>
<td>circumstances</td>
<td></td>
<td>The cardholder legal conditions</td>
<td>No</td>
</tr>
<tr>
<td>limit</td>
<td></td>
<td>The cardholder ristrict of representation</td>
<td>No</td>
</tr>
<tr>
<td>registration</td>
<td></td>
<td>The cardholder representation registry data</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
</table>

<div id="SELLOAlto" style="padding-top: 60px;"><h2>SELLOAlto<h2></div>

Electronic seal certificate for Spanish Public Administrations, intended for the electronic signature usually qualified in unattended processes, and issued on a cryptographic card or token.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>1</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the smartcard one.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>SELLOAlto</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>smartcard_sn</td>
<td></td>
<td>The smartcard serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_identifier</td>
<td></td>
<td>The organization identifier</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td>SELLO ELECTRONICO</td>
<td>The cardholder first organizational unit</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_2</td>
<td></td>
<td>The cardholder second organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>organizational_unit_3</td>
<td></td>
<td>The cardholder third organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>responsible_serial</td>
<td></td>
<td>The responsible serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_email</td>
<td></td>
<td>The responsible email</td>
<td>No</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>process_application</td>
<td></td>
<td>The application name that will use the certificate</td>
<td>Yes</td>
</tr>
<tr>
<td>description</td>
<td></td>
<td>A description</td>
<td>No</td>
</tr>
<tr>
<td>representation</td>
<td></td>
<td>The cardholder legal representation document</td>
<td>Yes</td>
</tr>
<tr>
<td>empowerment</td>
<td></td>
<td>The cardholder legal representation level</td>
<td>Yes</td>
</tr>
<tr>
<td>circumstances</td>
<td></td>
<td>The cardholder legal conditions</td>
<td>No</td>
</tr>
<tr>
<td>limit</td>
<td></td>
<td>The cardholder ristrict of representation</td>
<td>No</td>
</tr>
<tr>
<td>registration</td>
<td></td>
<td>The cardholder representation registry data</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
</table>

<div id="SelloOrganoAltoNubeQ" style="padding-top: 60px;"><h2>SelloOrganoAltoNubeQ<h2></div>

Electronic seal certificate for Spanish Public Administrations, intended for the electronic signature usually qualified in unattended processes, and issued in the centralized custody system of Uanataca certificates.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>2</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the cloud one.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>SelloOrganoAltoNubeQ</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_identifier</td>
<td></td>
<td>The organization identifier</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td>SELLO ELECTRONICO</td>
<td>The cardholder first organizational unit</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_2</td>
<td></td>
<td>The cardholder second organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>organizational_unit_3</td>
<td></td>
<td>The cardholder third organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>responsible_serial</td>
<td></td>
<td>The responsible serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_email</td>
<td></td>
<td>The responsible email</td>
<td>No</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>process_application</td>
<td></td>
<td>The application name that will use the certificate</td>
<td>Yes</td>
</tr>
<tr>
<td>description</td>
<td></td>
<td>A description</td>
<td>No</td>
</tr>
<tr>
<td>representation</td>
<td></td>
<td>The cardholder legal representation document</td>
<td>Yes</td>
</tr>
<tr>
<td>empowerment</td>
<td></td>
<td>The cardholder legal representation level</td>
<td>Yes</td>
</tr>
<tr>
<td>circumstances</td>
<td></td>
<td>The cardholder legal conditions</td>
<td>No</td>
</tr>
<tr>
<td>limit</td>
<td></td>
<td>The cardholder ristrict of representation</td>
<td>No</td>
</tr>
<tr>
<td>registration</td>
<td></td>
<td>The cardholder representation registry data</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
</table>


## Peru

<html>
<table>
  <tr>
    <th>Profile</th><th>Description</th><th>Element</th>
  </tr>
  <tr>
    <td><a href="#PEPNCiudadano">PEPNCiudadano</a></td><td>Natural person</td><td>Software/Smartcard/Token</td>
  </tr>
  <tr>
    <td><a href="#PEPNPerteneciente">PEPNPerteneciente</a></td><td>Natural person belonging to an organization</td><td>Software/Smartcard/Token</td>
  </tr>
  <tr>
    <td><a href="#PEPNRepresentante">PEPNRepresentante</a></td><td>Legal entity representative</td><td>Software/Smartcard/Token</td>
  </tr>
  <tr>
    <td><a href="#PEPNColegiado">PEPNColegiado</a></td><td>Natural person belonging to a professional association</td><td>Software/Smartcard/Token</td>
  </tr>
  <tr>
    <td><a href="#PEFacturacion">PEFacturacion</a></td><td>Legal entity for electronic invoicing</td><td>Software/Smartcard/Token</td>
  </tr>
  <tr>
    <td><a href="#PESElectronico">PESElectronico</a></td><td>Legal entity for unassisted signature</td><td>Cloud/Software/Smartcard/Token</td>
  </tr>
</table> 
</br> 
</html>


<div id="PEPNCiudadano" style="padding-top: 60px;"><h2>PEPNCiudadano<h2></div>

Certificate of natural person, destined to authentication and digital signature.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>[0, 1]</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>PEPNCiudadano</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>smartcard_sn</td>
<td></td>
<td>The smartcard serial number</td>
<td>No</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
DNI – National identity document.
CEX - Immigration Card.
PAS - Passport.
</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
</table>

<div id="PEPNPerteneciente" style="padding-top: 60px;"><h2>PEPNPerteneciente<h2></div>

Certificate of a natural person belonging to or linked to a company or organization, intended for authentication and digital signature.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>[0, 1]</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>PEPNPerteneciente</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>smartcard_sn</td>
<td></td>
<td>The smartcard serial number</td>
<td>No</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[DNI,CEX,PAS]</td>
<td>
The cardholder Id Document Type.
DNI – National identity document.
CEX - Immigration Card.
PAS - Passport.
</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_number</td>
<td></td>
<td>The cardholder document number</td>
<td>No</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>title</td>
<td></td>
<td>The cardholder professional title</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td></td>
<td>The cardholder first organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_identifier</td>
<td></td>
<td>The organization identifier</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>id_responsible_document_type</td>
<td>[DNI,CEX,PAS]</td>
<td>
The responsible document type.
DNI – National identity document.
CEX - Immigration Card.
PAS - Passport.
</td>
<td>Yes</td>
</tr>
<tr>
<td>id_responsible_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The responsible document country</td>
<td>Yes</td>
</tr>
<tr>
<td>id_responsible_document_number</td>
<td></td>
<td>The responsible document number</td>
<td>No</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
</table>

<div id="PEPNRepresentante" style="padding-top: 60px;"><h2>PEPNRepresentante<h2></div>

Certificate of legal entity representative, intended for authentication and digital signature.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>[0, 1]</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>PEPNRepresentante</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>smartcard_sn</td>
<td></td>
<td>The smartcard serial number</td>
<td>No</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[DNI,CEX,PAS]</td>
<td>
The cardholder Id Document Type.
DNI – National identity document.
CEX - Immigration Card.
PAS - Passport.
</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_number</td>
<td></td>
<td>The cardholder document number</td>
<td>No</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>title</td>
<td></td>
<td>The cardholder professional title</td>
<td>Yes</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_identifier</td>
<td></td>
<td>The organization identifier</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
</table>

<div id="PEPNColegiado" style="padding-top: 60px;"><h2>PEPNColegiado<h2></div>

Certificate of a natural person linked to a professional association, destined to authentication and digital signature.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>[0, 1]</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>PEPNColegiado</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>smartcard_sn</td>
<td></td>
<td>The smartcard serial number</td>
<td>No</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[DNI,CEX,PAS]</td>
<td>
The cardholder Id Document Type.
DNI – National identity document.
CEX - Immigration Card.
PAS - Passport.
</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_number</td>
<td></td>
<td>The cardholder document number</td>
<td>No</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td>Colegiado</td>
<td>The cardholder first organizational unit</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_2</td>
<td></td>
<td>The cardholder second organizational unit</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_3</td>
<td></td>
<td>The cardholder third organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>title</td>
<td></td>
<td>The cardholder professional title</td>
<td>Yes</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_identifier</td>
<td></td>
<td>The organization identifier</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>id_responsible_document_type</td>
<td>[DNI,CEX,PAS]</td>
<td>
The responsible document type.
DNI – National identity document.
CEX - Immigration Card.
PAS - Passport.
</td>
<td>Yes</td>
</tr>
<tr>
<td>id_responsible_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The responsible document country</td>
<td>Yes</td>
</tr>
<tr>
<td>id_responsible_document_number</td>
<td></td>
<td>The responsible document number</td>
<td>No</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
</table>

<div id="PEFacturacion" style="padding-top: 60px;"><h2>PEFacturacion<h2></div>

Certificate of legal entity, intended only for electronic invoicing processes.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>[0, 1]</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>PEFacturacion</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>smartcard_sn</td>
<td></td>
<td>The smartcard serial number</td>
<td>No</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[DNI,CEX,PAS]</td>
<td>
The cardholder Id Document Type.
DNI – National identity document.
CEX - Immigration Card.
PAS - Passport.
</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_number</td>
<td></td>
<td>The cardholder document number</td>
<td>No</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>title</td>
<td></td>
<td>The cardholder professional title</td>
<td>Yes</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_identifier</td>
<td></td>
<td>The organization identifier</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td></td>
<td>The cardholder first organizational unit</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_2</td>
<td></td>
<td>The cardholder second organizational unit</td>
<td>No</td>
</tr>
</table>

<div id="PESElectronico" style="padding-top: 60px;"><h2>PESElectronico<h2></div>

Certificado de persona jurídica, destinado a procesos de firma desasistidos dentro de una empresa u organización.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>[0, 1, 2]</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>PESElectronico</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>smartcard_sn</td>
<td></td>
<td>The smartcard serial number</td>
<td>No</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_identifier</td>
<td></td>
<td>The organization identifier</td>
<td>Yes</td>
</tr>
<tr>
<td>process_application</td>
<td></td>
<td>The application name that will use the certificate</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[DNI,CEX,PAS]</td>
<td>
The cardholder Id Document Type.
DNI – National identity document.
CEX - Immigration Card.
PAS - Passport.
</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_number</td>
<td></td>
<td>The cardholder document number</td>
<td>No</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
</table>


## Global

<html>
<table>
  <tr>
    <th>Profile</th><th>Description</th><th>Element</th>
  </tr>
  <tr>
    <td><a href="#PFSoftNC">PFSoftNC</a></td><td>Natural person</td><td>Software</td>
  </tr>
  <tr>
    <td><a href="#PFnubeNC">PFnubeNC</a></td></td><td>Natural person</td><td>Cloud/Smartcard/Token</td>
  </tr>
  <tr>
    <td><a href="#SELLOsoftNC">SELLOsoftNC</a></td></td><td>Electronic seal</td><td>Software</td>
  </tr>
  <tr>
    <td><a href="#SELLOnubeNC">SELLOnubeNC</a></td><td>Electronic seal</td><td>Cloud/Smartcard/Token</td>
  </tr>
</table> 
</br> 
</html>

<div id="PFSoftNC" style="padding-top: 60px;"><h2>PFSoftNC<h2></div>

Certificate of a natural person issued on a cryptographic container in P12 format and intended for authentication and electronic signature.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>0</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the software element.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>PFSoftNC</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_description</td>
<td></td>
<td>The cardholder document description</td>
<td>No</td>
</tr>
<tr>
<td>id_document_issuer</td>
<td></td>
<td>The cardholder document issuer</td>
<td>No</td>
</tr>
<tr>
<td>id_document_number</td>
<td></td>
<td>The cardholder document number</td>
<td>No</td>
</tr>
<tr>
<td>residence_address</td>
<td></td>
<td>The cardholder address of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_city</td>
<td></td>
<td>The cardholder city of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_province</td>
<td></td>
<td>The cardholder province of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence</td>
<td></td>
<td>The cardholder country of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_postal_code</td>
<td></td>
<td>The cardholder postal code of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_state</td>
<td></td>
<td>The cardholder state of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_district</td>
<td></td>
<td>The cardholder district of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_canton</td>
<td></td>
<td>The cardholder canton code of residence</td>
<td>No</td>
</tr>
</table>

<div id="PFnubeNC" style="padding-top: 60px;"><h2>PFnubeNC<h2></div>

Certificate of a natural person issued in the centralized custody system of Uanataca certificates or on a smartcard or a cryptographic token, and intended for authentication and electronic signature.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>[1, 2]</td>
<td>
    Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
    This profile only allows the cloud element.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>PFnubeNC</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_2</td>
<td></td>
<td>The cardholder second surname</td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_description</td>
<td></td>
<td>The cardholder document description</td>
<td>No</td>
</tr>
<tr>
<td>id_document_issuer</td>
<td></td>
<td>The cardholder document issuer</td>
<td>No</td>
</tr>
<tr>
<td>id_document_number</td>
<td></td>
<td>The cardholder document number</td>
<td>No</td>
</tr>
<tr>
<td>residence_address</td>
<td></td>
<td>The cardholder address of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_city</td>
<td></td>
<td>The cardholder city of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_province</td>
<td></td>
<td>The cardholder province of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence</td>
<td></td>
<td>The cardholder country of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_postal_code</td>
<td></td>
<td>The cardholder postal code of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_state</td>
<td></td>
<td>The cardholder state of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_district</td>
<td></td>
<td>The cardholder district of residence</td>
<td>No</td>
</tr>
<tr>
<td>residence_canton</td>
<td></td>
<td>The cardholder canton code of residence</td>
<td>No</td>
</table>

<div id="SELLOsoftNC" style="padding-top: 60px;"><h2>SELLOsoftNC<h2></div>

Electronic seal certificate issued in a cryptographic container in P12 format and intended for electronic signature, usually in unassisted processes.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>0</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>SELLOsoftNC</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_identifier</td>
<td></td>
<td>The organization identifier</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td></td>
<td>The cardholder first organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>responsible_serial</td>
<td></td>
<td>The responsible serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_email</td>
<td></td>
<td>The responsible email</td>
<td>No</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>process_application</td>
<td></td>
<td></td>
<td>Yes</td>
</tr>
<tr>
<td>description</td>
<td></td>
<td>A description</td>
<td>No</td>
</tr>
<tr>
<td>representation</td>
<td></td>
<td>The cardholder legal representation document</td>
<td>Yes</td>
</tr>
<tr>
<td>empowerment</td>
<td></td>
<td>The cardholder legal representation level</td>
<td>Yes</td>
</tr>
<tr>
<td>circumstances</td>
<td></td>
<td>The cardholder legal conditions</td>
<td>No</td>
</tr>
<tr>
<td>limit</td>
<td></td>
<td>The cardholder ristrict of representation</td>
<td>No</td>
</tr>
<tr>
<td>registration</td>
<td></td>
<td>The cardholder representation registry data</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
</table>


<div id="SELLOnubeNC" style="padding-top: 60px;"><h2>SELLOnubeNC<h2></div>

Electronic seal certificate issued in the centralized custody system of Uanataca certificates or on a smartcard or a cryptographic token, and intended for electronic signature, usually in unassisted processes.

<table>
<th>Field</th>
<th>Value</th>
<th>Description</th>
<th>Mandatory</th>
<tr>
<td>secure_element</td>
<td>[1, 2]</td>
<td>
Represents the device where the keys will be enrolled and can assume the values of 0, 1 or 2 that respectively are Software, Smartcard and Cloud.
This profile only allows the cloud element.
</td>
<td>Yes</td>
</tr>
<tr>
<td>profile</td>
<td>SELLOnubeNC</td>
<td>Represents the profile of the request</td>
<td>Yes</td>
</tr>
<tr>
<td>validity_time</td>
<td>[1,3,365,730,1095,1825]</td>
<td>It's the certificate validity expressed in days</td>
<td>Yes</td>
</tr>
<tr>
<td>registration_authority</td>
<td></td>
<td>The Registration Authority id</td>
<td>Yes</td>
</tr>
<tr>
<td>scratchcard</td>
<td></td>
<td>The scratchcard serial number that will be associated to the Request</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_type</td>
<td>[IDC,PAS,PNO,TIN]</td>
<td>
The cardholder Id Document Type.
IDC - Identification based on national identity card number.
PAS - Identification based on passport number.
PNO - Identification based on (national) personal number (national civic registration number).
TIN - Tax Identification Number according to the European Commission.</td>
<td>Yes</td>
</tr>
<tr>
<td>id_document_country</td>
<td>ISO 3166-1 alpha-2</td>
<td>The cardholder id document country</td>
<td>Yes</td>
</tr>
<tr>
<td>serial_number</td>
<td></td>
<td>The cardholder serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>given_name</td>
<td></td>
<td>The cardholder given name</td>
<td>Yes</td>
</tr>
<tr>
<td>surname_1</td>
<td></td>
<td>The cardholder first surname</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_country</td>
<td></td>
<td>The organization country</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_identifier</td>
<td></td>
<td>The organization identifier</td>
<td>Yes</td>
</tr>
<tr>
<td>organization_name</td>
<td></td>
<td>The organization name</td>
<td>Yes</td>
</tr>
<tr>
<td>organizational_unit_1</td>
<td></td>
<td>The cardholder first organizational unit</td>
<td>No</td>
</tr>
<tr>
<td>email</td>
<td></td>
<td>The cardholder email</td>
<td>Yes</td>
</tr>
<tr>
<td>mobile_phone_number</td>
<td></td>
<td>The cardholder mobile phone number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_name</td>
<td></td>
<td>The name of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_first_surname</td>
<td></td>
<td>The first surname of the organization representative</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_second_surname</td>
<td></td>
<td>The second of the organization representative</td>
<td>No</td>
</tr>
<tr>
<td>responsible_serial</td>
<td></td>
<td>The responsible serial number</td>
<td>Yes</td>
</tr>
<tr>
<td>responsible_email</td>
<td></td>
<td>The responsible email</td>
<td>No</td>
</tr>
<tr>
<td>organization_email</td>
<td></td>
<td>The organization email</td>
<td>Yes</td>
</tr>
<tr>
<td>process_application</td>
<td></td>
<td>The application name that will use the certificate</td>
<td>Yes</td>
</tr>
<tr>
<td>description</td>
<td></td>
<td>A description</td>
<td>Yes</td>
</tr>
<tr>
<td>representation</td>
<td></td>
<td>The cardholder legal representation document</td>
<td>Yes</td>
</tr>
<tr>
<td>empowerment</td>
<td></td>
<td>The cardholder legal representation level</td>
<td>Yes</td>
</tr>
<tr>
<td>circumstances</td>
<td></td>
<td>The cardholder legal conditions</td>
<td>No</td>
</tr>
<tr>
<td>limit</td>
<td></td>
<td>The cardholder ristrict of representation</td>
<td>No</td>
</tr>
<tr>
<td>registration</td>
<td></td>
<td>The cardholder representation registry data</td>
<td>No</td>
</tr>
<tr>
<td>organization_address</td>
<td></td>
<td>The organization address</td>
<td>No</td>
</tr>
<tr>
<td>organization_city</td>
<td></td>
<td>The organization city</td>
<td>No</td>
</tr>
<tr>
<td>organization_province</td>
<td></td>
<td>The organization province</td>
<td>No</td>
</tr>
<tr>
<td>organization_postal_code</td>
<td></td>
<td>The organization postal code</td>
<td>No</td>
</tr>
</table>

# Postman collection

A postman collection is available as a support for a quick start.<br>

<a href="https://cdn.bit4id.com/es/uanataca/public/ra/Uanataca_RA_Postman.zip">Registration Authority Postman collection download</a>
<br>

> Postman settings

It is required to add the authentication API certificate provided according to the environment.
The certificates  are added in:

**Postman settings > Certificates > Client certificates**

The required fields are:

`host` and `port`: The endpoint host and port regarding the environment

- **access.bit4id.org** in port **13035** for test environment
- **api.uanataca.com** in port **443** for production environment

`CRT file`: the file containing the public certificate

`KEY file`: the file containing the certificate private key

For test environment is also required to disable the SSL certificate verification in:. 

**Postman settings > General > Request**

`SSL certificate verification` must be set OFF.

<div id="APIReference" style="padding-top: 60px;"><h1>API Reference<h1></div>
