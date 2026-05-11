# Blue Basalt 🪨💻

**A Hardened System Foundation for the Modern Defender.**

Blue Basalt is a manual, multi-phase deployment stack for Debian 13 (Trixie). This repository provides a structured sequence for establishing a hardened system environment—integrating network-level intrusion detection (PSAD), brute-force mitigation (Fail2Ban), and encrypted system alerting (msmtp) alongside a modernized suite of local AI, development, and geospatial tools. It serves as a comprehensive guide for transforming a base OS into a fortified and fully-equipped workstation.

  

## 

----------

🟦 **Phase 1: Repository & System Prep**

Configure system sources and install the official Docker engine.

### 1. System Sources

Open the sources list:

  
  
```bash
sudo nano /etc/apt/sources.list
```  
  

  

Paste exactly:
  
  
```Bash
deb http://deb.debian.org/debian/ trixie main contrib non-free non-free-firmware
deb http://security.debian.org/debian-security trixie-security main contrib non-free non-free-firmware  
deb http://deb.debian.org/debian/ trixie-updates main contrib non-free non-free-firmware
```  

Update the package database:

  
  

```Bash
sudo apt update  
```  

### 2. Official Docker Installation

Remove conflicting legacy packages:

```Bash
sudo apt remove docker.io docker-doc docker-compose podman-docker containerd runc  
```  

Install requirements and set up the repository:

  
  

```Bash
sudo apt install ca-certificates curl  
```
```Bash
sudo install -m 0755 -d /etc/apt/keyrings  
```
```Bash
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc  
```
```Bash
sudo chmod a+r /etc/apt/keyrings/docker.asc  
```
```Bash
echo  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian trixie stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null  
```  

Install Docker Engine:

  
  

```Bash
sudo apt update && sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin  
```  

## 

----------

🟦 **Phase 2: Software & Permissions**

Deployment of core security tools and firmware.

-   Direct .debs: Install Google Earth Pro and VirtualBox 7.2 using `wget` and `sudo apt install ./*.deb`  
      
    
-   Master Package List:  
      
    ```Bash  
    sudo apt install firmware-misc-nonfree firmware-realtek build-essential wireshark vlc mpv gimp keepassxc thunderbird hexchat gh nmap arp-scan btop ufw fail2ban flowblade audacity ffmpeg npm qgis librecad kiwix filezilla qbittorrent  
      ```
    
-   Wireshark Fix:
	- ```sudo dpkg-reconfigure wireshark-common``` and select YES.  
      
    
-   Permissions: Add user to necessary security and virtualization groups:  
      
    ```Bash  
    sudo usermod -aG docker,vboxusers,wireshark,plugdev $USER  
    ```  
    

## 

----------

🟦 **Phase 3: Deno, Ollama & yt-dlp Configuration**

Modern runtimes and media extraction settings.

-   Deno:  
	- ```curl -fsSL https://deno.land/install.sh | sh ``` 
      
    
-   Brave:  
	- ```curl -fsS https://dl.brave.com/install.sh | sh ``` 
      
    
-   Ollama:  
	- ```curl -fsSL https://ollama.com/install.sh | sh  ```
      
    

### yt-dlp Setup

Install the latest binary:

  
  

```Bash
sudo wget "https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp" -O "/usr/local/bin/yt-dlp"  
```
```Bash
sudo chmod +x "/usr/local/bin/yt-dlp"  
```  

Configure yt-dlp:

```Bash
mkdir -p ~/.config/yt-dlp && nano ~/.config/yt-dlp/config  
```  

  

Paste content exactly:

  
```  

--js-runtime deno  
-f "bestvideo[ext=mp4]+bestaudio[ext=m4a]/best[ext=mp4]/best"  
-o "~/Downloads/YouTube/%(uploader)s/%(playlist_title|'Misc')s/%(upload_date)s - %(title)s [%(id)s].%(ext)s"  
--embed-metadata  
--write-subs  
--sub-langs "en.*"  
 ``` 

## 

----------

🟦 **Phase 4: Secure Mail Transport (msmtp)**

Configuring secure system mail alerts.

1.  Install:  ```sudo apt install msmtp msmtp-mta ca-certificates``` 
      
    
2.  Config:  ```sudo nano /etc/msmtprc```  
      
    
3.  Settings:  
      
    ```  
    defaults  
    auth on  
    tls on  
    tls_trust_file /etc/ssl/certs/ca-certificates.crt  
    logfile /var/log/msmtp.log  
      
    account default  
    host smtp.gmail.com  
    port 587  
    from your-email@example.com  
    user your-email@example.com  
    password your-app-password  
    ```  
    
