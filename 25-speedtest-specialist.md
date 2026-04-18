# 25 - Speedtest (Ookla) Specialist

## Introduction

Since its inception in 2006, Ookla’s Speedtest.net has become the preeminent platform for measuring internet performance worldwide, boasting over 67 billion tests and more than 11 million unique daily users. Its comprehensive testing capabilities span bandwidth, latency, jitter, and quality of experience metrics across video streaming, gaming, and general network responsiveness. The platform's unique strength lies in its combination of a massive global server network exceeding 16,000 nodes, rigorous measurement methodologies, and a proprietary client-server architecture optimized for both accuracy and scalability. This document provides an exhaustive technical analysis of the Speedtest.net architecture and methodology, focusing on the core measurement mechanisms, TCP test components, protocol details, result calculation algorithms, server architecture, multi-stage latency measurement, and legacy HTTP fallback testing.

## 1. Core Measurement Methodology

The fundamental premise of Speedtest.net’s methodology is to capture the **full throughput capacity** of the user's network connection, alongside accurate latency and jitter metrics, through a **foreground testing** paradigm. Unlike background or passive testing approaches — which often use small file transfers or CDN-based content that do not saturate the connection — Speedtest is designed to flood the network interface actively, pushing the connection to its maximum utilization. This approach ensures that the test reflects the **realistic maximum quality of service (QoS)** available to the user, encompassing the device, local network, last-mile ISP, and broader internet path.

Foreground testing is initiated explicitly by user interaction, triggering a multi-phase test that includes latency measurement, download saturation, and upload saturation. To achieve full connection saturation, Speedtest dynamically scales the number of TCP connections and adjusts data chunk sizes and buffer parameters in real-time. The system is architected to support speeds up to 10 Gbps, accommodating emerging technologies such as 5G carrier aggregation and fiber optic broadband, which require sustained high-throughput demands to activate network features fully.

By employing a global network of over 16,000 geographically distributed servers — strategically placed to minimize additional network latency — Speedtest minimizes extraneous travel time and accurately reflects the user's last-mile and transit performance. This server distribution also enables multi-server testing, where simultaneous connections to multiple servers are established to saturate the network beyond what a single server could achieve.

## 2. TCP Test Components

The Speedtest TCP test is composed of three distinct phases: **Latency/Jitter measurement**, **Download phase**, and **Upload phase**. Each phase is optimized to accurately capture network performance under different conditions.

### Latency and Jitter Measurement

Latency, defined as the bidirectional round-trip time (RTT) in milliseconds, is measured by the client sending timestamped packets to the server, which immediately replies. The RTT is calculated as the elapsed time between sending and receiving the echo. To ensure statistical significance and reduce the impact of transient network anomalies, multiple ping messages are sent, and the lowest RTT is selected as the representative latency measurement. Jitter, reflecting the variability in latency, is derived from the range and standard deviation of RTTs across these samples.

### Download Phase

The download test initiates by establishing multiple concurrent TCP connections to the selected server, all communicating over **port 8080**, a non-privileged port chosen for universal accessibility and to avoid conflicts with common HTTP services. The client initially requests a data chunk from the server and continuously monitors the throughput, dynamically adjusting the chunk size and TCP buffer window to maximize network utilization.

During the first half of the test duration, the client may spawn additional threads (TCP connections) if the observed throughput indicates that existing threads are insufficient to saturate the connection fully. This adaptive scaling overcomes the limitations imposed by TCP slow start and congestion control algorithms, which can throttle throughput when limited to a single connection.

The client uses real-time feedback from TCP acknowledgments and throughput calculations to incrementally increase the chunk size and the receive window advertised to the server, enabling more data to be in-flight and thus better utilizing high-bandwidth, high-latency paths.

### Upload Phase

The upload phase mirrors the download test in methodology but in reverse data flow. The client establishes multiple TCP connections and sends data chunks to the server over port 8080. As with download testing, the client dynamically adjusts chunk sizes and buffer windows based on the rate of acknowledgments and throughput metrics.

Additional connections are spawned during the first half of the upload test if necessary to saturate the upload capacity of the network. The server measures the incoming data streams, enabling accurate assessment of the upload throughput even in the presence of network asymmetries or high latency.

