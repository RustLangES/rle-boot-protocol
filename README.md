# Rle boot protocol

RLE boot protocol is a standard aimed at providing a strict method for communicating information from the bootloader to the kernel. It is based on standards such as Limine’s, although it adds its own elements, and the ones taken from Limine are modified to adapt to the vision of the boot protocol.

RLE has the philosophy of being simple, flexible, and pedantic. Any implementation of RLE can be added to the repository as long as it complies with the standard. Any type of feature is welcome; this is done with the aim of avoiding fragmentation, although it must first be submitted as an issue. If you want to contribute to RLE, please read CONTRIBUTING.md

# Note

All structures are packed. Look up how to mark a structure as packed in your programming language. In languages like Rust, mark your structures that interact with the protocol using ``#[repr(C, packed)]``

This specification for structures uses the following format name: type which can be easily adapted to C as type name. For example, if a part of the specification uses ``num: u32``, it can be adapted to ``uint32_t num``.

# Revisions

The protocol manages the version in which the kernel is to boot. In this case—similar to Limine—it is handled as follows: `1` (there is no default selection, so the kernel must explicitly know which protocol version it intends to use).

The `.revision` section must be present and contain the following list:
```
[ 0xA3F1C7D4B9826E5F, 0x7D4E9B3A1C6F8D20, BASE_REVISION_NUMBER ]
```

This list consists of `u64` (unsigned 64‑bit) integers. The bootloader must read this section, verify that the two magic numbers are correct, and then boot the kernel using the version specified by `BASE_REVISION_NUMBER`.

# Requests/Response

RLE agrees with Limine that requests are an excellent way to obtain data from the bootloader, and for developer convenience we use requests. A simple way to define requests is that they are hooks to tell the bootloader to look for information requested by the kernel. An important detail is that there can only be one request of the same type; if the bootloader finds two requests, it must refuse to boot.

All requests, including the start and end markers, go in the .requests section.

All of this is done through structures, which usually follow this base. Some may add more information, which will be documented later.

```
id: u64 // 64-bit unsigned magic number used by the bootloader to recognize what information to retrieve
state: RequestState // note: this is u8
response: u64 // address pointing to the response header
```

If the request could not be completed successfully, then the bootloader must leave everything as it was (except for state, which must be modified to indicate the cause).

If everything goes well, the bootloader must fill the response field with a VALID and readable memory address that contains the header.

RequestState indicates the status of the request. Its purpose is to help the kernel understand why the response is invalid. Its entries are as follows:
```
NONE = 0 // this is not used by the bootloader but by the kernel when defining a structure
OK = 1  // everything went well
UNSUPPORTED = 2 // the request is not supported by the hardware
UNKNOWN_ID = 3  // the request's magic number is invalid
```
All enum values are u8 (unsigned 8-bit integers).

# Start/end markers
Requests will only begin to be read when a start marker is present; everything before the start marker will be ignored. The end marker defines up to where the requests will be read—everything outside these two markers will be disregarded.

An important clarification is that there can only be one start marker and one end marker. If the bootloader encounters more than one of either marker, it must refuse to boot.

The correct way to use requests is as follows:
```
start_marker
// requests
end_marker
```
## Start header
The start header shall consist of a sequence of four `u64` (unsigned 64‑bit) integers, in the following order:
```
[
  0xC7A1D3F4B9826E5F
  0x9E4B7C2A1F6D8B30
  0x5D3F8A7E2C1B9D44
  0xA84E1B3C7D9F2036
]
```

## End header
The end header  shall consist of a sequence of four `u64` (unsigned 64‑bit) integers, in the following order:
```
[
  0xF2B4C8D1A73E9F60
  0x3D9A7E4B1C58B2E7
  0x8E1F6C3A9B04D7A2
  0x7ACD2E9F1348B6C5
]
```

# Entry memory layout

The executable is recommended to be loaded at 0xffffffff80000000. Lower half executables are not allowed, nor are relocations. The kernel must be loaded in the higher half. The bootloader must be responsible for properly configuring the HHDM, and the kernel must be loaded in that region. By convention, the HHDM will be loaded at 0xFFFF800000000000 (on x86_64 architectures). NOTE to the reader: it should be discussed how to handle this for x86.

# Memory regions

This section defines how memory regions are segmented within the boot protocol. The bootloader does not guarantee a specific order; however, it does ensure that each type accurately represents its intended purpose. For instance, the bootloader must guarantee to the executable that any entry marked as USABLE is indeed free and does not contain critical information.

A memory entry is marked as:

```
RESERVED = 0  // reserved by the firmware, not usable
BAD_MEMORY = 1  // memory that cannot be used due to physical damage, not usable
RESPONSES = 2 // the region where the bootloader stores all responses; may be used once the response data is no longer needed
EXECUTABLES = 3 // contains all kernel-related data, not usable
MODULES = 4 // contains kernel modules; this region should be omitted if no modules were passed, and may be used after all modules have been read
USABLE = 5 // free and usable memory, contains no initial data
FRAMEBUFFER = 6 // this memory is mapped to the framebuffer, not usable
ACPI_RECLAIMABLE = 7 // memory used by ACPI; may be reclaimed once ACPI tables are no longer in use
ACPI_NVS = 8 // this section is permanently used by ACPI, cannot be used
```


# Machine state at entry

Additionally, the bootloader must load a GDT; the IDT is not the responsibility of the bootloader and MUST NOT be loaded by it. The executable must ensure that a valid IDT is loaded.

The GDT configuration is the same as in limine:

Null descriptor 16-bit code descriptor. Base = 0, limit = 0xffff. Readable. 16-bit data descriptor. Base = 0, limit = 0xffff. Writable. 32-bit code descriptor. Base = 0, limit = 0xffffffff. Readable. 32-bit data descriptor. Base = 0, limit = 0xffffffff. Writable. 64-bit code descriptor. Base and limit irrelevant. Readable. 64-bit data descriptor. Base and limit irrelevant. Writable.

IF flag, VM flag, and direction flag are cleared on entry; other flags are undefined.

PG is enabled (cr0), PE is enabled (cr0), PAE is enabled (cr4), WP is enabled (cr0), LME is enabled (EFER), NX is enabled (EFER) if available. If 5-level paging is requested and available, then 5-level paging is enabled (LA57 bit in cr4).

The A20 gate is opened.

Legacy PIC (if available) and IO APIC IRQs (only those with delivery mode fixed (0b000) or lowest priority (0b001)) are all masked.

In EFI systems, boot services must be exited before booting.

rsp points to the stack; the bootloader must ensure to the kernel that rsp is a valid address. The default size is 64 KiB, but the kernel may specify its own stack size via a request.

## TODO: requests list
