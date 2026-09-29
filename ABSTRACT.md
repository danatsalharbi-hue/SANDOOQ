# Abstract — Sandooq

**Sovereign, end to end encrypted cloud storage, owned by the people who use it.**

**Author:** Dana Alharbi · **Programme:** SAIF · **Country:** Kingdom of Saudi Arabia

---

## Abstract

Cloud storage has become a permanent rental. Users pay every month for space they never own, their files are copied to data centres in other countries, and in most services the provider holds the encryption keys, so privacy depends on a policy rather than on mathematics.

Sandooq is a self hosted, end to end encrypted cloud storage platform built inside Saudi Arabia, combined with a marketplace that turns unused storage into income. Two ideas hold it together:

1. **Privacy by architecture.** Files are encrypted on the user's own device before they move anywhere, so the operator, the host, and anyone who reaches the disk can only ever see unreadable encrypted blocks.
2. **Storage as local infrastructure.** Anyone with unused capacity, an old laptop, an external drive or a NAS, can run a node and earn a monthly payout for the space they are not using.

## The two node architecture, and why Tailscale matters

The current proof of concept runs on **two nodes**:

| Node | Location | Exposure |
|---|---|---|
| **Public node** | A rented server with a public IP address | Reachable from anywhere over HTTPS, which is what real users need |
| **Private node** | A Mac at home | No public address, previously reachable only when a phone was on the same Wi-Fi |

Exposing a public server to the internet means constant scanning, credential attacks and a large attack surface. Leaving the private node inside the home network means it can only be used from one place.

**Tailscale resolves both problems at once.** It builds a private, encrypted network, called a tailnet, between the project's own devices using WireGuard. Each device receives a stable `100.x.y.z` address and is authenticated before it can join. With both nodes inside the tailnet:

- The **public node's SSH access can be closed to the whole internet**, so administration happens only over the encrypted tailnet. The public internet then sees one thing only: HTTPS on port 443 for real users.
- The **private node becomes reachable from anywhere**, from the phone or the server, without opening a single port on the home router and without a public address. It stops being a Wi-Fi only experiment and becomes part of the network.
- The two nodes can **federate and back each other up over the tailnet**, turning the decentralisation claim from a drawing into a working two node mesh.
- Testers can be invited into a private beta **without exposing any service publicly**.

Tailscale does not replace end to end encryption. Cryptomator protects the contents of files from everyone, including the operator. Tailscale protects the network path and removes unnecessary exposure. The two layers are complementary, and together they give the project a security posture that is defensible in front of a technical judge.

## Honest limits

- Tailscale is **private, not public**. Regular users still reach the project through HTTPS on the public node; the tailnet is for operators, nodes and invited testers. Tailscale Funnel can publish a service publicly for a demonstration.
- The free tier covers roughly three users and one hundred devices, which fits a pilot of this size.
- Metadata remains visible to a host, and a host can always delete data. Encryption prevents reading, not deletion.
- A private network reduces exposure; it does not remove the need for updates, backups and access control.

## Conclusion

Sandooq is a working proof of concept for a storage model that is sovereign, verifiable and economically local: encrypted on the device, hosted in the Kingdom, supplied by the community, and administered over a private network instead of an exposed one. Its next milestones are a second federated node, a minimum host earning flow, and a one command installer so the system can be reproduced by anyone.
