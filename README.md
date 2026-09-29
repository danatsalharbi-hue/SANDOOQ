# Sandooq

**Sovereign, end to end encrypted cloud storage, owned by the people who use it.**

A Saudi based alternative to paid iCloud storage, and a marketplace that turns idle storage into income.
Short abstract: see [`ABSTRACT.md`](ABSTRACT.md).

> **نبذة بالعربية:** صندوق منصة تخزين سحابي سعودية، مشفّرة من الطرف إلى الطرف. تمنح المستخدم مساحة خاصة على خوادم داخل المملكة، وتتيح لأي شخص يملك مساحة تخزين غير مستخدمة أن يؤجّرها ويحصل على دخل شهري. الخصوصية هنا جزء من التصميم: المشغّل لا يملك مفاتيح التشفير، لذلك لا يستطيع قراءة ملفات المستخدمين، ولا حتى أنا. الوصول إلى الخوادم الإدارية يتم عبر شبكة **Tailscale** الخاصة والمشفّرة، بحيث لا تُفتح منافذ غير ضرورية على الإنترنت.

---

## Why this exists

**I ran out of storage again.** There are only ever two options on the screen: delete photos I actually want, or pay for more space. I chose "pay" so many times that I stopped counting. Then one question would not go away:

> Why am I renting room on a computer I will never see, when I already own storage that sits there doing nothing?

That question became Sandooq.

## What Sandooq is

Sandooq has two halves, and they are the reason it is a product and not just a project:

- **Storage for users.** Self hosted, end to end encrypted cloud space, hosted inside Saudi Arabia, reachable from any device.
- **Income for hosts.** Anyone with spare capacity, an old laptop, an external drive, a NAS, can run a node and earn a monthly payout for the space they are not using.

Privacy is not a promise here. It is an architectural constraint: **the operator cannot read user files.**

## The process behind Sandooq

This is the exact journey a file takes:

```
1. Your device        the file is encrypted with your key, before it travels anywhere
        |
        v
2. Encrypted tunnel   HTTPS and TLS for users, SSH or the Tailscale tailnet for operators
        |
        v
3. Sandooq node       accounts, quotas and federation, but no key, so no way to read anything
        |
        v
4. Storage            unreadable encrypted blocks on disk
        |
        v
5. Only your device   can decrypt the file again
```

**On the supply side:** a host installs the host app, declares capacity, passes a readiness check for uptime, encryption and speed, goes live with a price and a trust score, and is paid monthly for the space actually used.

**On the demand side:** a user signs up, installs the app, picks a node by price, latency and trust, creates an encrypted vault, and uploads without thinking about any of the above.

**What the storage actually holds:**

```
dirid.c9r
bm6btr7rD3T77XmbqcYX...i1A=.c9r
eVTAQlXO25LMgAvH8HV...As=.c9r
Ca8nPRNRD_za8st_8nXB...eAl=.c9r
```

No readable names, no images, no text.

## Security architecture: two nodes, one private network

The proof of concept runs on **two nodes**, and this is deliberate, because the two of them prove different things.

| Node | What it is | How it is exposed |
|---|---|---|
| **Public node** | A rented server with a public IP address | HTTPS on port 443 for real users. SSH is **not** exposed to the internet. |
| **Private node** | A Mac at home | No public address at all. It used to work only when the phone was on the same Wi-Fi. |

Both nodes are joined to a **Tailscale tailnet**, a private encrypted network built on WireGuard. Each device gets a stable `100.x.y.z` address and must be authenticated before it joins.

**What this changes:**

- **Smaller attack surface.** The server's SSH port can be closed to the whole internet, so scanners and credential attacks have nothing to reach. Operators connect over the tailnet instead. The public internet sees HTTPS and nothing else.
- **The private node becomes useful.** The Mac at home is now reachable from the phone or from the server, from anywhere in the country, without opening a single port on the home router and without needing a public address. It goes from a Wi-Fi only experiment to a real second node.
- **Federation becomes real.** Two nodes on one private network can federate and back each other up over the tailnet, which turns the decentralisation claim into something that can be demonstrated.
- **Private beta for testers.** Invited testers can join the tailnet and use the service without any public exposure, which is exactly what a first pilot needs.

