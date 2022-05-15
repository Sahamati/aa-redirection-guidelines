---
description: Specification for redirection request from FIU to Account Aggregator
---

# Request Specification

## **URL** Parameters

> **\*** required parameters

| Name              | Type   | Description                                                                                                                                                                                                                                                                                         |
| ----------------- | ------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **fi**\*          | String | Unique requestor identifier. This will be encrypted using Base64/XOR along with **reqdate** field.                                                                                                                                                                                                  |
| **requestorType** | Enum   | Type of the requestor. This field will hold the values - FIU & LSP. The ENUM field would be expanded to accommodate new participants for any future requirements. This will be encrypted using Base64/XOR along with **reqdate** field.                                                             |
| **reqdate**\*     | String | **reqdate** field format will be ddmmyyyyhh24misss in UTC. For example, 06-Apr-2021, 10:40am and 756 milliseconds in IST should be formatted as 060420210510756 in UTC.The receiving server needs to validate this timestamp to be 180 secs of current time and beyond this request will be invalid |
| **ecreq**\*       | String | Base64 encoded & encrypted request parameters ([see details below](request-specification.md#ecreq-encrypted-path-parameters))                                                                                                                                                                       |

## ecreq (encrypted path parameters)

Below are the required parameters that will be encrypted using AES256 encryption algorithm.

| **Parameter name** | **Parameter type** | **Parameter description**                                                                                                                                                                                                                                                                                                                  |
| ------------------ | ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| txnid              | String             | UUID txnid. Uniquely identifies a particular redirection event. This same value will be returned by AA to requestor in the ecres txnid field.                                                                                                                                                                                              |
| sessionid          | String             | Value that represents a ‘state’ (or session) on the requestor end. This value is opaque to AA and will be returned as is to the requestor by AA in ecres sessionid field.                                                                                                                                                                  |
| userid             | String             | The AA user id ( Refer to A] below )                                                                                                                                                                                                                                                                                                       |
| redirect           | String             | Requestor Url that AA needs to call back after the user has provided consent in the AA domain. The value of this parameter should be URL encoded if the value contains url parameters. This is required in order to remove ambiguity between the parameters of ecreq (separated by ‘&’ character) with the parameters in the redirect url. |
| srcref             | Array              | Array of consent handle id(s), as returned by AA server to the /Consent request api invoked by the FIU(s) on the AA prior to this redirection call.                                                                                                                                                                                        |
| fipid              | Array              | Optional, can be blank. If FIU already has the information that citizen wants to discover accounts of a particular FIP beforehand, then they can pass this info to AA as a comma separated array in curly braces. It saves user clicks and enhances journey experience                                                                     |

### userid

The userid can be the mobile no with the aa handle e.g. 9999988888@aa

#### For existing AA user

The userid should be with the AA handle e.g. userid@aa
