# Speedtest CLI Command Reference

## 1. Introduction

The `speedtest` Command Line Interface (CLI) is a powerful, highly configurable tool designed for network performance measurement and diagnostics. Developed to provide developers, system administrators, and network engineers with a robust mechanism for testing internet bandwidth, latency, and packet loss, the `speedtest` CLI offers an extensive array of commands, flags, and configuration options. This comprehensive reference guide details every aspect of the `speedtest` CLI, providing deep insights into its architecture, command syntax, advanced usage patterns, and integration capabilities.

Whether you are automating network health checks, integrating bandwidth testing into monitoring dashboards, or troubleshooting complex network anomalies, the `speedtest` CLI delivers the precision and flexibility required for enterprise-grade network analysis. This document serves as the definitive resource for mastering the `speedtest` CLI, covering everything from basic execution to advanced server selection, output formatting, and network interface binding.

## 2. Installation and Setup

Before diving into the command reference, it is essential to ensure that the `speedtest` CLI is correctly installed and configured in your environment. The CLI is available across multiple platforms, including Linux, macOS, and Windows.

### 2.1. Linux Installation

For Debian/Ubuntu-based systems, the `speedtest` CLI can be installed via the official APT repository:

```bash
sudo apt-get update
sudo apt-get install curl
curl -s https://packagecloud.io/install/repositories/ookla/speedtest-cli/script.deb.sh | sudo bash
sudo apt-get install speedtest
```

For RHEL/CentOS-based systems, use the YUM repository:

```bash
curl -s https://packagecloud.io/install/repositories/ookla/speedtest-cli/script.rpm.sh | sudo bash
sudo yum install speedtest
```

### 2.2. macOS Installation

On macOS, the recommended installation method is via Homebrew:

```bash
brew tap teamookla/speedtest
brew update
brew install speedtest --force
```

### 2.3. Windows Installation

For Windows users, the `speedtest` CLI can be downloaded as a standalone executable from the official website or installed via a package manager like Scoop or Chocolatey.

## 3. Core Concepts and Architecture

Understanding the underlying architecture of the `speedtest` CLI is crucial for leveraging its full potential. The CLI operates by establishing multiple concurrent TCP connections to a selected test server, measuring the time taken to transfer a specific payload of data.

### 3.1. Test Phases

A standard `speedtest` execution consists of several distinct phases:

1.  **Configuration Retrieval:** The CLI fetches the global configuration, including client IP, ISP details, and a list of available test servers.
2.  **Server Selection:** Based on latency and geographic proximity, the CLI selects the optimal server for the test. This can be overridden using specific flags.
3.  **Latency and Jitter Measurement:** The CLI performs a series of ICMP or TCP pings to measure the round-trip time (latency) and the variance in latency (jitter).
4.  **Download Test:** The CLI establishes multiple connections to the server and downloads a payload, calculating the maximum sustained throughput.
5.  **Upload Test:** Similar to the download test, the CLI uploads a payload to the server to measure upstream bandwidth.
6.  **Result Reporting:** The final metrics are compiled and presented in the requested output format (e.g., human-readable text, JSON, CSV).

## 4. Global Flags and Options

The `speedtest` CLI supports a wide range of global flags that modify its behavior, output formatting, and network interactions. These flags can be combined to create highly customized test scenarios.

### 4.1. General Options

*   `-h, --help`: Displays the comprehensive help message, listing all available commands and flags.
*   `-V, --version`: Prints the current version of the `speedtest` CLI and exits.
*   `-L, --servers`: Retrieves and displays a list of available test servers near the client's location. This is useful for identifying specific server IDs for targeted testing.
*   `--selection-details`: Provides detailed information about the server selection process, including the latency measurements used to determine the optimal server.

### 4.2. Output Formatting

The `speedtest` CLI excels in its ability to format output for various use cases, from interactive terminal sessions to automated data ingestion pipelines.

*   `-f, --format <FORMAT>`: Specifies the output format. Supported formats include:
    *   `human-readable` (default): A visually appealing, text-based output suitable for interactive use.
    *   `csv`: Comma-separated values, ideal for importing into spreadsheets or databases.
    *   `tsv`: Tab-separated values.
    *   `json`: A structured JSON object containing all test metrics, perfect for programmatic parsing and integration with monitoring tools (e.g., Prometheus, Grafana).
    *   `jsonl`: JSON Lines format, where each test result is a single JSON object on a new line, useful for streaming data.
    *   `json-pretty`: A beautifully formatted, indented JSON output for human inspection.
