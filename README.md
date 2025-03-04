# Free Range Routing (FRR) Exporter

Prometheus exporter for FRR version 3.0+ that collects metrics by using `vtysh` and exposes them via HTTP, ready for collecting by Prometheus.

## Getting Started
To run frr_exporter:
```
./frr_exporter [flags]
```

To view metrics on the default port (9342) and path (/metrics):
```
http://device:9342/metrics
```

To view available flags:
```
usage: frr_exporter [<flags>]

Flags:
  -h, --help                Show context-sensitive help (also try --help-long and --help-man).
      --collector.bgp.peer-types
                            Enable scraping of BGP peer types from peer descriptions (default:
                            disabled).
      --web.listen-address=":9342"
                            Address on which to expose metrics and web interface.
      --web.telemetry-path="/metrics"
                            Path under which to expose metrics.
      --frr.vtysh.path="/usr/bin/vtysh"
                            Path of vtysh.
      --collector.bgp       Collect BGP Metrics (default: enabled).
      --collector.ospf      Collect OSPF Metrics (default: enabled).
      --collector.bgp6      Collect BGP IPv6 Metrics (default: enabled).
      --collector.bgpl2vpn  Collect BGP L2VPN Metrics (default: disabled).
      --log.level="info"    Only log messages with the given severity or above. Valid levels: [debug,
                            info, warn, error, fatal]
      --log.format="logger:stderr"
                            Set the log target and format. Example: "logger:syslog?appname=bob&local=7"
                            or "logger:stdout?json=true"
      --version             Show application version.
```

Promethues configuraiton:
```
scrape_configs:
  - job_name: frr
    static_configs:
      - targets:
        - device1:9342
        - device2:9342
    relabel_configs:
      - source_labels: [__address__]
        regex: "(.*):\d+"
        target: instance
```

## Collectors
To disable a default collector, use the `--no-collector.$name` flag, or
`--collector.$name` to enable it.

### Enabled by Default
Name | Description
--- | ---
BGP | Per VRF and address family (currently support unicast only) BGP metrics:<br> - RIB entries<br> - RIB memory usage<br> - Configured peer count<br> - Peer memory usage<br> - Configure peer group count<br> - Peer group memory usage<br> - Peer messages in<br> - Peer messages out<br> - Peer active prfixes<br> - Peer state (established/down)<br> - Peer uptime
BGP IPv6 | Per VRF and address family (currently support unicast only) BGP IPv6 metrics:<br> - RIB entries<br> - RIB memory usage<br> - Configured peer count<br> - Peer memory usage<br> - Configure peer group count<br> - Peer group memory usage<br> - Peer messages in<br> - Peer messages out<br> - Peer active prfixes<br> - Peer state (established/down)<br> - Peer uptime
BGP L2VPN | Per VRF and address family (currently support EVPN only) BGP L2VPN EVPN metrics:<br> - RIB entries<br> - RIB memory usage<br> - Configured peer count<br> - Peer memory usage<br> - Configure peer group count<br> - Peer group memory usage<br> - Peer messages in<br> - Peer messages out<br> - Peer active prfixes<br> - Peer state (established/down)<br> - Peer uptime
OSPFv4 | Per VRF OSPF metrics:<br> - Neighbors<br> - Neighbor adjacencies

### BGP: frr_bgp_peer_types_up
FRR Exporter exposes a special metric, `frr_bgp_peer_types_up`, that can be used in scenarios where you want to create Prometheus queries that can report on the number of types of BGP peers that are currently established, such as for Alert Manager. To implement this metric, a JSON formatted description with a 'type' element must be configured on your BGP group. FRR Exporter will then aggregate all BGP peers that are currently established and configured with that type.

For example, if you want to know how many BGP peers are currently established that provide internet, you'd set the description of all BGP groups that provide internet to `{"type":"internet"}` and query Prometheus with `frr_bgp_peer_types_up{type="internet"})`. Going further, if you want to create an alert when the number of established BGP peers that provide internet is 1 or less, you'd use `sum(frr_bgp_peer_types_up{type="internet"}) <= 1`.

To enable `frr_bgp_peer_types_up`, use the `--collector.bgp.peer-types` flag. 

## Development
### Building
```
go get github.com/tynany/frr_exporter
cd ${GOPATH}/src/github.com/prometheus/frr_exporter
go build
```

## TODO
 - Collector and main tests
 - OSPF6
 - ISIS
 - Additional BGP SAFI
 - Feel free to submit a new feature request
