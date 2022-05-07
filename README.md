# ubuntu-yubikey
### Enjoy peace of mind and save time while getting better security.
- Do these steps in order to verify your installation, then consider following [Hardening Instructions](https://github.com/nruddick/ubuntu-yubikey/edit/main/README.md#hardening--) (Difference being your keys are installed in `/etc`) 

## ⚙️ Install 
- Install the libpam-u2f package, create the keys and sync with your yubikey:
```
sudo apt-get install libpam-u2f &&\
mkdir -p ~/.config/Yubico &&\
pamu2fcfg > ~/.config/Yubico/u2f_keys
```
- When your device begins flashing, touch the metal contact to confirm the association.

## 🔑🔑 Extra Keys
```
pamu2fcfg -n >> ~/.config/Yubico/u2f_keys
```
- When your device begins flashing, touch the metal contact to confirm the association.

---

# 🧯 Testing (do not skip)
### ⚠️ This will save you from accidentally getting locked out of Ubuntu
- Open a new Terminal. 
  - In the new Terminal, run: 
```
sudo echo test
```
  - When prompted, enter your password and press `Enter`. 
    - Even with the correct password, the authentication should fail as the U2F Key is not plugged in. 
    - If the authentication succeeds without the U2F Key, that indicates the U2F PAM module was not installed or there is a typo in the changes you made to `/etc/pam.d/sudo`. 
  - Insert your device.
  - Open a new Terminal and re-run:
```
sudo echo test
```
  - When prompted, enter your password and press `Enter`. 
    - Then, touch the metal contact on your U2F Key when it begins 🔑 flashing.
### Congratulations! 
- If the password was accepted this time you have configured the system correctly and can continue on to the next section for requiring the U2F Key to login. 
  - 🗒️ Note: if you do not want to require the U2F Key to run the `sudo` command, remove the line you added to the `/etc/pam.d/sudo` file.
    - ❔ Not sure if switching `required` to `optional` would do.

---

# 🤠 The Fun Part!

## 🧑‍💻 `sudo`
```
sudo vi /etc/pam.d/sudo
```
- Add this line below after the `@include common-auth` line:
```
auth       required   pam_u2f.so
```

## 💻 Desktop Login
- _If you use LightDM substitute:_ `/etc/pam.d/lightdm`
```
sudo vi /etc/pam.d/gdm-password
```
- Add this line below after the `@include common-auth` line:
```
auth       required   pam_u2f.so
```

## ⌨️ TTY
- I assumed this is to protect remote ssh
```
sudo vi /etc/pam.d/login
```
- Add the line below after the “@include common-auth” line.
```
auth       required   pam_u2f.so
```

## 📓 Logging
```
sudo touch /var/log/pam_u2f.log &&\
sudo vi /etc/pam.d/gdm-password
```
- At the line that contains `..pam_u2f.so` replace the line with:
```
auth       required   pam_u2f.so debug debug_file=/var/log/pam_u2f.log” 
```
- This adds ` debug debug_file=/var/log/pam_u2f.log”` to the end of the line.
- Each subsequent login event will have the debug log saved in the `/var/log/pam_u2f.log` file.
  - Check log: `tail /var/log/pam_u2f.log`

## 🔐 Hardening
- If you would like to add additional layer of security you can change the output of the u2f_keys file to an area of the OS where you'll need sudo permission to edit the file ( e.g. /etc ). 
  - After creating a directory named Yubico ( e.g. `/etc/Yubico` ). You can then move the file from `~/.config/Yubico` to `/etc/Yubico` by running:
```
sudo mv ~/.config/Yubico/u2f_keys /etc/Yubico/u2f_keys
```
  - Once the u2f_keys file is moved to a safer location the PAM file will need to be modified so that u2f PAM module can find the u2f_keys file, This is done by adding ` authfile=/etc/Yubico/u2f_keys` to the end of the line for `auth       required   pam_u2f.so`. 
  - ⚠️ Warning: Please note that once you modify the /etc/pam.d/sudo file to require the YubiKey if you were to lose or misplace the YubiKey you will not be able to modify or change the file to remove the YubiKey requirement. 
  - ⚙️ Normal:
```
auth       required   pam_u2f.so
```
  - ⚙️ Hardening:
```
auth       required   pam_u2f.so authfile=/etc/Yubico/u2f_keys
```
  - 🐛 Debugging:
```
auth       required   pam_u2f.so debug debug_file=/var/log/pam_u2f.log”
```
  - 🐛 Hardening + Debugging:
```
auth       required   pam_u2f.so authfile=/etc/Yubico/u2f_keys debug debug_file=/var/log/pam_u2f.log”
```
### Quick Edit
```
#sudo &&\
sudo vi /etc/pam.d/sudo &&\
#desktop login &&\
sudo vi /etc/pam.d/gdm-password &&\
#TTY login &&\
sudo vi /etc/pam.d/login
```