4.  Secure Permissions:  
      
    ```Bash  
    sudo chown root:msmtp /etc/msmtprc  
    ```
    ```Bash
    sudo chmod 640 /etc/msmtprc  
    ```
    ```Bash  
    sudo touch /var/log/msmtp.log  
    ```
    ```Bash
    sudo chown root:msmtp /var/log/msmtp.log  
    ```
    ```Bash
    sudo chmod 660 /var/log/msmtp.log  
    ```
    ```Bash
    sudo usermod -aG msmtp $USER  
    ```  
      
    (Note: You must log out and back in for group changes to take effect).  
      
    
5.  Fix AppArmor: Create a local override to allow logging:  
      
    ```Bash  
    sudo nano /etc/apparmor.d/local/usr.bin.msmtp  
    ```  
    Add: `/var/log/msmtp.log rwkl,`
    
    Reload:
    ```Bash 
    sudo apparmor_parser -r /etc/apparmor.d/usr.bin.msmtp  
    ```  
    

## 

----------

🟦 **Phase 5: Log Analysis (Logwatch)**

Automated system security reporting.

1.  Install:  ```sudo apt install logwatch```  
      
    
2.  Override:  ```sudo mkdir -p /etc/logwatch/conf && sudo nano /etc/logwatch/conf/logwatch.conf```  
      
    
3.  Paste content:  
      
    ```  
    Output = mail  
    MailTo = your-email@example.com  
    MailFrom = Logwatch  
    Detail = High  
    Range = yesterday  
    Service = All  
    mailer = "/usr/sbin/sendmail -t"  
     ``` 
    

## 

----------

🟦 **Phase 6: Active Defense (fail2ban & ufw)**

Hardening the network perimeter.

1.  Install:  ```sudo apt install fail2ban ufw ``` 
      
    
2.  Config:  ```sudo nano /etc/fail2ban/jail.local ``` 
      
    
3.  Paste Jail Settings:  
      
    ```  
    [DEFAULT]  
    ignoreip = 127.0.0.1/8 ::1 192.168.1.0/24  
    bantime = 1h  
    findtime = 10m  
    maxretry = 5  
    destemail = your-personal-email@example.com  
    sender = your-secondary-email@gmail.com  
    mta = sendmail  
    action = %(action_mwl)s  
      
    [sshd]  
    enabled = true  
    port = ssh  
    logpath = %(sshd_log)s  
    backend = %(sshd_backend)s  
    ```  
    
4.  Activate:  
      
    ```Bash  
    sudo systemctl restart fail2ban  
    ```
    ```Bash
    sudo systemctl enable --now ufw  
    ```
    ```Bash
    sudo ufw enable  
    ```
    ```Bash
    sudo ufw allow ssh  
    ```
    ```Bash
    sudo ufw logging medium  
    ```  
    

## 

----------

🟦 **Phase 7: Environment Polish & Cleanup**

Optimizing user workflow and removing bloat.

-   Bash Aliases: Add to ~/.bashrc:  
      
    ```Bash  
    alias ai='ollama run gemma4:e2b'  
    alias ytdlp-other='yt-dlp --js-runtime deno --no-config -o "~/Downloads/Videos/%(extractor_key)s/%(uploader|Unknown Uploader)s/%(title)s [%(id)s].%(ext)s"'  
    ```  
    
-   Cleanup:  
      
    ```Bash  
    sudo apt purge evolution  
    ```
    ```Bash
    sudo apt purge yt-dlp  
    ```
    ```Bash
    sudo apt autoremove && sudo apt clean  
    ```  
    
-   Verification:  
      
    

-   Test Mail: 
	- ```echo "Subject: Stack Test" | sendmail -v your-email@example.com```
    
-   Test Fail2ban: 
	- ```sudo fail2ban-client status sshd```
    

## 

----------

🟦 **Phase 8: Intrusion Detection (psad)**

Detecting port scans and suspicious activity.

1.  Install:  ```sudo apt install psad iproute2```  
      
    
2.  Config:  ```sudo nano /etc/psad/psad.conf```  
      
    
3.  Primary Settings:  
      
	```Bash    
	-   EMAIL_ADDRESSES: your-email@example.com;
	    
	-   HOME_NET: 192.168.1.0/24, 172.17.0.0/16, 10.0.2.0/24, 192.168.56.0/24;
	    
	-   IFCFGTYPE: iproute2;
	    
	-   IPT_SYSLOG_FILE: /var/log/ufw.log;
	    
	-   ALERT_ALL: N;
	    
	-   EMAIL_LIMIT: 5;
	    
	-   MIN_DANGER_LEVEL: 3;
	```

4.  Interface Monitoring: Add to bottom of file:  
      
    ```Bash
    PROTECT_INTERFACE enp2s0,wlp0s20f3;  
    HOME_NET_INTERFACES enp2s0,wlp0s20f3;  
    ```  
    
5.  Start & Check:  
      
    ```Bash  
    sudo psad --sig-update  
    ```
    ```Bash
    sudo systemctl restart psad  
    ```
    ```Bash
    sudo psad -S
    ```
