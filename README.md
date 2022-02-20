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

Then create `/etc/default/sepaman` and set the value according to your needs. The following example values are reasonable defaults if you want to start with. **Note that all settings must be present.**

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
SEPA_FWMARK_CHAIN=none

# Load IP XFRM policies and states using SPI ID and keys from this directory
SEPA_IPSEC_KEY_DIR=/etc/ipsec/keys.d

# Create key files if they don't exist
SEPA_IPSEC_KEY_CREATE=true
```

## Usage

Specify the following configurations for an interface you want to configure with SepaMan. Note that you should comment out options that you don't use, since ifupdown doesn't allow options without arguments.

```conf
auto Example
iface Example inet static
    address 192.0.2.1/32

    # Interface type. Used in `ip tunnel add` or `ip link add`
    # The whole interface is ignored by SepaMan if this option is missing.
    sepa-type common # required, only "common" is supported at present

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

    # Override default routing rule priorities from /etc/default/sepaman.
    # All keys are optional (since there are default values)
    sepa-fromaddr-prio
    sepa-oif-prio
    sepa-fwmark-prio

    # Load IPsec Security Policies and Associations
    # IPsec is optional and is applied only when sufficient parameters have been given

    # You can either:
    # 1. Use explicit source and destination addresses
    sepa-ipsec-local 192.0.2.1
    sepa-ipsec-remote 192.0.2.2
    # 2. Or, use addresses from the tunnel interface
    sepa-ipsec-auto tunnel
```

## License

The MIT License
