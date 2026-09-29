# Sandooq

**Sovereign, end to end encrypted cloud storage, owned by the people who use it.**

A Saudi based alternative to paid iCloud storage, and a marketplace that turns idle storage into income.

> **نبذة بالعربية:** سندوق منصة تخزين سحابي سعودية، مشفّرة من الطرف إلى الطرف. تمنح المستخدم مساحة خاصة على خوادم داخل المملكة، وتتيح لأي شخص يملك مساحة تخزين غير مستخدمة أن يؤجّرها ويحصل على دخل شهري. الخصوصية هنا جزء من التصميم: المشغّل لا يملك مفاتيح التشفير، لذلك لا يستطيع قراءة ملفات المستخدمين، ولا حتى أنا.

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
2. Encrypted tunnel   HTTPS and TLS for apps and web, SSH for administration and transfer
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
Ca8nPrNRD_za8st_8nXB...eAl=.c9r
```

No readable names, no images, no text.

## Setup guides

Full instructions live in **[SETUP.md](SETUP.md)**. There are three parts:

| Part | What it covers | Time |
|---|---|---|
| **1. Laptop as a server** | Share a folder from a Mac over SMB, and reach it on your Wi-Fi | 10 minutes |
| **2. Server plus phone** | Create SSH keys, connect from the phone with Termius, install the cloud platform, create accounts, and reach it from the Files app | 30 minutes |
| **3. Encryption on the phone** | Create a Cryptomator vault on the server, upload into it, and verify the files are unreadable | 15 minutes |

Quick summary of each:

**1. Laptop as a server:** System Settings, General, Sharing, File Sharing on, then Options and tick SMB, then add a shared folder and set permissions. Note the `smb://` address. This only works on your own Wi-Fi, because private addresses such as `192.168.1.11` cannot be reached from outside.

**2. Server plus phone:** generate an Ed25519 key with `ssh-keygen`, send only the public key to the server, then in Termius create a host with the IP, port `52156`, your username and your key. Install the cloud platform with snap, turn on HTTPS, and create one account per person with a quota. The iPhone Files app speaks SMB only, so use SMB or an SFTP file provider app such as Secure ShellFish.

**3. Encryption on the phone:** in Cryptomator create a vault, store it inside the server or Nextcloud folder, set a strong vault password, unlock it, and copy files into the Cryptomator location in the Files app. Then open the storage folder and confirm you only see `.c9r` scrambled names.

## Who can read what

| Actor | Can read the files? | What they can see |
|---|---|---|
| The user | **Yes**, they hold the key | Everything they own |
| Node operator or host | **No** | Encrypted blocks, file counts, rough sizes, activity |
| Other users | **No** | Only what is explicitly shared |
| Network eavesdropper | **No** | Scrambled traffic |
| Thief with the disk | **No** | Unreadable storage |

**Stated honestly:** a host can still observe metadata, and whoever controls a disk can delete data. What no one can do is read it.

## Tech stack

| Layer | Used |
|---|---|
| Terminal and administration | Termius (iPhone, Mac), macOS Terminal, SSH keys (Ed25519) |
| File access | Apple Files app, Samba (SMB), SFTP file providers (Secure ShellFish, Owlfiles) |
| Encryption | Cryptomator vaults (end to end), full disk encryption (at rest), HTTPS and TLS plus SSH (in transit) |
| Cloud platform | Nextcloud or Seafile, deployed with Docker or snap |
| Private networking | Tailscale, plus Funnel for public demos |
| Transport | SSH on a custom port, SFTP, HTTPS, SMB, WebDAV |

## Roadmap

| Phase | Goal |
|---|---|
| 0. Proof of concept | One working node: HTTPS, accounts, quotas, an encrypted vault, and a live host rental demo |
| 1. Private pilot | 10 to 30 testers, onboarding, backups, monitoring |
| 2. Multi user product | Self service signup, two factor, mobile apps, billing, support |
| 3. Federation | Multiple nodes, a node directory, the in app node picker |
| 4. Market launch | Host app, reputation, payouts, commission |
| 5. Scale | A national network, B2B and white label, partnerships, compliance |

## Business model

- **Subscriptions:** free starter tier, paid plans in SAR below mainstream providers.
- **Marketplace commission:** a percentage of every rental between a user and a host.
- **Family and group plans:** a shared pool, one payer, many users.
- **Schools and small businesses:** private, PDPL aligned storage.
- **Managed hosting and white label:** setup and maintenance for organisations, licensing to local brands and ISPs.

## Repository contents

- `README.md` — this file
- `SETUP.md` — the full step by step build guide
- `glossary.md` — a bilingual technical glossary
- `Sandooq-Report.pdf` — the project report for SAIF

## What this project needs next

Ordered by importance. The first three are the difference between a demo and a real product.

1. **One running instance, properly.** Installed on the server, reached over HTTPS with a real domain, and with a working demo account. Everything else is theory until this exists.
2. **A second node.** Federation is the central claim of the project. Two nodes that can see each other and share one file turn the map from a drawing into a fact.
3. **A minimum host flow.** A small script or app that lets someone plug in a drive, register capacity, accept encrypted writes, and see a usage report. The marketplace is the heart of the idea, so it needs a working minimum, not a plan.
4. **A one command installer.** Docker Compose or a shell script so anyone can reproduce the setup on their own machine. This is what turns one person's server into a project other people can run, and it is also what lets a judge check the work.
5. **Backups and a health check.** A second copy of the data and a simple uptime page, so a live demo cannot fail on stage.
6. **A thirty second privacy proof.** A rehearsed sequence: upload a file, show the storage, show the `.c9r` names, decrypt it again. Same result every time, without improvising.
7. **Real numbers.** Cost per gigabyte, host payout, and the break even point. The figures in this README are illustrative, and judges will ask for the real ones.
8. **Arabic onboarding.** The first screen a Saudi user sees should be in Arabic, not English with a translation underneath.
9. **A privacy policy and terms of use.** Short, plain and honest, written with PDPL in mind. It also protects the project legally.
10. **Three real pilot users.** Not friends who say it is nice, but people who upload something real and come back a week later.
11. **A two minute demo video**, and a rehearsed three minute pitch, with the live demo also practised offline.

**What it does not need right now:** more slides, more ideas, or a bigger network. It needs one working instance, one second node, and one host earning their first riyal.

## Status

Proof of concept. A single node runs with HTTPS, accounts, quotas and an encrypted vault. Next: a federation test with two nodes, then the host earning demo.

## Author

**Dana Alharbi** — SAIF project, Kingdom of Saudi Arabia.

*Sandooq. Storage you own. Privacy you can verify. Income that stays in the Kingdom.*
