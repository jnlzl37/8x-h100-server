# 8x H100 Server: How Much GPU Memory and Bandwidth You Get, Whether It's 6U or 8U, Rent or Buy, and What to Verify Before You Order

An 8x H100 server gives you **640 GB of GPU memory** (8 × 80 GB HBM3 on SXM5 modules), **900 GB/s of bidirectional NVLink** between any two GPUs in the chassis, and eight PCIe Gen5 x16 links back to the host CPUs. Those three numbers determine everything else about the machine: why the chassis is 6U or 8U tall, why it needs six 3,000 W power supplies, and why the decision between renting and buying usually comes down to utilization rather than sticker price.

There are three ways to get one. You can buy a barebone 8-GPU system from an OEM: Dell builds the PowerEdge XE9680 in a 6U air-cooled chassis, Supermicro builds the SYS-821GE-TNHR in 8U. You can rent a dedicated 8-GPU machine by the month. Or you can run the same eight GPUs on a cloud provider's hourly meter. The silicon is the same in all three cases. What differs is who owns the power bill, the lead time, and the failure.

Cloud GPU prices change constantly, so instead of quoting a rate that will be wrong by the time you read it, this article sticks to what can be checked: card-level and system-level specs, the rack, power and cooling requirements that catch people out, the one published hourly data point worth using as a reference, and the questions to ask before you commit. The signup credit attached to the link below is a **$300 credit**; eligibility, expiry, and which products it covers are stated on the provider's own pages, and those terms matter more than the headline number.

