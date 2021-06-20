Course Link: https://www.youtube.com/playlist?list=PLSE8ODhjZXjbohkNBWQs_otTrBTrjyohi


# Relational Model #

## Disk oriented architecture ##
 - Slotted pages
 - Log structured orgnanization
   - HBase, Cassandra, LevelDB, RocksDB, S3
   - Build indexes, compact log.

## Tuple storage ##
  - Tuple is seq of bytes. DBMS interprets bytes into attributes and values
  - Overflow storage pages
  - External storage(Blob) (no durability/transactions)

## OLTP v/s OLAP ##
### OLTP  ###
Simple queries read/update small amount of data
### OLAP ###
Queries read large amounts of data spanning multiple entities

## n-ary Storage Model(NSM) ##
-  DBMS stores all attributes for a single tuple contiguously in a page.

* Advantages
   * Fast inserts, updates, and deletes.
   * Good for queries that need the entire tuple.

* Disadvantages
   * Not good for scanning large portions of the table and/or a subset of the attributes.

## Decomposition Storage Model(Columnar) ##
The DBMS stores the values of a single attribute for all tuples contiguously in a page.
   * Ideal for OLAP workloads where read-only queries perform large scans over a subset of the table’s attributes.

* Advantages
  * Reduces the amount wasted I/O because the DBMS only reads the data that it needs.
  * Better query processing and data compression (more on this later).
* Disadvantages
  * Slow for point queries, inserts, updates, and deletes because of tuple splitting/stitching.

* Examples
  * SybasIQ, Vertica, Druid, Clickhouse, Pinot, MonetDB

## Buffer Pool Manager ##