## 3. Protocol Details

### Port Usage and Connection Management

Speedtest employs TCP connections primarily over port 8080. This port selection balances accessibility — avoiding privileged ports that require elevated permissions — with compatibility across various firewalls and network configurations. The use of TCP ensures reliable, ordered delivery, which is essential for accurate throughput and latency measurement.

### Chunk Sizing and Buffer Adjustments

The client continuously monitors instantaneous throughput and TCP-level acknowledgments to dynamically adjust the size of data chunks requested (download) or sent (upload). Initial chunk sizes are conservative to accommodate slower connections and avoid overwhelming the network buffers. As the test progresses and throughput stabilizes, chunk sizes increase, sometimes exponentially, to maximize utilization.

TCP window scaling is integral to this process. The client adjusts the receive window size advertised to the server, allowing more unacknowledged data to be in flight. This mechanism is crucial for high-bandwidth, high-latency connections where default TCP window sizes would otherwise limit throughput. By leveraging TCP window scaling, Speedtest effectively counters the bandwidth-delay product limitations.

Buffer sizes on both client and server are tuned dynamically. On the client side, socket receive buffers are adjusted to match the increasing chunk sizes, ensuring that the application layer can process incoming data without inducing artificial bottlenecks. Similarly, the server adjusts send buffers in response to client window size advertisements, maintaining a steady data flow.

### Threading Logic

Speedtest's threading model is adaptive. Initially, the test starts with a minimal number of TCP connections (often one or two). If throughput measurements indicate underutilization of the available bandwidth, additional threads are spawned up to a maximum configured limit (typically four for HTTP fallback testing and potentially more for TCP tests).

This approach allows the test to overcome TCP slow start on individual connections and better utilize multi-core client devices by parallelizing data transfers. The client aggregates throughput across all threads, adjusting chunk sizes independently per thread based on per-connection feedback.

In a high-level pseudocode representation, the threading logic can be abstracted as:

```pseudo
initialize threads = 1
while test_duration_not_reached:
    measure throughput across threads
    if throughput < expected and threads < max_threads:
        threads += 1
        establish new TCP connection
    for each thread:
        adjust chunk size based on throughput feedback
    sleep short_interval
aggregate throughput across all threads for final calculation
```

This dynamic threading ensures that the test adapts to a broad spectrum of network conditions and device capabilities.

## 4. Result Calculation Algorithm

Once the download and upload phases conclude, Speedtest processes the collected throughput samples to produce the final speed result. This post-processing involves rigorously filtering out outliers and ensuring statistical robustness.

The algorithm first sorts all throughput samples collected during the test in descending order. The two fastest samples are removed to eliminate any anomalous spikes caused by transient network bursts or measurement artifacts. Subsequently, the bottom 25% (approximately the slowest quarter of samples) are discarded, removing periods of network congestion or transient slowdowns that do not represent typical performance.

The remaining middle 72-73% of samples are then averaged to compute the final reported download or upload speed. This approach balances sensitivity to consistent throughput with the removal of spurious deviations, providing an accurate and stable metric.

Mathematically, if \( S = \{s_1 \geq s_2 \geq ... \geq s_n\} \) is the sorted set of samples:

\[
S' = S[3 : \left\lceil 0.75 n \right\rceil]
\]

