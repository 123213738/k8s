```bash
sudo groupadd -g 2000 denny
sudo useradd -u 2001 -m denny -g denny
systemctl disable firewalld
systemctl stop firewalld
```
