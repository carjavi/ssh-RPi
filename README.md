<p align="center"><img src="./img/ssh.png" height="200" alt=" " /></p>
<h1 align="center"> SSH RPi </h1> 
<h4 align="right">February 24</h4>
<img src="https://img.shields.io/badge/OS-Linux%20GNU-yellowgreen">
<img src="https://img.shields.io/badge/OS-Windows%2011-blue">

![Raspberry Pi](https://img.shields.io/badge/-RaspberryPi-C51A4A?style=for-the-badge&logo=Raspberry-Pi)
![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)

<br>

# Remote access RPi
## Para habilitar SSH en RPi Raspbian, puedes hacer lo siguiente:
Abrir una ventana de terminal

1. Escribir ```sudo raspi-config``` en la ventana de terminal
2. Seleccionar ```Opciones de interfaz```
3. Navegar y elegir ```SSH```
4. Elegir la opción ```Yes```
5. Pulsar ```OK```
6. Finalmente, pulsar ```Finish``` 

Alternativamente podemos usar ```systemctl``` para iniciar el servicio
```
sudo systemctl enable ssh 
sudo systemctl start ssh
```

### Habilitar SSH sin conexión
No es necesario que la Raspberry Pi esté conectada para la habilitación del SSH. Podemos hacerlo colocando un archivo de nombre «ssh», sin extensión alguna, en la partición de arranque de una tarjeta SD desde otro dispositivo. Cuando se inicie la Raspberry Pi realiza una búsqueda del archivo «ssh». Si encuentra el archivo, el SSH pasará a habilitarse y el archivo se suprimirá. Lo que haya dentro del archivo no importa en absoluto, pudiendo estar completamente vacío.

## Ver usuarios/hostname/ip
### users
```
cat /etc/passwd | grep home                             //List Only Users With Home Directories
cat /etc/passwd                                         //List All Users Including System Users 
awk -F ":" '/home/ {print $1}' /etc/passwd | sort       //List Only Usernames
```
### hostname
```
hostname
```
### ip
```
hostname -I
ifconfig
```



## Estableciendo conexión SSH
```
ssh <USER>@<IP-ADDRESS>
ssh hostname@ip
```
Pueden pasar ahora dos cosas:

1. Si recibimos un ```connection timed out erro```r existe la posibilidad que la IP introducida esté equivocada. Deberemos corregirla.
2. Si la conexión tiene éxito, veremos una advertencia de seguridad/autenticidad. Debemos escribir ```yes``` para continuar. Esta advertencia solo aparecerá la primera vez que nos conectamos.

> :memo: **Note:** Next you will be prompted for the password for the pi login: the default password on Raspberry Pi OS is raspberry.

<br>


# Passwordless SSH Access

It is possible to configure your Raspberry Pi to allow access from another computer without needing to provide a password each time you connect. To do this, you need to use an SSH key instead of a password. To generate an SSH key:

Checking for Existing SSH Keys
First, check whether there are already keys on the computer you are using to connect to the Raspberry Pi:
```
ls ~/.ssh
```
If you see files named id_rsa.pub or id_dsa.pub then you have keys set up already, so you can skip the 'Generate new SSH keys' step below.

## Generate new SSH Keys
To generate new SSH keys enter the following command:
```
ssh-keygen
```
Upon entering this command, you will be asked where to save the key. We suggest saving it in the default location (~/.ssh/id_rsa) by pressing Enter.

You will also be asked to enter a passphrase, which is optional. The passphrase is used to encrypt the private SSH key, so that if someone else copied the key, they could not impersonate you to gain access. If you choose to use a passphrase, type it here and press Enter, then type it again when prompted. Leave the field empty for no passphrase.

Now look inside your .ssh directory:
```
ls ~/.ssh
```
and you should see the files ```id_rsa``` and ```id_rsa.pub```:
```
authorized_keys  id_rsa  id_rsa.pub  known_hosts
```
The id_rsa file is your private key. Keep this on your computer.

The id_rsa.pub file is your public key. This is what you share with machines that you connect to: in this case your Raspberry Pi. When the machine you try to connect to matches up your public and private key, it will allow you to connect.

Take a look at your public key to see what it looks like:
```
cat ~/.ssh/id_rsa.pub
```
It should be in the form:
```
ssh-rsa <REALLY LONG STRING OF RANDOM CHARACTERS> user@host
```


## Copy your Key to your Raspberry Pi
Using the computer which you will be connecting from, append the public key to your authorized_keys file on the Raspberry Pi by sending it over SSH:
```
ssh-copy-id <USERNAME>@<IP-ADDRESS>
```
> :memo: **Note:** During this step you will need to authenticate with your password.

Alternatively, if ssh-copy-id is not available on your system, you can copy the file manually over SSH:

```
cat ~/.ssh/id_rsa.pub | ssh <USERNAME>@<IP-ADDRESS> 'mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys'
```

If you see the message ssh: ```connect to host <IP-ADDRESS>port 22: Connection refused```  and you know the IP-ADDRESS is correct, then you may not have enabled SSH on your Raspberry Pi. Run ```sudo raspi-config``` in the Raspberry Pi’s terminal window, enable SSH, then try to copy the files again.

Now try ```ssh <USER>@<IP-ADDRESS>``` and you should connect without a password prompt.

If you see a message "Agent admitted failure to sign using the key" then add your RSA or DSA identities to the authentication agent ssh-agent then execute the following command:

```
ssh-add
```

> :memo: **Note:* You can also send files over SSH using the scp (secure copy) command.

## Adjust Directory Permissions
If you can’t establish a connection after following the steps above there might be a problem with your directory permissions. First, you want to check the logs for any errors:
```
journalctl -f
# might return:
Nov 23 12:31:26 raspberrypi sshd[9146]: Authentication refused: bad ownership or modes for directory /home/pi
```
If the log says Authentication refused: bad ownership or modes for directory /home/pi there is a permission problem regarding your home directory. SSH needs your home and ~/.ssh directory to not have group write access. You can adjust the permissions using chmod:
```
chmod g-w $HOME
chmod 700 $HOME/.ssh
chmod 600 $HOME/.ssh/authorized_keys
```
Now only the user itself has access to .ssh and .ssh/authorized_keys in which the public keys of your remote machines are stored.


<br>
<br>

# Setting SSH Ubuntu Server 20.04 on RPi

1. Create Folder <br>
Create the directory if required
```
sudo mkdir ~/.ssh
sudo nano ~/.ssh/authorized_keys
```

2. Permissions
```
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

3. Verify Password Authentication
```
cat /etc/ssh/sshd_config | grep PasswordAuthentication
```
Debe decir "Yes". sino cambiar con el siguiente comando:

```
sudo nano /etc/ssh/sshd_config
```
Change ```PasswordAuthentication``` from no to ```Yes```. Save the file and restart the SSH service

4. Service ssh restart
```
sudo systemctl restart ssh
```

> :bulb: **Tip:** Copying the Public Key to Your Ubuntu Server and create folder (one command)
```
cat ~/.ssh/id_rsa.pub | ssh username@remote_host "mkdir -p ~/.ssh && touch ~/.ssh/authorized_keys && chmod -R go= ~/.ssh && cat >> ~/.ssh/authorized_keys"
```


Creating and Using SSH Keys with GUI: https://www.purdue.edu/science/scienceit/ssh-keys-windows.html

<br>

## Troubleshooting SSH UBUNTU 20.4

### SSH Permission Denied (publickey) 
probar si esta corriendo el servivio de SSH
```
sudo systemctl status sshd
```

ver la llave publica
```
cat ~/.ssh/id_rsa.pub
```

### Password Authentication
If public key authentication is not working, you can temporarily enable password authentication to troubleshoot further. Open the SSH configuration file on the remote server:
```
sudo nano /etc/ssh/sshd_config
```
Change ```PasswordAuthentication``` from no to ```Yes```. Save the file and restart the SSH service

```
sudo systemctl restart ssh
```

<br>


## How to Set Up SSH Key on Windows 10 
(with PowerShell / GitBash Terminal)

1. generate SSH keypair
```
ssh-keygen
```

2. Copying the Public Key to Your Ubuntu Server
```
ssh-copy-id username@server_ip
```
si hay error revisar Troubleshooting

 3. Authenticating to Your Ubuntu Server Using SSH Keys
```
ssh username@remote_host
```



<br>

## Copying Files to your Raspberry Pi from SSH
```
scp myfile.txt pi@192.168.1.3:
```
Copy the file to the /home/pi/project/ directory on your Raspberry Pi (the project folder must already exist):
```
scp myfile.txt pi@192.168.1.3:project/
```
### Copying Files from your Raspberry Pi
Copy the file myfile.txt from your Raspberry Pi to the current directory on your other computer:
```
scp pi@192.168.1.3:myfile.txt .
Copying Multiple Files
scp myfile.txt myfile2.txt pi@192.168.1.3:
scp *.txt pi@192.168.1.3:
scp m* pi@192.168.1.3:
scp m*.txt pi@192.168.1.3:

```

### Copying a Whole Directory
Copy the directory project/ from your computer to the pi user’s home folder of your Raspberry Pi at the IP address 192.168.1.3
```
scp -r project/ pi@192.168.1.3:
```

<br>

---
Copyright &copy; 2022 [carjavi](https://github.com/carjavi). <br>
```www.instintodigital.net``` <br>
carjavi@hotmail.com <br>
<p align="center">
    <a href="https://instintodigital.net/" target="_blank"><img src="./img/developer.png" height="100" alt="www.instintodigital.net"></a>
</p>




