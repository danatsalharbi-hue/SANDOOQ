# SETUP.md — How to build Sandooq, step by step

This guide has four parts:

1. Turn a laptop into a server
2. Set up the public server and reach it from the phone
3. Add Tailscale, the secure private network
4. Add encryption on the phone

Part 1 is the simplest. Part 2 is the real product. Part 3 is what makes it secure and usable from anywhere. Part 4 is what makes it private.

---

## Part 1. Turn your laptop into a server (Mac)

This is the fastest way to see the idea working. On its own it only works on your own Wi-Fi. Part 3 fixes that.

1. **Make a folder to share.** Open Finder and create a folder in your home folder, for example `Cloud`.
2. **Open File Sharing.** Apple menu, System Settings, General, Sharing, then turn **File Sharing** on.
3. **Enable SMB.** Click the **i** next to File Sharing, then **Options**, and tick **Share files and folders using SMB**. Tick your own account in the list. Click Done.
4. **Choose what to share.** Under **Shared Folders** click **+** and add your new `Cloud` folder.
5. **Set permissions.** On the right, under **Users**, set your account to **Read & Write**, and set **Everyone** to **No Access**.
6. **Write down the address.** The Sharing screen shows something like `smb://Danas-MacBook-Air.local`. Also get the IP address. Open Terminal and run:
   ```
   ipconfig getifaddr en0
   ```
   That prints your local address, like `192.168.1.11`.
7. **Optional, for SSH from the phone.** In the same Sharing screen, turn on **Remote Login**. Now the laptop also accepts SSH on port 22.
8. **Keep it awake.** System Settings, Lock Screen, and turn on **Prevent automatic sleeping on power adapter when the display is off**. Also turn on **Wake for network access** in Sharing.

**Result:** the laptop is now a storage server on your Wi-Fi.

**Why this is only step one:** an address like `192.168.1.11` is private. Nobody outside your Wi-Fi can reach it, and many Saudi home connections sit behind CGNAT, which means you do not even have your own public address. Do not try to fix that by opening ports on the router. Part 3 fixes it properly.

---

## Part 2. Set up the public server and reach it from your phone

This is the version that works from anywhere in Saudi Arabia.

### 2.1 Get access to the server

You need four things from the server owner (or from your hosting panel):

- the **IP address**
- the **SSH port** (ours is `52156`, not the usual 22)
- your **username**
- your **public key** added to their account

### 2.2 Create your SSH key on the laptop

Open Terminal and run:

```
ssh-keygen -t ed25519 -C "my-laptop"
```

Press Enter to accept the default location, then set a passphrase (recommended). Show the public key:

```
cat ~/.ssh/id_ed25519.pub
```

Copy that line and send it to the server owner. **Never send the private key**, the file without `.pub`.

### 2.3 Connect from the phone with Termius

1. Install **Termius** from the App Store.
2. Open **Keychain**. Create a new key (Ed25519) and name it `my-iphone`, or import an existing key.
3. Copy the **public key** from Termius and send it to the server owner too. You need one key per device.
4. Go to **Hosts**, tap **+**, and fill in:
   - **Address:** the server IP
   - **Port:** `52156`
   - **Username:** your username
   - **Key:** select `my-iphone` (switch authentication from Password to Key)
5. Tap **Save**, then tap the host to connect. The first time it asks about a fingerprint, type **yes** once.
6. You should land on a prompt like `dana@server:~$`. Type `pwd` and press Enter.

### 2.4 Install the cloud platform on the server

On Ubuntu or Debian:

```
sudo apt update
sudo apt install -y snapd
sudo snap install nextcloud
```

Then create your admin account and turn on HTTPS:

```
sudo nextcloud.manual-install your-admin-name 'a-strong-password'
sudo nextcloud.enable-https self-signed
```

Open `https://` plus your server IP in a browser. The certificate warning is expected with `self-signed`. The smooth version needs a domain name and ports **80** and **443** open:

```
sudo nextcloud.enable-https lets-encrypt
```

### 2.5 Make accounts for other people

In the Nextcloud admin panel, go to **Users** and create one account per person, each with a **quota** (for example 5 GB). One person, one account. Never share your own login.

### 2.6 Use it from the phone

- Install the **Nextcloud** app (free) and log in with the account you created. Files sync automatically.
- The **iPhone Files app** speaks **SMB** only; it cannot do SSH or SFTP directly. Two options:
  - **SMB:** if the server runs Samba, use Files, then **Connect to Server**, then `smb://` plus the address.
  - **SFTP:** install a file provider app such as **Secure ShellFish** or **Owlfiles**, add an SFTP connection with the server IP, port `52156`, your username and your key. It then appears as a folder inside Files.

---

## Part 3. Add Tailscale, the secure private network

This is the part that makes the project secure and removes the "only on my Wi-Fi" limitation. Tailscale builds a private encrypted network, called a **tailnet**, between your own devices using WireGuard. Each device gets a stable `100.x.y.z` address and must be authenticated to join.

### 3.1 Create the account

Go to **tailscale.com** and sign up (Google, GitHub, Microsoft or email). Use the **same account** on every device, because that account defines the network. The free tier covers roughly three users and one hundred devices, which is enough for a pilot.

