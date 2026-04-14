# moxie
Moxie 2 – DNS Packet Crafting and Network Testing Utility

Overview:
Moxie 2 is a low-level DNS packet construction and transmission tool written in C. It is designed for educational use, controlled laboratory testing, and defensive security research. The project provides a practical demonstration of how raw network packets can be manually constructed and transmitted without relying on the operating system’s standard networking stack.
By working directly with IP and UDP headers and crafting DNS queries from scratch, Moxie 2 offers insight into how network protocols operate at a fundamental level. It is particularly useful for students, developers, and security professionals who want to deepen their understanding of packet structure, checksum computation, and traffic generation techniques in a controlled environment. This tool is not intended for use on networks or systems without explicit authorization. It should only be used in environments where you have full ownership or permission to conduct testing.

Core Capabilities:

Moxie 2 focuses on manual packet construction and controlled transmission. It builds complete IPv4 and UDP packets, embeds a DNS query payload, and sends them using raw sockets. The implementation includes support for multi-threaded packet transmission and configurable timing, allowing users to simulate controlled traffic patterns. Key capabilities include manual construction of IPv4 headers, UDP headers, and DNS query packets; encoding of domain names into DNS label format; configurable packet count and transmission delay; multi-threaded execution for parallel packet sending; randomized packet attributes such as transaction IDs, source ports, and TTL values; and full checksum calculation for both IP and UDP layers using a pseudo-header.

How It Works:

The program constructs each packet from the ground up. A DNS query is first assembled by encoding the domain name into the correct label format and attaching a DNS header and question section. This payload is then wrapped inside a UDP segment with a randomized source port.
An IPv4 header is built around the UDP segment, including fields such as total length, protocol type, and source and destination addresses. The source address is user-defined, allowing experimentation with how packets appear at the network level. Check sums are calculated manually to ensure packet integrity. Packets are transmitted using a raw socket with IP header inclusion enabled, meaning the kernel does not modify the headers. Multiple threads divide the workload, each responsible for sending a portion of the total packet count. Timing between packets is controlled with a configurable delay and a small randomized jitter to avoid perfectly uniform traffic patterns.

System Requirements:
Moxie 2 is intended for use on Unix-like systems that support raw sockets. It has been tested on Linux environments.
Requirements include a GCC-compatible compiler, POSIX threading support, and root or elevated privileges to allow raw socket operations.

Compilation:
To compile the program, use the following command:
gcc -o moxie2 moxie2.c -lpthread
gcc -o moxie2 moxie2.c -pthread (safer and more complete alternative)
This will produce a single executable named moxie2.
The program is executed from the command line and requires both a source IP address and a domain name.
sudo ./moxie2 -s <source_ip> -d [options]
Required arguments:
-s or --spoof specifies the source IP address to be placed in the packet headers.
-d or --domain specifies the domain name to be queried.
Optional arguments:
-c or --count sets the number of packets to send. The default is 10.
-D or --delay sets the delay between packets in milliseconds. The default is 100.
-h or --help displays the usage information.

Example
sudo ./moxie2 -s 192.168.1.10 -d example.com -c 20 -D 50

This example sends 20 packets with a 50 millisecond delay between transmissions. Operational Notes
Because Moxie 2 uses raw sockets, it must be run with elevated privileges. The program does not rely on the system’s DNS resolver and instead sends packets directly to a predefined list of public DNS servers. The tool does not process or capture responses. It is strictly a transmission utility designed to demonstrate packet creation and outbound traffic behavior. Safety and Responsible Use

Moxie 2 is intended strictly for legitimate and ethical purposes. Acceptable use cases include local lab experimentation, academic study, protocol analysis, and authorized network testing. It must not be used to disrupt services, generate unauthorized traffic, or target systems without explicit permission. Misuse may violate laws and regulations and can result in serious consequences. If you are unsure whether your use case is appropriate, do not proceed.

Limitations:
The current implementation is intentionally minimal and focused on demonstrating core concepts. It supports only IPv4 and does not include DNS response handling or validation. The list of DNS revolvers is static and hard-coded, and there is no configuration for dynamic discovery or customization. Error handling is basic by design and(also by design) the tool does not include advanced safeguards such as rate limiting or traffic shaping beyond simple delay and jitters, it could be thought of as some POC that is semi-functional

Future Development:
There are several areas where the project could be extended. Adding DNS response parsing would allow for round-trip analysis. Support for additional query types and protocols such as TCP-based DNS could broaden its usefulness. Introducing configurable resolver lists and improved logging would make the tool more flexible and informative. From a safety perspective, implementing optional safeguards such as rate limiting, interface binding, or restricting execution to local environments would improve responsible usability.

Disclaimer
This software is provided for educational and research purposes only. The author assumes no responsibility for misuse, damage, or legal consequences resulting from the use of this tool. Users are solely responsible for ensuring their actions comply with all applicable laws and regulations.

A test run may look something like this: 
┌───────────────────────────────────────────────────────────────┐
│                        MOXIE 2 v1.0                           │
│           DNS Packet Crafting Test Environment                │
├───────────────────────────────────────────────────────────────┤
│ Mode           : Controlled Lab Simulation                    │
│ Source IP      : 192.168.1.10                                 │
│ Query Domain   : example.com (ANY)                            │
│ Threads        : 4                                            │
│ Packet Count   : 20                                           │
│ Delay          : 50 ms (+ jitter)                             │
├───────────────────────────────────────────────────────────────┤
│ Initializing raw socket...                    [ OK ]           │
│ Building DNS query payload...                [ OK ]           │
│ Encoding domain labels...                    [ OK ]           │
│ Calculating checksums...                     [ OK ]           │
│ Launching worker threads...                  [ OK ]           │
├───────────────────────────────────────────────────────────────┤
│ [Thread 1] Sent packet  1 | ID: 48231 | → 8.8.8.8:53           │
│ [Thread 2] Sent packet  2 | ID: 11902 | → 1.1.1.1:53           │
│ [Thread 3] Sent packet  3 | ID: 55012 | → 9.9.9.9:53           │
│ [Thread 4] Sent packet  4 | ID: 33177 | → 8.8.4.4:53           │
│ [Thread 1] Sent packet  5 | ID: 90214 | → 208.67.222.222:53    │
│ [Thread 2] Sent packet  6 | ID: 12003 | → 1.0.0.1:53           │
│ ...                                                           │
│ [Thread 3] Sent packet 19 | ID: 77123 | → 8.8.8.8:53           │
│ [Thread 4] Sent packet 20 | ID: 44301 | → 9.9.9.9:53           │
├───────────────────────────────────────────────────────────────┤
│ Transmission complete.                                        │
│ Total packets sent : 20                                       │
│ Average rate       : ~18 packets/sec                          │
│ Errors             : 0                                        │
└───────────────────────────────────────────────────────────────┘

Can't say for sure as I have due to laws been unable to test it: unless someone with a big enough network would be able to test on my behalf and please share finding's(even if only a little), or let me test it.

License This tool is provided under the MIT License. Feel free to modify and use it for your own ethical penetration testing purposes.
