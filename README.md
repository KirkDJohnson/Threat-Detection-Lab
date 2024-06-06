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
On the 118th <br/>
<img src="" height="100%" width="100%" alt="Threat Detection Lab"/>
<br />
<br />
Text<br/>
<img src="" height="100%" width="100%" alt="Threat Detection Lab"/>
<br />
<br />
Text<br/>
<img src="" height="100%" width="100%" alt="Threat Detection Lab"/>
<br />
<br />
Text<br/>
<img src="" height="100%" width="100%" alt="Threat Detection Lab"/>
<br />
<br />

<h2>Thoughts</h2>
Text

