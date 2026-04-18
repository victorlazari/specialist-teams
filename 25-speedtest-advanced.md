# 25 - Speedtest (Ookla) Advanced Specialist File

## Introduction

Speedtest by Ookla stands as the industry-defining benchmark for internet performance measurement, with a legacy spanning nearly two decades and an unparalleled global footprint. Since its inception in 2006, Speedtest has facilitated over 67 billion tests across more than 190 countries, leveraging a vast network of over 16,000 strategically deployed servers worldwide. This extensive infrastructure underpins its ability to deliver precise, consumer-grade Quality of Service (QoS) assessments and enterprise-grade insights through Speedtest Intelligence.

This document provides an advanced, technical deep dive into the architectural, methodological, and practical aspects of Speedtest as it pertains to cutting-edge network technologies such as 5G and fiber-optic broadband, super-fast connections exceeding 10 Gbps, multi-server latency measurements, advanced threading logic, and CLI integration. It further explores the complexities of video experience testing, mobile network QoS sampling, and the requirements and architecture for hosting custom OoklaServer instances. The content herein is intended for network engineers, systems architects, and technical specialists seeking an exhaustive understanding of Speedtest’s capabilities and methodologies.

---

## 1. Measuring 5G, Fiber, and Super-Fast Connections: Addressing the Limitations of Traditional Testing

The evolution of network technologies towards 5G cellular and fiber-optic broadband, alongside the commercialization of multi-gigabit services, introduces significant challenges in accurately measuring full throughput capacity. Traditional speed testing methodologies, which often rely on downloading small static files from content delivery networks (CDNs), are inherently flawed when applied to these high-speed environments. Such approaches fail primarily due to the TCP slow start mechanism and insufficient test duration, which do not allow connections to ramp up to their maximum throughput.

### The TCP Slow Start Challenge

TCP slow start is a congestion control algorithm designed to prevent network congestion by initially limiting the amount of data sent over a connection and gradually increasing it based on acknowledgments from the receiver. While this mechanism is essential for network stability, it causes initial test samples to underrepresent the actual throughput potential during short-duration tests or when transferring small files. This is especially problematic for 5G and fiber connections where the theoretical bandwidth may reach multiple gigabits per second, but the test fails to capture sustained speeds due to the limited data window.

### Dynamic Connection Scaling for Multi-Gigabit Throughput

Ookla’s proprietary solution to this problem is dynamic connection scaling, implemented within both the Speedtest client and OoklaServer. The client initiates multiple concurrent TCP connections to the server, initially starting with a limited number of threads (typically two). During the test’s first half, the client continuously monitors throughput and, if the measured bandwidth exceeds certain thresholds (e.g., 4 Mbps), it dynamically spawns additional connections up to a maximum of four or more threads for HTTP fallback or multiple connections for TCP tests. This multithreaded approach aggregates bandwidth across connections, effectively circumventing TCP slow start limitations on individual streams and allowing testing to saturate connections up to and beyond 10 Gbps.

The server side is designed to handle these concurrent streams, coordinating seamless aggregation and ensuring accurate measurement of download and upload speeds without packet loss or jitter artifacts induced by overloaded server resources.

### Why Small Files Fail

Small file downloads typically do not provide sufficient data volume to allow TCP congestion windows to expand fully. As a result, the measured speeds reflect the initial ramp-up phase rather than steady-state throughput. In contrast, Speedtest’s methodology involves sustained data transfers with adaptive chunk sizes and continuously optimized buffer sizes, allowing for real-time throughput maximization across multiple threads and servers.

Furthermore, 5G network architectures involve carrier aggregation, which combines multiple frequency bands dynamically to boost throughput. This aggregation requires sustained data demand to activate, which small file tests cannot trigger, resulting in underreported speeds.

### Example: Multi-Threaded Download Test Behavior

