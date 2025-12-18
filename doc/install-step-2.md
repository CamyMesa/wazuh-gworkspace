> [!NOTE]  
> this wodle was adapted for an all in one Wazuh architecture and may require (slight) adaptation for other deployment methods - 

# add required Python libraries


```
apt install python3-googleapi python3-google-auth

```

The wodle requires the `google-api-python-client` Python library, which is not distributed with the standard Wazuh distribution.
on.

# install wodle
Clone this repo in the directory `/var/ossec/wodles`
```

> git clone https://github.com/CamyMesa/gworkspace.git
> ls
wazuh-gworkspace/
>chown -R root:wazuh /var/ossec/wodles/gworkspace
>chmod -R 750 /var/ossec/wodles/gworkspace
```


Copy the file with the GCP service account key you have created previously, by pasting the contents of the JSON file inside the service_account_key.json

```
cd /var/ossec/wodles/gworkspace/wodle
touch service_account_key.json
cat service_account_key.json
```


Also configure your Google Workspace service account in a file named config.json:
```
touch config.json


{
    "service_account": "<E-MAIL OF YOUR GOOGLE WORKSPACE SERVICE ACCOUNT>"
}

Change the privileges if needed

chown -R root:wazuh config.json
chmod -R 750 config.json
chown -R root:wazuh service_account_key.json
chmod -R 750 service_account_key.json

root@WazuhLab4:/var/ossec/wodles/gworkspace/wodle# ll
total 36
drwxr-x--- 2 root wazuh 4096 dic 10 09:43 ./
drwxr-x--- 6 root wazuh 4096 dic  2 17:08 ../
-rwxr-x--- 1 root wazuh   51 dic  2 16:46 config.json*
-rwxr-x--- 1 root wazuh   48 dic  2 16:45 .gitignore*
-rwxr-x--- 1 root wazuh  594 dic  2 17:06 gworkspace*
-rwxr-x--- 1 root wazuh 9267 dic 10 09:43 gworkspace.py*
-rwxr-x--- 1 root wazuh 2366 dic  2 16:46 service_account_key.json*
```

You can test that the wodle works by running it and checking that it outputs log events in JSON format. The `--unread` parameter ensures that the historical messages will be left unread for the next run. 
```
./gworkspace -a admin --unread | ./gworkspace -a all --unread
```

# add rules
Events only generate alerts if they are matched by a rule. Go to the rules configuration and create a new rules file `0685-gworkspace_rules.xml` and fill it with the contents of [/rules/0685-gworkspace_rules.xml](/rules/0685-gworkspace_rules.xml).

# change ossec.conf
Add this wodle configuration to `/var/ossec/etc/ossec.conf` to ensure that the wodle is called periodically by Wazuh. 
```
  <wodle name="command">
    <disabled>no</disabled>
    <tag>gworkspace</tag>
    <command>/var/ossec/wodles/gworkspace/wodle/gworkspace -a all -o 2</command>
    <interval>10m</interval>
    <ignore_output>no</ignore_output>
    <run_on_start>yes</run_on_start>
    <timeout>0</timeout>
  </wodle>
```

This will run the wodle every 10 minutes. Running it more often will be more resource-intensive for Google (every run requires at least one API call for each of the 30-odd service types, such as 'Meet', 'Drive', etc) and running it is less often will mean that events arrive with more delay. More delay also means that the `@timestamp` fields are (more) incorrect, since Wazuh does not allow the decoder to map a field to `@timestamp`, but fills it with the time of alert injection. The `data.timestamp`contains the real timestamp of the event. This means the information is retained but the events may show up in the Wazuh dashboards a couple of minutes after their actual occurrence.

The wodle keeps track of the most recent event that has been extracted for each service type, and will start extracting from that time point on at the next extraction. The `-o` parameter configures the offset, or the maximum number of hours to go back in time. If the offset goes back too far in history, the extraction will return a lot of data and may time out the first time you run it. And if the offset is too short it will result in missed events, should the wodle stop running for longth than that period.

Restart the server for the changes to take effect, for example using the `Restart cluster` button in the `Server Management` > `Status` menu.

You should start seeing new events show up in the Threat hunting module. You can filter for `data.gworkspace.application: *` to make it easier to see.