> **Short version:** if you need to run something this week and you don't want to touch a data center, start with an hourly cloud instance or a dedicated rental. If your eight GPUs will be busy more than roughly 70% of the time, a dedicated machine or a purchased system is the cheaper shape within a year. Almost nobody knows their utilization before they measure it, so measure first.
>
> 👉 [See Vultr's current GPU instances and new-account credit](https://bit.ly/VuLtr)

## What is actually inside an 8x H100 server?

The building block is NVIDIA's HGX H100 8-GPU baseboard: eight H100 Tensor Core GPUs plus **four third-generation NVSwitch** chips. Every GPU has multiple fourth-generation NVLink ports wired to all four switches, so any H100 can talk to any other H100 concurrently at full NVLink speed. That topology is why the "8x" in 8x H100 matters more than the card count: eight GPUs connected through PCIe alone would give you a fraction of the bandwidth.

Here is the single-GPU spec sheet for the SXM5 variant, which is what goes into these servers:

| Spec | H100 SXM5 |
| --- | --- |
| Memory | 80 GB HBM3, 3.35 TB/s |
| GPU-to-GPU | NVLink, 900 GB/s bidirectional |
| Link to host | PCIe Gen5, 128 GB/s |
| FP64 / FP64 Tensor Core | 34 / 67 TFLOPS |
| TF32 Tensor Core | 989 TFLOPS |
| BF16 / FP16 Tensor Core | 1,979 TFLOPS |
| FP8 / INT8 Tensor Core | 3,958 TFLOPS / TOPS |
| Max TDP | up to 700 W, configurable |
| MIG partitioning | up to 7 instances at 10 GB each |
| Form factor | SXM |

Multiply by eight and you get the system-level figures NVIDIA published for the HGX H100 8-GPU board: **32,000 TFLOPS FP8, 16,000 TFLOPS FP16, 480 TFLOPS FP64**, 3.6 TB/s of bisection bandwidth, and roughly 3x better all-reduce throughput than the equivalent HGX A100 system thanks to NVSwitch multicast and SHARP in-network reductions. Those numbers come from a 2022 launch post that carries an explicit caveat at the bottom: *"All performance numbers are preliminary based on current expectations and subject to change in shipping products."* Treat them as vendor targets, not benchmarks you can reproduce on day one.

## Sparse or dense: which TFLOPS number is on the listing?

The 1,979 TFLOPS FP16 figure for one H100 is the sparsity-inclusive number. Structured sparsity roughly doubles throughput on paper, and vendor tables footnote it inconsistently. One third-party spec sheet lists the 8-GPU node at **7,912 TFLOPS BF16 dense**, which is 8 × 989, exactly half the sparse number. Both are correct; they describe different measurement conventions.

This matters when you compare quotes. A reseller advertising "16 PFLOPS FP16" for the same box you saw quoted at "7.9 PFLOPS" is not selling you a slower machine. If you put the two numbers in one spreadsheet without normalizing, you'll draw the wrong conclusion about which system is faster, and if you're sizing a training run against a published throughput target, you'll be off by 2x.

## The listing says 80 GB. Which 80 GB?

H100 comes in two memory configurations, and they are not interchangeable:

- **SXM5:** 80 GB HBM3 at 3.35 TB/s, 900 GB/s NVLink, up to 700 W. This is what goes into HGX 8-GPU baseboards.
- **PCIe:** 80 GB HBM2e at roughly 2 TB/s, with far less interconnect bandwidth, because the cards sit in standard slots instead of on an NVSwitch fabric.

Total memory is 640 GB either way, so a listing that says "8× 80 GB, 640 GB total" can hide which one you're getting. The trap is real: a provider-published case study from unihost describes an 8× H100 NVLink server with "640 GB total HBM2e." The capacity is right, the memory type is wrong for an SXM machine, and that kind of label error is common in secondhand listings. If a quote mentions HBM2e or 2 TB/s per GPU, you're looking at PCIe cards and the 900 GB/s NVLink figure does not apply.

## Power, weight and cooling: the part of the bill nobody quotes

The Supermicro SYS-821GE-TNHR is a useful reference because its datasheet publishes the numbers most sellers leave out.

**Dimensions and mass.** 8U rackmount, 14" × 17.2" × 33.2" (355.6 × 437 × 843.28 mm). Net weight **166 lbs (75.3 kg)**, shipping weight 225 lbs. Two people and a server lift, plus a floor-loading check if your racks aren't rated for a dense GPU row.

**Power.** Six 3,000 W Titanium (96%) supplies in a 4+2 redundant arrangement, with an 8× 3,000 W (4+4) option. By the datasheet arithmetic that's on the order of 12 kW of supply capacity per chassis. Four chassis per rack, which is how Supermicro lays out its 256-GPU cluster, works out to roughly 48 kW of supply capacity per rack, fed by **34 × 208V 60A three-phase PDUs** across nine 48U, 750 mm × 1200 mm racks. If your facility can't deliver three-phase power at that density, the GPUs are not the constraint.

**Cooling.** Air-cooled, with ten heavy-duty fans, rated for an operating range of **10 °C to 35 °C** (50 °F to 95 °F) and 8–90% non-condensing humidity. Thirty-five degrees is the ceiling, not a target; a colo hall that drifts toward the top of that range will push fan speeds up and clocks down.

**Memory and expansion.** 32 DIMM slots, up to **8 TB of DDR5-5600 ECC**, plus eight PCIe 5.0 x16 low-profile slots and two full-height half-length slots. Here's the part people miss: in a properly configured node, **all eight of those PCIe 5.0 slots go to 1:1 400Gb/s networking, one NIC per GPU** (ConnectX-7 class), which is what enables GPUDirect RDMA and GPUDirect Storage so data lands in GPU memory without bouncing through the host. Buy the box with a single 100G NIC and you've paid for eight GPUs that can't be fed.

One more item from the product page: this chassis requires **onsite service**. Supermicro also offers free remote testing, which is worth asking about for any 8-GPU machine, whether you're buying or renting. Burn-in before delivery beats debugging a dead NVLink switch in week two.

## From 8 GPUs to 256: when one box stops being enough

A single 8-GPU node covers a surprising amount: NVIDIA's own guidance puts BERT-Large and Mask R-CNN comfortably within one node's memory. The line gets crossed when the model itself doesn't fit. Recommendation models with terabyte-scale embedding tables and large mixture-of-experts language models need aggregate memory across nodes, and that's where the interconnect becomes the bottleneck rather than the GPU.

Two ways to scale past eight:

**NVLink-Network.** A version of the HGX H100 8-GPU board connects outward through OSFP LinkX cables to an external NVLink switch, forming a domain of up to **256 GPUs that share NVLink semantics**. The published comparison: a 256-GPU H100 pod versus a 256-GPU A100 pod means a 32x larger NVLink domain, 1,024 PFLOPS FP8, and 70 TB/s of bisection bandwidth. If your workload does all-reduce on a scale that crosses node boundaries every step, this is the difference between linear and disappointing scaling.

**Standard InfiniBand or Ethernet fabric.** Supermicro's 32-node reference unit packs **256 H100 or H200 GPUs** (20 TB of HBM3 with H100, 36 TB with H200) into four 8U systems per rack, nine racks total, with eight Quantum-2 400G NDR leaf switches and four spines in a non-blocking topology. That unit is designed to be an increment: 32 nodes today, thousands of nodes later, same leaf-spine design.

The practical test for whether you need any of this: if your job fits in 640 GB with room for activations and you're not sharding a model across nodes, a single 8-GPU box is the right purchase and the fabric is an expense you can defer.

## What does an 8x H100 server cost per hour, and when does renting beat buying?

The most reusable published data point is Lambda Cloud's August 2023 listing of **$2.59 per GPU-hour** for an on-demand HGX H100 8x SXM instance. That's **about $20.72 per hour for the whole node** (my arithmetic, not their copy), or roughly **$15,100 for a month of continuous use** at 730 hours. That rate is more than three years old and should be treated as a reference point, not a current quote.

The number that actually decides rent-versus-buy is utilization. Take that $20.72/hour node running around the clock and assume your team keeps it 40% busy, which is a normal starting point for a shared research box. You're paying for 5,840 GPU-hours a month and using about 2,336 of them, so your effective cost per GPU-hour of real work is around **$6.48** instead of $2.59.

The unihost case study is interesting for exactly this reason. An enterprise AI team moved its baseline GPU workloads off public cloud onto a dedicated 8× H100 NVLink server in the Netherlands (2× Xeon Gold 6448Y, 640 GB GPU memory, 2 TB RAM, 2 TB NVMe, 10 Gbps network) and reported average utilization of paid GPU capacity rising from **roughly 35–45% to over 70%**, with cloud kept for burst capacity. It's a vendor's own customer story, so read it as an illustration rather than a survey, and note the memory mislabel mentioned earlier. The mechanism it describes is still the right one: hourly billing punishes idle capacity, and a dedicated box you own the whole time converts a variable bill into a fixed one.

| Form | What you get | What bites | Price signal | Fits whom |
| --- | --- | --- | --- | --- |
| Purchase: Dell PowerEdge XE9680 | 8× H100 80 GB SXM5, 2× Xeon 48C, 6U air-cooled | Rack, three-phase power, onsite service, capital outlay | No public list price | Steady load, own data center |
| Purchase: Supermicro SYS-821GE-TNHR | 8× H100 80 GB SXM5, 8U air-cooled, 32 DIMM up to 8 TB | 166 lbs, ~12 kW per chassis, 10–35 °C envelope | No public list price | Same, plus a preference for 8U airflow |
| Dedicated rental | 8× H100 80 GB NVLink, 2 TB RAM, local NVMe | Monthly commitment, one region, less elastic | Configurations published, prices on request in the listings checked | Continuous baseline training |
| Hourly cloud | An 8-GPU node by the hour | Rate type (on-demand, preemptible, contract), utilization | $2.59/GPU-hr for 8× H100 SXM, Lambda Cloud, Aug 2023 | Experiments, spikes, unknown duration |
| Hourly cloud: Vultr | GPU instances billed hourly; current models, node sizes and credit terms are on the site | Confirm whether the rate you see is on-demand, preemptible or a pre-paid contract rate | See live listing  [current GPU lineup and credit](https://bit.ly/VuLtr) | Teams that want to measure utilization before signing anything |

One caution on that last column: hourly H100 rates published by different sources for the same provider often disagree, and the usual reason is that one number is an on-demand rate while the other is preemptible or tied to a pre-paid contract. Before you budget, check which one you're looking at.

About the $300 credit specifically: at the one verified node rate above ($20.72/hour), $300 covers roughly **15 hours** of an eight-GPU machine. It's a trial budget, not a discount on a training run. Check the eligibility and expiry terms before you plan around it.

👉 [Check Vultr's current on-demand rate and the credit terms](https://bit.ly/VuLtr)

## Who should not buy an 8x H100 server

Most teams reading this. Run the test honestly:

- **Your model fits on one or two GPUs.** Fine-tuning a 7B model with LoRA, running inference for a small product, or doing classical ML doesn't need eight SXM5 cards, $15k a month, or a 166 lb chassis. Rent one or two GPUs and revisit when the workload outgrows them.
- **Your load is intermittent.** A few training runs a quarter, batch jobs, demos. Idle eight-GPU capacity costs the same as busy capacity, and the case study above shows utilization moving from 35% to 70% is worth more than any per-hour discount.
- **You have no rack space or three-phase power.** A dedicated rental exists precisely for this situation.
- **You need to shard a model across more than 8 GPUs and haven't checked the fabric.** Eight fast GPUs behind a shared 100G link will underperform a slower node with proper 1:1 networking. Sort the interconnect budget before the GPU budget.

## Before you order: seven things to verify

1. **SXM5 with HBM3, or PCIe with HBM2e?** Ask for the module type and the per-GPU memory bandwidth. 80 GB alone doesn't tell you.
2. **Total memory and per-GPU bandwidth.** 640 GB at 3.35 TB/s per GPU is the SXM5 target; anything materially below that is a different product.
3. **Network topology.** Eight PCIe 5.0 slots at 1:1 400Gb/s per GPU, or one shared uplink? This decides whether multi-node training works at all.
4. **Rate type and billing granularity.** On-demand hourly, preemptible, or a pre-paid contract? Rounding to the minute or the hour?
5. **Minimum term and exit.** Month-to-month, or a commitment that outlives your project?
6. **Credit terms, if you're using one.** Eligibility, expiry date, and whether GPU hours are covered.
7. **Remote burn-in.** Supermicro offers free remote testing on its 8U system; ask any provider for a stress test before you accept delivery, and watch for ECC errors and NVLink link-down events.

## The decision rule

Model fits on one or two GPUs: rent small, don't buy eight. Eight GPUs needed for a distributed job but utilization unknown: run hourly for two weeks and measure, which is what the credit is for. Utilization stable above roughly 70%: go dedicated or buy, because the fixed cost beats the meter at that point. Multi-node training: verify the fabric before you verify the price, since the network is what you'll actually be waiting on.

👉 [Start with a Vultr GPU instance and check what the $300 credit covers](https://bit.ly/VuLtr)
