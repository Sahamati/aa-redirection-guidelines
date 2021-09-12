---
description: Specification for redirection response from the AA back to FIU
---

# Response Specification

Following URL parameters would need to be accepted on FIU side.

{% api-method method="get" host="https://my-fiu.com/webview" path="" %}
{% api-method-summary %}
WebView Response Redirection
{% endapi-method-summary %}

{% api-method-description %}

{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter type="string" name="ecres" required=true %}
Encrypted parameters \(see below\)
{% endapi-method-parameter %}

{% api-method-parameter name="resdate" type="string" required=true %}
format will be ddmmyyyyhh24misss UTC
{% endapi-method-parameter %}

{% api-method-parameter name="fi" type="string" required=true %}
Unique AA identifier. This will be encrypted using Base64/XOR along with resdate field
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```text

```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

## ecres \(encrypted path parameters\)

Below are the parameters that will be encrypted using AES256 encryption algorithm

| **Parameter name** | **Parameter type** | **Parameter description** |
| :--- | :--- | :--- |
| txnid | String | UUID txnid \( To be sent back from the request \) |
| sessionid | String | Value of sessionid received in the ‘ecreq’ field in the request. |
| userid | String | The AA user id |
| status | String | The status ‘S’ for success and ‘F’ for failure |
| errorcode | String | Refer the errorcodes table below |
| srcref | String | The consent handle id received in the ecreq ‘srcref’ field |

## Error codes

The following errorcodes are returned by the AA to FIU when the user is redirected back to the FIU application.

| **Error code** | **Message** | **Status Parameter** | **Mandatory** | **When is this returned** |
| :--- | :--- | :--- | :--- | :--- |
| 0 | Success | S | Y | When the user has accepted the consent |
| 1 | Consent is rejected | F | Y | When the user rejects the consent |
| 2 | Consent not available | F | Y | Consent request not found with the AA |
| 3 | Invalid request | F | Y | The redirection request has invalid data |
| 4 | User authentication failed | F | N | User is not able to authenticate self |
| 5 | Consent in pending state | F | N | User neither approved or rejected the consent |

