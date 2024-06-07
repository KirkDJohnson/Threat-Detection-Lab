<h1>Threat Detection Lab IN PROGRESS</h1>



<h2>Description</h2> 
Text

<br />


<h2>Languages and Utilities Used</h2>

- <b>jq</b> 
- <b>Browserling</b>
- <b>Linux Terminal</b>
- <b>Virus Total</b>
- <b>Wireshark</b>
<h2>Environments Used </h2>

- <b>Oracle Virtual Box for Virtual Machine</b>
- <b>Kali Linux</b>
<h2>Lab Overview</h2>

To begin I moed the files to file to a new folder, unzipped it, and used the cat command to examine the log file and found that it is in json format.<br/>
<img src="https://github.com/KirkDJohnson/Threat-Detection-Lab/assets/164972007/b577e4d7-1e4c-4a00-b094-66045d3429bd" height="100%" width="100%" alt="Threat Detection Lab"/>
<br />
<br />
Knowing that reviewing the logs in this format would be tedious and ineeficient, I used the tool jq which is a json processer. What initally caught my eye is signature field within alert which I was investigate further to find what potentially occured.<br/>
<img src="https://github.com/KirkDJohnson/Threat-Detection-Lab/assets/164972007/a9520706-ce01-4ef1-8d74-df0ef06adf3a" height="100%" width="100%" alt="Threat Detection Lab"/>
<img src="https://github.com/KirkDJohnson/Threat-Detection-Lab/assets/164972007/d36659aa-7ad4-488f-9a5b-86d2b8eb20d5" height="100%" width="100%" alt="Threat Detection Lab"/>
<br />
<br />
With jq I filterd for all logs from the alert.signature, sorted them by the number in which they appeared in the logs to see what alert signautre was triggered the most without the duplicates. What I dicvoered from this is a large amount ofg web traffic from the user agent python-requests a suspicous user agent compared to firefox or chrome, and the second occurence being a potentional exploit of a known CVE.<br/>
<img src="https://github.com/KirkDJohnson/Threat-Detection-Lab/assets/164972007/32458183-48ec-43f3-a815-3f5a30d1abde" height="100%" width="100%" alt="Threat Detection Lab"/>
<br />
<br />
After noting the CVE for further investigation, I pivoted to the IP traffic seeing who which address had the most connections with the host, where I did a similar search of sorting them by the number of time they appear in the logs. I then filterd the alert singature by IP address to match the IP to any mlicious behavior.<br/>
<img src="https://github.com/KirkDJohnson/Threat-Detection-Lab/assets/164972007/bc629936-bd24-429a-be94-c2544ce11a8a" height="100%" width="100%" alt="Threat Detection Lab"/>
<img src="https://github.com/KirkDJohnson/Threat-Detection-Lab/assets/164972007/f98116c0-709f-4640-be3a-a3e4ca697ae2" height="100%" width="100%" alt="Threat Detection Lab"/>
<br />
<br />
Now I know the suspected malicoius IP addresses, I conducted further threat intelligence on the suspected CVE used to attack the webserver. CVE-2022-25237 was ranked a 9.8 critical vulnerability as by appending a string to end of a URL, users with no privileges can access privilged API endpoints ultimately allowing for remote code execution. Other arrticle and the proof of concept also mention the string "il8ntranslation" with either a forward slash or semi-colobn in front which causes the exploit. <br/>
<img src="https://github.com/KirkDJohnson/Threat-Detection-Lab/assets/164972007/5d831bcc-a6bb-4e29-a4cb-45ea8180dffa" height="100%" width="100%" alt="Threat Detection Lab"/>
<img src="https://github.com/KirkDJohnson/Threat-Detection-Lab/assets/164972007/3f869dda-105e-4bcd-8806-72a7c7a5f466" height="100%" width="100%" alt="Threat Detection Lab"/>
<br />
<br />
With what I had uncovered in the logs and threat intelligence I moved to the packet capture to try and see when the attack happened and other possible indicators of attack or compromise. Looking at statistics -> conversations and sorting by the number of packets, the IP from the logs with the python-requests and CVE staging was the top talker. Taking a bit of a detour and looking at Virus Total for any community information about this IP showed that ut was clean but related to a malicious trojan. <br/>
<img src="https://github.com/KirkDJohnson/Threat-Detection-Lab/assets/164972007/46a5b0bc-9c62-4ec1-aa53-3a4bb7df2659" height="100%" width="100%" alt="Threat Detection Lab"/>
<img src="https://github.com/KirkDJohnson/Threat-Detection-Lab/assets/164972007/7051e8ac-b26e-4f7a-8bb7-bd80d5c053f1" height="100%" width="100%" alt="Threat Detection Lab"/>
<br />
<br />
Filtering for the suspecetd malicious IP, first showed a considerable amount of TCP SYN requests targeting known ports in which the server responded with RST, ACK, potnetionally indicating a port scan. However, knowing attakc was agaisnt a web server, I added http to the filter and discovered a lot of POST requst traffic to the server. <br/>
<img src="https://github.com/KirkDJohnson/Threat-Detection-Lab/assets/164972007/7d003096-04e6-4723-8d3e-f148c41e6c92" height="100%" width="100%" alt="Threat Detection Lab"/>
<img src="https://github.com/KirkDJohnson/Threat-Detection-Lab/assets/164972007/d01d58c9-d666-4a4c-8ee4-fb289d9f0fae" height="100%" width="100%" alt="Threat Detection Lab"/>
<br />
<br />
Follwing the first HTTP streams shows the a user logging into "/bontia/loginservice" with a username of a complex email and password. Examining more HTTP streams shows similar results, the user/attacker is using different credntials against the login service. However, rather than being a brute force attack of trying the most common passwords and usernames, this attacks appears to be crential dumping where already compromised centials are then used in a seperate attack as here.<br/>
<img src="https://github.com/KirkDJohnson/Threat-Detection-Lab/assets/164972007/7ed48469-5bb5-4ab8-9bc6-28d987ecae12" height="100%" width="100%" alt="Threat Detection Lab"/>
<img src="https://github.com/KirkDJohnson/Threat-Detection-Lab/assets/164972007/5cea7fc6-7faf-4d52-b0da-50958eef36f2" height="100%" width="100%" alt="Threat Detection Lab"/>
<br />
<br />
On the 118th login attempt, the attack successfully login into the webserver with the credentials of "seb.broom" signified by the HTTP 204 response. Next in the HTTP stream, a POST request to the bontia/API/pageUpload, the attacker appended to the URL ";il18ntranslation" which as seen earlier confirms the attacker leverageing CVE-2022-25237. <br/>
<img src="https://github.com/KirkDJohnson/Threat-Detection-Lab/assets/164972007/8ea484b7-9adc-4a91-b37c-b1dc82a7186c" height="100%" width="100%" alt="Threat Detection Lab"/>
<img src="(https://github.com/KirkDJohnson/Threat-Detection-Lab/assets/164972007/52556617-65fa-4c9c-8afa-92a421d9cce7" height="100%" width="100%" alt="Threat Detection Lab"/>
<img src="https://github.com/KirkDJohnson/Threat-Detection-Lab/assets/164972007/d7bbde97-63dd-4b38-9b3d-3e489f2b7e55" height="100%" width="100%" alt="Threat Detection Lab"/>
<br />
<br />Within the log file and Wireshark, the IP address 138[.]199[.]59[.]221 was seen using "python-request" into the webserver the same user agent by the other malicious IP and in Wireshark had similar amount of traffic ot the webserver so I filered for that IP with HTTP to analyze the traffic. Examing the first few packets and it is immedaitely clear that this is a malicious user, likely the same threat actor as the previous IP as there were POST requesets to the /bonita/loginservice followed by "i18ntranslation" appended to the URL.<br/>
<img src="https://github.com/KirkDJohnson/Threat-Detection-Lab/assets/164972007/c5116953-9bce-40d7-9243-16c8bcfebf15" height="100%" width="100%" alt="Threat Detection Lab"/>
<br />
<br />
With it now clear that this IP was also involved in the attack, I begun following the HTTP streams and discovered that it found the this IP used the saem correct cernditals first to gain access to the webserver after that used the exploit<br/>
<img src="https://github.com/KirkDJohnson/Threat-Detection-Lab/assets/164972007/7df163c6-6103-4b2c-93f7-a39183ee063b" height="100%" width="100%" alt="Threat Detection Lab"/>
<br />
<br />
Noticing this IP address had HTTP GET requests I filtered for it and discovered three GET requests, one to cat the /etc/passwd file, a wget command to the website pastes.io a file sharing website, and a bash command to run this donwloaded file. When following both HTTP streams it was apparent that both GET requests were successful given the HTTP 200 response meaning OK.<br/>
<img src="https://github.com/KirkDJohnson/Threat-Detection-Lab/assets/164972007/5dcfb06b-40d0-4239-a93c-54149aa151fb" height="100%" width="100%" alt="Threat Detection Lab"/>
<img src="https://github.com/KirkDJohnson/Threat-Detection-Lab/assets/164972007/65a14d63-fb16-42b4-90a8-8de0add5e859" height="100%" width="100%" alt="Threat Detection Lab"/>
<img src="https://github.com/KirkDJohnson/Threat-Detection-Lab/assets/164972007/ad2e95a1-c6a1-4e0c-b0c2-05f84200b550" height="100%" width="100%" alt="Threat Detection Lab"/>
<br />
<br />
Using the cat command with /etc/passwd with show all the user accounts of the webserver, so I pivoted my focus to the wget command of the pastes.io website to see what was downloaded. Using the website Browserling, allows users to see render the webpage in a sandboxxed enviornment. <br/>
<img src="https://github.com/KirkDJohnson/Threat-Detection-Lab/assets/164972007/b264f477-b4f1-4996-8aab-54cd4e81cb9b" height="100%" width="100%" alt="Threat Detection Lab"/>
<img src="https://github.com/KirkDJohnson/Threat-Detection-Lab/assets/164972007/3260b5e1-036f-45ff-8ede-626d2d4de664" height="100%" width="100%" alt="Threat Detection Lab"/>
<img src="" height="100%" width="100%" alt="Threat Detection Lab"/>
<br />
<br />
Within Wireshark, after the script was run, it is clear the attacker established an ssh session where their activity was ecnrypted. <br/>
<img src="" height="100%" width="100%" alt="Threat Detection Lab"/>
<br />
<br />
Lastly, within Wireshark, after the ssh session, the attacker downloads and uses nmap, a network scanner, and then ran an TCP SYN scan likely to identify other networks and ports to pivot to, to continue the attack.<br/>
<img src="https://github.com/KirkDJohnson/Threat-Detection-Lab/assets/164972007/49471a23-9d91-4e5a-b110-3f8c27a047e4" height="100%" width="100%" alt="Threat Detection Lab"/>
<img src="https://github.com/KirkDJohnson/Threat-Detection-Lab/assets/164972007/409aa18e-8a9f-470e-9b50-94773963069e" height="100%" width="100%" alt="Threat Detection Lab"/>
<br />
<br />
Once the investigation was complete, I search the MITRE ATT&CK framework and found that the attacker used Technqiue 1098.004, manipulation of SSH auithorized keys.<br/>
<img src="https://github.com/KirkDJohnson/Threat-Detection-Lab/assets/164972007/8895a1a2-30c0-4453-9cb3-4c2f955d8fb4" height="100%" width="100%" alt="Threat Detection Lab"/>
<br />
<br />

<h2>Thoughts</h2>
This CTF/walkthrough was similar to many I have done in the past; however, it introduced me to a very useful tool, jq, which I will use in the future for dealing with large JSON files. Moreover, the given scenario and questions provided just enough information to complete the CTF without hand-holding, allowing me to use Wireshark and conduct the threat intelligence myself on the CVE. I then switched to Wireshark to uncover how the CVE was used by multiple IP addresses as part of the attack chain. The initial access was a credential stuffing brute force attack, followed by the exploitation of CVE-2022-25237. The attacker then modified the SSH authorized keys for persistence and used cat /etc/passwd as part of their objective. The CTF gave me a great view of how an attacker moved through the kill chain and used several different avenues in their attacks. The more labs and CTFs like these I do, the more confident I become with Wireshark.

