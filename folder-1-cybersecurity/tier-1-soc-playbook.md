# Tier-1 SOC Playbook: Detecting and Containing DDoS & Mass Assignment Attacks using NIST SP 800-61

This standard operating playbook defines the containment protocol for neutralizing high-impact REST API security incidents and privilege escalation attacks within a Security Operations Center (SOC) environment.

* * *

## 1\. Incident Metadata & Triage Intake

Security analysts must populate baseline parameters immediately upon alert triage prior to initiating active mitigation routines.

| Metadata Field | Registry Value |
| --- | --- |
| **Incident ID** | `IR-2026-99128` |
| **Timestamp (UTC)** | `2026-07-28T14:30:00.128Z` |
| **Target URI** | `{{base_url}}/api/v1/billing_info/plan_details` |
| **Source IP Address** | `192.168.14.88` |
| **Host Gateway Agent** | `API-GW-PROD-04` |
| **Triage Classification** | High-Severity API Input Abuse |

* * *

## 2\. Raw Payload Capture & Attack Scope

### Captured Gateway Payload

```json
{
  "username": "johndoe",
  "billing_info": {
    "street": "123 Main St",
    "plan_details": {
      "plan_id": "premium-tier",
      "is_billing_exempt": true
    }
  }
}
```

### Attack Vector Analysis

*   **Primary Vulnerability:** Nested Parameter Auto-Binding (Mass Assignment / OWASP API6:2023).
    
*   **Exploit Mechanism:** The target endpoint recursively parses incoming JSON trees. The attacker exploited this nested structure to inject the `is_billing_exempt: true` attribute deep within the child model path (`billing_info.plan_`[`details.is`](http://details.is)`_billing_exempt`).
    
*   **Financial Risk Impact:** The unauthorized privilege bypass allowed the provision of premium services without recording valid payment methods, creating a financial exposure risk of approximately $14,500 per event.
    

## 3\. Prioritized Containment Workflow (NIST SP 800-61)

```plaintext
[ Preparation ] ➔ [ Detection & Analysis ] ➔ [ Containment Checklist ] ➔ [ Recovery ] ➔ [ Post-Incident Activity ]
```

### Step 1: Containment

*   **Identity Access Revocation:** Revoke the compromised JSON Web Token (JWT) from the active session cache immediately.
    
*   **Database Isolation:** Sever application-to-database write pathways if anomalous outbound write commands persist.
    
*   **Perimeter Blocking:** Apply a drop rule for IP `192.168.14.88` at the Web Application Firewall (WAF) perimeter boundary.
    

### Step 2: Eradication

*   **Database Purge:** Execute an explicit SQL update query resetting `is_billing_exempt` attributes to `false` for modified accounts within the incident window.
    
*   **Input Schema Enforcement:** Inject a strict JSON Schema validation layer at the entry controller of the endpoint to discard un-whitelisted attributes prior to model serialization.
    

### Step 3: Recovery

*   **Container Reversion:** Revert API application containers to the last verified stable software build tag.
    
*   **Regression Testing:** Execute automated Postman regression suites to confirm the endpoint returns a `400 Bad Request` validation failure when receiving attack payloads.
    

### Step 4: Verification & Auditing

*   **Perimeter Audit:** Issue a test request from IP `198.51.100.72` and verify the perimeter firewall returns a `403 Forbidden` response.
    
*   **Node Status Audit:** Confirm EDR telemetry reports database host `db-srv-01` in an active "Quarantine Mode" status.
    
*   **Identity Audit:** Verify OAuth user record `usr_sydney_coder_8f4g` displays an "unauthorized" and "revoked" status in the identity provider directory.
