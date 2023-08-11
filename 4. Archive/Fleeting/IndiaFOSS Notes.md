# IndiaFOSS Notes
> Created: 2022-07-23 09:24

# Day 1

## Kovid Goyal

Inventor of Calibre. Working on a terminal emulator. Kitty?

## Mon.School

Look up Anand's course "The Joy Of Programming"

## Rudra Saraswat

Project lead for Ubuntu Unity. Creator of Unity Remix. 12 years old.

Android apps at the age of 5. Mother introduced him to programming.

## Nabarun Paul

Kubernetes maintainer and Senior Engineer at VMWare. PyCon India Technical Lead.

- Can we not automate certain portions of this instead of having 6 teams?
> Probably not. It's high impact and regular, global releases with outside community users.
- To stick to the timeline, how do they decide to cut features?
> Give a grace period of 2-3 days extra in case a feature is nearly complete. If it's still not done at the end, then we revert _if_ it's user impacting. Else we leave it in and complete on the next release. Decisions for what features to add in the release is delegated to each group's lead.
- Tradeoff between time taken to make a decision, and quality of decisions?
> Decide on n days for consensus when describing the decision to be taken, reach out to all necessary stakeholders and inform them about the time limit to read the email and reply. Do not block decisions. Take input of leadership to decide how to word the decision, how much time to give, and leaning by default for the decision.

Seems to be a significantly sized release process. 

- Master blocking release: blocks releases, all have to be green to proceed (e.g. conformance tests)
- `krel` Kubernetes Release toolbox. Build the code, generate binaries and platform images. 
- `kpromo` Kubernetes project artifact promoter. For staging/prod promotion.

Do, Delegate, Defer. 

## Self Hosting

Self Hosting using Nomad cluster.

Works at Zerodha.

Cubans got banned from using Notion due to sanctions.

+ Ansible. Harden ssh, installing base software
+ Terraform. Managing infra, achieving consistency between desired state and real world state. Basically IaC.
+ Nomad. Simple workload orchestrator and scheduler. Lightweight and simple alternative to K8s. 

Why Nomad? Was using K8s, went down deep complexity hell. Deployed first app with Nomad in a few minutes. Single binary executable with a UI.

Nomad Agent. Look up Raft Consensus. 🚩

Works very well with other Hashicorp software.

- Have you had any issues with uptime or reliability in general?
> Yes but don't host very critical stuff unless you know how to keep uptime perfect.

Restic. To encrypt data at rest and upload anywhere. Backblaze B2 for cloud storage (cheapest option available?)

![[IMG_20220723_122731.jpg]]

Security. If it should not be public facing, don't expose to www. Prefer to use VPN or mesh networks instead of IP whitelists. Tighten firewall rules. Use strong passwords, periodic updates to app and OS.

To start with, pick up something simple like Adguard or Pihole and host it. Go with the simplest way.

For hardware, droplet from DO or Hetzner. Alternatively, RasPi but those are getting expensive these days.

## Blaze OSS - Akash

Works at Razorpay.

Blaze. Peer to peer file sharing webapp. Very easy to use.

https://blaze.now.sh

## Kailash Nadh: Dictionaries

Free open dictionaries.

For Indian languages we don't have good mainstream usable online dictionaries.

Olam: English / Malayalam dictionary 

Alar.ink: Kannada / English dictionary 

Stemming. Algorithm where a word stemmed collapses into its root form. `swimming --> swim`

Postgres TSVECTOR. Text Search Vector. Built-in stemming and supports English, German and a few others. Zero Asian language support.

Postgres `@@`. 🚩

Metaphone. Phonetic properties of a word.

```
swimming: SWMNK
elephant: ELFNT
elefant: ELFNT
```

MLPhone. Metaphones for Malayalam, with different levels of specificity. KNPhone for Kannada.

License. OdbL and CC.

- Are there other options in the data licensing space?
> OdbL is the defacto for open data licensing. Not many other options at present, Creative Commons normally used for open content. OdbL allows licensing with mixed copyright on content.

Dictpress. Building open source dictionaries for any language.

