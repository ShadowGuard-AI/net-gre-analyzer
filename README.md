![Banner](banner.png)

# net-gre-analyzer
Analyzes GRE (Generic Routing Encapsulation) packets in a PCAP file to extract tunnel information and encapsulated protocols. It reads Ethernet frames carrying IPv4 with protocol number 47 (GRE) and reports the GRE flags, the encapsulated protocol, and the payload length. Focused on Basic network operations.

## Install
```bash
git clone https://github.com/ShadowGuard-AI/net-gre-analyzer.git
cd net-gre-analyzer
```

Python 3, standard library only. There is nothing to install.

## Usage
```bash
python3 main.py PCAP_FILE [-v] [-o OUTPUT] [-l LIMIT]
```

## Parameters
- `pcap_file`: Path to the PCAP file containing GRE packets. Required.
- `-v`, `--verbose`: Enable verbose logging (debug level).
- `-o`, `--output`: Output file to save extracted data. Optional; results are appended, one entry per GRE packet.
- `-l`, `--limit`: Limit the number of packets to analyze. Optional.
- `-h`, `--help`: Show help message and exit.

## Exit codes
- `0`: the file was read to the end, or the limit was reached.
- `1`: the PCAP file is missing, has an invalid magic number, or a packet header fails to unpack.

## Limits, as the code states them
- Only Ethernet frames carrying IPv4 are examined; other frames are skipped at debug level.
- The limit counts every frame examined, GRE or not, not just GRE packets analysed, so on a general capture the run can stop before it reaches much GRE.
- Packet headers are always unpacked as little-endian, even though two of the accepted magic values mark big-endian and nanosecond files; those captures get a garbage captured length and the read desynchronises.
- A routing field present in a GRE header is reported as not implemented and the packet is skipped, and checksum, key, and sequence values affect only header size math, never the printed result.

## Fix in Wave 12
The encapsulated protocol was looked up with `getservbyport` on an EtherType, after an extra `ntohs` swap, so a GRE packet carrying IPv4 (EtherType 0x0800) errored with "port/proto not found" instead of reporting a protocol. Verified before the fix on a crafted PCAP with one GRE packet. After the fix, the same file reports "Encapsulated Protocol: IPv4 (EtherType: 0x800)" with a 5 byte payload, and known EtherTypes (IPv4, IPv6, ARP, MPLS, VLAN, LLDP) are named from a small table, with anything else reported as Unknown.

## License
MIT. See [LICENSE](LICENSE).
