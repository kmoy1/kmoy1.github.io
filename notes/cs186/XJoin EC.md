# Technical Report EC: XJoin Implementation

XJoin is an advanced and pipelineable join algorithm, and one of the subjects of our technical report. Its main benefit is that continues join work even when input record arrival is slow or erratic. Below is the documentation for my implementation of XJoin in code. I decided to implement my own version of a database (with all due respect to MOOCBase) in Java.

XJoin is based on Symmetric Hash Join which was originally designed to allow a high degree of pipelining in parallel database systems. Unfortunately, SHJ requires that both $R$ and $S$'s hash tables fit in memory. Thus SHJ cannot be used if $R$ and $S$'s hash tables extend beyond memory constraints. XJoin fixes this by allowing *partitions* of $R$ and $S$'s hash tables to be flushed to disk. It does this by partitioning inputs. 

Extensions for XJoins that differ from the standard MOOCBase joins: 

- **Implementing our background join process with delayed input (records).** 
- **Implementing multiple threads.**
- Flushing records to disk.
- Ensuring output join relation is correct, without duplicate records



