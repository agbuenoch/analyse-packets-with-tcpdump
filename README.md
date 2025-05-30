# analyse-packets-with-tcpdump
This project used the command-line tool `tcpdump` to capture and analyse live network traffic from a Linux virtual machine. It identifies `network interfaces` to capture network packet data. It uses `tcpdump` to filter live network traffic and capture network traffic using tcpdump. Lastly, it filters the captured packet data.

## Step 1: Identify Network Interfaces.
Run `pwd` to print the current working directory as pointed to by the 1st arrow. To list all files and directories in the current working directory, run `ls -l`, which lists all the directory contents, as pointed to by the 2nd arrow. There is only one file `sample.pcap` in the `/home/analyst directory`.<br>
**View: [Step1A](screenshots/Step1A)**

We must first of all identify the network interfaces that can be used to capture network packet data. Run the command 
```bash
sudo ifconfig
```
to identify the available interfaces. The 1st and 2nd arrows point to the `eth0` and `lo` network interfaces, respectively. The Ethernet network interface is identified by the entry with the eth prefix.<br>
**View: [Step1B](screenshots/Step1B)**

Alternatively, we can use `tcpdump` to identify the interface options available for packet capture.
```bash
sudo tcpdump -D
```

This command may be useful on systems that do not include the `ifconfig` command. The screenshot below lists all available network interfaces we can use to capture packets, as pointed out by the arrow. The network interfaces include `eth0, any, lo, nflog, nfqueue`. Notice that `eth0` and `any` network interfaces are up and running.<br>
**View: [Step1C](screenshots/Step1C)**

- tcpdump = This is a command-line packet capture tool.
- -D = This will display a numbered list of all available interfaces.

**What does the "any" interface mean?**
The "any" interface is neither a physical nor a virtual network adapter. It is a special pseudo-interface used by tcpdump to listen on all available interfaces simultaneously.

When you are not sure which interface traffic will come through (e.g., `eth0, wlan0, lo`), you can use:
```bash
sudo tcpdump -i any
```

This will capture traffic across all interfaces, such as:
- `eth0`: Ethernet
- `wlan0`: Wireless
- `lo`: Loopback
- `docker0` or `br-xxxx`: Docker or bridge interfaces

The limitation is, it cannot capture the link-layer header (e.g., Ethernet headers). So, if you need to inspect Ethernet frames, use a specific interface (e.g., `-i eth0`) instead. Another limitation is that you won’t see MAC addresses when capturing on `any` interface.

Example:
```bash
sudo tcpdump -i any port 80
```
This captures all `HTTP traffic` on all interfaces. This is great for general network monitoring or troubleshooting.

## Step 2: Inspect the network traffic of a network interface with tcpdump.
The command-line tool `tcpdump` is used to filter live network packet traffic on an interface. Let us filter live network packet data from the `eth0` interface with <br>
```bash
sudo tcpdump -i eth0 -v -c5
```

Run `tcpdump` with the following options or flags:
- `-i eth0`: Capture data specifically from the eth0 interface.
- `-v`: Display detailed packet data.
- `-c5`: Capture 5 packets of data.
**View: [Step2A](screenshots/Step2A)**

Let's take a detailed look at the packet information that this command has returned. Five packets were captured as pointed to by the numbered arrows, with each packet starting with a `time stamp` (in Hours, Minutes, and Seconds, e.g. 06:37:28.000526), followed by the protocol type, `IP`.

At the start of the packet output, tcpdump reported that it was listening on the `eth0` interface, and it provided information on the `link type` and the capture size in `bytes`:<br>
**View: [Step2B](screenshots/Step2B)**

Looking at the first packet, which is highlighted in Red-Orange as pointed to by the 1st arrow, the first field is the packet's `timestamp`, followed by the protocol type, `IP`. The verbose option, `-v`, has provided more details about the IP packet fields, such as `TOS, TTL, offset, flags`, internal protocol type (in this case, `TCP (6)`), and the `length` of the outer IP packet in bytes:<br>
**View: [Step2C](screenshots/Step2C)**

The specific details about these fields are beyond the scope of this article. But you should know that these are properties that relate to the IP network packet. 

In the next section, the data shows the systems that are communicating with each other:<br>
**View: [Step2D](screenshots/Step2D)**

By default, `tcpdump` will convert `IP addresses` into `names`, as in the screenshot. The name of your Linux virtual machine, also included in the command prompt, appears here as the source for one packet and the destination for the second packet. In your live data, the name will be a different set of letters and numbers.

The direction of the arrow `>` indicates the direction of the traffic flow in this packet. Each system name includes a suffix with the port number (`.5000` in the screenshot), which is used by the source and the destination systems for this packet.

The remaining data filters the header data for the inner TCP packet:<br>
**View: [Step2E](screenshots/Step2E)**

The `flags` field identifies `TCP flags`. In this case, the `P` represents the `push` flag, and the period `.` indicates it's an `ACK` flag. This means the packet is pushing out data.

The next field is the `TCP checksum value`, which is used for detecting errors in the data.

