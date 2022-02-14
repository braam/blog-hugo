---
title: "Merge PDF Files With Linux"
date: 2022-02-14T15:24:22+01:00
draft: false
tags:
- linux
---

Easy trick to remember how to join/merge multiple pdf files into one big pdf file.
Thanks to the poppler-utils we have a few pdf tools including **pdfunite**, packet details: [https://packages.debian.org/sid/poppler-utils](https://packages.debian.org/sid/poppler-utils)

Install the tools with:
```bash
sudo apt install poppler-utils
```

After this we can merge multiple pdf files together, using following syntax:
```bash
pdfunite 1.pdf 2.pdf 3.pdf output.pdf
```

The file **output.pdf** will contain all pdf files joined together.