*   `--output-header`: When used with `csv` or `tsv` formats, this flag includes a header row detailing the column names.
*   `-p, --progress <yes|no>`: Enables or disables the interactive progress bar during the test. Disabling the progress bar is recommended for automated scripts to prevent terminal clutter.

### 4.3. Server Selection and Filtering

By default, the `speedtest` CLI automatically selects the best server based on latency. However, you can explicitly control the server selection process.

*   `-s, --server-id <ID>`: Forces the CLI to use a specific test server identified by its unique ID. You can obtain server IDs using the `-L` flag. Multiple server IDs can be provided as a comma-separated list to test against multiple servers sequentially.
*   `-o, --host <HOSTNAME>`: Specifies a custom test server by its hostname or IP address. This is particularly useful for testing against internal or private speedtest servers.

### 4.4. Network Configuration

For advanced network diagnostics, the `speedtest` CLI provides granular control over network interfaces and protocols.

*   `-i, --interface <INTERFACE>`: Binds the test to a specific local network interface (e.g., `eth0`, `wlan0`, `en0`). This is essential for testing multi-homed systems or specific network paths.
*   `-I, --ip <IP_ADDRESS>`: Binds the test to a specific local IP address.
*   `--source-ip <IP_ADDRESS>`: Specifies the source IP address for the test connections.
*   `--ca-certificate <PATH>`: Specifies the path to a custom CA certificate bundle for verifying SSL/TLS connections to the test servers.

## 5. Advanced Usage and Examples

This section provides detailed examples of how to utilize the `speedtest` CLI for various scenarios, ranging from basic bandwidth checks to complex automated monitoring.

### 5.1. Basic Bandwidth Test

The most common use case is a simple, interactive bandwidth test. Executing the command without any arguments will automatically select the best server and display the results in a human-readable format.

```bash
speedtest
```

**Expected Output:**

