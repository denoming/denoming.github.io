---
title: "Video Data Insights"
date: 2025-02-24
categories: [Linux]
tags: [video]
---
# Formats

## RGB

`RGB` is an additive color model where red, green, and blue intensities are added together in different combinations to produce a exhaustive set of colors.

`RGB888` --> `R` is an 8 bit value varies from 0 to 255. Same for `G` and `B`. `RGB565` --> Here `R` is of 5 most significant bits from `R` (8 bits) of `RGB888`. Here `G` is of 6 most significant bits from `G` (8 bits) of `RGB888`. Here `B` is of 5 most significant bits from B (8 bits) of `RGB888`.

`RGB888` to `RGB565`:
```
short int rgb565_pixel;
rgb565_pixel = ((R >> 3) << 11) | ((G >> 2) << 5) | (B >> 3);
```

## YUV

### Planar

`YUV` is the color format where you can have a complete separation of brightness and color components from `RGB` format. `Y` represents brightness components where as `Cb` and `Cr` represents color components. In memory, `Y` followed by `Cb` and followed by `Cr`:
`[Y1Y2......][Cb1Cb2......][Cr1Cr2.......]`

### Semi Planar

YUV420
In memory, `Y` followed by interleaved data of `Cb` and `Cr`, looks as below:
`[Y1Y2......][Cb1Cr1Cb2Cr2......]`

Y = width x height pixels (bytes)
Cb = Y / 4 pixels (bytes)
Cr = Y / 4 pixels (bytes)
Total num pixels (bytes) = width * height * 3 / 2

### Interleaved

In case of `YUV422` (e.g. `YUYV`) interleaved data, it looks as below:
`Y1U1Y2V1 Y3U2Y4V2 ... ...`



