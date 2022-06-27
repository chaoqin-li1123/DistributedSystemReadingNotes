# Describe the Problem
A database that has flexible row key, column key and version number for each entry.

# How They Solve the Problem
- A multi-dimensional sorted map, indexed by row key, column key and version, “row_key + column_key + version”: value
- Break the table into tablets: a continuous group of row. Data of a tablet is on a single server.
- Hierachical data indexing, root/metadata/data.
- Group column together by column family, data in the same column family stored in the same file for lookup locality. 
- Use distributed lock service Chubby for cluster management.
- Operation on a row is atomic.

# Engineering Tricks
- Exploit immutability, read-only SSTable doesn't require synchronization.
- Cache and prefetch tablet location from metadata server.
- Use bloom filter to filter out invalid lookup.


