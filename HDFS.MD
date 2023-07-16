# HDFS (Hadoop Distributed File System):

- Handles big files:
    - Breaks files into blocks (default block size is 128 MB).
    - Stores multiple copies of each block across several computers for fault tolerance.

- Architecture:
    - Single Name Node:
        - Keeps track of block locations and metadata.
    - Multiple Data Nodes:
        - Store the actual blocks of data.
        - Communicate with each other to maintain block replication and consistency.

- Reading a file:
    1. Client contacts the Name Node to retrieve the file's block locations.
    2. Client retrieves the file directly from the respective Data Nodes.

- Writing a file:
    1. Client communicates with the Name Node:
        - Name Node creates an entry for the new file.
    2. Client contacts a single Data Node.
    3. Data Nodes coordinate with each other to write the file in parallel.
    4. Once the data is stored, Data Nodes report block locations to the client.
    5. Client updates the Name Node with the block locations.

- Name Node resilience:
    - Backup metadata:
        - Metadata is written to local disk and a separate NFS (Network File System) for recovery purposes.
    - Secondary Name Node:
        - Maintains a merged copy of the edit log from the primary Name Node.
    - HDFS Federation:
        - Multiple Name Nodes manage separate namespace volumes.
        - If one Name Node fails, only a portion of the data is affected.
    - HDFS High Availability:
        - Hot standby Name Node using shared edit log stored in a different file system (not HDFS).
        - ZooKeeper is used to track the active Name Node.
        - Clients first consult ZooKeeper to determine the active Name Node.
        - Requires complex configuration and ensures only one Name Node runs at a time.

- How to use HDFS:
    - UI (e.g., Ambari):
        - Utilize the user interface provided by tools like Ambari to interact with HDFS.
        - The UI offers a graphical representation of the file system, allowing users to browse, upload, and download
          files.
    - Command-Line Interface:
        - Access HDFS using the command-line interface (CLI) provided by Hadoop.
        - Use commands such as `hdfs dfs` to perform various operations, such as listing files, copying files, or
          creating directories.
    - HTTP / HDFS Proxies:
        - Access HDFS using the HTTP protocol through HDFS proxies.
        - The HDFS proxies provide REST APIs that can be utilized for file operations, such as reading, writing, and
          listing files.
    - Java Interface:
        - Utilize the Java API provided by Hadoop to interact with HDFS programmatically.
        - Develop custom Java applications to read, write, and manipulate files stored in HDFS.
    - NFS Gateway:
        - Mount an HDFS cluster as another server using the NFS gateway.
        - This allows HDFS to be accessed as a regular file system through the standard file system interfaces.
        - Users can interact with HDFS by simply accessing the mounted directory, just like any other local file system.

## Basic HDFS Terminal Commands

Use the following HDFS terminal commands to interact with the Hadoop Distributed File System:

- `hadoop fs --<command>`: Execute HDFS command
- `hadoop fs -mkdir ml-100k`: Create a directory in HDFS
- `hadoop fs -ls`: List directory contents in HDFS
- `hadoop fs -copyFromLocal <local_file> <hdfs_path>`: Copy a file from the local filesystem to HDFS
- `hadoop fs -rm <hdfs_path>`: Remove a file from HDFS
- `hadoop fs -rmdir <hdfs_path>`: Remove a directory from HDFS
- `hadoop fs`: Show help and available commands