```bash
# Pseudocode illustrating dynamic thread scaling during download test
init_threads = 2
max_threads = 4
current_threads = init_threads
test_duration_seconds = 10
elapsed_time = 0

while elapsed_time < test_duration_seconds:
    throughput = measure_current_throughput()
    if throughput > threshold and current_threads < max_threads:
        spawn_additional_thread()
        current_threads += 1
    adjust_chunk_size_based_on_throughput()
    elapsed_time += time_delta
end
```

---

## 2. Multi-Server Latency and Latency Under Load: Understanding Network Responsiveness

Latency measurement is a cornerstone of Speedtest’s QoS analysis, providing insight into network responsiveness beyond mere bandwidth metrics. Ookla measures latency in three primary scenarios: base latency to a single server, multi-server latency involving multiple proximal servers, and latency under load conditions during maximum throughput testing.

### Base Latency Measurement

Base latency is determined by calculating the round-trip time (RTT) for small packets between the client and a single selected server, typically over TCP port 8080 or via ICMP ping where available. To mitigate transient network fluctuations, Speedtest repeats latency probes multiple times and selects the lowest recorded RTT as the representative value, reflecting the best-case network responsiveness.

### Multi-Server Latency

To provide a more comprehensive assessment of network latency, Speedtest also measures RTTs to multiple nearby servers. This approach helps identify routing inefficiencies, peering issues, or regional network congestion by comparing latency profiles across different endpoints within the same geographic or network vicinity.

### Latency Under Load

Perhaps the most revealing latency measurement occurs under load, where latency is assessed concurrently with high-volume download and upload traffic. This reflects real-world conditions where network congestion and bufferbloat can significantly impact responsiveness. Speedtest measures ping during both upload and download phases, capturing the impact of network utilization on latency.

### Latency Measurement Algorithm

The latency measurement process involves sending timestamped small packets to the server and measuring the time until the corresponding acknowledgment is received. These measurements are repeated, often dozens of times, to capture jitter and transient latency spikes. The process distinguishes between idle latency (before throughput tests), upload latency (during upload test), and download latency (during download test).

---

## 3. Speedtest CLI Implementation and Integration: Linux-Native, Structured Output, and Observability

The Speedtest CLI is the official command-line interface provided by Ookla for native Linux environments, including Ubuntu, Debian, Fedora, CentOS, and FreeBSD, as well as ARM-based architectures like Raspberry Pi. This tool facilitates automated and programmatic speed testing suitable for server environments, continuous monitoring, and integration with observability platforms.

### Native Linux Implementation

Unlike browser-based Speedtest clients that rely on JavaScript and web protocols, the CLI is a compiled binary written for native execution on Linux systems. This reduces overhead, improves test accuracy by avoiding browser-induced limitations, and enables integration into scripting and system management workflows.

### Structured JSON Output

One of the CLI’s key advantages is its support for structured JSON output, facilitating easy parsing and ingestion by monitoring systems and data pipelines. This output includes detailed test results such as download/upload speeds in bits per second, latency values in milliseconds, packet loss percentages, server metadata, and the test timestamp.

Here is an example JSON output from a Speedtest CLI run:

```json
{
  "type": "result",
  "timestamp": "2025-10-15T13:45:30Z",
  "ping": {
    "jitter": 1.23,
    "latency": 12.5
  },
  "download": {
    "bandwidth": 9500000000,
    "bytes": 1125000000,
    "elapsed": 12000
  },
  "upload": {
    "bandwidth": 2000000000,
    "bytes": 250000000,
    "elapsed": 12000
  },
  "packetLoss": 0,
  "isp": "Example ISP",
  "interface": {
    "internalIp": "192.168.1.2",
    "name": "eth0",
    "macAddr": "00:11:22:33:44:55",
    "isVpn": false,
    "externalIp": "203.0.113.45"
  },
  "server": {
    "id": 12345,
    "name": "Example Server",
    "location": "Seattle, WA",
    "country": "US",
    "host": "speedtest.example.net",
    "port": 8080,
    "ip": "198.51.100.1"
  },
  "result": {
    "id": "abcdef12-3456-7890-abcd-ef1234567890",
    "url": "https://www.speedtest.net/result/c/abcdef12-3456-7890-abcd-ef1234567890"
  }
}
```

