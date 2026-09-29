# SETUP.md — How to build Sandooq, step by step

This guide has three parts:

1. Turn a laptop into a server
2. Set up the server and reach it from the phone
3. Add encryption on the phone

You do not need all three to start. Part 1 is the simplest, Part 2 is the real product, Part 3 is what makes it private.

---

## Part 1. Turn your laptop into a server (Mac)

This is the fastest way to see the idea working. It only works on your own Wi-Fi.

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

**Why this is only step one:** an address like `192.168.1.11` is private. Nobody outside your Wi-Fi can reach it, and many Saudi home connections sit behind CGNAT, which means you do not even have your own public address. That is why the real product runs on a public server.

---

## Part 2. Set up the server and reach it from your phone

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
5. Tap **Save**, then tap the host to connect.
6. The first time it asks about a fingerprint. Type **yes** once.
7. You should land on a prompt like `dana@server:~$`. Type `pwd` and press Enter to confirm you are really on the server.

### 2.4 Install the cloud platform on the server

This is the software that turns a plain server into a cloud with accounts, quotas and apps. On Ubuntu or Debian:

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

Open `https://` plus your server IP in a browser. You will get a certificate warning with `self-signed`, that is expected. The smooth version is a domain name plus:

```
sudo nextcloud.enable-https lets-encrypt
```

Your server must have ports **80** and **443** open for this, and a domain name pointing at it.

### 2.5 Make accounts for other people

In the Nextcloud admin panel, go to **Users** and create one account per person, each with a **quota** (for example 5 GB). One person, one account. Never share your own login.

### 2.6 Use it from the phone

- Install the **Nextcloud** app (free) on the phone and log in with the account you created. Files sync automatically.
- On the **iPhone Files app**: it speaks **SMB** only. It cannot do SSH or SFTP directly. Two options:
  - **SMB:** if the server runs Samba, connect with Files, then **Connect to Server**, then type `smb://` plus the address.
  - **SFTP:** install a file provider app such as **Secure ShellFish** or **Owlfiles**, add an SFTP connection with the server IP, port `52156`, your username and your key. It then appears as a folder inside the Files app.

### 2.7 Optional, access without opening ports

Install **Tailscale** on the server, the phone and the laptop, and sign in with the same account:

```
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
tailscale ip -4
```

Now every device can reach the server at its `100.x.y.z` address from anywhere, and you can close the public SSH port if you want. Tailscale is private, not public.

---

## Part 3. Add encryption on the phone (Cryptomator)

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

**The rule to remember:** the vault protects your files from the storage and from other people. It does not hide them from you, because you hold the key. If you can open your own vault, that is normal.

**Warning:** if you lose the vault password, the files are gone forever. Nobody can reset it, not even the server owner.

---

## The whole process in one picture

```
Laptop or phone
   encrypts the file with your key
        |
        v
Encrypted tunnel (HTTPS / SSH / TLS)
        |
        v
Sandooq node (accounts, quotas, no key)
        |
        v
Storage (unreadable encrypted blocks)
        |
        v
Only your device can decrypt it again
```

**Supply side:** someone with spare storage installs the host app, declares capacity, passes a readiness check, goes live in the node list, and earns a monthly payout for the space actually used.

**Demand side:** a user signs up, installs the app, picks a node by price, latency and trust, creates a vault, and uploads.

---

## Honest limits

- A host can still see metadata: how many files, roughly how big, and when you were online.
- Anyone who controls a disk can delete data. Encryption stops reading, not deletion.
- Cryptomator's iPhone app over an SMB mount has known reliability issues. If it misbehaves, point the vault at a different storage connection, such as SFTP.
