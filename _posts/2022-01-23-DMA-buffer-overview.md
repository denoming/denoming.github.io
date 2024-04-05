---
title: "DMA buffer overview"
date: 2022-01-23
categories: [Engineering,Embedded]
tags: [dma,esp32]
---

# DMA buffer

## DMA buffer size

**Small DMA buffer means more interrupts, more work for the CPU**
**Large DMA buffer means less interrupts, less work for the CPU**

Data transfer rate:
```
(16bits * 2) / 8 * 44.1KHz = 176KBytes/s
```

* `16bits` - bits per sample;
* `2` - sampling in stereo (left and right channel);
* `44.1KHz` - frequency.

For example, if we have DMA buffer size equal the size of 8 samples the interrupt frequency equal:
```
8 / 44.1KHz ~ 181us
```
Every `181us` the CPU will be interrupted.

For example, if we have DMA buffer size equal the size of 1024 samples the interrupt frequency equal:
```
1024 / 44.1KHz ~ 23ms
```
Every `23ms` the CPU will be interrupted.

**Large Buffer = More Time, but there is a tradeoff - latency**

We need to wait for the DMA transfer to complere before we can start reading from the buffer.
By default the minimum size of the DMA buffer is 8 samples and at most 1024 samples. The size of the DMA buffer relates to `dma_buf_len` field within `i2s_config_t` structure.

The number of bytes that are actually being used:
```
bits_per_sample/8 * num_channels * dma_buf_len * dma_buf_count
```

The more memory is allocated for the DMA buffer the less memory is available for code. Internal SRAM usually have 328Kbytes memory size.

## DMA buffer count

The more number of DMA buffer we have the more time to process data is available.

For example, if we only one DMA buffer:
```
1 / 44KHz ~ 22us
```
We will have only `22us` before the next banch of data will come. Thus we will have only `22us` period to process the data.

## Conclusions

For example we need to transfer audio to remote server. The processing of data on the remote server takes around `100ms`. Therefore, the total rate equals:
```
100ms * 44.1Hz = 4410 samples
4410samples * 2channels * 16bits/8 = 17KBytes
```

The size `17KBytes` is a lower limit of the DMA buffer. We need a space equals `17KBytes` to store while we sending the data to the remote server.

* `dma_buf_len` - trade off between latency and CPU load;
* `dma_buf_count` - tradeoff between allowed processing time and memory use (minimum of 2).