### Integration with Observability Platforms

The CLI’s JSON output can be piped into data collectors such as Prometheus exporters, ELK Stack pipelines, or cloud-native monitoring tools like Datadog and Splunk. This enables continuous network performance monitoring, alerting on degradations, and historical trend analysis.

### Automated Monitoring Bash Script Example

The following bash script demonstrates periodic Speedtest CLI execution with JSON output redirected to a log file, facilitating scheduled monitoring via cron jobs or systemd timers:

```bash
#!/bin/bash
LOG_FILE="/var/log/speedtest/speedtest_$(date +%Y%m%d).jsonl"
SPEEDTEST_BIN="/usr/local/bin/speedtest"

# Run Speedtest CLI and append JSON output to log file
$SPEEDTEST_BIN --format json >> $LOG_FILE 2>&1

# Optionally, add timestamp for each entry if CLI output lacks precise timing
echo "" >> $LOG_FILE
```

This script can be scheduled to run at fixed intervals, producing a JSON Lines (JSONL) file suitable for ingestion by log processors.

---

## 4. Video Experience Testing: Capturing Adaptive Bitrate Playback Metrics

Beyond raw bandwidth and latency, Speedtest extends its measurement capabilities to assess Quality of Experience (QoE) for video streaming, a critical application for modern broadband users. Video experience testing leverages real video player instances embedded within the Speedtest Android and iOS applications.

### Adaptive Bitrate Streaming (ABR) Dynamics

Video streaming services commonly employ adaptive bitrate algorithms that adjust video quality in real time based on current network throughput and latency conditions. Speedtest’s video experience module simulates this behavior by streaming video content encoded at multiple bitrates and measuring switching behavior, buffering events, startup delay, and rendered quality.

The embedded player collects granular telemetry during playback, including the number and duration of quality switches, rebuffering frequency, and playback smoothness. These metrics provide a robust proxy for the user’s actual viewing experience, bridging the gap between synthetic network tests and real-world application performance.

### Mobile and Fixed Network Scenarios

Video testing accounts for the unique characteristics of mobile networks, including variable coverage, handoffs, and signal fluctuations. By conducting tests on both Wi-Fi and cellular connections, Speedtest captures the differential impact of network type on video QoE. This data aids operators and content providers in optimizing CDN placement, encoding strategies, and network management policies.

---

## 5. Mobile Network Samples: Radio QoS, Signal, Coverage, and CDN Performance

Speedtest’s mobile network sampling goes beyond traditional speed tests to incorporate metrics specific to cellular radio QoS and coverage. This includes measuring signal strength, availability, and the performance of content delivery networks (CDNs) under typical mobile conditions.

### Radio QoS Sampling

By integrating with mobile device APIs and partnering with manufacturers, Speedtest can detect 5G NR states, LTE bands, carrier aggregation status, and signal metrics such as Reference Signal Received Power (RSRP) and Signal-to-Noise Ratio (SNR). These data points contextualize speed results, explaining variability and enabling network operators to diagnose coverage gaps or capacity constraints.

### Coverage and Availability Metrics

Speedtest aggregates millions of consumer-initiated tests and passive measurements to build detailed, crowdsourced maps of cellular coverage and quality. These maps inform both consumers and operators, highlighting areas with strong or poor network presence and supporting targeted infrastructure investments.

### CDN Performance Testing

By measuring latency and throughput to CDN endpoints, Speedtest evaluates the efficiency of content delivery. This includes examining cache hit rates, server responsiveness, and peering quality. Such insights assist content providers in optimizing CDN configurations to improve streaming and download performance on mobile networks.

