# SepaMan

## Installation

Copy `sepaman` to `/usr/local/sbin` and create the following symlinks:

```shell
SEPAMAN=/usr/local/bin/sepaman
ln -s "$SEPAMAN" /etc/network/if-pre-up.d/
ln -s "$SEPAMAN" /etc/network/if-up.d/
ln -s "$SEPAMAN" /etc/network/if-down.d/
ln -s "$SEPAMAN" /etc/network/if-post-down.d/
```

Then create `/etc/default/sepaman` and set the value according to your needs. The following example values are used as defaults if omitted.

```shell
# System-wide SepaMan settings

# Priority for routing rules like `from <addr> table <table>`
SEPA_FROMADDR_PRIORITY=3

# Priority for routing rules like `from all oif <iface> table <table>`
SEPA_OIF_PRIORITY=3

# Priority for routing rules like `from all fwmark <mark>/<mask> table <table>`
SEPA_FWMARK_PRIORITY=4

# Firewall mark mask
SEPA_FWMARK_MASK=0xFFFFFFFF

# The iptables chain in "mangle" table to append rules into. It's recommended to use a custom chain than the default.
# An empty string or the special value "none" (without quotes) disables this feature.
# Note that it is not possible to use a chain named "none"
# Example: iptables -t mangle -A "$SEPA_FWMARK_CHAIN" -i "$IFACE" -j MARK --set-xmark "$SEPA_FWMARK/$SEPA_FWMARK_MASK"
SEPA_FWMARK_CHAIN=PREROUTING
```

## Usage

Specify the following configurations for an interface you want to configure with SepaMan. Note that you should comment out options that you don't use, since ifupdown doesn't allow options without arguments.

```conf
auto Example
iface Example inet static
    address 192.0.2.1/32

    # Interface type. Used in `ip tunnel add` or `ip link add`
    # The whole interface is ignored by SepaMan if this option is missing.
    sepa-type # required, "gre" or "wg"

    # Routing table. Creates routing rules for this table.
    # Used in `ip route add table` and `ip rule add table`
    # It's recommended to omit this option and use the default value,
    #   and edit /etc/iproute2/rt_tables instead.
    sepa-table # optional, defaults to `$IFACE`
    
    # Firewall Mark. Creates fwmark-based routing rules and "mangle" iptables rules
    sepa-fwmark # optional, no rules are created if omitted
    sepa-fwmark-mask # optional, overrides the system-wide setting

    # iptables chain for firewall mark
    sepa-fwmark-chain # overrides the system-wide setting. Special value "none" is accepted

    ############################
    # GRE-specific configuration
    sepa-local 192.0.2.2
    sepa-remote 192.0.2.3
    sepa-gre-key # optional, "unkeyed GRE" is used when omitted
    sepa-gre-ttl # optional

    ##################################
    # WireGuard-specific configuration
    # Does not yet support multiple peers for the same interface
    sepa-wg-port # optional, listen-port for WireGuard
    sepa-wg-fwmark # optional, as in `wg set fwmark`
    sepa-wg-private-key /etc/wireguard/example.key
    sepa-wg-peer # required, base64-encoded public key of the peer
    sepa-wg-preshared-key # optional
    sepa-wg-endpoint # optional
    sepa-wg-persistent-keepalive # optional
    #sepa-wg-allowed-ips # NOT IMPLEMENTED, 0.0.0.0/0 and ::/0 are used
```

## License

The MIT License
