# wazuh-gworkspace

This Wazuh wodle integrates **all Google Workspace audit events**, including:

- Google Drive
- Google Groups
- Google Calendar
- Admin
- All other products visible in **Google Workspace Reporting**
- **Google Workspace Alert Center**

---

<img width="1509" height="665" alt="image" src="https://github.com/user-attachments/assets/2348c56c-d9f9-4ad5-bfb3-d4991d4bbe11" />

---

## Advantages vs Standard Wazuh GCP Integration

- ✅ **No complex Pub/Sub configuration required**
- ✅ Integrates **all auditable Google Workspace events** available in Google Workspace Reporting  
  (Drive, Calendar, Admin, Groups, etc.)
- ✅ Integrates alerts from **Google Workspace Alert Center**
- ✅ Includes **predefined rules with sensible alert levels**, aligned with equivalent actions in the O365 integration

---

## Disadvantages / Limitations

- ❌ Covers **Google Workspace only**, not Google Cloud Platform (GCP)
- ❌ **Batch-driven** instead of event-driven  
  → introduces a delay between event occurrence and ingestion
- ❌ The `@timestamp` reflects **ingestion time**, not event time  
  → original event time is stored in `data.timestamp`

---

## Installation

1. **Create a Service Account & OAuth Client** in Google Workspace
2. **Install the Wodle**

---

## Frequently Asked Questions

### What if I have several Google Workspace tenants?
What if I have several Google Workspace tenants?
Just follow the installation procedure several times. So:

- Create a service account in each tenant

- Create separate directories
/var/ossec/wodles/gworkspace-tenant-A/
/var/ossec/wodles/gworkspace-tenant-B/
etc

- Create the respective service accounts, and place them in the service_account_key.json of their directories.
- In ossec.conf create separate <wodle> entries, where the <command> is changed:
 

```bash
 <wodle name="command">
    <disabled>no</disabled>
    <tag>gworkspace</tag>
    <command>/var/ossec/wodles/gworkspace-tenant-A/gworkspace -a all -o 2</command>
    <interval>10m</interval>
    <ignore_output>no</ignore_output>
    <run_on_start>yes</run_on_start>
    <timeout>0</timeout>
  </wodle>
```

All the events include a data.gworkspace.customerId that identifies the Google Workspace customer. If you want a specific label you can add a <tag>name</tag> to the ossec.conf.

