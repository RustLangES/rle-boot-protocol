# Boot tables
A **boot table** is a memory region that contains information related to the boot process. There are three types of tables:

- **Master Table**
- **Extensions Table**
- **Modules Table**

Each table holds specific data, but only the **master table** is accessible at boot time. It contains metadata about how to locate and access the other tables, as well as general boot information.

Every table must begin with its **header**, and the **minimum size** of any table is the size of its header. The **bootloader** must guarantee the following to the **kernel** when reading the tables:

- The addresses of the tables are correct
- The memory region containing the tables always includes a header, even if the table is empty
- The master table contains valid addresses to the other tables

## Accessing the Tables

To access a table, the **master table** is used and must be placed in the following CPU registers:

- `rsi`: address of the master table
- `rax`: size of the master table

The bootloader must ensure that these registers contain accurate information for the kernel to proceed.

## Table Checksums

Each table must include a `u16` field representing its **checksum**. The bootloader is responsible for ensuring that **all headers** contain the correct checksum values.

The expected checksums are:

| Table Type       | Checksum Value |
|------------------|----------------|
| Master Table     | `0xAAFF`       |
| Extensions Table | `0xFFBB`       |
| Modules Table    | `0xBBAA`       |

## Master table Header
The structure of the master table is as follows:

-   checksum: u16

-   extensions_table_addr: u64

-   extensions_table_size: u16

-   modules_table_addr: u64
-   modules_table_size: u16


The bootloader **MUST ENSURE** that all information is correct, including both the checksum and the memory addresses.

extensions_table_addr: contains a valid address that must never be null for the extensions table. This address points to the beginning of the header. The kernel must perform a read of extensions_table_size to obtain the header of the extensions table.

modules_table_addr: just like extensions_table_addr, it is a valid address that must never be null and points to the header of the modules table. The kernel must perform a read of modules_table_size to obtain the header of the modules table.