### 3.2 Install it on the public server

Connect with Termius and run:

```
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

The second command prints a link. Open it, log in, and **approve** the device. Then find the server's private address:

```
tailscale ip -4
```

You will get something like `100.101.102.103`. Write it down.

### 3.3 Install it on the private node (the Mac) and the phone

- **Mac:** install Tailscale from tailscale.com/download or the Mac App Store, sign in, and turn it on. Then also run the same two commands in Terminal if you want it available to command line tools.
- **iPhone:** install **Tailscale** from the App Store, sign in with the same account, and turn the switch on.

Now open Tailscale on the phone: all three devices should appear on the same network.

### 3.4 Reach the private node from anywhere

Before Tailscale, the home Mac was reachable only from the same Wi-Fi. Now use its `100.x.y.z` address instead of `192.168.x.x`:

- **SSH:** in Termius, create a host with the `100.x.y.z` address of the Mac, port `22` (Remote Login must be on), and your Mac username.
- **Files app:** connect to `smb://100.x.y.z` and log in with your Mac account.

This works on mobile data, from another city, on a different Wi-Fi, with **no ports opened on the home router**.

### 3.5 Close the public SSH port on the server

This is the security win. Until now, port `52156` was exposed to the entire internet, where scanners try passwords constantly. Once you can reach the server over the tailnet:

1. Confirm you can SSH to the server using its `100.x.y.z` address.
2. Only then, close the public SSH port in the host firewall or control panel.

After this, the public internet can reach **only HTTPS on port 443**, which is what real users need. Administration, federation and backups all run inside the tailnet.

Optional trick: Tailscale has its own SSH, so you can skip key management for admin access:

```
sudo tailscale up --ssh
```

Test it before you rely on it.

### 3.6 Access control, briefly

In the Tailscale admin console you can:

- **approve** or remove devices,
- define **ACLs** so a device can only reach what it needs,
- see which device is connected and when.

For a project like this, keep it simple: operators reach everything, testers reach only the service port.

### 3.7 Optional, a public link for a demo day

For SAIF demo day, if you want a public `https://` link without opening ports:

1. In the admin console, go to **DNS** and turn on **HTTPS Certificates**.
2. On the server:
   ```
   sudo tailscale funnel 443 on
   ```
   It prints a public link such as `https://your-server.your-tailnet.ts.net`.

### 3.8 Troubleshooting

- `sudo tailscale status` shows your devices and their connection state.
- `sudo systemctl status tailscaled` tells you if the service is running.
- `sudo tailscale up --reset` makes it ask you to log in again.
- If a device shows offline, make sure the Tailscale app is running on it.

**Honest limit:** Tailscale is private, not public. Real users still reach the public node over HTTPS. The tailnet is for operators, nodes and invited testers.

---

## Part 4. Add encryption on the phone (Cryptomator)

This is the part that makes the storage unable to read your files.

1. **Install Cryptomator** on the iPhone (paid app) and, if you want, on the laptop (free).
2. Tap **+**, then **Create New Vault**.
3. Give it a name, for example `Private`.
4. When it asks **where** to store the vault, choose the **server or Nextcloud folder**, not just the phone. Create a folder for it inside the share.
5. Set a **strong vault password**. It must be different from your server or Nextcloud password. Save it in the **Passwords** app.
6. The vault now appears in Cryptomator. Tap it and **unlock** it.
7. A **Cryptomator** location appears inside the Files app. Copy your files **there**, not into the plain share. Encryption happens on the phone before anything is uploaded.
8. **Verify it.** On the laptop or server, open the folder that holds the vault. You should see scrambled names like `dirid.c9r` and files ending in `.c9r`. No images, no readable filenames.
9. **Test first.** Use two or three unimportant files before you trust it with anything important.

**The rule to remember:** the vault protects your files from the storage and from other people. It does not hide them from you, because you hold the key.

**Warning:** if you lose the vault password, the files are gone forever. Nobody can reset it, not even the server owner.

---

## The whole system in one picture

```
        Tailscale tailnet  (WireGuard, encrypted, device authenticated)

  Public node  <-------------->  Private node  <-------------->  Phone
  public IP, HTTPS for users      Mac at home, no ports open      admin + own files
  SSH closed to the internet      100.x.y.z address only          100.x.y.z address only

        The internet sees:  HTTPS on 443, and nothing else

  File path:
  device -> encrypt with your key -> encrypted tunnel -> node -> unreadable blocks
  -> only your device can decrypt it again
```

**Two different layers of protection, and both are needed:**

- **Tailscale** protects the network path and reduces exposure. It keeps attackers off the administration ports and lets private nodes work from anywhere.
- **Cryptomator** protects the contents. Even the operator, even a stolen disk, sees nothing readable.

---

## Honest limits

- A host can still see metadata: how many files, roughly how big, and when you were online.
- Anyone who controls a disk can delete data. Encryption stops reading, not deletion.
- Tailscale is private, not public, and the free tier has device limits.
- Cryptomator's iPhone app over an SMB mount has known reliability issues. If it misbehaves, point the vault at a different storage connection, such as SFTP.