---

## 6. Building a Custom OoklaServer Host: Hardware, Connectivity, and Transit Requirements

Organizations seeking to deploy their own Speedtest server nodes can leverage OoklaServer, the proprietary server daemon powering the global Speedtest network. Hosting an OoklaServer instance requires careful consideration of hardware specifications, network connectivity, and transit quality to ensure accurate and reliable measurements.

### Hardware Specifications

The minimum hardware requirements depend on the target market segment and expected test volumes. For smaller markets or low-traffic environments, servers with 1 Gbps upstream and downstream capacity suffice. However, in developed markets or metropolitan areas, the recommended baseline is at least 10 Gbps full-duplex connectivity to accommodate high concurrency and super-fast connections.

Server-grade hardware should feature multi-core CPUs capable of handling multiple concurrent TCP connections and data streams, high-performance NVMe SSDs to support logging and caching without bottlenecks, and sufficient RAM (16 GB or more) to optimize network stack performance. Network interface cards (NICs) supporting offloading features and low-latency operation are preferred.

### Transit Quality

The network transit path from the OoklaServer host to the internet backbone must exhibit low latency, minimal packet loss, and stable throughput. Hosting providers should guarantee SLA-backed connectivity with redundant paths to prevent service degradation. Peering arrangements with major ISPs and CDNs enhance test accuracy by reducing the number of intermediate hops.

### Software Environment

OoklaServer runs on modern Linux distributions optimized for network performance. The server software manages dynamic connection scaling, multi-threaded TCP sessions, and stable test termination logic. Continuous monitoring scripts and automated health checks ensure uptime and performance stability.

---

## 7. Advanced Threading Logic: Optimizing Throughput and Overcoming Browser Limitations

The Speedtest client employs sophisticated threading logic to balance test accuracy, resource utilization, and compatibility across diverse environments.

### Thread Count Determination

Tests begin with two threads for both download and upload phases. This conservative approach minimizes HTTP overhead and resource consumption on slower connections. During the initial phase, the client measures real-time throughput, and if it exceeds a 4 Mbps threshold, it dynamically increases the thread count to four or more, depending on the connection type and protocol.

This dynamic scaling is critical for accurately measuring high-bandwidth connections, as multiple threads aggregate bandwidth to circumvent TCP congestion control limitations inherent to single-stream tests.

### Browser Limitations and Secondary URLs

Legacy browsers such as Internet Explorer 7 and Firefox 2 impose severe constraints on concurrent connections and HTTP pipelining, limiting the number of simultaneous requests and thus throttling speed test accuracy. To address this, Speedtest utilizes secondary URLs and additional connection pooling to distribute requests and maintain throughput measurement fidelity.

This approach involves directing test traffic to multiple endpoints and coordinating data streams across them, effectively mitigating browser-imposed threading limitations while preserving the integrity of the test.

---

## Architectural Analysis of 5G Testing Challenges

Evaluating 5G network performance involves multiple complexities arising from the unique characteristics of 5G NR technology, network slicing, and carrier aggregation.

Firstly, 5G networks employ dynamic spectrum allocation and multi-layered carrier aggregation, where multiple frequency bands are combined to maximize throughput. Triggering carrier aggregation requires sustained data demand, which short or small file tests fail to provide, resulting in artificially low speed measurements.

Secondly, 5G latency is influenced by the deployment of edge computing and network slicing. Different slices might prioritize ultra-reliable low latency communications (URLLC) or enhanced mobile broadband (eMBB), affecting latency and throughput characteristics. Speedtest’s multi-server latency measurements help reveal these differences by testing multiple endpoints representing different slices or network segments.

Thirdly, 5G coverage areas are heterogeneous, with mmWave bands providing ultra-high speeds but limited range and sub-6 GHz bands offering broader coverage with lower throughput. Speedtest integrates radio QoS measurements, including signal strength and band awareness, to contextualize throughput results with the underlying radio environment.