```
        Tailscale tailnet  (WireGuard, encrypted, device authenticated)

  Public node  <-------------->  Private node  <-------------->  Phone
  public IP, HTTPS for users      Mac at home, no ports open      admin + own files
  SSH closed to the internet      100.x.y.z address only          100.x.y.z address only

        The internet sees:  HTTPS on 443, and nothing else
```

**Important:** Tailscale does not replace end to end encryption. Cryptomator protects file **contents** from everyone, including the operator. Tailscale protects the **network path** and removes unnecessary exposure. They are two different layers, and the project needs both.

**Honest limit:** Tailscale is private, not public. Real users still reach the service over HTTPS on the public node. The tailnet is for operators, nodes and invited testers. Tailscale Funnel can publish a service publicly for a short demonstration.

## Setup guides

Full instructions live in **[SETUP.md](SETUP.md)**. There are four parts now:

| Part | What it covers | Time |
|---|---|---|
| **1. Laptop as a server** | Share a folder from a Mac over SMB, and reach it on your Wi-Fi | 10 minutes |
| **2. Server plus phone** | Create SSH keys, connect from the phone with Termius, install the cloud platform, create accounts, and reach it from the Files app | 30 minutes |
| **3. Tailscale, the secure layer** | Join both nodes and your phone to one private network, reach the home node from anywhere, and close the public SSH port | 15 minutes |
| **4. Encryption on the phone** | Create a Cryptomator vault on the server, upload into it, and verify the files are unreadable | 15 minutes |

Quick summary:

**1. Laptop as a server:** System Settings, General, Sharing, File Sharing on, then Options and tick SMB, then add a shared folder and set permissions. This alone only works on your own Wi-Fi, because private addresses such as `192.168.1.11` cannot be reached from outside. Part 3 fixes that.

**2. Server plus phone:** generate an Ed25519 key with `ssh-keygen`, send only the public key to the server, then in Termius create a host with the IP, port `52156`, your username and your key. Install the cloud platform with snap, turn on HTTPS, and create one account per person with a quota. The iPhone Files app speaks SMB only, so use SMB or an SFTP file provider app such as Secure ShellFish.

**3. Tailscale:** install it on the server, the Mac and the phone, sign in with one account, and note each device's `100.x.y.z` address. Now the home node is reachable from anywhere, the server's SSH port can be closed to the internet, and the two nodes can federate over the tailnet.

**4. Encryption on the phone:** in Cryptomator create a vault, store it inside the server or Nextcloud folder, set a strong vault password, unlock it, and copy files into the Cryptomator location in the Files app. Then open the storage folder and confirm you only see `.c9r` scrambled names.

## Who can read what

| Actor | Can read the files? | What they can see |
|---|---|---|
| The user | **Yes**, they hold the key | Everything they own |
| Node operator or host | **No** | Encrypted blocks, file counts, rough sizes, activity |
| Network eavesdropper | **No** | Scrambled traffic, and the admin traffic is inside the tailnet |
| Port scanner on the internet | **Nothing to reach** | HTTPS on the public node only, no SSH |
| Other users | **No** | Only what is explicitly shared |
| Thief with the disk | **No** | Unreadable storage |

**Stated honestly:** a host can still observe metadata, and whoever controls a disk can delete data. What no one can do is read it.

## Tech stack