This section also includes the `sequence` and `acknowledgement numbers`, the window `win` size, and the `length` of the inner TCP packet in bytes.

## Step 3: Capture network traffic with tcpdump.
Let's use `tcpdump` to save the captured network data to a packet capture file.

In the previous command, we used tcpdump to stream all network traffic. Here, we will use a filter and other tcpdump configuration options to save a small sample that contains only web `TCP port 80` network packet data.

Capture packet data into a file called `networktraffic.pcap` using:
```bash
sudo tcpdump -i eth0 -nn -c9 port 80 -w networktraffic.pcap &
```

Press the `ENTER` key to get your command prompt back after running this command. Let us first of all run `ls -l` to list out the files and directories currently in the home directory, as pointed out by the 1st arrow, there is only `sample.pcap`. This command runs in the background, but some output text will appear in your terminal as pointed to by the 2nd arrow.
**View: [Step3A](screenshots/Step3A)**

This command will run tcpdump in the background with the following options:

- `-i eth0`: Capture data from the eth0 interface.
- `-nn`: Do not attempt to resolve IP addresses or ports to names. This is best practice from a security perspective, as the lookup data may not be valid. It also prevents malicious actors from being alerted to an investigation.
- `-c9`: Capture 9 packets of data and then exit.
- `port 80`: Filter only port 80 traffic. This is the default HTTP port.
- `-w networktraffic.pcap`: Save the captured data to the named file.
- `&`: This is an instruction to the Bash shell to run the command in the background.

Notice that the `networktraffic.pcap` file has been created as pointed to by the 3rd arrow above, but the file at this stage is empty because we have not currently generated any web traffic that will be captured into the file; this is why the `value` of `networktraffic.pcap file length is currently `empty` or `zero 0`, circled in `yellow`. Remember, the command above is an instruction for web traffic (`port 80`) to be captured from the `eth0` network interface into `networktraffic.pcap` file.

Use `curl` to generate some HTTP (`port 80`) traffic. When the curl command
```bash
curl kc7cyber.com
```
is used like this to open a website; it generates some HTTP (`TCP port 80`) traffic that can be captured as pointed to by the 4th arrow.

To verify that packet data has been captured after running `curl kc7cyber.com`, run the command `ls -l capture.pcap` and this will return the output pointed to by the fifth arrow above, but notice that the value `zero 0` circled in `yellow colour` as pointed to by the third arrow has now changed to `977` circled in `blue colour`, this implies that the command `curl kc7cyber.com` generated some HTTP (`port 80`) traffic and was captured into our `networktraffic.pcap` file that is running in the background: 

## Step 4: Filter the captured packet data.
Let's use tcpdump to filter data from the packet capture file we saved previously in `Step 3 above`.

Use the tcpdump command, this time without the option `-v`, i.e. verbose.
```bash
sudo tcpdump -nn -r networktraffic.pcap
```

to filter the packet header data from the `networktraffic.pcap` capture file. You will notice that no detailed information about IP is presented here, and the output return is a bit less compared to when we use the `-v` option/flag. As pointed to by the 1st arrow, tcpdump is reading the packets from the packets saved in `networktraffic.pcap`.<br>
**View: [Step4A](screenshots/Step4A)**

This command will run tcpdump with the following options:

- `-nn`: Disable port and protocol name lookup.

- `-r`: Read capture data from the named file, i.e. networktraffic.pcap.

If the `-v` option was included, it would have displayed detailed packet data.

You must specify the `-nn` switch again here, as you want to make sure tcpdump does not perform name lookups of either `IP addresses` or `ports`, since this can alert threat actors.

Use the tcpdump command 
```bash
sudo tcpdump -nn -r capture.pcap -X
```

to filter the extended packet data as pointed to by the 3rd arrow from the `networktraffic.pcap` capture file.<br>
**View: [Step4B](screenshots/Step4B)**

This command will run tcpdump with the following options:
- `-nn`: Disable port and protocol name lookup.
- `-r`: Read capture data from the named file i.e `networktraffic.pcap`.
- `-X`: Display the hexadecimal and ASCII output format packet data. Security analysts can analyse hexadecimal and ASCII output to detect patterns or anomalies during malware analysis or forensic analysis.

> Hexadecimal, also known as hex or base 16, uses 16 symbols to represent values, including the digits 0-9 and letters A, B, C, D, E, and F. American Standard Code for Information Interchange (ASCII) is a character encoding standard that uses a set of characters to represent text in digital form.

##  Summary.
This article explains how to use the network protocol analyser command line tool `tcpdump` to capture and analyse network traffic. First, we identify a network interface, then use tcpdump to filter and capture live network traffic. We explain the tcpdump command and its options, and how to interpret the output. Lastly, we explain how to save captured network data to a file and filter the data.

## LinkedIn Article.
- [Analyse Packets with tcpdump](https://www.linkedin.com/pulse/kali-linux-users-account-management-enoch-agbu-hhzof)

## Connect with me.
[🔗 LinkedIn](https://www.linkedin.com/in/agbuenoch)<br>
[🔗 X](https://www.x.com/agbuenoch)
