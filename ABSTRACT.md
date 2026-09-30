# Abstract — SANDOOQ

**Sovereign, end-to-end encrypted cloud storage, owned by Saudis who use it.**

**Author:** Dana Turki Alharbi · Grade 9, Numou Education Center — Creativity Oasis, Alkhobar · **Programme:** SAIF · **Country:** Kingdom of Saudi Arabia

---

Your photos are sitting in a foreign data centre right now. Your device runs out of space, and the only two buttons on screen are *delete* or *pay*. That message is not an accident: it is a business model, and it has become the default way the world stores its memories.

SANDOOQ is a Saudi-based, self-hosted cloud, very similar to built-in iCloud — except the server is yours. Files are encrypted end-to-end with **Cryptomator (AES-256)**, and nodes stay private over **Tailscale** and SSH, so even the operator cannot peek. Privacy you can prove, seamless backup, private file sharing on any device, extra storage for free. Decentralized by design, with a potential business model that lets anyone rent out spare space and earn. Your cloud. Your rules.

Two ideas hold it together:

1. **Privacy by architecture.** Files are encrypted on the user's own device before they move anywhere, so the operator, the host, and anyone who reaches the disk can only ever see unreadable encrypted blocks.
2. **Storage as local infrastructure.** Anyone with unused capacity — an old laptop, an external drive, a NAS — can run a node and earn a monthly payout for the space they are not using.

## The two-node architecture, and why Tailscale matters

The proof of concept runs on **two nodes**: a **public node** (a rented server with a public IP, serving real users over HTTPS) and a **private node** (a Mac at home with no public address). Exposing a public server means constant scanning; leaving the private node inside the home network means it is reachable from only one place.

**Tailscale resolves both at once.** It builds a private, encrypted **tailnet** between the project's devices using WireGuard. Each device gets a stable `100.x.y.z` address and must authenticate before joining. With both nodes inside the tailnet:

- The **public node's SSH access is closed to the whole internet**; administration happens only over the tailnet. The public internet sees one thing: HTTPS on port 443.
- The **private node becomes reachable from anywhere** with no ports opened on the home router.
- The two nodes can **federate and back each other up** over the tailnet, turning decentralisation from a drawing into a working mesh.
- Testers can be invited to a private beta **with no public exposure**.

Tailscale does not replace end-to-end encryption. Cryptomator protects file contents from everyone, including the operator. Tailscale protects the network path and removes unnecessary exposure. The two layers are complementary.

## Honest limits

- Tailscale is **private, not public**. Users reach the service over HTTPS on the public node; the tailnet is for operators, nodes and invited testers.
- Metadata remains visible to a host, and a host can always delete data. Encryption prevents reading, not deletion.
- A private network reduces exposure; it does not remove the need for updates, backups and access control.

## Conclusion

SANDOOQ is a working proof of concept for a storage model that is sovereign, verifiable and economically local: encrypted on the device, hosted in the Kingdom, supplied by the community, and administered over a private network instead of an exposed one. Its next milestones are a second federated node, a minimum host-earning flow, and a one-command installer so anyone can reproduce the system.

*SANDOOQ. Storage you own. Privacy you can verify. Income that stays in the Kingdom.*
