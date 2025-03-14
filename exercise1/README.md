#  Basic Forwarding

## Table of Contents

- [Basic Forwarding](#basic-forwarding)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Prerequisites](#prerequisites)
  - [Step 1: Run the (incomplete) starter code](#step-1-run-the-incomplete-starter-code)
    - [A note about the control plane](#a-note-about-the-control-plane)
  - [Step 2: Implement L3 forwarding](#step-2-implement-l3-forwarding)
  - [Step 3: Run your solution](#step-3-run-your-solution)
  - [What's Next?](#whats-next)
    - [Troubleshooting](#troubleshooting)
      - [Cleaning up Mininet](#cleaning-up-mininet)
  - [Relevant Documentation](#relevant-documentation)

## Introduction

The objective of this exercise is to copmlete the imlpementation of a P4 program that implements basic
forwarding. To keep things simple, we will just implement forwarding for IPv4.

With IPv4 forwarding, the switch must perform the following actions for every
packet: (i) update the source and destination MAC addresses, (ii) decrease the time-to-live (TTL) in the IP header, and (iii) forward the packet out of the appropriate port.

Your switch will have a single table, which the control plane will populate with
static rules. Each rule will map an IP address to the MAC address and output
port for the next hop. We have already defined the control plane rules, so you
only need to implement the data plane logic of your P4 program.

We will use the following topology for this exercise. It is a single pod of a
fat-tree topology and henceforth referred to as pod-topo:
![pod-topo](./mininet/pod-topo.png)

Our P4 program will be written for the V1Model architecture implemented on
P4.org's bmv2 software switch. The architecture file for the V1Model can be
found at:
[p4c/p4include/v1model.p4](https://github.com/p4lang/p4c/blob/main/p4include/v1model.p4).
This file describes the interfaces of the P4 programmable elements in the
architecture, the supported externs, as well as the architecture's standard
metadata fields. We encourage you to take a look at it.

> **Spoiler alert:** There is a reference solution in the `solution`
> sub-directory. Feel free to compare your implementation to the reference.

## Prerequisites

## Step 1: Run the (incomplete) starter code

The directory with this README also contains a skeleton P4 program,
`p4src/main.p4`, which initially drops all packets. Your job will be to extend
this skeleton program to properly forward IPv4 packets.

Before that, let's compile the incomplete `main.p4` and bring up a switch in
Mininet to test its behavior.

1. In your shell, run:

   ```bash
   make p4-build
   make proto-build
   make start
   ```

This will:

   - compile `main.p4`, and
   - start the pod-topo in Mininet and configure all switches with the
     appropriate P4 program + table entries, and
   - configure all hosts with the commands listed in
     [pod-topo/topology.json](./pod-topo/topology.json)

2. You can now access the Mininet command prompt with the command `make mn-cli`.
   Try to ping between hosts in the topology:

   ```bash
   mininet> h1 ping h2
   mininet> pingall
   ```

3. Press Ctrl+D leave to detach from the Mininet command line. Then, to stop
   mininet:

   ```bash
   make stop
   ```

   And to delete all pcaps, build files, and logs:

   ```bash
   make clean
   ```

The ping failed because each switch is programmed according to `main.p4`, which
drops all packets on arrival. Your job is to extend this file, so it forwards
packets.i Everytime you want to test your solution, you need to recompile the P4 
code, running `make p4-build`, and restart the Mininet instance, running `make restart`.

### A note about the control plane

A P4 program defines a packet-processing pipeline, but the rules within each
table are inserted by the control plane. When a rule matches a packet, its
action is invoked with parameters supplied by the control plane as part of the
rule.

In this exercise, we have already implemented the control plane logic for you.
As part of bringing up the Mininet instance, the `make start` command will
install packet-processing rules in the tables of each switch. These are defined
in the `mininet/sX-runtime.json` files, where `X` corresponds to the switch number.

**Important:** We use P4Runtime to install the control plane rules. The content
of files `sX-runtime.json` refers to specific names of tables, keys, and actions,
as defined in the P4Info file produced by the compiler (look for the file
`p4src/build/.p4info.txt` after executing `make p4-build`). Any changes in the P4
program that add or rename tables, keys, or actions will need to be reflected in
these `sX-runtime.json` files. We provide a minimal P4 GRPC python client under `util/simple_conrtoller.py`

## Step 2: Implement L3 forwarding

The `main.p4` file contains a skeleton P4 program with key pieces of logic
replaced by `TODO` comments. Your implementation should follow the structure
given in this file---replace each `TODO` with logic implementing the missing
piece.

A complete `main.p4` will contain the following components:

1. Header type definitions for Ethernet (`ethernet_t`) and IPv4 (`ipv4_t`).
2. **TODO:** Parsers for Ethernet and IPv4 that populate `ethernet_t` and
   `ipv4_t` fields.
3. An action to drop a packet, using `mark_to_drop()`.
4. **TODO:** An action (called `ipv4_forward`) that:
   1. Sets the egress port for the next hop.
   2. Updates the Ethernet destination address with the address of the next hop.
   3. Updates the Ethernet source address with the address of the switch.
   4. Decrements the TTL.
5. **TODO:** A control that:
   1. Defines a table that will read an IPv4 destination address, and invoke
      either `drop` or `ipv4_forward`.
   2. An `apply` block that applies the table.
6. **TODO:** A deparser that selects the order in which fields inserted into the
   outgoing packet.
7. A `package` instantiation supplied with the parser, control, and deparser.
   > In general, a package also requires instances of checksum verification and
   > recomputation controls. These are not necessary for this tutorial and are
   > replaced with instantiations of empty controls.

## Step 3: Run your solution

Follow the instructions from Step 1. This time, you should be able to
successfully ping between any two hosts in the topology.

## What's Next?

Other questions to consider:

- How would you enhance your program to respond to ARP requests?
- How would you enhance your program to support traceroute?
- How would you enhance your program to support next hops?
- Is this program enough to replace a router? What's missing?

### Troubleshooting

There are several problems that might manifest as you develop your program:

1. `main.p4` might fail to compile. In this case, `make p4-build` will report
   the error emitted from the compiler and halt.

2. `main.p4` might compile but fail to support the control plane rules in the
   `s1-runtime.json` through `s3-runtime.json` files that `make start` tries to
   install using P4Runtime. In this case, `make start` will report errors if
   control plane rules cannot be installed. Use these error messages to fix your
   `main.p4` implementation.

3. `main.p4` might compile, and the control plane rules might be installed, but
   the switch might not process packets in the desired way. The `logs/sX.log`
   files contain detailed logs that describe how each switch processes each
   packet. The output is detailed and can help pinpoint logic errors in your
   implementation.

#### Cleaning up Mininet

In the latter two cases above, `make start` may leave a Mininet instance running
in the background. Use the following command to clean up these instances:

```bash
make stop
```

## Relevant Documentation

The documentation for P4_16 and P4Runtime is available
[here](https://p4.org/specs/)

All exercises in this repository use the v1model architecture, the
documentation for which is available at:

1. The BMv2 Simple Switch target document accessible
   [here](https://github.com/p4lang/behavioral-model/blob/master/docs/simple_switch.md)
   talks mainly about the v1model architecture.
2. The include file `v1model.p4` has extensive comments and can be accessed
   [here](https://github.com/p4lang/p4c/blob/master/p4include/v1model.p4).
