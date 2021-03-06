+++
date = "2019-01-13"
publishdate = "2019-01-15"
title = "Oracle, AWS, and Azure Benchmarking Shootout"
author = "Matt Jaffee"
author_twitter = "mattjaffee"
author_img = "2"
image = "/img/blog/why-oci/banner.jpg"
overlay_color = "green" # blue, green, or light
+++

_Psst - when you're done with this, [check out part 2](../cloud-bench-redux)._

For those unfamiliar, Pilosa is an open source, distributed index which is
designed to help analyze huge datasets at "UI latency". Basically we want all
queries to return in under a second regardless of complexity.

This is a fairly tall order, and we're in the midst of launching
[Molecula](https://www.molecula.com), a managed service around Pilosa that makes
these kinds of low latency analytics a reality. We're very interested in which
cloud providers—and which instance configurations on each provider—have the best
overall price/performance ratio.

For this initial foray, we've run a massive suite of Pilosa-centric benchmarks
on 10 different configurations spread across Azure, AWS, and Oracle's cloud (OCI).

Now, when it comes to choosing a cloud provider, Oracle probably isn't the first
company that comes to mind. I think most people would agree that the order is
something like:

1. AWS
2. Azure
3. Google Cloud Platform
4. Others...

We recently had the good fortune of being chosen to participate in the [Oracle
Startup Accelerator](https://www.oracle.com/startup/) program, and they've
strongly encouraged us to put OCI to the test.

<div class="note">
Disclaimer: While we are in the Oracle Startup Accelerator, Azure and AWS
have also provided us with free credits as part of their respective startup
programs and we've used those to run these benchmarks.
</div>

As we started to take it for a spin, we were pleasantly surprised by a number of things:

1. It emulates AWS in a lot of ways, so it feels fairly familiar.
2. It was built from the ground up to work with [Terraform](https://www.terraform.io/) which is muy bueno.
3. The costs were quite competitive, especially considering the amount of memory and SSD available.

Check out this table:

| Cloud | Type            | n | $/hr  | RAM | Threads | Fast Disk/node |
|-------|-----------------|--:|------:|----:|--------:|---------------- |
| OCI   | VM.Standard2.16 | 3 | $3.06 | 720 |      96 | N/A            |
| OCI   | VM.DenseIO2.16  | 3 | $6.12 | 720 |      96 | 12.8 TB NVME   |
| OCI   | BM.Standard2.52 | 1 | $3.32 | 768 |     104 | N/A            |
| OCI   | BM.HPC2.36      | 2 | $5.40 | 768 |     144 | 6.7 TB NVME    |
| Azure | F32s v2         | 3 | $4.06 | 192 |      96 | 256 GB SSD     |
| Azure | F16             | 6 | $4.78 | 192 |      96 | 256 GB SSD     |
| Azure | Standard_E64_v3 | 2 | $7.26 | 864 |     128 | 864 GB SSD     |
| AWS   | c4.8xlarge      | 3 | $4.77 | 180 |     108 | EBS only       |
| AWS   | c5.9xlarge      | 3 | $4.59 | 216 |     108 | EBS only       |
| AWS   | r5d.12xlarge    | 2 | $6.91 | 768 |      96 | 1.8 TB NVME    |


In particular, a 2-node HPC2.36 cluster on OCI is comparable in price to a
3-node c5d.9xlarge on AWS, but has 5x the SSD space, significantly more
processors, and triple the memory.

These basic numbers don't tell the whole story of course - there are hundreds,
or even thousands of different hardware and software choices that a cloud
provider has to make when building their offering. Networking fabric, disk make
and model, processor type, motherboard, memory speed, network card, and
hypervisor just to name a few - not to mention the myriad configuration options,
any of which might have a drastic impact on performance. Some of these things
are published, or can be determined from inside of an instance, but many or even
most of them are intentionally abstracted away from the user. The only really
reliable way to see how different providers stack up is to run your workload on
them and measure the performance!

Pilosa is a fairly interesting piece of software to benchmark as it works out a
few different areas of the hardware. During data ingest and startup, sequential
disk I/O is of critical importance. During most queries, Pilosa is heavily CPU
bound due to its almost unhealthy obsession with bitmap indexes. It's also
written in Go and eats cores for breakfast - it will squeeze every inch of CPU
performance out of a machine. Pilosa holds its bitmap data in RAM, so memory
bandwidth is also a significant factor. Network throughput is probably the least
important aspect, though latency between hosts can make up a significant
percentage of query times for simpler queries.

### Setup 

For our main Pilosa benchmarks, we decided to use the venerable [Billion Taxi
Ride](https://www.pilosa.com/blog/billion-taxi-ride-dataset-with-pilosa/)
dataset which is large enough to provide a realistic workload, without being
unwieldy.

In order to aid repeatability and speed deployment, we produced a set of
parameterized [Terraform](https://www.terraform.io/) modules for each cloud
provider so that we could easily launch clusters with varying instance sizes.
Provisioning the machines was taken care of by a suite of
[Ansible](https://www.ansible.com/) playbooks. All of this is contained in the
`terraform` and `ansible` subdirectories of our
[infrastructure](https://github.com/pilosa/infrastructure) repository on Github.

We ended up with the 10 different configurations listed above which contain a
mix of compute, memory, and storage optimized instances. Necessarily, there are
many configurations that we *didn't* run, but this selection should give us a
good sampling of the various CPU and storage options available on these clouds.
We would very much welcome any suggestions for additional configurations which
might be more performant or cost effective.

After loading about 100GB (half) of the taxi data into each cluster, we ran 20
iterations of each of the following queries on each of our configurations:

1. `TopN(distance, Row(pickup year=2011))` - Number of rides for each integer distance (in miles) in the year 2011
2. `TopN(distance)` - Number of rides for each integer distance in miles.
3. `TopN(cab type)` - Number of rides in each cab type (yellow or green)
4. `GroupBy(year, passengers, distance)` - Number of rides for every combination of year, number of passengers, and distance in miles
5. `GroupBy(cab type, year, month)` - Number of rides for every combination of cab type, year, and month.
6. `GroupBy(cab type, year, passengers)` - Number of rides for every combination of cab type, year, and number of passengers.
7. `Count(Union(Row(year=2012), Row(year=2013), Row(month=March)))` - Number of rides in 2012, 2013, or March of any year.
8. `Count(Intersect(Row...)...)` -  Number of rides matching a nested boolean combination of 29 values.

We also ran Pilosa's extensive suite of micro-benchmarks which test some compute
and I/O related functions with fewer variables to consider.

### Results

First, let's take a look at raw performance and see which configuration was fastest for each query on average over the 20 iterations.

| cloud | instance type    | nodes | query                                             |
|-------|------------------|------:|--------------------------------------------------- |
| AWS   | r5d.12xlarge     |     2 | TopN(distance, Row(pickup year=2011))             |
| AWS   | c5.9xlarge       |     3 | TopN(distance)                                    |
| AWS   | c5.9xlarge       |     3 | TopN(cab type)                                    |
| OCI   | BM.HPC2.36       |     2 | GroupBy year, passengers, dist                    |
| AWS   | r5d.12xlarge     |     2 | Group By cab type, year, month                    |
| AWS   | r5d.12xlarge     |     2 | Group By cab type, year, passengers               |
| AWS   | c5.9xlarge       |     3 | Count Union of 3 rows                             |
| Azure | Standard_F32s_v2 |     3 | Count Of intersection of unions involving 29 rows |

Wow! AWS is looking pretty good here, taking 6 of the 8 benchmarks evenly split
across their compute and memory optimized instances. Interestingly, the compute
optimized instances are winning at what are probably the 3 simplest queries
while the r5 instances are dominating some of the heavier queries. The OCI HPC
instances snuck in on one of the 3 field "Group By" queries, and Azure's compute
optimized Standard_F32s won the long segmentation query.

To get an idea of how much variation there is between the hardware types, this
is the relative performance of each configuration for one of the "Group By"
queries:

![RawGroupByPerf](/img/blog/why-oci/raw-query-perf-horizontal.png)
*Group By Cab, Year, Passengers*


This is a good start, but it has a few problems. Since the data set fits into
memory on all the configurations, any extra memory is effectively wasted. It
also doesn't exercise disk performance at all, and most importantly, it doesn't
take cost into account. This last is fairly easy to remedy; instead of looking
at the minimum average latency for each query, we can look at the minimum
average latency multipled by the cluster cost. With the right unit conversions,
this effectively gives us how much it would cost to run 1 Million of each given
query back to back on a particular configuration... or as we like to call it,
"Dollars per Megaquery". Let's look at which configuration has the best $/MQ for
each of these benchmarks:


| cloud | instance type    | nodes | query                                             |   $/MQ |
|-------|------------------|------:|---------------------------------------------------|--------: |
| OCI   | VM.Standard2.16  |     3 | TopN(distance, Row(pickup year=2011))             |   $8.08 |
| AWS   | c5.9xlarge       |     3 | TopN(distance)                                    |  $13.17 |
| AWS   | c5.9xlarge       |     3 | TopN(cab type)                                    |   $3.65  |
| OCI   | BM.HPC2.36       |     2 | GroupBy year, passengers, dist                    | $281.12 |
| Azure | Standard F32s v2 |     3 | Group By cab type, year, month                    |   $5.51 |
| Azure | Standard F32s v2 |     3 | Group By cab type, year, passengers               |   $8.13 |
| OCI   | VM.Standard2.16  |     3 | Count Union of 3 rows                             |   $3.59 |
| OCI   | VM.Standard2.16  |     3 | Count Of intersection of unions involving 29 rows | $141.75 |

It appears that OCI's VM.Standard2.16 is the king of cost effectiveness. This is
particularly interesting as these instances have 240 GB of memory to go with
their 32 hyperthreads whereas AWS's c5.9xlarge (36 hyperthreads) and Azure's
F32s have 72GB and 64GB respectively. So, although Oracle's Standard2.16 VMs are
running on slower processors than the 3+ GHz Xeon Platinums that AWS and Azure
have, Oracle is able to offer them at a significantly lower price and with far
more memory.

![FilteredTopN](/img/blog/why-oci/filtered-topn-dpmq-horizontal.png)
*Filtered TopN*


![Segmentation](/img/blog/why-oci/29seg-dpmq-horizontal.png)
*29 Value Segmentation*

It's also worth noting that Azure's F32s machines performed best in cost
effectiveness on two pretty intensive queries, while AWS's c5.9xlarges topped
the two simplest queries which are most likely to be affected by network
performance rather than computation. More dedicated testing would probably be
needed to determine whether AWS's networking provides consistently lower latency
between VMs, or whether this is just a fluke. We did use the placement group
feature for AWS, but did not use any comparable features (if they exist) for the
other clouds.

![GroupByCabYearPassengers](/img/blog/why-oci/groupby-dpmq-horizontal.png)
*GroupBy Cab, Year, Passengers*


### Microbenchmarks

The main Pilosa package contains hundreds of micro-benchmarks, many of which
involve some form of data ingestion. Depending on the parameters, these are both
CPU and disk intensive.

These benchmarks take advantage of at most 32 hyperthreads, so the per-thread
performance of a CPU is more important than the total performance.

The most compute intensive benchmarks are dominated by the c5.9xlarge which
isn't too surprising, but I would have expected similar performance out of
Azure's F32s as they are also running high-end Intel processors.

![CPU](/img/blog/why-oci/intersection-count-horizontal.png)
*CPU Intensive*

Those benchmarks which are both compute and I/O intensive are mostly swept by
the r5d.12xlarge which is also running Xeon Platinum (at a slightly lower
clock), and has 2 SSDs which we bonded into a single logical volume using LVM
striping.

![CPU and I/O](/img/blog/why-oci/import-concurrent-horizontal.png)
*CPU and I/O Intensive*

The pure I/O benchmarks (literally just writing a file) were taken by Oracle's
DenseIO instances which also have 2 SSDs striped by LVM. This would seem to
indicate that Oracle's disks are slightly faster, or its I/O subsytem slightly
more optimized than AWS. On another note, the c5.9xlarge instances on EBS
actually fare pretty well compared to the NVME SSDs - only about a factor of 2
slower.

![I/O](/img/blog/why-oci/filewrite-horizontal.png)
*I/O Intensive*


### Puzzles

A number of things didn't quite add up that I'd like to dig into more:

- Azure didn't perform as well as expected overall - especially in I/O. I'm not
  sure what type of SSDs Azure uses for its temporary storage, but we should
  probably look into other storage options.
- OCI's BM.Standard2.52 performed poorly. On paper it looks extremely cost
  effective though: 104 hyper threads, ~~2 massive SSDs~~, and more RAM than you
  can shake a stick at for $3.31/hr. With the same processor as the
  `VM.Standard2.16` models, its lack of performance is confusing.
  
<div class="note">
Correction: The <code>BM.Standard2.52</code> does not have SSDs. That would be the
<code>BM.DenseIO2.52</code> which has 8 (!!) NVMe SSDs for a whopping 51.2 TB of total
storage, and otherwise identical specs. It's about twice the cost at $6.63/hr.
This doesn't explain, however, why it's 15-20% slower than the <code>VM.*.16</code> models
on the IntersectionCount (single threaded CPU performance) benchmark.
</div>

- The r5d.12xlarge won the Filtered TopN query by a healthy margin, but got 6th
  in the 29 value segmentation by a factor of 4. Those two queries aren't
  terribly different from a workload perspective, however.

### Conclusion and Future Work

For pure cost-effectiveness, Oracle is pretty compelling, especially given the
amount of memory and SSD you get compared to other cloud providers. The I/O
performance is also excellent.

For pure speed, AWS seems to provide the fastest processors, although it's
unclear why Azure's offering wasn't more competitive in this area - on paper it
seemed like they should be nearly equal.

Overall, we've barely scratched the surface as to the types of tests we could
run, and there are certainly more instance configurations that we could try. A
good exercise would be to measure baseline performance of each subsystem in
isolation (e.g. simple disk write with `dd`, simple network test with a TCP
based `ping` tool, etc.), and then compare that with our assumptions about what
the various Pilosa benchmarks *should* be testing.

For those who wish to look at the data more closely, we've been munging all the
results on [data.world](https://data.world/jaffee/benchmarks), which has been a
pretty nice platform for sharing data and various queries as well as producing
charts. Click "Launch Workspace" on the top right to start playing with the
data - you will have to create a (free) account.

For those who wish to try to reproduce or expand on these benchmarks, we have a
variety of tooling available in our [Infrastructure repository.](https://github.com/pilosa/infrastructure/tree/master/terraform/examples)
This helps with deploying servers, installing and running Pilosa, loading data,
and running the benchmarks.

Thanks for reading, and please reach out if you have any questions!

_Update: Part 2 with GCP, memory bandwidth, and more is [here!](../cloud-bench-redux)_

_Jaffee is a lead software engineer at Pilosa. When he’s not burning cloud credits for science, he enjoys jiu-jitsu, building mechanical keyboards, and spending time with family. Follow him on Twitter at [@mattjaffee](https://twitter.com/mattjaffee?lang=en)._

_Banner Photo by Daniele Levis Pelusi on Unsplash_

