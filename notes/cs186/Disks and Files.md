# Disks and Files

### Memory and Disk

Data that a database processes must already exist in memory (RAM). However, memory is limited: if data becomes too big, we'll find ourselves running out of space very soon. We need a bigger storage area for this data: disks. **Disks** are used to cheaply store all of a database’s data persistently, which means that even in cases of shutdown, the data is saved. However, there is a large cost whenever data is accessed from disk or when new data is written to disk. 

### Files, Pages, Records

The basic unit of data for relational databases are **records** (rows). Records are organized into **relations** (tables). Relations can be modified, deleted, searched, or created in memory, usually via a language like SQL.

The basic unit of data for disk, on the other hand, is a **page**. This is the smallest possible size of transfer from disk to memory and vice versa. 

We now have to change our relational database structure slightly to be compatible with disk. Each relation is stored in its own file, and records become pages in the file. The database receives metadata about these relations and records by the relation schema and access pattern, such as the type of file used and how pages and records are organized in the file and page respectively. 

### Choosing File Types

We said one of the metadata that our database determined was the type of file used. There are two main types of files: **Heap Files** and **Sorted Files.** The file type chosen will really depend on the minimum **I/O cost** associated with a relation's **access pattern**. Remember that access patterns for a given relation consist of insert, delete, and scan operations to that relation. 

An **I/O** is a unit of "cost" equivalent to one page read from disk, which is also equivalent to one page write to disk. 

 ### Heap Files

**Heap files** do not have any ordering of pages in the file **or** records on the page. There are two main implementations of a heap file:

- **Linked List:** Each data page contains records (as always) but also contains a **free space tracker**, as well as **pointers** to next pages. The **header page** is located at the start of the file, and splits the pages into two linked lists: **full pages** and **free pages**. These partitions are self-explanatory: full pages are pages full with records and free pages are pages with space available for records. When free pages become full, they move to the front of full pages LL. 
- **Page Directory**: This format implements a **linked list of header pages**, resembling a page directory (go figure). Each header page contains entries that both point to a single data page AND contain metadata about free space in that page. Pages no longer need to contain pointers. 

In a linked list heap file implementation, we would have to read each page (header included) from disk to see if there's enough space to insert a record. Remember that this costs an I/O for each page encountered during our traversal!

In a page directory heap file implementation, however, we only need to read a header page to find the free space!  **Header pages themselves only tell us about free space and give pointers to data pages**, so all reading a header page will do is allow us to *lock on* to a sufficient data page to insert into. Header pages ***are not*** a collection of data pages themselves.  We still need 2 I/Os to read and write (load and edit) a data page from disk. Note we also have to use another I/O to update the header afterwards.

### Sorted Files

**Sorted files** order pages as well as order records on those pages. In fact, since all records are sorted, data pages will be sorted accordingly (by range). 

**Searching through sorted files takes logN I/Os**. Since we know all records are sorted, we can simply use binary search to find a record.

**Inserting takes logN + N I/Os** on the average case. logN for finding the correct page to insert on, and another N (on average) to account for insertion pushing back records.

### Record Types

Records can either be **Fixed Length Records** (FLRs) or **Variable Length Records**(VLRs). FLRs can only contain fixed-length fields (integer, boolean, etc.) while VLRs can contain fixed length AND variable length fields (varchar). VLRs store fixed-length fields before variable-length fields. To keep track of the variable length fields, a **record header** is used for VLRs that contain pointers pointing to the end of variable length fields. 

The record type is completely determined by the relation schema, i.e. the column types. Logically, all FLRs with the same schema must be equal in the number of bytes. However, VLRs are not all uniform size. 

### Page Formats

Pages can either contain FLRs or VLRs. Pages can also be **packed** or **unpacked**.

For pages with FLRs, a page header is used to store the number of records on that page. If the page is packed, then this is all that's needed, since we know all FLRs have uniform size (we can compute the offset easily by just multiplying # records by record size) If the page is unpacked, however, we might also need a **bitmap** that tells us with slots are filled (1) and which are not (0). Deletion is as easy as setting the corresponding bit to 0.

For pages with VLRs, it's a little bit trickier to manage, since we don't really know a record's size. So VLRs use a **page footer** with a **slot directory**. This slot directory contains three important pieces of information (right to left): **slot count**, **free space pointer**, and entries. Each entry maps to a current record, and consists of a (record pointer, record length) pair. Insertion is the same no matter whether the VLR page is packed or not: insert the record at the location pointed to by FSP, then set an entry for that record.

Deletion, on the other hand, depends on packing. If a VLR page is unpacked, to delete a record, we simply update the slot directory by removing the corresponding entry. However, if the page is packed, we must **also** shift all records in front of the deleted record back by that deleted entry's length, and also shift those records' entries 1 position in the slot directory.

Note that deleted VLRs will still count in the slot count, since those are still slots for records to be inserted. 