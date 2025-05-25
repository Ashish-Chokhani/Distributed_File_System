# Distributed File System (DFS)

## Overview
This project implements a distributed file management system using MPI for parallelism and fault tolerance. The system provides functionalities such as uploading, retrieving, and searching files, along with heartbeat monitoring, failover, and recovery mechanisms. The system is implemented using MPI, where:
- **Rank 0** acts as the central metadata server.
- **Ranks 1 to N-1** serve as storage nodes.

---
## Features
### 1. File Partitioning & Three-Way Replication
- Files are split into fixed-size chunks (32 bytes each).
- Each chunk is stored on three different nodes for redundancy.
- No requirement to maintain replication in case of failures.

### 2. Load Balancing
- Chunks are evenly distributed across nodes.
- No node stores more than one replica of the same chunk.

### 3. Heartbeat Mechanism & Failure Detection
- Each node periodically sends a heartbeat signal to the metadata server.
- If a node fails to send a heartbeat for more than 3 seconds, it is marked as down.

### 4. Failover & Recovery Operations
- **Failover**: Simulates the failure of a storage node, stopping requests and heartbeats.
- **Recovery**: Restores a failed node and its ability to process requests and send heartbeats.

### 5. File Operations
- **Upload**: Files are partitioned and stored with replication.
- **Download**: Chunks are retrieved and reassembled.
- **Search**: Locates file chunks containing a specific word.

### 6. Error Handling
- If an error occurs during operations, output `-1`.
- Examples: Searching for a non-existent file, downloading a file with missing chunks, etc.

---
## Commands & Input Format
Commands are taken from **stdin**:
- `upload <file_name> <absolute_file_path>` – Uploads a file to ADFS.
- `retrieve <file_name>` – Retrieves a file and reassembles it.
- `search <file_name> <word>` – Searches for a word in a file.
- `list file <file_name>` – Lists nodes storing chunks of a file.
- `failover <rank>` – Simulates failure of a node.
- `recover <rank>` – Restores a previously failed node.
- `exit` – Terminates the program.

---
## Output Format
- Success outputs `1`.
- Errors output `-1`.
- Specific outputs for commands such as `list file`, `search`, and `retrieve`.

---
## Additional Notes
- The metadata server (Rank 0) is always operational.
- Maximum file size: ~1-2MB.
- Heartbeat messages sent every 1-2 seconds.
- Search functionality provides exact word matches only.

---

## How to Run

### 1. Compile the Code
```bash
mpic++ -std=c++17 ./DFS.cpp -o exec
```

### 2. Run the DFS System
```bash
mpirun -np <num_processors> ./exec
```

### 3. Testing with Provided Test Cases
Navigate to the relevant directory:
```bash
cd scripts
chmod +x ./DFS_test.sh
./DFS_test.sh
```
---

## Example Usage
### **Test Case 1** (File Upload & Retrieval)
#### **Input:**
```
upload a.txt testcases/a.txt
list file a.txt
failover 1
failover 2
retrieve a.txt
search a.txt abcd
exit
```
#### **Output:**
```
1
0 3 1 2 3
1 3 1 2 4
2 3 1 3 4
0 3 1 2 3
1 3 1 2 4
2 3 1 3 4
1
1
abcd efgh ijkl abcd abcd defg dsds abcddufhef ebhfeihf udefheifh abcde
3
0 15 20
```

### **Test Case 2** (Failover & Recovery)
#### **Input:**
```
upload a.txt testcases/a.txt
failover 1
list file a.txt
recover 1
list file a.txt
exit
```
#### **Output:**
```
1
0 3 1 2 3
1 3 1 2 4
2 3 1 3 4
1
0 2 2 3
1 2 2 4
2 2 3 4
1
0 3 1 2 3
1 3 1 2 4
2 3 1 3 4
```

---

### Parallelization Strategy

1. **Upload Functionality**:
   - The file is divided into chunks, and each chunk is distributed across worker nodes.
   - Chunks are replicated to ensure fault tolerance.
   - The master node coordinates the chunk distribution and metadata management.
   - Worker nodes handle the actual storage of chunks.

2. **Retrieve Functionality**:
   - The master node queries metadata to locate chunks for the requested file.
   - Worker nodes send the chunks back to the master node.
   - The master node reconstructs the file and provides it to the user.

3. **Search Functionality**:
   - The master node distributes the search request to all worker nodes holding chunks of the file.
   - Each worker node searches its local chunks for the requested word and reports results back to the master node.
   - Results are aggregated by the master and presented to the user.

4. **Heartbeat Monitoring**:
   - Worker nodes periodically send heartbeat signals to the master.
   - The master tracks the last heartbeat time for each worker to monitor their availability.

5. **Failover Mechanism**:
   - If a worker node fails (detected via missing heartbeats), the master initiates failover.
   - Replicated chunks are redistributed to available nodes to maintain redundancy.

6. **Recovery Mechanism**:
   - When a failed node recovers, the master updates the metadata and reintegrates the node into the system.

---

## Highlights of the Program

- **Distributed File Storage**:
  - Files are divided into chunks and distributed across multiple nodes.
  - Replication ensures high availability and fault tolerance.

- **Dynamic Load Balancing**:
  - The master node balances the workload across available nodes by tracking their loads.

- **Fault Tolerance**:
  - The system monitors node availability and performs failover to ensure service continuity.

- **Efficient Search**:
  - Parallel search allows fast retrieval of matching results across distributed data.

- **Scalability**:
  - The system supports adding nodes dynamically, improving scalability and performance.

---

## Complexity Analysis

### Original Algorithm
- **Time Complexity**: (O(n)\) for single-threaded file processing, where \(n\) is the file size. The search as well as retrieve operation requires O(n) complexity as well
- **Space Complexity**: \(O(n)\) additional space for storing an entire file

### Parallel Algorithm
- **Time Complexity**: Search as well retrieve requires \(O(n / p)\), where \(p\) is the number of processes (worker nodes).
- **Space Complexity**: O(N*replication_factor), accounting for chunks processed concurrently, ensuring fault tolerance
- **Individual Complexity**:
  - Upload: \(O(p * REPLICATION\_FACTOR)\)
  - Retrieve: \(O(N)\), where \(N\) is the file size
  - Search: \(O(N/p)\), as all nodes are queried in parallel.

---

## Assumptions
1.	File Naming:
   - All file names in the system are unique. Duplicate file names are not allowed.
2.	File Content:
	- File content does not contain any punctuation marks.
3.	Word Size:
	- Words in the files are guaranteed to have a length of at least 1 character.
4.	System Configuration:
	- The number of worker nodes (p) is known and remains constant during system execution.


