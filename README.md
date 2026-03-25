# moxie
It is a DNS DOS tool for ethical testing designed to help people clean up the OPEN DNS resolver issue.
DNS Amplification Test Tool (ETHICAL USE ONLY)

The DNS Amplification Test Tool is a network security testing tool designed to simulate DNS amplification attacks. This tool uses DNS servers to amplify traffic in order to test the security of a target network or service. This tool is for ethical testing only. Unauthorized use can be illegal in many jurisdictions. This tool works by sending specially crafted DNS query packets with a spoofed source IP address (typically the victim's address). When the DNS server receives these requests, it responds with large responses (amplifying the original request). The goal is to test if the target DNS server can handle these amplified responses.


Spoofed DNS queries with random query IDs.
Sends queries to a list of open DNS resolvers to amplify traffic. Supports multi-threaded operation to increase packet-sending speed. Adjustable packet count and delay between packets. Ethical use only: Must be used with explicit permission on targets you own or have authorization to test. Do not use this tool on any system or network without explicit written permission. Unauthorized use of this tool can be illegal and unethical. Always ensure that you have permission before running any penetration testing tools.


System Requirements: 
To run the DNS Amplification Test Tool, you must meet the following system requirements:
Linux/Unix based system (or WSL on Windows).
Root privileges (because raw sockets are required).
A C compiler (e.g., gcc) to compile the source code. Required libraries:
No external dependencies or libraries are required beyond standard C libraries.


Installation
Clone the repository or download the source code:
git clone https://github.com/ZeroDayGang-gh05t5h311/br33dingSlut
cd br33dingSlut

Compile the source code:
Run the following command to compile the C code using gcc:
gcc -o dns_amplification_tool br33dingSlut.c -pthread
This will generate the executable dns_amplification_tool.
Install: There's no formal installation process. You just need to compile the program and run it.

Configuration
The tool comes with a hardcoded list of open DNS resolvers. You can modify this list by changing the gather_dns_resolvers function inside the code.

The current default list of DNS resolvers includes:
Google DNS: 8.8.8.8, 8.8.4.4
Cloudflare DNS: 1.1.1.1, 1.0.0.1
OpenDNS: 208.67.222.222
Quad9 DNS: 9.9.9.9

If you want to use additional resolvers, you can modify this list manually in the gather_dns_resolvers() function, or you can extend the program to read resolvers from a configuration file or a remote source.

Usage
To run the tool, execute it as root with the following parameters:
sudo ./dns_amplification_tool -s <spoofed_source_ip> -d <target_domain> [OPTIONS]
Command-line Options:
-s <IP>
Required: The IP address to spoof as the source address (this is typically the victim's address).
-d <domain>
Required: The domain name you wish to query using the ANY DNS query type (e.g., example.com).
-c <n>
Optional: Number of packets to send. Default is 10. Example: -c 1000 will send 1000 packets.
-D <ms>
Optional: Delay between packets in milliseconds. Default is 100. Example: -D 50 will introduce a 50 ms delay.
-h
Optional: Displays help information.
Example Command:
sudo ./dns_amplification_tool -s 192.168.1.50 -d isc.org -c 20 -D 80
This will send 20 DNS ANY query packets to a random DNS server (from the list of open resolvers) with the source IP spoofed as 192.168.1.50, and a delay of 80 milliseconds between each packet.
Source Code Explanation
1. gather_dns_resolvers function:
This function gathers the list of DNS resolvers that will be used for amplification. By default, it uses a hardcoded list of open DNS servers, but you can modify this list as needed.
void gather_dns_resolvers(const char ***dns_servers) {
    static const char *default_dns[] = {
        "8.8.8.8",    // Google DNS
        "1.1.1.1",    // Cloudflare DNS
        "8.8.4.4",    // Google DNS
        "9.9.9.9",    // Quad9 DNS
        "1.0.0.1",    // Cloudflare DNS (alternate)
        "208.67.222.222", // OpenDNS
        NULL          // Null terminator for the list
    };
    *dns_servers = default_dns;
}

The DNS Packet Construction:
The tool constructs DNS ANY query packets and sends them using raw sockets(why re-invent the wheel?). The packet structure includes:

IP header: Contains the source IP (spoofed) and destination IP (the resolver’s IP).
UDP header: The DNS query is sent via UDP port 53 (standard for DNS).
DNS header: A DNS header with a random query ID and flags set for recursion.
The tool sends the constructed packets to the chosen DNS resolvers, and these DNS servers will respond with large DNS responses, amplifying the traffic.

// Build the DNS question
int name_len = encode_dns_name(qname, domain);  // Encodes the domain name
q->qtype = htons(255);  // ANY query type
q->qclass = htons(1);   // IN (Internet) class
3. Threading:
The tool uses multi-threading to send packets in parallel. This makes it more efficient and increases the volume of requests it can generate.
pthread_t threads[MAX_THREADS];
for (int i = 0; i < MAX_THREADS; i++) {
    pthread_create(&threads[i], NULL, send_packet, (void *)&data[i]);
}
4. Random DNS Resolver Selection:
For each packet, the target DNS resolver is randomly chosen from the list of available DNS servers. This ensures that the packets are sent to different servers for amplification.
const char *target_ip = dns_servers[rand() % 6]; // Choose from the list of DNS servers
5. Packet Sending:
The tool sends the crafted packet to the chosen DNS resolver using the sendto() system call.
if (sendto(sock, packet, packet_len, 0, (struct sockaddr *)&dest, sizeof(dest)) < 0) {
    fprintf(stderr, "sendto failed: %s\n", strerror(errno));
} else {
    printf("[+] Sent packet %3d | ID: %5u | src:%s → dst:%s:53\n", i, ntohs(dns->id), spoof_ip, target_ip);
}

Important Notes
Running as Root: This tool requires root privileges because it needs to create raw sockets.

Ethical Use: Only run this tool on networks or systems that you own or have explicit permission to test. Unauthorized use is illegal and unethical.

DNS Amplification Test: This tool is designed to simulate DNS amplification attacks. It's used to test the ability of DNS servers to handle large volumes of traffic, which could help identify vulnerabilities in the server’s defense mechanisms.
Customize DNS Resolvers: You can modify the list of DNS resolvers directly in the code or implement a configuration file to dynamically load resolver IPs.

Impact on Target: While testing, ensure that the target DNS server is capable of handling the volume of traffic you’re generating. Do not overwhelm a system that is not authorized for testing.

License This tool is provided under the MIT License. Feel free to modify and use it for your own ethical penetration testing purposes.
