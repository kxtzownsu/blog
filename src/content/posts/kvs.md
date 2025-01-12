---
title: KVS - Kernel Version Switcher
published: 2024-09-25
description: 'a small paragraph on how this was found :spoob:'
image: ''
tags: [shim, rma, chromeos, chromebook]
category: 'chromebook'
draft: false 
lang: 'en'
---

# KVS - Kernel Version Switcher
a small writeup on how I found this


### Backstory: 
so, around February 2024, I came up with an idea in Whelement. A way to switch the kernver on Chromebooks. <br>
If we could do this, we could bypass Google's downgrade protections and allow anyone to downgrade to R107 and use SH1mmer again. 

### Problems:
- Due to the patch in R111 that disabled writing to TPM indexes in dev reco, this was impossible with FWMP (enrolled). <br>
- I had no clue how to write to the TPM at the time

### How I did it
I knew at the time that `chromeos-tpm-recovery` switched the kernver back to 0x00010001, so I dived into the ChromiumOS source code to find out how. (*its how i found tpmc as well :3*) <br>
Something in that file immediately struck my eyes, [one of the last lines](https://chromium.googlesource.com/chromiumos/platform/vboot_reference/+/master/utility/chromeos-tpm-recovery#217) in the file, it mentioned the RW space for the kernver. 

I started going through the functions to figure out what reset_rw_space did. First, I found out that `$secdata_kernel` was actually the infamous `0x1008` as well! <br>
After that, I went to the `reset_rw_space` function's source and found out the args were `reset_rw_space <index> <bytes in a string>`. <br>
This meant that `0x1008` was the index and `02  4c 57 52 47  1 0 1 0  0 0 0  55` was.. something. <br>
<sup>Yeah, at the time I didn't know this, but that was actually the hex values for the kernver v0.2 struct.</sup> <br>

I figured out that `reset_rw_space` actually called another function, `write_space`. This newly-discovered function is what actually wrote the data to the TPM, and so, thats where I found the actual command that's being ran when you run `chromeos-tpm-recovery`, `tpmc write 0x1008 "02 4c 57 52 47 01 00 01 00 00 00 00 55"`.

At the time, I had a [fakemurk](https://github.com/MercuryWorkshop/fakemurk)'d chromebook which somehow had gotten set to kernver 0. This is how I was first able to obtain the kernver 0 hex and how it was discovered. A few people from Titanium Network who had kernver 2 and 3 dumped their values and sent it to me. I then ran the files through hexdump and [hexed.it](https://hexed.it) and put them into a file.

### Finale
thats basically it, after this, I wrote my TUI, my builder, and another thing thats still private (Kernver Generator)

tysm for reading! <3 