\[
\text{Final Speed} = \frac{1}{|S'|} \sum_{i=3}^{\left\lceil 0.75 n \right\rceil} s_i
\]

where the indices exclude the two fastest samples and the bottom 25% of samples.

This filtering methodology applies to both TCP and HTTP fallback testing, ensuring consistency across protocols.

## 5. The Speedtest Server Network Architecture

### Global Server Network

Ookla operates a vast, globally distributed server network consisting of over 16,000 servers spanning more than 190 countries and all inhabited continents. This network encompasses more than 7,000 distinct host networks, including ISPs, mobile operators, hosting providers, and academic institutions. The servers are strategically deployed in major population centers to minimize latency and maximize test accuracy.

Server nodes vary in capacity based on market demands. Smaller markets typically feature servers with minimum 1 Gbps upstream and downstream connectivity. Developed markets and major metropolitan areas are provisioned with 10 Gbps or higher capacity servers to accommodate dense user populations and high-speed broadband connections. Large operators may deploy 40 Gbps or 100 Gbps server infrastructure to serve ultra-high-speed clients and enterprise customers.

### OoklaServer Daemon

Each server runs the proprietary **OoklaServer** daemon, a TCP-based server application designed to communicate efficiently with Speedtest clients. The daemon incorporates several key features:

1. **Dynamic Connection Scaling**: The server supports multiple concurrent TCP connections per client session, dynamically allocating resources to match client threading levels.

2. **Stable Stop Early Test Termination**: To optimize bandwidth use, the server can terminate tests early once the client has collected sufficient data for an accurate measurement, preventing unnecessary network load.

3. **Server-Side Upload Measurement**: Upload throughput is measured on the server by aggregating incoming data streams, enabling precise measurement even in scenarios with high latency or asymmetric network paths.

The daemon implements a proprietary protocol layered over TCP that supports chunked data transfer, buffer negotiation, and connection management to facilitate the test.

### Server Selection Algorithm

When a user initiates a test, the Speedtest client selects an optimal test server using a multi-criteria algorithm. The primary objective is to select a server that best represents the user's **last-mile connection quality** and the broader transit network.

The algorithm considers geographic proximity, server response times, network path characteristics, and server load and availability. Servers are classified as **on-net** if they reside within the user's ISP network or peering arrangement, providing a measure of last-mile performance. **Off-net** servers, located outside the user's immediate network, facilitate measurement of transit and peering quality.

The selection process ensures the server chosen provides a representative and accurate measurement while minimizing latency introduced by extraneous network hops.

### Server Monitoring and Performance Assurance

Ookla employs automated monitoring systems that continuously evaluate server health, bandwidth capacity, and responsiveness. Servers that fail performance thresholds or exhibit unstable connectivity are automatically decommissioned from the testing pool to maintain network integrity.

## 6. Measuring Ping at 3 Stages and Gaming Guidelines

Speedtest uniquely measures latency (ping) at three distinct stages during a test: **Idle**, **Download**, and **Upload**. This multi-stage measurement provides a comprehensive view of network responsiveness under varying load conditions.

- **Idle Ping**: Measured at the beginning of the test before any data transfer begins. This represents the baseline network latency when the connection is not under load.

- **Download Ping**: Measured concurrently during the download phase. Latency here reflects the network responsiveness while the downlink is saturated, revealing the impact of congestion and buffering.

- **Upload Ping**: Measured during the upload phase, representing latency under uplink load.

This tri-stage latency measurement is especially valuable for applications sensitive to delay and jitter, such as online gaming and real-time communications. By assessing latency under idle and loaded conditions, Speedtest provides insights into network stability and suitability for latency-critical use cases.

### Gaming Latency Guidelines

Based on empirical data gathered from billions of tests, Ookla provides latency thresholds for gaming performance:

| Latency Range (ms) | Gaming Experience          |
|--------------------|---------------------------|
| 0 - 59             | Winning                   |
| 60 - 129           | In the game               |
| 130 - 199          | Struggling                |
| 200+               | Game over (unplayable)    |

These guidelines help users and network engineers assess whether their connection is adequate for online gaming and identify when network improvements are necessary.

## 7. HTTP Legacy Fallback Testing Mechanics

While the primary Speedtest methodology employs TCP-based testing optimized for accuracy and scalability, a fallback mechanism using HTTP is implemented to maximize compatibility with restrictive network environments or legacy systems.

### HTTP Latency Measurement

HTTP latency is measured by timing the duration to receive an HTTP response from the server. The client issues multiple HTTP GET requests to small binary files, recording the time between request initiation and response completion. The lowest observed latency is selected to minimize the influence of transient network delays.

### HTTP Download Testing

The HTTP download test operates by downloading small binary files to estimate connection speed. Initially, the client downloads a small file to gauge throughput and select an appropriately sized file for the main test. To prevent caching at any network intermediate or the client itself, random strings are appended to the request URLs.

Throughput samples are collected at a high frequency, up to 30 times per second, ensuring fine-grained measurement granularity. These samples are aggregated into 20 equal slices, each representing 5% of the total samples. After discarding outliers, the remaining slices are averaged to produce the final download speed.

### HTTP Upload Testing

For upload testing, the client sends small chunks of random data via HTTP POST requests. An initial estimate phase determines the optimal chunk size for the connection. The test uses up to four HTTP threads to parallelize uploads, minimizing overhead on slower connections by reducing thread count when necessary.

As with download testing, throughput samples are sorted, and the fastest half are averaged to derive the final upload speed.

### Threading in HTTP Testing

To mitigate HTTP overhead and optimize performance, Speedtest employs a dynamic threading model for HTTP fallback testing. If the pre-test estimated speed is below 4 Mbps, the test uses two threads; otherwise, four threads are employed. This selective threading balances resource use with measurement accuracy.

## TCP vs HTTP Testing: A Comparative Overview

| Aspect                  | TCP Testing                               | HTTP Fallback Testing                   |
|-------------------------|-----------------------------------------|----------------------------------------|
| Protocol                | Proprietary TCP-based protocol           | Standard HTTP over TCP                  |
| Port                    | 8080                                     | 80 or 443 (depending on configuration) |
| Connection Threads      | Dynamic, scalable (up to multiple higher threads) | Up to 4 threads                        |
| Chunk Size Management   | Adaptive chunk sizing and buffer scaling | Pre-defined chunk sizes based on pre-test |
| Latency Measurement     | Multiple RTT pings, lowest value chosen  | HTTP response time, multiple samples   |
| Throughput Sampling     | Continuous real-time throughput calculation | Samples aggregated into 20 slices      |
| Accuracy                | High, supports saturation up to 10 Gbps | Moderate, fallback for restrictive networks |
| Use Case                | Default for modern devices/networks      | Fallback for legacy or restricted environments |
| Overhead                | Low, optimized for bulk data transfer    | Higher due to HTTP headers and POST overhead |

## Deep Technical Explanation: TCP Window Scaling and Threading in Speedtest.net

The core challenge in accurately measuring high-speed internet connections lies in overcoming the intrinsic limitations of TCP congestion control and flow control mechanisms, especially the **bandwidth-delay product (BDP)**. The BDP quantifies the amount of data that must be in flight to fully utilize the network path:

\[
\text{BDP} = \text{Bandwidth (bits/s)} \times \text{RTT (s)}
\]

For example, a 1 Gbps connection with 50 ms RTT requires approximately 6.25 MB of data in transit to saturate the link.

TCP window size regulates the maximum amount of unacknowledged data sent on the connection. Default TCP window sizes without scaling (typically 64 KB) are insufficient for high BDP paths. To address this, the TCP Window Scaling option (RFC 1323) allows advertising a window scale factor, expanding the effective window size up to 1 GB.

In Speedtest, the client monitors incoming acknowledgments and network conditions to adjust socket receive buffer sizes and advertise increased window sizes dynamically during tests. This adaptation ensures that the TCP stack can maintain a sending rate sufficient to saturate the connection.

Threading complements window scaling by establishing multiple parallel TCP connections, each with its own window. This approach mitigates limitations of single connection slow start and enables aggregate throughput to approach or exceed the sum of individual connection capacities.

The dynamic scaling of both window sizes and number of threads during Speedtest phases ensures that the test accurately reflects the maximum achievable throughput, regardless of underlying TCP stack or network conditions.

## Conclusion

Ookla’s Speedtest.net architecture and methodology represent a pinnacle of internet performance measurement technology. By combining foreground testing that saturates the full connection capacity, adaptive TCP testing with dynamic chunk sizing and threading, a vast global server network, and robust result calculation algorithms, Speedtest provides unparalleled accuracy and reliability. Its tri-stage latency measurement and gaming guidelines offer actionable insights for latency-sensitive applications, while HTTP fallback testing guarantees compatibility across diverse network environments.

Understanding the deep technical underpinnings—from TCP window scaling to server selection algorithms—is essential for specialists aiming to design, optimize, or interpret broadband performance tests in the modern, heterogeneous internet landscape. Speedtest.net remains the definitive standard for QoS assessment worldwide, powered by a sophisticated interplay of network engineering, protocol design, and statistical rigor.