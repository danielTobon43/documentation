---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.3'
      jupytext_version: 1.13.8
  kernelspec:
    display_name: Python 3
    name: python3
---

```python id="7LkFelkcVVeM"
# Created by Imad El Hanafi : https://imadelhanafi.com
```

<!-- #region id="7KYvm7elVabs" -->
# Check GPU status

Make surre to use : GPU runtime mode (Runtime->Change Runtime type -> python3 + GPU
)


<!-- #endregion -->

```python id="-nwY1_gYVcpc" colab={"base_uri": "https://localhost:8080/"} outputId="96cc1a2f-bfbe-4ef5-d252-0efe3046b9f1"
# Check nvidia and nvcc cuda compiler

!nvidia-smi
!/usr/local/cuda/bin/nvcc --version
```

<!-- #region id="FT66Rd6yV3ib" -->
#Mount Goolge Drive
<!-- #endregion -->

```python id="tUCE2A_DVeMe" colab={"base_uri": "https://localhost:8080/"} outputId="7a56a501-5e70-4bf4-a810-4f06cc97bfd2"
# link to google drive

from google.colab import drive
drive.mount('/content/gdrive/')

```

```python id="7j2NLxcrV9hj"
#check that Gdrive is mounted

!ls '/content/gdrive/My Drive/ssh_files/'
```

<!-- #region id="Ftyme-AIYFgK" -->
#Setup SSH port forwarding
<!-- #endregion -->

```python id="C0t3EVVaWbUJ" colab={"base_uri": "https://localhost:8080/"} outputId="fcd080fb-65ad-4910-9453-f52c55124dc7"
#1 - setup ssh/user 


#Generate a random root password
import random, string
password = ''.join(random.choice(string.ascii_letters + string.digits) for i in range(30))


#Setup sshd
! apt-get install -qq -o=Dpkg::Use-Pty=0 openssh-server pwgen > /dev/null

#Set root password
! echo root:$password | chpasswd
! mkdir -p /var/run/sshd
! echo "PermitRootLogin yes" >> /etc/ssh/sshd_config
! echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config

print("username: root")
print("password: ", password)

#Run sshd
get_ipython().system_raw('/usr/sbin/sshd -D &')
```

```python id="H_1iQGAtYvCq"
# 2 - Download Ngrok

! wget -q -c -nc https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip
! unzip -qq -n ngrok-stable-linux-amd64.zip
```

```python id="68P192JwZBtF" colab={"base_uri": "https://localhost:8080/"} outputId="d38dd72a-5227-4934-81ba-7f8af9195f66"
# 3 - setup Ngrok - authtoken

#Ask token
print("Get your authtoken from https://dashboard.ngrok.com/auth")
import getpass
authtoken = getpass.getpass()

#Create tunnel
get_ipython().system_raw('./ngrok authtoken $authtoken && ./ngrok tcp 22 &')
```

<!-- #region id="N5sGgljXaTjL" -->
Congratulations, you are ready to go. 
On Ngrok interface https://dashboard.ngrok.com/status you'll find the tcp address and the port 

connect using the following : 

```
ssh root@0.tcp.ngrok.io -p [ngrok_port]

> then enter the password generated previously

```
<!-- #endregion -->

```python id="Lz16iXUw0xJk"
export LD_PRELOAD=/usr/lib64-nvidia//libnvidia-ml.so
```

```python id="ACzGZ2_MaSQi"
# When done, kill Ngrok

!kill $(ps aux | grep './ngrok' | awk '{print $2}')
```
