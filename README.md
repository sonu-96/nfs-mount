# nfs-mount
```
## Step 1 — Downloading and Installing the Components - on the Host Server
sudo apt update
sudo apt install nfs-kernel-server
## On the Client
sudo apt update
sudo apt install nfs-common
## Step 2 — Creating the Share Directories on the Host
#### Example 1: Exporting a General Purpose Mount
sudo mkdir /var/nfs/demo -p
ls -dl /var/nfs/demo
Output
drwxr-xr-x 2 root root 4096 Apr 17 23:51 /var/nfs/demo
--- NFS will translate any root operations on the client to the nobody:nogroup credentials as a security measure. Therefore, you need to change the directory ownership to match those credentials. ---
sudo chown nobody:nogroup /var/nfs/demo
ls -dl /var/nfs/demo
Output
drwxr-xr-x 2 nobody nogroup 4096 Apr 17 23:51 /var/nfs/demo
## Step 3 — Configuring the NFS Exports on the Host Server
sudo vim /etc/exports
/var/nfs/demo    client_ip(rw,sync,no_subtree_check) --- put into exports file ---
sudo systemctl restart nfs-kernel-server
## Step 4 — Adjusting the Firewall on the Host
sudo ufw status
Output
Status: inactive
sudo ufw enable
sudo ufw status
Output
Status: active
sudo ufw allow ssh --- for preventing not accessing ssh after firewall allow for nfs ---
sudo ufw allow from client_ip to any port nfs
sudo ufw status
Output
Status: active

To                         Action      From
--                         ------      ----
SSH                        ALLOW       Anywhere                 
2049                       ALLOW       32.0.13.24        
SSH (v6)                   ALLOW       Anywhere (v6)
## Step 5 — Creating Mount Points and Mounting Directories on the Client
sudo mkdir -p /nfs/demo
sudo mount host_ip:/var/nfs/demo /nfs/demo
df -h
Output
Filesystem                   Size  Used Avail Use% Mounted on
tmpfs                        198M  972K  197M   1% /run
/dev/vda1                     50G    5G   45G  10% /
tmpfs                        900M     0  900M   0% /dev/shm
tmpfs                        5.0M     0  5.0M   0% /run/lock
/dev/vda15                   105M  5.3M  100M   5% /boot/efi
tmpfs                        198M  4.0K  198M   1% /run/user/1000
10.108.0.7:/var/nfs/demo      10G  3G      7G  30% /nfs/demo

du -sh /nfs/demo --- This shows us that the contents of the entire demo directory is using the available space.

## Step 6 — Testing NFS Access create file demo.txt
sudo touch /nfs/demo/demo.test
ls -l /nfs/demo/demo.test
Output
-rw-r--r-- 1 nobody nogroup 0 Apr 18 00:02 /nfs/demo/demo.test
## Step 7 — Mounting the Remote NFS Directories at Boot
sudo vim /etc/fstab
host_ip:/var/nfs/demo    /nfs/demo   nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
man nfs   ---- for checking more details of nfs ----
## Step 8 — Unmounting an NFS Remote Share
sudo umount /nfs/demo
df -h
Output
Filesystem                   Size  Used Avail Use% Mounted on
tmpfs                        198M  972K  197M   1% /run
/dev/vda1                     50G    5G   45G  10% /
tmpfs                        900M     0  900M   0% /dev/shm
tmpfs                        5.0M     0  5.0M   0% /run/lock
/dev/vda15                   105M  5.3M  100M   5% /boot/efi
tmpfs                        198M  4.0K  198M   1% /run/user/1000

```
