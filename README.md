<p align="center">
  <img src="assets/logo.svg" alt="PortScanner Logo" width="540">
</p>

# PortScanner

Multithreaded TCP port scanner written in C leveraging OpenMP parallelism and non-blocking POSIX sockets for fast asynchronous timeout resolution.

## Architecture

* **Parallel Dispatch**: Distributes target port sweeps across all available hardware cores via `#pragma omp parallel for`.
* **Asynchronous Sockets**: Employs `fcntl(O_NONBLOCK)` and `select()` with a 1-second connect timeout to prevent socket hangs on firewalled or dropped endpoints.
* **Banner Extraction**: Inspects initial payload bytes with a 2-second timeout window to fingerprint common protocol banners (SSH, HTTP, FTP, SMTP, MySQL).
* **Dual-Stack Resolution**: Resolves target hostnames using `getaddrinfo` with `AF_UNSPEC` for IPv4 and IPv6 compatibility.
* **Thread-Safe Console**: Serializes output streams inside OpenMP critical sections to prevent interleaved terminal logs.

## Compilation

### Linux (GCC)

```bash
gcc -O2 -fopenmp port_scanner.c -o port_scanner
```

### macOS (Clang with libomp)

```bash
clang -O2 -Xpreprocessor -fopenmp -lomp port_scanner.c -o port_scanner
```

## Usage

```bash
./port_scanner <target-host>
```

### Example

```bash
./port_scanner scanme.nmap.org
```

Sample output:

```text
Port Open : 22 | Service: SSH
Port Open : 80 | Service: HTTP
Port Open : 443 | Service: Unknown
Port Filtered : 23
Port Filtered : 135
Port Filtered : 445
```

## License

MIT License. See LICENSE for details.
