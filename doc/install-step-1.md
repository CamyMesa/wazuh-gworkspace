# On GCP

## Create an GCP service account
First you will create a **GCP service account**. This service account will use **Domain-wide delegation** to impersonate a **Google Workspace service account** to access to the events and alerts.

At [https://console.cloud.google.com/](https://console.cloud.google.com/) create a new project called `Wazuh`.

<img width="2372" height="838" alt="image" src="https://github.com/user-attachments/assets/65e0c9c4-7596-4a2e-a63c-af0101364cb9" />


In the [Enabled API & services](https://console.cloud.google.com/apis/library) enable:
* the [Admin SDK API](https://console.cloud.google.com/apis/library/admin.googleapis.com) 
* the [Alert Center API](https://console.cloud.google.com/apis/api/alertcenter.googleapis.com)

In the [API & Services menu](https://console.cloud.google.com/apis) select the [Credentials screen](https://console.cloud.google.com/apis/credentials) and create a **service account** with an **Service account name** of for example `Wazuh monitoring`. You will need the `client id` later on.

<img width="998" height="728" alt="image" src="https://github.com/user-attachments/assets/163ec846-4757-4719-b714-de3e98f2f502" />


Once the service account is created, select the "Keys" option from the horizontal menu, and add a key. 

<img width="1307" height="626" alt="image" src="https://github.com/user-attachments/assets/f8e69b45-412d-4f90-9375-82222f9dcfec" />

<img width="595" height="350" alt="image" src="https://github.com/user-attachments/assets/81877cf4-f4cf-4c94-98d0-9cf59a0a2a05" />

### Be sure to save the key in your password manager since you cannot download it afterwards.

# On Google Workspace

## Create a Google Workspace service account
Now, in the Google Workspace admin allow the GCP service account [Domain-wide delegation](https://admin.google.com/ac/owl/domainwidedelegation) by entering the `client id`, and giving it two scopes:
* `https://www.googleapis.com/auth/admin.reports.audit.readonly`
* `https://www.googleapis.com/auth/apps.alerts`

<img width="888" height="525" alt="image" src="https://github.com/user-attachments/assets/fa74b9b8-d136-4039-bcc3-a06d5d9529df" />


Then you will create a Google Workspace service account that will serve as a service account used by Wazuh to connect. We will make sure to not give this account a Google Workspace licence, both for financial and security reasons.

In the [Organizational units](https://admin.google.com/ac/orgunits) screen create an OrgUnit called `Service accounts`.

<img width="1578" height="732" alt="image" src="https://github.com/user-attachments/assets/512e7490-0803-4564-a105-0aefdb1f5f50" />

Change the [Licence settings](https://admin.google.com/ac/billing/licensesettings) for that OrgUnit so that licenses are not automatically assigned. Note that the sliders do not appear until your mouse is over the option.

<img width="841" height="324" alt="image" src="https://github.com/user-attachments/assets/a1041052-5649-4454-8d74-5053653ceadd" />

Create a new user in that OrgUnit, naming it for example **First name** `SVC Wazuh` and **Last name** `Monitoring` with an e-mail address of `svc-wazuh-monitoring@yourdomain.com`. 
* in **Admin roles and privileges**
* Asign a new role named SVC Wazuh Monitoring and add the following privileges:
  * in `Alert Center` check `View access`
  * check the `Reports` role
 
Add this role to the svc-wazuh-monitoring account

<img width="412" height="206" alt="image" src="https://github.com/user-attachments/assets/5351cb84-a731-4a69-8780-5ce36a1289e7" />


* in **Licences** double-check that the user does not have a paying licence
* in **Apps** double-check that the user does not have access to apps that it shouldn't
* you can throw away the password for that user, it will be the GCP service account that impersonates this user

Depending on your MFA policies you may also need to change other OrgUnit-wide settings for the OrgUnit, so that you can turn off **2-step verification** or **require password change** in the **Security** tab sheet.

Hope you kept the JSON file containing the key of the GCP service account, you will use it in the [next step](/doc/install-step-2.md).

