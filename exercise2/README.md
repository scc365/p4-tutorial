# Implementing a P4 Calculator

## Introduction

The objective of this tutorial is to implement a basic calculator
using a custom protocol header written in P4. The header will contain
an operation to perform and two operands. When a switch receives a
calculator packet header, it will execute the operation on the
operands, and return the result to the sender. The P4 language implement a limited set of arithmetic operations. To expand coverage, we will also explore how packets can be send to the controller, in order to extend support. 

## Step 1: Run the (incomplete) starter code

The directory with this README also contains a skeleton P4 program,
`p4src/main.p4`, which initially drops all packets.  Your job will be to
extend it to properly implement the calculator logic.

As a first step, compile the incomplete `p4src/main.p4` and bring up a
switch in Mininet to test its behavior.

1. In your shell, run:
2. 
   ```bash
   make p4-build
   make proto-build
   make start
   ```

   This will:
   * compile `main.p4`, and

   * start a Mininet instance with one switches (`s1`) connected to
     two hosts (`h1`, `h2`).
   * The hosts are assigned IPs of `10.0.1.1` and `10.0.1.2`.

3. We've written a small Python-based driver program that will allow
you to test your calculator. You can run the driver program directly
from the Mininet command prompt:

```
mininet> h1 python3 /mininet/calc.py 5+5
Didn't receive response
```

3. You can express different expressions using the second CLI parameter.
 The calc program will parse your expression, and
prepare a packet with the corresponding operator and operands. It will
then send a packet to the switch for evaluation. When the switch
returns the result of the computation, the test program will print the
result. However, because the calculator program is not implemented,
you should see an error message.

```
mininet> h1 python3 /mininet/calc.py 5&5
Didn't receive response
```

## Step 2: Implement the Calculator

To implement the calculator, you will need to define a custom
calculator header, and implement the switch logic to parse header,
perform the requested operation, write the result in the header, and
return the packet to the sender.

We will use the following header format:

             0                1                  2              3
      +----------------+----------------+----------------+---------------+
      |      P         |       4        |     Version    |     Op        |
      +----------------+----------------+----------------+---------------+
      |                              Operand A                           |
      +----------------+----------------+----------------+---------------+
      |                              Operand B                           |
      +----------------+----------------+----------------+---------------+
      |                              Result                              |
      +----------------+----------------+----------------+---------------+


-  P is an ASCII Letter 'P' (0x50)
-  4 is an ASCII Letter '4' (0x34)
-  Version is currently 0.1 (0x01)
-  Op is an operation to Perform:
 -   '+' (0x2b) Result = OperandA + OperandB
 -   '-' (0x2d) Result = OperandA - OperandB
 -   '&' (0x26) Result = OperandA & OperandB
 -   '|' (0x7c) Result = OperandA | OperandB
 -   '^' (0x5e) Result = OperandA ^ OperandB


We will assume that the calculator header is carried over Ethernet,
and we will use the Ethernet type 0x1234 to indicate the presence of
the header.

Given what you have learned so far, your task is to implement the P4
calculator program. There is no control plane logic, so you need only
worry about the data plane implementation.

A working calculator implementation will parse the custom headers,
execute the mathematical operation, write the result in the result
field, and return the packet to the sender.

## Step 3: Run your solution

Follow the instructions from Step 1.  This time, you should see the
correct result:

```bash
mininet> h1 python /mininet/calc.py 10+10
[<__main__.Token instance at 0x7ffffca68710>, <__main__.Token instance at 0x7ffffca687a0>, <__main__.Token instance at 0x7ffffca688c0>]
###[ Ethernet ]### 
  dst       = 00:04:00:00:00:00
  src       = 08:00:00:00:01:01
  type      = DCA
###[ P4calc ]### 
     P         = 'P'
     Four      = '4'
     version   = 0x1
     op        = '+'
     operand_a = 10
     operand_b = 10
     result    = 3735927486
###[ Raw ]### 
        load      = ' '

20
```

## Step 4: Implement a P4 controller

The P4 language supports the following operators: '+', '-', '&', '|', '^'.
Nonetheless, the multiplication operation cannot be supported by the controller.
To implement this functionality, you need to use a controller. We provide a P4
controller skeleton implemented in Python in the folder (`controller.py`). You
can run the controller with the following command.

```bash
python3 controller.py --p4info p4src/build/p4info.txt --bmv2-json p4src/build/bmv2.json
```

You need first to modify your P4 program to send packets to the controller in
the `operation_mult` action. You then need to process appropriately the response
packet in your controller code and use a packet out message to send it to the
switch. The Python file `util/lib/p4_cli/helper.py` offers a number of methods to
construct and transmit a packet out message.

## Relevant Documentation

The documentation for P4_16 and P4Runtime is available [here](https://p4.org/specs/)

All exercises in this repository use the v1model architecture, the documentation for which is available at:
1. The BMv2 Simple Switch target document accessible [here](https://github.com/p4lang/behavioral-model/blob/master/docs/simple_switch.md) talks mainly about the v1model architecture.
2. The include file `v1model.p4` has extensive comments and can be accessed [here](https://github.com/p4lang/p4c/blob/master/p4include/v1model.p4).