```text
   Speedtest by Ookla

     Server: Example ISP - City, State (id = 12345)
        ISP: Your Internet Provider
    Latency:    12.34 ms   (0.45 ms jitter)
   Download:   543.21 Mbps (data used: 654.3 MB)
     Upload:   123.45 Mbps (data used: 145.6 MB)
Packet Loss:     0.0%
 Result URL: https://www.speedtest.net/result/c/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

### 5.2. Automated JSON Reporting

For integrating `speedtest` results into a monitoring system, the JSON output format is highly recommended. Disabling the progress bar ensures clean output.

```bash
speedtest -f json -p no
```

**Expected Output (Truncated):**

```json
{
  "type": "result",
  "timestamp": "2023-10-27T10:00:00Z",
  "ping": {
    "jitter": 0.45,
    "latency": 12.34,
    "low": 11.5,
    "high": 13.2
  },
  "download": {
    "bandwidth": 67901250,
    "bytes": 654300000,
    "elapsed": 10005
  },
  "upload": {
    "bandwidth": 15431250,
    "bytes": 145600000,
    "elapsed": 10002
  },
  "packetLoss": 0,
  "isp": "Your Internet Provider",
  "interface": {
    "internalIp": "192.168.1.100",
    "name": "eth0",
    "macAddr": "00:11:22:33:44:55",
    "isVpn": false,
    "externalIp": "203.0.113.50"
  },
  "server": {
    "id": 12345,
    "host": "speedtest.example.com",
    "port": 8080,
    "name": "City",
    "location": "State",
    "country": "US",
    "ip": "198.51.100.20"
  },
  "result": {
    "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "url": "https://www.speedtest.net/result/c/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "persisted": true
  }
}
```

### 5.3. Testing Specific Network Interfaces

In a server environment with multiple network interfaces (e.g., a management interface and a high-speed data interface), you can isolate the test to a specific interface.

```bash
speedtest -i eth1
```

This command ensures that all test traffic is routed through the `eth1` interface, providing an accurate measurement of that specific network path's performance.

### 5.4. Generating CSV Data for Analysis

To collect data over time for analysis in a spreadsheet or database, the CSV format with headers is ideal.

```bash
speedtest -f csv --output-header >> speedtest_results.csv
```

By appending the output to a file (`>>`), you can run this command periodically (e.g., via a cron job) to build a historical dataset of network performance.

### 5.5. Targeted Server Testing

If you suspect routing issues to a specific geographic region or ISP, you can force the `speedtest` CLI to use a server in that location. First, list the available servers:

```bash
speedtest -L
```

Identify the desired server ID (e.g., `9876`), and then execute the test:

```bash
speedtest -s 9876
```

## 6. Troubleshooting and Error Handling

While the `speedtest` CLI is generally robust, you may encounter errors due to network connectivity issues, firewall restrictions, or server unavailability.

### 6.1. Common Error Codes

*   **Configuration Error:** The CLI was unable to retrieve the global configuration. This usually indicates a lack of internet connectivity or a DNS resolution failure.
*   **Server Selection Error:** The CLI could not find a suitable test server. This may occur if all nearby servers are offline or if your firewall is blocking the necessary ports.
*   **Socket Error:** A network connection was unexpectedly closed or timed out during the test. This can be caused by unstable network conditions or aggressive firewall rules.

### 6.2. Diagnostic Steps

1.  **Verify Connectivity:** Ensure that the system has basic internet connectivity by pinging a reliable external host (e.g., `ping 8.8.8.8`).
2.  **Check DNS Resolution:** Verify that DNS resolution is functioning correctly (e.g., `nslookup speedtest.net`).
3.  **Inspect Firewall Rules:** The `speedtest` CLI requires outbound TCP and UDP access on ports 80 and 8080. Ensure that your firewall or security groups allow this traffic.
4.  **Enable Verbose Logging:** While the `speedtest` CLI does not have a dedicated verbose flag, capturing the standard error output can provide additional context for debugging.

## 7. Integration with Monitoring Systems

The `speedtest` CLI's JSON output format makes it an excellent candidate for integration with modern monitoring and observability platforms.

### 7.1. Prometheus and Grafana Integration

A common pattern is to use a wrapper script or a dedicated exporter (e.g., `speedtest-exporter`) that periodically executes the `speedtest` CLI, parses the JSON output, and exposes the metrics in the Prometheus exposition format.

Key metrics to track include:

*   `speedtest_download_bandwidth_bytes`: The measured download bandwidth.
*   `speedtest_upload_bandwidth_bytes`: The measured upload bandwidth.
*   `speedtest_ping_latency_milliseconds`: The measured latency.
*   `speedtest_ping_jitter_milliseconds`: The measured jitter.
*   `speedtest_packet_loss_ratio`: The percentage of packet loss.

By visualizing these metrics in Grafana, network administrators can establish baselines, configure alerts for performance degradation, and correlate network performance with other system metrics.

## 8. Security Considerations

When deploying the `speedtest` CLI in an enterprise environment, several security considerations should be addressed.

### 8.1. Data Privacy

The `speedtest` CLI transmits network performance data, including your public IP address and ISP information, to the test servers and the central Speedtest infrastructure. Ensure that this aligns with your organization's data privacy policies.

### 8.2. Resource Consumption

Bandwidth testing is inherently resource-intensive. Executing frequent speed tests can consume significant network bandwidth and potentially impact other critical services sharing the same network link. It is recommended to schedule automated tests during off-peak hours or implement rate limiting.

### 8.3. Execution Privileges

The `speedtest` CLI generally does not require root or administrator privileges to execute. However, binding to specific network interfaces or IP addresses may require elevated permissions depending on the operating system's security configuration. Always adhere to the principle of least privilege when configuring automated execution.

## 9. Conclusion

The `speedtest` CLI is an indispensable tool for network performance measurement, offering a rich set of features, flexible output formatting, and robust integration capabilities. By mastering the commands, flags, and advanced usage patterns detailed in this comprehensive reference guide, network professionals can effectively diagnose issues, monitor performance trends, and ensure the optimal operation of their network infrastructure. From simple interactive tests to complex automated monitoring pipelines, the `speedtest` CLI provides the precision and reliability required for modern network management.