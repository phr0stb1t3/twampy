# twampy

**Python implementation of TWAMP/TWAMP-light (RFC 5357) and STAMP (RFC 8762)**

twampy is a network measurement tool for latency, jitter, and packet loss testing. Originally developed to validate Nokia SR OS and SR Linux TWAMP implementations, it provides a standards-compliant implementation of the Two-Way Active Measurement Protocol and Simple Two-Way Active Measurement Protocol.

## About this fork

This repository is an unofficial fork of [nokia/twampy](https://github.com/nokia/twampy).
Nokia does not maintain, support or endorse this fork.
Report problems with the changes below in this repository, not upstream.

The fork starts from upstream release 1.3.2 (commit `885b986`).
The table lists every change that this fork adds to upstream.

| Change | What it does | Commit | Upstream status |
|--------|--------------|--------|-----------------|
| Sender busy-wait fix | The session sender blocks in `select()` until the next probe is due. Upstream spins between the reply and the next probe and uses one full CPU core per sender session. In a loopback test, CPU time for 10 probes at 1 s dropped from 9.06 s to 0.07 s. | `bf44790` | Proposed as [nokia/twampy#19](https://github.com/nokia/twampy/pull/19) |
| Sub-millisecond send interval | `--interval` (sender and controller) accepts fractional milliseconds, with a minimum of 0.05 ms (50 µs). It rejects 0, negative values, `nan`, `inf` and text. Upstream accepts whole milliseconds only (minimum 1 ms) and accepts 0 and negative values. | `a094510` | Fork only |
| Padding generation | `zeros()` returns `bytes(n)` instead of packing a new list for each packet. The bytes on the wire do not change. | `a094510` | Fork only |
| Documentation | The configuration guide states the correct default interval (100 ms, not 1000 ms) and the new range. The README has this section, and `LICENSE` has a modification notice. | `a094510` and later | Fork only |

Below about 0.5 ms, the default Linux timer slack (50 µs) and scheduler latency make the gaps between packets uneven.
The average rate does not change.
To remove timer slack, run the sender with a real-time policy, for example `chrt -f 50`.

To see the full difference from upstream:

```bash
git remote add upstream https://github.com/nokia/twampy.git
git fetch upstream
git diff upstream/master master
```

## Features

- ✅ **Multiple Operation Modes** - Controller, Control Client, Session Sender, Reflector
- ✅ **Dual Stack** - IPv4 and IPv6 support
- ✅ **QoS Testing** - DSCP/TOS marking for quality of service validation
- ✅ **Flexible Sizing** - Custom padding, jumbo frames, IMIX traffic patterns
- ✅ **RFC Compliant** - Implements RFC 5357 (TWAMP), RFC 8762 (STAMP), RFC 8972 (STAMP Extensions)
- ✅ **No Dependencies** - Uses Python standard library only

## Quick Start

### Installation

```bash
# Clone and install
git clone https://github.com/phr0stb1t3/twampy
cd twampy
pip install -e .
```

### Basic Usage

```bash
# Start a reflector
twampy responder :862

# Run a test (from another terminal/host)
twampy controller 192.168.1.100
```

Example output:
```
===============================================================================
Direction         Min         Max         Avg          Jitter     Loss
-------------------------------------------------------------------------------
  Outbound:       92.89ms    196.63ms     95.15ms       576us      0.0%
  Inbound:            0us         0us         0us         0us      0.0%
  Roundtrip:        339us    103.53ms      1.91ms       638us      0.0%
-------------------------------------------------------------------------------
                                                    Jitter Algorithm [RFC1889]
===============================================================================
```

## Command Overview

| Command | Description |
|---------|-------------|
| `controller` | Full TWAMP controller (control + sender) |
| `sender` | TWAMP light session sender |
| `responder` | TWAMP light reflector |
| `controlclient` | TWAMP control client only |
| `dscptable` | Display DSCP/QoS reference table |

## Common Options
```bash
# Send 1000 packets at 10ms intervals with DSCP EF marking
twampy sender 192.168.1.100 --count 1000 --interval 10 --dscp ef

# Run controller with verbose logging
twampy controller 192.168.1.100 --verbose

# Start reflector on custom port
twampy responder :20000
```

## Requirements

- Python 3.11 or higher (tested with 3.11 and 3.14)
- No external dependencies (standard library only)

## Documentation

📚 **[Full Documentation](https://nokia.github.io/twampy/)** (or see [docs/](docs/))

- [Installation Guide](docs/user-guide/installation.md) - Detailed installation instructions
- [Usage Guide](docs/user-guide/usage.md) - Examples and use cases
- [Configuration Reference](docs/user-guide/configuration.md) - All parameters explained
- [Error Codes](docs/reference/error-codes.md) - Protocol error reference
- [DSCP Table](docs/reference/dscp-table.md) - QoS values reference

## Platform Support

| Platform | Status | Notes |
|----------|:------:|-------|
| Linux | ✅ Full | All features supported |
| Windows | ✅ Full | Requires registry mod for DSCP |
| macOS | ⚠️ Partial | DF flag not supported |
| FreeBSD | ⚠️ Partial | DF flag not supported |

## Limitations

- Software timestamping only (typical accuracy: 100μs - 2ms)
- Unauthenticated mode only (no encryption/authentication)


twampy can be imported and used in Python programs:

```python
import twampy

# Your code here (API documentation coming soon)
```

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

## License

Licensed under the BSD-3-Clause License. See [LICENSE](LICENSE) for details.

The original code is copyright Nokia. The fork changes listed in [About this fork](#about-this-fork) are copyright phr0stb1t3 and use the same BSD-3-Clause license.
The Nokia copyright notice, the license conditions and the disclaimer are unchanged.
The use of the Nokia name in this README identifies the upstream project only and is not an endorsement.

## Author

**Sven Wisotzky** - Nokia (original author)

Fork changes: phr0stb1t3

## References

- [RFC 5357 - TWAMP](https://tools.ietf.org/html/rfc5357)
- [RFC 8762 - STAMP](https://tools.ietf.org/html/rfc8762)
- [RFC 8972 - STAMP Optional Extensions (Session ID)](https://tools.ietf.org/html/rfc8972)
- [RFC 4656 - OWAMP](https://tools.ietf.org/html/rfc4656)
- [RFC 1889 - RTP (Jitter Calculation)](https://tools.ietf.org/html/rfc1889)