GH ajinasokan/ditto mobile app 

## OCaml

Derived from ML (metalanguage), functional programming.

Eio. Effects based concurrent IO.

## Appsmith

Low code app building, for dashboards, CRM etc.

## No Cost Test Environment

SigNoz, Prashant Shahi

LocalTunnel. Open source tunneling tool, easy to make a local webserver available to the public.

## Transactional Systems
Presto. Framework with FP (Haskell?)

## Hasura

GraphQL layer to abstract away datastores.

# Day 2

## eBPF

Barun, working at Accuknox.

Containers paired with an orchestrator like Kube.

OWNKIT IT IS PRONOUNCED OWNKIT.

Container Runtime Security. Monitor, alert, take action.

Kernel tracing / auditing. `auditd, ftrace, strace, eBPF`

https://ebpf.io/

KubeArmor. https://kubearmor.io/

KubeArmor runs privileged as part of the host. KubeArmor will work as a daemon on individual hosts, the actual agent runs on individual hosts.

## Storage Advancements

https://indiafoss.net/2022/cfp/submissions/how-advancements-in-storage-are-triggering-innovations-in-opensource-and-vi

Nitesh Shetty.

The Linux OS stack is optimized towards older, sequential read/write devices which were slower.

SPDK. Exposed PCIe registers to userspace. 

Multiqueue with scheduler on Linux for concurrent parallel data queuing. Polled queue to avoid context switch and reduce latency.

IO_uring. Scalable async IO infra. 

SSD limitations. Log-on-log, where you end up writing more data on the disk than the host requests to write. Internally you are writing more data than you require (over provisioning). https://en.m.wikipedia.org/wiki/Write_amplification

Multistream.

ZNS. Host can group data based on usage and frequency. SSD usually don't like load which is mixed, like data that is written very often along with data that is rarely written. It reduces the lifetime of the SSD. 

KV (key value) SSD. Device capable of storing key value pairs.

Computational SSD. The device itself can do offloaded computations. 

At present the stack is not mature enough to handle all the new storage hardware improvements. If the application wants to consume it, it needs to trickle down in 

xNVMe. User space library for memory allocation and IO submission. Abstracts the underlying IO engine.

IO_uring. Gives direct asynchronous access to underlying nVMe hardware.

## Liberated Hardware

https://indiafoss.net/2022/cfp/submissions/experiences-in-connecting-securing-and-optimising-enterprise-networks-with

Unmukti Technology.

MWAN3. Load balancing, failover, directing network traffic through specific links.

Radio links for internet. Packet loss if you lose LoS with base station. 10-20% packet loss can happen. 

Traffic prioritisation with qos-scripts. OpenWRT has a package for setting traffic priority.

Deep packet inspection with libndpi.

OpenVPN. BFD (via bird4) for detecting internet failure for sub-second failover.

Forward Error Correction with UDPSpeeder, for dealing with packet loss in transmission.

Charcoal.io, for centrally managed ACLs.

## Hardware Hacking

https://indiafoss.net/2022/cfp/submissions/hacking-hardware-using-free-software-and-gnu-linux

liberatedhardware.org

Open Source Hardware (OSHWA)

## Open Source Hardware

https://indiafoss.net/2022/cfp/submissions/how-we-built-an-opensource-hardware-product-from-india

paperd.ink. Low power open source ePaper development board.

https://www.crowdsupply.com/

## FOSS UPI

https://indiafoss.net/2022/cfp/submissions/how-do-we-get-a-foss-upi-mobile-app

User can directly talk to NPCI, or via a bank, or via a TSP (technical service provider, like GPay).

Cannot programmatically call USSD codes on iOS (can be on Android but requires specific permissions). USSD is also insecure and does not have encryption between you and the cell cover. Jio doesn't support `*99#` either.

https://github.com/librefin-in

## Network Security Panel Discussion

Dashboards and software accidentally published on the internet.

Use network monitoring.

https://zeek.org/
https://suricata.io/

https://www.veracode.com/security/arp-spoofing ARP spoofing.



----

## References
1. https://indiafoss.net/2022/schedule