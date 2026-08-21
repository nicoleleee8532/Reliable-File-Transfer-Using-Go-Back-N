A clean, comprehensive `README.md` formatted for your repository:

---

# Reliable File Transfer using Go-Back-N Protocol

**Course**: Computer Networks (100071011) | **Academic Year**: 2023–2024-2

**Institution**: Beijing Institute of Technology — School of Computer

**Author**: 李恩心 (Li Enxin) | **Student ID**: 1820232039 | **Class**: 86132301

**Instructor**: 阮思捷 (Ruan Sijie)

**Date**: May 16, 2026

---

## 1. Project Overview

This project implements a reliable, bidirectional file transfer system over UDP using the **Go-Back-N (GBN)** sliding window protocol. The system reliably transfers large files between hosts despite packet loss, corruption, or out-of-order delivery.

The implementation follows the GBN specification described in *Computer Networks* (A.S. Tanenbaum, 5th Edition, Chapter 3, Protocol 5).

---

## 2. Protocol Design

### 2.1 Go-Back-N Mechanism

* **Sliding Window**: Maintained at sender side with size `SWSize`.


* **State Variables**:
* `base`: Sequence number of the oldest unacknowledged packet.


* `nextseqnum`: Sequence number assigned to the next packet to send.




* **Acknowledgments**: Cumulative ACKs.


* **Timeout & Retransmission**: On timeout, retransmits all outstanding packets in the range `[base, nextseqnum - 1]`.



### 2.2 PDU (Frame) Structure

Each UDP datagram encapsulates a custom PDU structure:

```text
+----------+----------+---------+------------+----------+
|  SeqNum  |  AckNum  |  Flags  |    Data    | Checksum |
|   (4B)   |   (4B)   |  (1B)   |  (<= 4KB)  |   (2B)   |
+----------+----------+---------+------------+----------+
```[cite: 2]

| Field | Size | Description |
| :--- | :--- | :--- |
| `SeqNum` | 4 bytes | Sequence number of the packet[cite: 2] |
| `AckNum` | 4 bytes | Cumulative acknowledgment number[cite: 2] |
| `Flags` | 1 byte | `0` = DATA packet, `1` = FIN packet[cite: 2] |
| `Data` | $\le$ 4 KB | Payload / file chunk data[cite: 2] |
| `Checksum` | 2 bytes | CRC-CCITT (`0x1021` polynomial) for error detection[cite: 2] |

---

## 3. Project Structure

```text
.
├── pdu.py                     # PDU encoding/decoding logic
├── checksum.py                # CRC-CCITT (0x1021) checksum implementation
├── sender.py                  # Core sender logic & GBN sliding window
├── receiver.py                # Core receiver logic & ACK handling
├── channel.py                 # Network channel abstraction
├── channel_wrapper.py         # Packet loss & corruption simulation wrapper
├── bidirectional_sender.py   # Supports simultaneous send/receive
├── multi_host_sender.py       # Multi-host transfer handling
├── utils.py                   # Common helper utilities
├── loganalyzier.py            # Log processing script
├── analysis.py                # Plotting & performance metrics generator
├── config.ini                 # Configuration file
├── create_test_file.py        # Generates test binaries (e.g., input.dat)
├── test_sender_receiver.py    # Test suit launcher
├── input.dat                  # Source file for transmission
├── output.dat / received.dat  # Target output files
├── sender_log.txt             # Transmission event log
└── receiver_log.txt           # Reception event log
```[cite: 2]

---

## 4. Configuration (`config.ini`)

Parameters are loaded dynamically from `config.ini`[cite: 2]:

```ini
[COMMUNICATION]
UDPPort = 42039
DataSize = 1024
SWSize = 8
Timeout = 1000
ErrorRate = 0
LostRate = 0
InitSeqNo = 0
```[cite: 2]

| Parameter | Value | Description |
| :--- | :--- | :--- |
| `UDPPort` | `42039` | Destination UDP communication port[cite: 2] |
| `DataSize` | `1024` | Data payload size per PDU in bytes[cite: 2] |
| `SWSize` | `8` | Sender sliding window size[cite: 2] |
| `Timeout` | `1000` | Retransmission timer in milliseconds[cite: 2] |
| `ErrorRate` | `0` | Corruption rate ($1/N$ packets corrupted)[cite: 2] |
| `LostRate` | `0` | Packet loss rate ($1/N$ packets dropped)[cite: 2] |
| `InitSeqNo` | `0` | Starting packet sequence number[cite: 2] |

---

## 5. Usage & Execution

### Prerequisites
* Python 3.8+[cite: 2]
* Standard libraries (`socket`, `struct`, `configparser`, `time`, `hashlib`)[cite: 2]

### Running File Transfer
1. **Generate Test Data**:
   ```bash
   python create_test_file.py
   ```[cite: 2]

2. **Start Receiver**:
   ```bash
   python receiver.py
   ```[cite: 2]

3. **Start Sender** (in a second terminal):
   ```bash
   python sender.py
   ```[cite: 2]

### Running Tests & Verification
* **Check File Integrity**:
  ```bash
  md5 input.dat received.dat
  ```[cite: 2]
* **Analyze Logs**:
  ```bash
  python loganalyzier.py
  python analysis.py
  ```[cite: 2]

---

## 6. Key Implementation Highlights

* **CRC-CCITT Error Detection**: Uses standard polynomial `0x1021` with initial seed `0xFFFF` to check payload validity[cite: 2]. Corrupted packets are discarded silently to allow sender timeout recovery[cite: 2].
* **Simulated Channel Wrapper**: Intercepts `sendto()` calls to introduce probabilistic bit corruption (`ErrorRate`) and packet drops (`LostRate`)[cite: 2].
* **Cumulative ACK Flow**: Receiver returns `AckNum = expected_seq`[cite: 2]. Un-corrupted out-of-order packets trigger duplicate ACKs for the last correctly received sequence[cite: 2].

---

## 7. Performance Overview

Testing performed on local loopback (`127.0.0.1`) with a 3 MB binary test file (`3,145,728` bytes / `3,072` packets)[cite: 2]:

| Test Scenario | Window Size | Error Rate | Loss Rate | Transmitted / Retransmitted | Time (s) | Throughput | MD5 Match |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Ideal** | 8 | 0% | 0% | 3,072 / 0 | 0.256s | ~11.7 MB/s | **PASS** |
| **Window 16** | 16 | 0% | 0% | 3,072 / 0 | 0.241s | ~12.4 MB/s | **PASS** |
| **10% Error** | 8 | 10% | 0% | ~3,400 / ~328 | 0.890s | ~3.7 MB/s | **PASS** |
| **10% Loss** | 8 | 0% | 10% | ~3,400 / ~340 | 0.920s | ~3.6 MB/s | **PASS** |

---

## 8. References
* Tanenbaum, A. S., & Wetherall, D. J. (2011). *Computer Networks* (5th ed.). Pearson Education. (Chapter 3, Protocol 5 - Go-Back-N)[cite: 2]
* Postel, J. (1980). *User Datagram Protocol*. RFC 768.[cite: 2]
* ITU-T Recommendation V.41: *Code-independent error control system* (CRC-CCITT standard).[cite: 2]

```
