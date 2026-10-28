# Reliable File Transfer Using Go-Back-N Protocol

A Python implementation of a reliable file transfer system operating over UDP sockets using the **Go-Back-N (GBN)** sliding window protocol. Developed for the Computer Networks course (2023–2024 Spring) at Beijing Institute of Technology.

---

## 📌 Metadata

* **Student ID:** 1820232039


* **Name:** 李恩心 (Li Enxin)


* **Class No.:** 86132301


* **Instructor:** 阮思捷 (Prof. Ruan Sijie)


* **Institution:** School of Computer Science, Beijing Institute of Technology


* **Date:** May 16, 2026



---

## 1. Project Overview

This project implements a bidirectional, reliable file transfer system over unreliable UDP transport. Based on Chapter 3 (Protocol 5) of *Computer Networks* (A.S. Tanenbaum, 5th Edition), the system guarantees data integrity and order across network conditions featuring packet loss and bit corruption.

### Key Features

* **Custom PDU Encapsulation:** Includes explicit sequence numbers, acknowledgments, flags, data, and checksum fields.
* **Sliding Window Control:** Go-Back-N mechanism supporting configurable window sizes.
* **Error Detection:** 16-bit CRC-CCITT polynomial (`0x1021`) checksum verification.
* **Network Simulation:** Built-in loss and error wrappers to test protocol resilience.
* **Logging & Analytics:** Full event logging and performance analysis scripts.

---

## 2. Protocol & Architecture

### Protocol Data Unit (PDU) Structure

Each UDP datagram carries a structured frame formatted as follows:

```text
+---------------+---------------+-----------+---------------+---------------+
| SeqNum (4B)   | AckNum (4B)   | Flags(1B) | Data (<=4KB)  | Checksum (2B) |
+---------------+---------------+-----------+---------------+---------------+

```

| Field | Size | Description |
| --- | --- | --- |
| `SeqNum` | 4 Bytes | Packet sequence number |
| `AckNum` | 4 Bytes | Cumulative acknowledgment number |
| `Flags` | 1 Byte | `0` = DATA packet, `1` = FIN packet |
| `Data` | $\le$ 4 KB | File chunk payload |
| `Checksum` | 2 Bytes | CRC-CCITT (`0x1021`) error detection |

### System Workflow

```text
HOST 1 (Sender)                                HOST 2 (Receiver)
  +-----------+                                  +------------+
  | Read File |                                  | UDP Socket |
  +-----+-----+                                  +-----+------+
        |                                              |
  +-----+-----+                                  +-----+------+
  | Split Data|                                  | CRC Check  |
  +-----+-----+                                  +-----+------+
        |                                              |
  +-----+-----+     UDP Datagram (PDU)          +------+-----+
  | GBN Window +------------------------------->+ In-Order?   |
  +-----+-----+                                 +---+----+---+
        ^                                           |    |
        |                                       Yes |    | No
        |                                           v    v
        |           Cumulative ACK              +---+----+---+
        +---------------------------------------+ Send ACK   |
                                                +------------+

```

---

## 3. Repository Structure

```text
.
├── README.md
├── docs/
│   ├── file-transfer-system-report.docx
│   └── file-transfer-system-presentation.pptx
├── archives/
│   └── source-project.zip
└── Week_01/Lab/
    └── Week_01_intro_to_python.ipynb
```

The complete original source project is preserved in `archives/source-project.zip`; the report and presentation are in `docs/`.

---

## 4. Configuration

Network and protocol parameters are managed via `config.ini`:

```ini
[COMMUNICATION]
UDPPort = 42039
DataSize = 1024
SWSize = 8
Timeout = 1000
ErrorRate = 0
LostRate = 0
InitSeqNo = 0

```

* **`UDPPort`**: Port used for UDP communication.
* **`DataSize`**: Data payload length in bytes (1 KB).
* **`SWSize`**: Maximum sliding window size.
* **`Timeout`**: Retransmission timer in milliseconds.
* **`ErrorRate`**: Corrupts 1 out of $N$ packets ($0$ disables corruption).
* **`LostRate`**: Drops 1 out of $N$ packets ($0$ disables loss).

---

## 5. Getting Started

### Prerequisites

* **Python**: `3.8+`
* **OS**: Linux, macOS, or Windows

### Running a Transfer

1. **Generate Test File:**
```bash
python create_test_file.py

```


2. **Start Receiver:**
```bash
python receiver.py

```


3. **Start Sender:**
```bash
python sender.py

```



### Verify File Integrity

Check that input and output files match:

```bash
md5 input.dat received.dat

```

---

## 6. Experimental Results

Tested on localhost using a **3 MB** test file (3,072 packets):

| Configuration | Timeouts | Retransmissions | Time (s) | Throughput (pkts/s) | Efficiency |
| --- | --- | --- | --- | --- | --- |
| **Window=4, 0% Loss** | 0 | 0 | 0.282 | 10,894 | 100% |
| **Window=8, 0% Loss** | 0 | 0 | 0.256 | 12,000 | 100% |
| **Window=16, 0% Loss** | 0 | 0 | 0.241 | 12,746 | 100% |
| **Window=8, 10% Error** | ~30 | ~328 | 0.890 | 3,820 | ~90% |
| **Window=8, 10% Loss** | ~35 | ~340 | 0.920 | 3,700 | ~89% |

---

## 7. References

1. Tanenbaum, A. S., & Wetherall, D. J. (2011). *Computer Networks* (5th ed.). Pearson. (Chapter 3, Protocol 5)
2. Kurose, J. F., & Ross, K. W. (2017). *Computer Networking: A Top-Down Approach* (7th ed.). Pearson.
3. Postel, J. (1980). *User Datagram Protocol*. RFC 768.
4. ITU-T Recommendation V.41 (2002). *Code-independent error control system*.
