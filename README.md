# nvme vps: what real IOPS and renewal prices show, and when a $2 SSD VPS makes more sense

Two very different people type "nvme vps" into a search bar. One runs a database that stutters every time traffic spikes, and storage is a genuine fix for that. The other just wants a small server for a blog, a Discord bot, or a personal VPN, noticed "NVMe" on a pricing page, and is trying to work out whether it justifies paying extra.

They should end up spending very different amounts of money, and most comparison posts never separate them. So here's the split up front: if your workload is IOPS-hungry — PostgreSQL, MySQL, Redis, busy WooCommerce, CI runners, container hosts — pay for storage with published, independent benchmark numbers. If it's basically anything else, a budget SSD VPS starting at $1.99/month will feel identical in daily use, and the money you save covers months of hosting.

The rest of this article backs that up with numbers: what NVMe actually changes on a VPS, why the word "NVMe" on a plan page proves nothing, what renewal prices do to your real costs, and which cheap plans are worth it in each case.

> **Quick picks**
>
> - **Fastest measured storage:** UpCloud's $5/mo Developer plan sustained 100,000+ 4K random read IOPS with sub-1 ms tail latency
> - **Lowest-priced genuine NVMe entry:** Vultr at $2.50/mo with 10 GB NVMe — NVMe is standard on every tier, across 32 datacenters
> - **Best IOPS per dollar:** Hetzner's CX22 at around $5/mo — 2 vCPU, 4 GB RAM, 41,000 read IOPS, 20 TB egress
> - **Cheapest overall server when you don't need NVMe:** RackNerd, SSD storage from $1.99/mo with no renewal-price jump — 👉 [check RackNerd's current VPS deals and datacenter list](https://bit.ly/my-rn)

## What NVMe actually buys you on a VPS

On paper the gap is large. NVMe drives push 4–6 GB/s of sequential throughput against roughly 500–600 MB/s for SATA SSD, and they handle 500K+ random IOPS where SATA tops out around 80–100K. NVMe is a protocol built for flash memory sitting on the PCIe bus, so it skips the SATA controller entirely — that's where most of the advantage comes from.

In practice, you feel it in three places. Docker image pulls finish 2–3x faster. `git clone` takes 40–60% less time. Database queries that live on random reads — which is most of what PostgreSQL, MySQL, and Redis actually do — come back sooner, and latency stays flat under load.

The catch is that a lot of VPS workloads never get near this. A static site, a file archive, a small API, a VPN endpoint, or a dev sandbox does tiny amounts of disk I/O. On those, the difference between 100,000 IOPS and 8,000 is invisible, because you're nowhere close to either ceiling. That one fact reshapes the whole "which NVMe VPS should I buy" question, and it's where the cheap SSD options below earn their place.

## "NVMe" on the plan page vs. the IOPS you actually get

Advertising NVMe and delivering usable IOPS are two different achievements. One recent round of fio benchmarking provisioned entry and mid-tier plans across 12 providers and found a spread from 6,000 4K random read IOPS on heavily oversold budget hosts up to 100,000+ on UpCloud's MaxIOPS tier. The test setup was reasonable: $5–$10 plans in a US East datacenter, 4K blocks at queue depth 32 with four parallel jobs, five runs over two days, medians reported. Anything below 15,000 read IOPS got excluded — below that line, "NVMe" is a sticker rather than a spec.

The mid-tier results also show why averages alone mislead:

| Plan (4 GB class) | 4K random read | 4K random write | p99 latency |
| --- | --- | --- | --- |
| UpCloud MaxIOPS | 110,000 | 78,000 | 0.9 ms |
| Hetzner CX32 | 48,000 | 32,000 | 1.6 ms |
| Vultr High Frequency | 45,000 | 30,000 | 1.5 ms |
| DigitalOcean Premium AMD | 44,000 | 29,000 | 1.7 ms |
| Linode | 40,000 | 26,000 | 1.9 ms |
| Contabo VPS M | 22,000 | 14,000 | 4.2 ms |

A plan that averages 50,000 IOPS but spikes to 50 ms at p99 will feel worse to real users than one holding a flat 30,000. Databases care about the worst case, because the worst case is what your slowest queries wait on.

You can check any of this yourself before committing. Vultr and UpCloud both bill hourly, so the test costs cents:


fio --rw=randread --bs=4k --iodepth=32 --numjobs=4 --runtime=300 --time_based --direct=1


Let it run the full five minutes, repeat it a couple of times, and hold the medians against the 15,000 IOPS bar. If a provider's entry plan fails that, its NVMe branding doesn't matter.

## Renewal prices: the part of the bill most comparisons skip

Here's a number worth staring at. A 2026 tally of 196 VPS plans on NVMe storage found a median entry price of $6.12/month, while the median renewal price — among the 32 hosts that actually publish one — is $10.19/month. That's a 66% jump, locked in on day one. Only 12 of those 196 hosts keep your price the same.

The concrete cases are uglier than the median. Bluehost's NVMe 8 plan costs $12.99/mo on a two-year term and renews at $28.99 — a 123% increase. Its NVMe 16 tier jumps 92%. Hostinger's $1.99 intro price renews at 2–3x once the term ends.

Now put that next to the actual cost of NVMe itself. Moving from a $1.99 SSD plan to a $5 measured-NVMe plan costs about $3 a month, or $36 a year. The median renewal inflation costs $4.07 a month, roughly $49 a year, on a plan you never upgraded. The "expensive" NVMe upgrade is frequently cheaper than the renewal hike on budget plans — which is why the second-year price deserves more attention than the first-year one.

A few providers sit on the right side of this. AlphaVPS's C2G plan — 2 vCPU, 2 GB RAM, 30 GB NVMe, 3 TB transfer — runs $2.90/mo and renews at the same price. RackNerd gets flagged in comparisons for exactly the same trait, which matters more the longer you keep a server. 👉 [Compare RackNerd's renewal pricing before you commit somewhere else](https://bit.ly/my-rn)

## RackNerd: the cheap end of the NVMe VPS conversation

RackNerd earns its spot in an NVMe VPS article for an unfashionable reason: its VPS plans ship SATA SSD storage starting at **$1.99/month**, and for the majority of workloads people actually run on small servers, that's plenty. KVM virtualization, a range of US datacenter locations plus Amsterdam, and pricing that doesn't reset at renewal make it the default answer for blogs, bots, VPNs, dev boxes, and low-traffic sites.

What it isn't is an IOPS play. RackNerd's storage is listed as SATA SSD, so if you're migrating a hot PostgreSQL database or a busy WooCommerce shop, this is the wrong tool and the specialists in the next section are the right ones. Saying that plainly is more useful than pretending a $2 plan competes with a six-figure IOPS tier.

| Plan | Price | Storage | Renewal | Order |
| --- | --- | --- | --- | --- |
| Entry KVM VPS | $1.99/mo | SATA SSD | same price, no jump | [Order the entry plan](https://bit.ly/my-rn) |

Beyond the entry plan, RackNerd's catalog is mostly annual KVM VPS deals with more RAM, more cores, and bigger storage allocations, and the lineup rotates through specials often enough that the order page is the only reliable price list. Those annual terms are where the per-GB pricing gets aggressive. 👉 [See RackNerd's current SSD VPS plans and specials](https://bit.ly/my-rn)

One pattern worth stealing if you do end up needing real NVMe: buy it only for the box that needs it. Put the database on a measured-NVMe plan and leave the application on cheap SSD compute. For a single small stack, one $5–6 NVMe plan is simpler than juggling two servers; once the app side grows, splitting them keeps each box cheap.

## Budget NVMe VPS plans, side by side

For readers who genuinely need the IOPS, here's how the cheap end of the real-NVMe market stacks up, using measured numbers instead of plan-page claims:

| Provider | Plan | Price | 4K read IOPS | Notes |
| --- | --- | --- | --- | --- |
| UpCloud | Developer | $5/mo | 100,000+ | MaxIOPS tier; p99 under 1 ms; 99.99% SLA with 10x credit |
| Hetzner Cloud | CX22 | ~$5/mo | 41,000 | 2 vCPU / 4 GB, 20 TB egress; best IOPS-per-dollar |
| Vultr | High Frequency HF1 | $6/mo | 39,000 | Sapphire Rapids CPU; 32 regions; hourly billing |
| DigitalOcean | Premium AMD 1GB | $7/mo | 38,000 | very flat tail latency |
| Linode (Akamai) | Shared 1GB | $5/mo | 32,000 | NVMe across shared plans; strong network |
| IONOS | VPS XS | $4/mo | 24,000 | cheapest plan that cleared the IOPS bar |
| KnownHost | Entry VPS | $5/mo | — | NVMe on every plan including entry; 2 TB bandwidth; operating since 2006 |
| Contabo | VPS S | $6.50/mo | 18,000 | 8 GB RAM; IOPS dip under contention |

UpCloud is the pick when storage performance is the whole point — nothing else in this price range sustained six figures. Hetzner wins on value if you can live with its thinner US and APAC coverage. Vultr makes sense when location count matters more than raw IOPS. Contabo trades storage speed for RAM, a fine trade for memory-bound work and a bad one for databases.

## FAQ

**How much faster is NVMe than SATA SSD on a VPS?**
Roughly 8–10x in sequential throughput and 5x or more in random I/O: 4–6 GB/s versus 500–600 MB/s, and 500K+ IOPS versus 80–100K. The real-world effects show up as 2–3x faster Docker pulls and 40–60% faster clones.

**Do I need NVMe for WordPress?**
Usually no. A typical WordPress site is comfortable around 5,000 IOPS, which even entry-level SSD storage handles. The exception is a busy WooCommerce store, where the database's write load genuinely benefits from NVMe.

**Does RackNerd use NVMe drives?**
RackNerd's plans are listed with SATA SSD storage, and that's a big part of how it reaches $1.99/mo. It's the right pick when disk speed isn't your bottleneck; for IOPS-bound work, choose a provider with published fio numbers.

**How do I verify disk speed before committing to a plan?**
Pick a provider with hourly billing, deploy in your target region, run the fio command above, and hold the result against the 15,000 read-IOPS floor. Watch p99 as well as the average, and do it inside the first hour while the test costs pennies.

## A 30-second decision guide

- **Static site, blog, small API, dev sandbox, VPN, game server** — skip the NVMe premium entirely. RackNerd's SSD plans from $1.99/mo with flat renewal pricing are the rational choice here, and the money saved outlasts most of these projects. 👉 [Open RackNerd's order page and pick your location](https://bit.ly/my-rn)
- **Production database or anything with latency-sensitive users attached** — pay for measured IOPS. UpCloud's $5 Developer plan for the fastest storage in this class, Hetzner's CX22 around $5 if value per IOPS matters more, Vultr's High Frequency line if you need regional spread.
- **Busy WooCommerce or CI-heavy workloads** — same answer as above, with one extra check: match the egress allowance to your traffic, because fast disk behind a slow network is the wrong bottleneck to pay your way out of.

Whichever way you go, two checks take five minutes and catch problems no plan page will admit to: confirm the renewal price for the exact billing term you're selecting, and run fio in the first hour after deploy.
