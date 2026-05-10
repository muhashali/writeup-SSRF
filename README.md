# Security Report: SSRF Vulnerability in Web-to-PDF Conversion Service

## 1. Vulnerability Overview
* **Vulnerability Type:** Server-Side Request Forgery (SSRF)
* **Target Application:** `utils.edu.stf` (Internal IP: `10.124.1.237`)
* **Affected Component:** PDF Generation Engine (URL-to-PDF Converter)
* **Severity:** High

## 2. Technical Description
The target host exposes a utility designed to convert web pages into PDF documents. This service accepts a user-provided URL, fetches the content via a server-side GET request, and renders the result. 

The application fails to implement sufficient validation of the input URL and lacks egress filtering. This allows an attacker to manipulate the server into initiating requests to internal infrastructure and loopback addresses (`127.0.0.1`) that are otherwise protected by network-level access control lists (ACLs).

## 3. Proof of Concept (PoC)
The vulnerability was exploited to access an internal service bound to a non-standard port on the local interface.

### Attack Vector:
1.  **Endpoint:** `http://10.124.1.237/`
2.  **Vector:** The "Convert via URL" input field.
3.  **Payload:** `http://127.0.0.1:9732`
4.  **Action:** Upon submission, the server fetched the contents of the local service at port 9732, rendered the response into a PDF, and delivered the document to the attacker.

### Results:
The generated PDF document contained the internal service's output, including the challenge flag:
`STF{9b9e7d88-2846-4841-b07c-9b09b29cc361}`

## 4. Impact Analysis
Successful exploitation of this SSRF vulnerability demonstrates the following risks:
* **Internal Reconnaissance:** Ability to scan internal ports and identify hidden services.
* **Access Control Bypass:** Direct interaction with services restricted to the local interface (e.g., administrative panels, metadata services).
* **Information Disclosure:** Exposure of sensitive system data or internal credentials.

## 5. Remediation Strategy
To secure the application against SSRF, the following controls are recommended:
* **Allow-listing:** Restrict the converter to a predefined list of trusted domains and protocols (e.g., only `https`).
* **Network Segregation:** Implement firewall rules to block the application server from reaching private IP ranges (RFC 1918) and loopback addresses.
* **Pre-Request Validation:** Resolve the provided URL's hostname to an IP address and verify it is not a private or local address before initiating the connection.
* **Isolated Environment:** Deploy the rendering engine in a low-privilege, sandboxed container with restricted network egress.
