# Developer Integration Manual: Secure Firewall REST API Gateway
This manual guides developers through authenticating, configuring mandatory security headers, executing rule-creation payloads, and programmatically resolving errors using the Secure Firewall API Gateway.

**Base Path:** `https://api.securegateway.com/v1`

* * *

## 1\. Authentication Workflow

To access protected endpoints, client applications must first obtain a cryptographically signed JSON Web Token (JWT) by submitting valid service credentials.

### Authentication Request

*   **Method:** `POST`
    
*   **Endpoint:** `https://api.securegateway.com/v1/auth/token`
    
*   **Headers:** `Content-Type: application/json`
    

```json
{
  "client_id": "srv_firewall_mgr_01",
  "client_secret": "sec_d98f7e21a2bc490e8a71f8fe0876cd3a"
}
```

### Successful Response (`201 Created`)

JSON

```plaintext
{
  "token_type": "Bearer",
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expires_in_seconds": 900
}
```

## 2\. Mandatory HTTP Security Headers

Integrating client systems must pass standard security headers on every HTTP transaction:

<table style="min-width: 75px;"><colgroup><col style="min-width: 25px;"><col style="min-width: 25px;"><col style="min-width: 25px;"></colgroup><tbody><tr><td colspan="1" rowspan="1"><p><strong>HTTP Security Header</strong></p></td><td colspan="1" rowspan="1"><p><strong>Required Value</strong></p></td><td colspan="1" rowspan="1"><p><strong>Defensive Purpose</strong></p></td></tr><tr><td colspan="1" rowspan="1"><p><strong>Strict-Transport-Security</strong></p></td><td colspan="1" rowspan="1"><p><code>max-age=63072000; includeSubDomains; preload</code></p></td><td colspan="1" rowspan="1"><p>Forces encrypted HTTPS connections, neutralizing Man-in-the-Middle (MitM) attacks.</p></td></tr><tr><td colspan="1" rowspan="1"><p><strong>X-Content-Type-Options</strong></p></td><td colspan="1" rowspan="1"><p><code>nosniff</code></p></td><td colspan="1" rowspan="1"><p>Disables MIME-type sniffing to prevent cross-site scripting (XSS) vectors.</p></td></tr><tr><td colspan="1" rowspan="1"><p><strong>Authorization</strong></p></td><td colspan="1" rowspan="1"><p><code>Bearer &lt;access_token&gt;</code></p></td><td colspan="1" rowspan="1"><p>Authorizes administrative requests at the gateway parsing boundary.</p></td></tr></tbody></table>

## 3\. Creating a Packet-Filtering Rule

Submit a `POST` request to `/firewall-rules` using schema-validated parameters.

Prevent Mass Assignment: The parameter `is\_system\_override` is strictly readOnly. Including this attribute in request payloads will trigger an immediate `400 Bad Request` validation error.

### Request Body (`POST /v1/firewall-rules`)

```json
{ "rule\_name": "allow\_https\_traffic", "destination\_ip": "198.51.100.42", "direction": "ingress", "network\_port": 443 }
```

### Successful Response (`201 Created`)

```json

{ "rule\_id": "8f4geev-e89b-12d3-a456-426614174000", "rule\_name": "allow\_https\_traffic", "destination\_ip": "198.51.100.42", "direction": "ingress", "network\_port": 443, "is\_system\_override": false }
```

## 4\. Error Resolution Catalog

Scenario: `INVALID\_PORT\_RANGE` (400 Bad Request) Occurs when the payload contains a port number outside the allowable range (1 to 65535).

```json
{ "error\_code": "INVALID\_PORT\_RANGE", "message": "The specified network port must be a valid integer between 1 and 65535.", "timestamp": "2026-10-24T18:42:01.105Z", "remediation\_reference": "[https://api.securegateway.com/v1/docs/errors#INVALID\_PORT\_RANGE](https://api.securegateway.com/v1/docs/errors#INVALID_PORT_RANGE)" }
```

**Programmatic Remediation Steps:**

1.  Inspect client-side form input validation rules to restrict port values prior to transmission.
    
2.  Enforce strict integer range constraints (1–65535 inclusive) in internal data models.
    
3.  Resubmit the corrected request payload to `/v1/firewall-rules`.
