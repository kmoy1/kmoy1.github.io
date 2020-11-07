# Disk and Files Disc WalkThru

**Q1 (T/F): When querying for a 16 byte record, exactly 16 bytes of data is read from disk. **

**A1: False.**

We only read PAGES from disk.

**Q2 (T/F): In a heap file, all pages must be filled to capacity except the last page. **

**A1: False.**

This is just a BS condition made to sound fancy and thus true. Heap files have its pages filled however. 

**Q3 (T/F): A slot directory that is 512 bytes can address 64 records in a page. **

**A1: False. **

Remember the slot directory is at the page footer for pages with VLRs, and consists of slot count, free space pointer, and entries. Entries are (record pointer, record length) - 8 bytes. So with 64 records we have 64*8 = 512 bytes taken up in the slot directory, leaving no room for the FSP or slot count. So this won't do.

**Q4 (T/F): In a page containing fixed-length records with no nullable fields, the size of the bitmap never changes.**

**A1: True.**

The size of the records is fixed, so we know exactly the maximal number of records on a page. So this means the size of the bitmap will certainly be fixed. 

## Benefits of a record header: VLRs

Remember that a record header contains pointers to the end of variable length fields. Also, remember FLRs come before all VLRs. 

**Q1 (T/F): A record header removes the need for a delimiter character to separate fields in the records**

**A1: True.**

This is literally the purpose of record headers' pointers.

**Q2 (T/F): A record header always matches or beats space cost when compared to FLR format.**

**A2: False.**

Record headers take space in themselves- in particular, 4 bytes per VLR pointer.  So if all the "variable" length records are actually just fixed, the record header is just useless and takes up space per record.

**Q3 (T/F): A record header can access any field without scanning the entire record.**

**A3: True.**

Simple pointer arithmetic. 

**Q4 (T/F): A record header has compact representation of null values.**

**A4: True.**

NULL fields actually don't take up space themselves. There's still a pointer in the header to it, though. 

## Fragmentation and Record Formats

**Fragmentation** occurs when records aren't packed-i.e. there's enough combined space between records on a page such that we can fit another record, but we can't because it's not contiguous space. 

**Q1: Is fragmentation an issue with packed fixed length record page format?**

**A1: Not at all.**

There's no way to get space between records because **records are compacted upon deletion.** (*packed* FLR)

**Q2: Is fragmentation an issue with slotted page VLRs?**

**A1: Definitely.**

Remember for slotted pages and its associated VLRs, we can't just stick a new record into a deleted record "slot" because the sizes may be different. 

**Q3: We usually use bitmaps for pages with fixed-length records. Why not just use a slotted page for pages with fixed-length records?**

**A3: Waste of space.** Slotted pages use a slot directory which contain pointers which are 4 byte addresses *each*. Bitmaps literally only have a bit per record- so we'll only take a byte or two total, probably. 

## Record Formats

```sql
CREATE TABLE Questions (
	qid integer PRIMARY KEY,
	answer integer,
	qtext text,
);
```

Assume the record header stores pointers to the end of variable length fields *only*. 

**Q1: Size of smallest possible record (in bytes)?**

**A1: 12 bytes**

Record header has one pointer to end of qtext (VLF). The qid and answer fields are ints => 4 bytes, and if we set qtext to NULL that's 0 bytes. In total, 12 bytes. 

Now let's say we add a bitmap to the beginning of our record header indicating null fields. Assume this bitmap is *padded* so we round up size (10 bits rounds to 16 bits or 2 bytes). 

**Q2: What is the largest possible record, assuming qtext NULL?**

**A2: 13 bytes.**

Again, 4 bytes for qtext pointer, 4 bytes for answer and qid. 3 bits for our bitmap needed, one bit for each field, which rounds up to 1 byte. So 4+4+4+1 = 13 in total. 

## Linked List Implementation IOs

Let's say linked-list heap file $A$ has 8 full pages and 8 pages with free space.

**Q1: In the *worst case,* how many IOs are required to *find* a page with enough free space?**

**A1: 9 IOs**

Read header page and all 8 data pages linked list (including the last page which contains the space). 

Now assume that after writing a page, the page becomes full and assume that the header page can insert at the beginning of the full pages linked list.

**Q2: How many IOs to write a record to the 8th page with free space?**

**A2: 14 IOs**

9 IOs to first find (read from disk) the 8th page. After we write it and flush (1 IO), we also need to **move this now full page to the beginning of the full pages linked list**. This means editing both the free page and full page linked list ptrs. 

- 1 IO to write the 7th free space page, setting its *next* pointer to NULL
- 1 IO to edit header page, since the number of pages w/ free space has decreased, and also because the location (and pointer) of our page is changing.

- 2 IOs to read the *previous* front of LL into memory and change it. 

**So when a page moves from free space to full space, we need IOs to edit the free space LL, the header, AND the full space LL!** Total: 9+1+1+1+2 = 14 IOs. 

## Page Directory Implementation IOs

Assume we have a page directory heap file $B$. One page in the directory can hold 16 page entries. There are 54 pages in file A in total.

**Q1: In the *worst case,* how many IOs are required to find a page with free space?**

**A1: 4 IOs**

There are *at worst* $\lceil 54/16 \rceil = 4$ header pages, assuming each header page is filled as much as possible. So in the worst case, we have to read all 4 header pages: 4 IOs. 

**Q2: In the worst case, how many IOs are required to write a record to a page with free space (assuming at least one free page exists)?**  

**A2: 7 IOs**.

4 IOs to traverse header pages. 2 IOs to read + write page with free space. 1 more IO to write to page directory.

