# Rle boot protocol

**RLE is a modern boot protocol designed to provide a standardized and strict way for the kernel to communicate with the information provided by the bootloader.**

## Who is the RLE boot protocol intended for?
It is aimed at hobbyist operating systems but also at real-world operating systems. RLE is designed to be as flexible as possible and to adapt to any use case.

## Dictionary

| Word | Meaning                             |
|------|-------------------------------------|
| u8  | Unsigned int 8 bits                |
| u16 | Unsigned int 16 bits, little-endian |
| u32 | Unsigned int 32 bits, little-endian |
| u64 | Unsigned int 64 bits, little-endian |
| struct | Data structure, always packed |

## Table of Contents

- [Boot tables](./boot-tables.md)
