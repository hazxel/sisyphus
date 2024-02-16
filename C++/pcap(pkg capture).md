

### open & close connection

- `pcap_open_live`
- `pcap_open_offline`
- `pcap_close`: close the connection, free related resources



### collect and process packet

- `pcap_dispatch`: return when process more than *cnt*
- `pcap_loop`: 