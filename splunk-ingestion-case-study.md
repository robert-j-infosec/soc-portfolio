# **Splunk Universal Forwarder Ingestion Failure – Investigation & Resolution**

## **1. Overview**
A Windows endpoint stopped sending logs to Splunk. The SOC needed to determine whether the issue was caused by the Universal Forwarder, network connectivity, configuration corruption, or certificate problems. This case study documents the investigation and restoration of log ingestion.

---

## **2. Initial Detection**
The issue was identified when:

- The host stopped appearing in Splunk searches  
- No new events were populating the `wineventlog` index  
- The forwarder status dashboard showed the endpoint as **missing**

This triggered a Tier 1 investigation.

---

## **3. Evidence Collected**
- `splunkd.log` from the endpoint  
- Windows Event Viewer (Application/System)  
- Forwarder service status  
- `outputs.conf` and `inputs.conf`  
- Indexer listener status (`9997`)  
- Deployment server configuration  

---

## **4. Key Findings**

### **4.1 Forwarder Service Running**
The SplunkForwarder service was active:

```
Get-Service SplunkForwarder
```

No service‑level issues were found.

---

### **4.2 Connection Failures in splunkd.log**
Repeated errors were observed:

```
ERROR TcpOutputProc - Connection to indexer failed
ERROR SSLCommon - Received fatal SSL alert
```

This indicated a **TLS handshake failure** between the forwarder and indexer.

---

### **4.3 Corrupted outputs.conf**
Reviewing the forwarder configuration revealed:

- Incorrect indexer hostname  
- Missing `server = <indexer-ip>:9997`  
- A deployment server push had overwritten the file with invalid values  

This explained the SSL errors and ingestion failure.

---

## **5. Root Cause**
A misconfigured `outputs.conf` was pushed from the deployment server, causing:

- TLS handshake failures  
- Incorrect indexer target  
- Forwarder unable to establish a secure connection  
- Complete ingestion failure  

---

## **6. Resolution Steps**

### **6.1 Corrected outputs.conf**
```
[tcpout]
defaultGroup = indexer_group

[tcpout:indexer_group]
server = <indexer-ip>:9997
```

---

### **6.2 Validated Indexer Listener**
```
netstat -an | find "9997"
```

---

### **6.3 Restarted the Forwarder**
```
splunk restart
```

---

### **6.4 Verified Successful Connection**
`splunkd.log` showed:

```
INFO TcpOutputProc - Connected to indexer
```

Events began flowing immediately.

---

## **7. Outcome**
- Log ingestion fully restored  
- Endpoint reappeared in Splunk searches  
- Deployment server configuration corrected  
- Root cause documented for SOC knowledge base  

---

## **8. Lessons Learned**
- Validate `outputs.conf` when ingestion stops  
- TLS errors often indicate hostname/certificate mismatches  
- Deployment server pushes can silently break forwarders  
- Monitoring dashboards should alert on missing forwarders within 15 minutes  

---

## **9. MITRE ATT&CK Context**
While not an attack, ingestion failures impact:

- **TA0002 – Execution visibility**  
- **TA0007 – Discovery visibility**  
- **TA0008 – Lateral Movement visibility**

Maintaining telemetry is critical for SOC detection capability.

---

## **10. Next Steps**
- Add automated checks for forwarder heartbeat failures  
- Implement configuration validation before DS pushes  
- Add alerting for TLS handshake errors in `splunkd.log`  

---