| Layer | Used |
|---|---|
| Terminal and administration | Termius (iPhone, Mac), macOS Terminal, SSH keys (Ed25519) |
| Private networking | **Tailscale**, a WireGuard based tailnet, with Funnel for public demos |
| File access | Apple Files app, Samba (SMB), SFTP file providers (Secure ShellFish, Owlfiles) |
| Encryption | Cryptomator vaults (end to end), full disk encryption (at rest), HTTPS and TLS plus SSH or WireGuard (in transit) |
| Cloud platform | Nextcloud or Seafile, deployed with Docker or snap |
| Transport | SSH on a custom port, SFTP, HTTPS, SMB, WebDAV |

## Roadmap

| Phase | Goal |
|---|---|
| 0. Proof of concept | Two nodes joined by a tailnet, HTTPS for users, accounts, quotas, an encrypted vault, and a host rental demo |
| 1. Private pilot | 10 to 30 testers, invited into the tailnet, plus onboarding, backups and monitoring |
| 2. Multi user product | Self service signup, two factor, mobile apps, billing, support |
| 3. Federation | Multiple nodes federating over the tailnet, a node directory, the in app node picker |
| 4. Market launch | Host app, reputation, payouts, commission |
| 5. Scale | A national network, B2B and white label, partnerships, compliance |

## What this project needs next

Ordered by importance. The first three are the difference between a demo and a real product.

1. **One running instance, properly.** Installed on the public node, reached over HTTPS with a real domain, and with a working demo account. Everything else is theory until this exists.
2. **The two node setup joined by Tailscale, working end to end.** The public node serving users over HTTPS, the private node reachable only inside the tailnet, and the server's SSH port closed to the internet. This is the security story, so it has to be true, not theoretical.
3. **A minimum host flow.** A small script or app that lets someone plug in a drive, register capacity, accept encrypted writes, and see a usage report. The marketplace is the heart of the idea, so it needs a working minimum, not a plan.
4. **A one command installer.** Docker Compose or a shell script so anyone can reproduce the setup on their own machine. This is what turns one person's server into a project other people can run, and it is also what lets a judge check the work.
5. **Backups and a health check.** A second copy of the data, and a simple uptime page, so a live demo cannot fail on stage.
6. **A thirty second privacy proof.** A rehearsed sequence: upload a file, show the storage, show the `.c9r` names, decrypt it again. Same result every time, without improvising.
7. **Real numbers.** Cost per gigabyte, host payout, and the break even point. The figures in this README are illustrative, and judges will ask for the real ones.
8. **Arabic onboarding.** The first screen a Saudi user sees should be in Arabic, not English with a translation underneath.
9. **A privacy policy and terms of use.** Short, plain and honest, written with PDPL in mind. It also protects the project legally.
10. **Three real pilot users.** Not friends who say it is nice, but people who upload something real and come back a week later.
11. **A two minute demo video**, and a rehearsed three minute pitch, with the live demo also practised offline.

**What it does not need right now:** more slides, more ideas, or a bigger network. It needs one working instance, two nodes joined securely, and one host earning their first riyal.

## Business model

- **Subscriptions:** free starter tier, paid plans in SAR below mainstream providers.
- **Marketplace commission:** a percentage of every rental between a user and a host.
- **Family and group plans:** a shared pool, one payer, many users.
- **Schools and small businesses:** private, PDPL aligned storage.
- **Managed hosting and white label:** setup and maintenance for organisations, licensing to local brands and ISPs.

## Repository contents

- `README.md` — this file
- `ABSTRACT.md` — a one page abstract of the project, including the two node security architecture
- `SETUP.md` — the full step by step build guide, including the Tailscale layer
- `glossary.md` — a bilingual technical glossary
- `Sandooq-Report.pdf` — the project report for SAIF

## Status

Proof of concept. Two nodes exist: a public server reached over HTTPS, and a private Mac node. Tailscale joins them into one private encrypted network, which lets the public node keep its SSH port closed and makes the private node usable from anywhere. Next: the host earning demo, a second federated node, and the one command installer.

## Author

**Dana Alharbi** — SAIF project, Kingdom of Saudi Arabia.

*Sandooq. Storage you own. Privacy you can verify. Income that stays in the Kingdom.*
