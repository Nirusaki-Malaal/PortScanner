# PortScanner

A fast, multithreaded TCP port scanner written in C. It quickly scans the top 1,000 most common ports using OpenMP for multi-core parallelism and non-blocking POSIX sockets for fast, asynchronous timeout handling. It also attempts basic service banner grabbing to identify services running on open ports.

## Features

* **Top 1,000 Ports**: Scans a curated list of the most common TCP services.
* **Parallel Execution**: Uses OpenMP (`#pragma omp parallel for`) to distribute port checks across all available CPU cores.
* **Non-blocking I/O**: Sockets use `O_NONBLOCK` and `select()` with a 1-second connect timeout to avoid hanging on unresponsive or filtered ports.
* **Banner Grabbing**: Inspects incoming bytes on open ports with a 2-second timeout to identify common services (SSH, FTP, SMTP, HTTP, MySQL).
* **IPv4 and IPv6 Support**: Resolves addresses using `getaddrinfo` with `AF_UNSPEC`.
* **Thread-safe Output**: Uses OpenMP critical sections to prevent garbled console logs.

## Requirements

* A C compiler (GCC recommended, or Clang)
* OpenMP support (`libgomp` on Linux, `libomp` on macOS)
* Linux or a POSIX-compliant environment

## Build Instructions

Compile the source code using GCC with OpenMP enabled and basic optimizations:

```bash
gcc -O2 -fopenmp port_scanner.c -o port_scanner
```

On systems using Clang (such as macOS with Homebrew OpenMP):

```bash
clang -O2 -Xpreprocessor -fopenmp -lomp port_scanner.c -o port_scanner
```

## Usage

Run the compiled binary by passing the target domain or IP address as the only argument:

```bash
./port_scanner <target-host>
```

### Examples

Scan a remote domain:

```bash
./port_scanner scanme.nmap.org
```

Scan a local IP address:

```bash
./port_scanner 192.168.1.1
```

### Sample Output

```text
Port Open : 22 | Service: SSH
Port Open : 80 | Service: HTTP
Port Open : 443 | Service: Unknown
Port Filtered : 23
Port Filtered : 135
Port Filtered : 445
```

## How It Works

1. **Target Resolution**: The host string is passed to `getaddrinfo()` to look up socket address structures for IPv4 or IPv6.
2. **Socket Setup**: Sockets are set to non-blocking mode using `fcntl(sock_fd, F_SETFL, O_NONBLOCK)`.
3. **Connection Check**: The scanner initiates `connect()`, then uses `select()` to wait up to 1 second on the write descriptor. It inspects `getsockopt()` for `SO_ERROR` to determine if the port is open (error 0) or filtered (timed out).
4. **Service Identification**: If a port is open, a secondary `select()` waits up to 2 seconds for initial banner bytes. The received banner is matched against simple signatures.
5. **Console Logging**: Open and filtered ports are reported to standard output inside an OpenMP critical block to keep console lines intact.

## Disclaimer

This tool is created for educational purposes, authorized security assessments, and network troubleshooting. Only run port scans against systems you own or have explicit permission to test.
