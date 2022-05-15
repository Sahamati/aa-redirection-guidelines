---
description: Specification for redirection response from the AA back to FIU
---

# Response Specification

## URL Parameters

Following URL parameters would need to be accepted on the requestor side.

> All URL query parameters below are **required**.

| Parameter Name | Type   | Description                                                                                   |
| -------------- | ------ | --------------------------------------------------------------------------------------------- |
| fi             | String | Unique AA identifier. This will be encrypted using Base64/XOR along with resdate field        |
| resdate        | String | format will be `ddmmyyyyhh24misss` UTC                                                        |
| fi             | String | Encrypted parameters ([see below](response-specification.md#ecres-encrypted-path-parameters)) |

## ecres (encrypted path parameters)

Below are the parameters that will be encrypted using AES256 encryption algorithm

| **Parameter name** | **Parameter type** | **Parameter description**                                                                                    |
| ------------------ | ------------------ | ------------------------------------------------------------------------------------------------------------ |
| txnid              | String             | UUID txnid ( To be sent back from the request )                                                              |
| sessionid          | String             | Value of sessionid received in the ‘ecreq’ field in the request.                                             |
| userid             | String             | The AA user id                                                                                               |
| status             | String             | The status ‘S’ for success ( atleast one consent is approved ) and ‘F’ for failure ( all consents declined ) |
| errorcode          | String             | The response code : 0 if status is ‘S’ and others for failure (Refer to Error Codes table below)             |
| srcref             | Array              | Array of accepted consent handle identifiers only in case of LSP                                             |

## Error codes

The following errorcodes are returned by the AA to FIU when the user is redirected back to the FIU application.

| **Error code** | **Message**                | **Status Parameter** | **Mandatory** | **When is this returned**                     |
| -------------- | -------------------------- | -------------------- | ------------- | --------------------------------------------- |
| 0              | Success                    | S                    | Y             | When the user has accepted the consent        |
| 1              | Consent is rejected        | F                    | Y             | When the user rejects the consent             |
| 2              | Consent not available      | F                    | Y             | Consent request not found with the AA         |
| 3              | Invalid request            | F                    | Y             | The redirection request has invalid data      |
| 4              | User authentication failed | F                    | N             | User is not able to authenticate self         |
| 5              | Consent in pending state   | F                    | N             | User neither approved or rejected the consent |
