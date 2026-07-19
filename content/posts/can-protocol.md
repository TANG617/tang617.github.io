---
title: CAN Protocol
description: An introduction to Classical CAN and CAN FD identifiers, arbitration, payloads, and frame formats.
publishDate: 2024-04-19T10:30:38+08:00
updatedDate: "2026-07-19T00:00:00+08:00"
tags:
  - CAN
---

# Classical CAN and CAN FD

CAN, or Controller Area Network, is a multi-master broadcast bus designed for reliable communication in electrically noisy environments. “CAN 2.0” commonly refers to Classical CAN. CAN FD extends the frame format with a larger payload and an optional faster data phase.

CAN supports two identifier lengths:

- the base frame format uses an 11-bit identifier;
- the extended frame format uses a 29-bit identifier.

The identifier describes the message and also participates in arbitration; it is not normally a device address. When multiple nodes transmit at the same time, the numerically lower identifier wins non-destructive bitwise arbitration because a dominant bit overwrites a recessive bit.

![Classical CAN and CAN FD frame overview](https://cdn.jsdelivr.net/gh/TANG617/images/202404191050406.png)

## Classical CAN

A Classical CAN data frame carries up to 8 data bytes. Its main fields include the start of frame, arbitration field, control field, data field, CRC, acknowledgement, and end of frame. CAN also defines remote, error, and overload frames.

CAN controllers use bit stuffing, CRC checking, acknowledgement monitoring, and error counters to detect faults. A controller that repeatedly disrupts the bus can eventually enter the bus-off state.

## CAN FD

CAN FD supports payloads of up to 64 bytes. The Flexible Data Rate Format (FDF) bit distinguishes a CAN FD frame, while the Bit Rate Switch (BRS) bit allows the arbitration phase and data phase to use different bit rates. The Error State Indicator (ESI) reports the transmitter's error state.

![CAN FD base-format data frame](https://cdn.jsdelivr.net/gh/TANG617/images/202404191052817.png)

CAN FD retains CAN arbitration principles, but a Classical CAN controller cannot generally receive a CAN FD frame as an ordinary Classical CAN frame. A mixed network therefore requires controllers and transceivers selected for the intended operating mode.

## Reference

- [STMicroelectronics: Introduction to FDCAN peripherals for STM32 product classes](https://www.st.com/resource/en/application_note/an5348-introduction-to-fdcan-peripherals-for-stm32-product-classes-stmicroelectronics.pdf)