Finally, the interplay between mobile device hardware, firmware, and the network significantly impacts test accuracy. Through partnerships with device manufacturers, Speedtest incorporates in-app 5G detection and specialized test workflows tailored to specific chipsets and radio implementations.

---

## Conclusion

Ookla's Speedtest represents a comprehensive, scientifically rigorous platform for measuring and analyzing internet network performance across a vast array of technologies and use cases. Its proprietary methodologies, including dynamic connection scaling, multi-server latency testing, and advanced threading logic, address the unique challenges posed by modern high-speed networks such as 5G and fiber-optic broadband.

The Speedtest CLI enables native, automated, and programmable testing with structured outputs suitable for integration with observability platforms, empowering network operators and enterprises to monitor and optimize network QoS continuously. Additionally, Speedtest’s video experience and mobile network sampling extend the scope of performance measurement into application-level QoE and radio-layer QoS domains.

For organizations interested in deploying custom Speedtest servers, OoklaServer offers a scalable, high-performance platform that demands robust hardware and network transit capabilities to ensure measurement accuracy and reliability.

In sum, the Speedtest ecosystem exemplifies a mature, end-to-end solution for internet performance testing that balances depth of analysis, global scale, and practical usability for both consumers and enterprises.

---

## Appendix A: Sample Speedtest CLI JSON Output

```json
{
  "type": "result",
  "timestamp": "2025-10-15T13:45:30Z",
  "ping": {
    "jitter": 1.23,
    "latency": 12.5
  },
  "download": {
    "bandwidth": 9500000000,
    "bytes": 1125000000,
    "elapsed": 12000
  },
  "upload": {
    "bandwidth": 2000000000,
    "bytes": 250000000,
    "elapsed": 12000
  },
  "packetLoss": 0,
  "isp": "Example ISP",
  "interface": {
    "internalIp": "192.168.1.2",
    "name": "eth0",
    "macAddr": "00:11:22:33:44:55",
    "isVpn": false,
    "externalIp": "203.0.113.45"
  },
  "server": {
    "id": 12345,
    "name": "Example Server",
    "location": "Seattle, WA",
    "country": "US",
    "host": "speedtest.example.net",
    "port": 8080,
    "ip": "198.51.100.1"
  },
  "result": {
    "id": "abcdef12-3456-7890-abcd-ef1234567890",
    "url": "https://www.speedtest.net/result/c/abcdef12-3456-7890-abcd-ef1234567890"
  }
}
```

---

## Appendix B: Automated Speedtest CLI Monitoring Script

```bash
#!/bin/bash
# File: speedtest_monitor.sh
# Purpose: Run Speedtest CLI periodically and log JSON output for monitoring

LOG_DIR="/var/log/speedtest"
LOG_FILE="${LOG_DIR}/speedtest_$(date +%Y%m%d).jsonl"
SPEEDTEST_BIN="/usr/local/bin/speedtest"

# Ensure log directory exists
mkdir -p $LOG_DIR

# Execute Speedtest CLI with JSON output
$SPEEDTEST_BIN --format json >> $LOG_FILE 2>&1

# Append a newline for JSONL formatting
echo "" >> $LOG_FILE
```

This script can be scheduled via cron as follows:

```cron
0 * * * * /usr/local/bin/speedtest_monitor.sh
```

which will collect data hourly, enabling historical network performance analysis.

---

## References

- Ookla Speedtest About: https://speedtest.net/about
- Ookla Speedtest Methodology: https://ookla.com/resources/guides/speedtest-methodology
- Speedtest Server Network Architecture: https://ookla.com/articles/speedtest-server-network
- Speedtest CLI: https://speedtest.net/apps/cli
- Assessing Accuracy: https://speedtest.net/help/guides/assessing-accuracy

---

*Document compiled with data current as of October 2025.*