
# DNS Privacy Clients :: dnsprivacy.org

> ## Excerpt
> As of release 239 systemd-resolved now supports opportunistic
DNS-over-TLS -
see   the resolved.conf man
page. The
release notes say:

---
-   [DOT](https://dnsprivacy.org/dns_privacy_clients/#dot)
    -   [Operating systems](https://dnsprivacy.org/dns_privacy_clients/#operating-systems)
    -   [Local forwarders](https://dnsprivacy.org/dns_privacy_clients/#local-forwarders)
        -   [Stubby](https://dnsprivacy.org/dns_privacy_clients/#stubby)
        -   [Unbound](https://dnsprivacy.org/dns_privacy_clients/#unbound)
        -   [Unbound/Stubby combination](https://dnsprivacy.org/dns_privacy_clients/#unboundstubby-combination)
        -   [Knot resolver](https://dnsprivacy.org/dns_privacy_clients/#knot-resolver)
        -   [Bind](https://dnsprivacy.org/dns_privacy_clients/#bind)
    -   [Mobile](https://dnsprivacy.org/dns_privacy_clients/#mobile)
    -   [Routers](https://dnsprivacy.org/dns_privacy_clients/#routers)
    -   [Browsers](https://dnsprivacy.org/dns_privacy_clients/#browsers)
    -   [Command line clients](https://dnsprivacy.org/dns_privacy_clients/#command-line-clients)
        -   [getdns](https://dnsprivacy.org/dns_privacy_clients/#getdns) 
        -   [LDNS (drill) 1.6.17](https://dnsprivacy.org/dns_privacy_clients/#ldns-drill-1617)
        -   [kdig](https://dnsprivacy.org/dns_privacy_clients/#kdig)
-   [DoH](https://dnsprivacy.org/dns_privacy_clients/#doh)
    -   [Desktop](https://dnsprivacy.org/dns_privacy_clients/#desktop)
    -   [Mobile](https://dnsprivacy.org/dns_privacy_clients/#mobile-1)
    -   [Browser](https://dnsprivacy.org/dns_privacy_clients/#browser)
-   [DoQ](https://dnsprivacy.org/dns_privacy_clients/#doq)
    -   [Desktop/mobile](https://dnsprivacy.org/dns_privacy_clients/#desktopmobile)

## DOT

### Operating systems

As of release 239 systemd-resolved [now supports opportunistic DNS-over-TLS](https://github.com/systemd/systemd/blob/master/NEWS) - see   [the resolved.conf man page](https://www.freedesktop.org/software/systemd/man/resolved.conf.html). The release notes say:

> systemd-resolved now supports DNS-over-TLS. It’s still turned off by default, use DNSOverTLS=opportunistic to turn it on in resolved.conf. We intend to make this the default as soon as couple of additional techniques for optimizing the initial latency caused by establishing a TLS/TCP connection are implemented.

However see [this ISOC article](https://www.internetsociety.org/blog/2018/12/dns-privacy-in-linux-systemd/) on some issues with this implementation.

### Local forwarders

#### Stubby

**Recommended: See the [DNS Privacy Daemon - Stubby](https://dnsprivacy.org/dns_privacy_daemon_-_stubby) web page for how to use Stubby as a local DNS Privacy stub resolver on your desktop or laptop!**

#### Unbound

Unbound can be run as a local caching forwarder, configured to use SSL upstream, however it cannot yet send several of the privacy related options (padding, ECS privacy) etc. The 1.7.1 release of Unbound supports authentication of upstream recursive resolvers using an authentication domain name (i.e. PKIX authentication) if a certificate bundle is configured. The 1.13.1 release can re-use upstream connections. An example minimal config is given below.

NOTE:

-   This uses Cloudflare for simplicity and testing purposes, **modify this to the resolver of your choice** from e.g. [the stubby config file](https://github.com/getdnsapi/stubby/blob/develop/stubby.yml.example)!
-   Update the path to the cert bundle to a locally installed cert.pem file so that connections can be authenticated. The path will depend on your OS and installation.

```
server:
    directory: "/etc/unbound"
    username: unbound
    chroot: "/etc/unbound"
    # logfile: "/etc/unbound/unbound.log"  #uncomment to use logfile.
    pidfile: "/etc/unbound/unbound.pid"
    # verbosity: 1      ## uncomment and increase to get more logging.
    # listen on local host, port 53
    interface: 127.0.0.1@53
    interface: 0::1@53
    prefetch: yes
    hide-identity: yes
    hide-version: yes
    do-not-query-localhost: no
    # specifiy a path to a local certificate bundle to authenticate connections
    tls-cert-bundle: "/etc/ssl/cert.pem"
    forward-zone:
        name: "."
        forward-addr: 1.1.1.1@853#cloudflare-dns.com
        forward-tls-upstream: yes
```

#### Unbound/Stubby combination

Some user combine Unbound (as a caching proxy with other features such as DNS Blacklisting) and Stubby (as fully featured TLS forwarder).

Or, if you want to set this up yourself, an example config for this is:

Unbound config

```
server:
  use-syslog: yes
  username: "unbound"
  directory: "/etc/unbound"
  trust-anchor-file: trusted-key.key
  root-hints: "/etc/unbound/root.hints"
  do-not-query-localhost:  no
forward-zone:
  name: "."
    forward-addr: 127.0.0.1@8053
    forward-addr: ::1@8053
```

Stubby config

```
resolution_type: GETDNS_RESOLUTION_STUB
dns_transport_list:
  - GETDNS_TRANSPORT_TLS
tls_authentication: GETDNS_AUTHENTICATION_REQUIRED
tls_query_padding_blocksize: 256
edns_client_subnet_private : 1
idle_timeout: 10000
listen_addresses:
  - 127.0.0.1@8053
  -  0::1@8053
round_robin_upstreams: 1
upstream_recursive_servers:
  ...
```

#### Knot resolver

As of the 2.0.0 release knot resolver can also forward queries over TLS!

#### Bind

Bind does not support TLS natively but can be configured to [run behind a local TLS proxy such as stunnel.](https://kb.isc.org/article/AA-01386/0/DNS-over-TLS.html)

Lars de Bruin has kindly created [a docker image](https://github.com/HyperDevil/docker-bind) which uses BIND as a caching local resolver with Stubby as a TLS forwarder.

### Mobile

| Platform | Status |
| --- | --- |
| **Android** | [Android supports DNS-over-TLS](https://android-developers.googleblog.com/2018/04/dns-over-tls-support-in-android-p.html) in the Android P Developer Preview. Also see this talkgiven by the Android developers at NDSS DNS Privacy workshop 2018: [Video](https://www.youtube.com/watch?v=9xO6kSgvtOQ&index=3&list=PL5i3urU7m33XR1GtR0UZWi4fdX01W2bOZ), [Slides](https://drive.google.com/open?id=0B7vYCtlP8e6KQ3RIYm40ZElzVHZJY0duYkZ4VUs4aFFGVzNF) |
|  | Quad 9 has an App: [Quad9 Connect](https://www.quad9.com/quad9-connect-on-google-play/) |
| **iOS** | [Work in underway on a Stubby iOS app](https://dnsdisco.com/iOS-dns-proxy-post.html), however it is currently blocked by an implementation restriction. |
|  | [Cloudflare has an app call 1.1.1.1](https://itunes.apple.com/us/app/1-1-1-1-faster-internet/id1423538627) - it does DoH by default but will also do DoT but only uses 1.1.1.1 |

### Routers

-   Set up [DNS-over-TLS forwarding on a Turris router](https://doc.turris.cz/doc/en/public/dns_knot_misc)
-   [OpenWRT (LEDE)](https://blog.cloudflare.com/dns-over-tls-for-openwrt/) 
-   [Asuswrt-Merlin](https://www.snbforums.com/threads/fork-asuswrt-merlin-374-43-lts-releases-v36ea.18914/page-398#post-426091)
    -   GitHub Repo: [https://github.com/Xentrk/Stubby-Installer-Asuswrt-Merlin](https://github.com/Xentrk/Stubby-Installer-Asuswrt-Merlin)
    -   Support Forum: [https://www.snbforums.com/threads/stubby-installer-asuswrt-merlin.49469/](https://www.snbforums.com/threads/stubby-installer-asuswrt-merlin.49469/)
    -   For information on how the settings were derived at, see the blog post at: [https://x3mtek.com/dns-over-tls-with-dnsmasq-and-stubby-on-asuswrt-merlin/](https://x3mtek.com/dns-over-tls-with-dnsmasq-and-stubby-on-asuswrt-merlin/)

### Browsers

[Tenta](https://tenta.com/) is a browser for Android that encrypts DNS queries using DNS-over-TLS

### Command line clients

If you want a DNS Privacy enabled command line tool or a library then choose from one of the following:

#### getdns 

-   [Website](https://getdnsapi.net/)
    -   getdns supports multiple features related to DNS privacy including persistent connections, strict and opportunistic privacy profiles and TLS authentication by hostname of SPKI pinset
-   **API spec:** [https://getdnsapi.net/spec.html](https://getdnsapi.net/spec.html)
-   **Source**:  [https://github.com/getdnsapi/getdns](https://github.com/getdnsapi/getdns)
    -   See the first few sections on the [DNS Privacy Daemon - Stubby](https://dnsprivacy.org/dns_privacy_daemon_-_stubby) page for instructions on how to install and build getdns as a local stub resolver with TLS support from source.
-   **API**: Use the api directly via C or any of the available [language bindings (Python, Java, nodejs, PHP)](https://github.com/getdnsapi/getdns/wiki/Language-Bindings)
-   **getdns\_query**: Use API directly, or use with the wrapper script getdns\_query (run ‘make getdns\_query’ then getdns\_query is found in the test directory):

```
    -   getdns_query @\<serverIP> -s -a -A -l T   (Pipelined TCP queries)
    -   getdns_query @\<serverIP> -s -a -A -l L   (Pipelined TLS queries)
    -   getdns_query @\<serverIP> -s -a -A -l LT  (Pipelined TLS queries with fallback to TCP)
    -   getdns_query @\~ -s -a -A -l L -m         (Pipelined TLS queries in strict mode using server hostname for authentication)
```

-   Daemon mode: see the DNS Privacy Daemon - Stubby page

#### LDNS (drill) 1.6.17

**Source**: ldns 1.6.17 source code available from this link to NLNet Labs: [ldns-1.6.7](http://www.nlnetlabs.nl/downloads/ldns/ldns-1.6.17.tar.gz)

**Patch**: Grab and apply the [patch to ldns-1.6.17](https://portal.sinodun.com/stash/projects/TDNS/repos/dns-over-tls_patches/browse/ldns-1.6.17_dns-over-tls.patch) from out git repository. Also see the notes [here](https://portal.sinodun.com/wiki/display/TDNS/DNS-over-TLS+patches#DNS-over-TLSpatches-LDNS:ldns.1.6.17_dns-over-tls.patch).

**Query**: To query this with drill use: (the IP address is used here simply to stop the server name resolution falling back to TCP because your local resolver doesn’t support DNS-over-TLS).

```
* drill -t          @<serverIP>  <query name>    (to see TCP query)
* drill -l -p1021   @<serverIP>  <query name>    (to see TLS query)
* drill -C          @<serverIP>  <query name>    (to see STARTTLS query)
* drill -C -D       @<serverIP>  <query name>    (to do a DNSSEC lookup using STARTTLS)
```

#### kdig

See [https://knot.readthedocs.io/en/stable/man\_kdig.html](https://knot.readthedocs.io/en/stable/man_kdig.html)

## DoH

### Desktop

-   Cloudflare have release two tools to provide DOH clients, see [https://developers.cloudflare.com/1.1.1.1/dns-over-https/cloudflared-proxy/](https://developers.cloudflare.com/1.1.1.1/dns-over-https/cloudflared-proxy/)
-   Frank Denis has a [dnscrypt-proxy](https://github.com/jedisct1/dnscrypt-proxy) (client proxy) that supports DoH, and there is a Windows client GUI called Simple DNSCrypt
-   _curl_ also supports DoH [https://github.com/curl/doh](https://github.com/curl/doh)
-   _kdig_  [also supports DoH since version 3.0](https://www.knot-dns.cz/docs/latest/html/man_kdig.html#examples)

### Mobile

-   There is an [Android App called ‘Intra’](https://play.google.com/store/apps/details?id=app.intra) which can be used to send all queries from the device over DOH to either Cloudflare or Google or a user configured resolver
-   [Cloudflare has an app call 1.1.1.1](https://itunes.apple.com/us/app/1-1-1-1-faster-internet/id1423538627) - it does DoH by default but will also do DoT but only uses 1.1.1.1

### Browser

-   **Firefox**
    -   Firefox 64.0 includes a configuration option where the URL of a DOH server can be specified and then all queries sent by Firefox will go to that server over DOH.
        -   It can be turned on in ‘Opportunistic mode’ via the Firefox->Preferences→Network Settings→Settings dialog (scroll to bottom to find the ‘Enable DNS-over-HTTP’ check box and URL). 
    -   Here are more details of [how it works and how to do more complex configuration e.g. strict mode](https://daniel.haxx.se/blog/2018/06/03/inside-firefoxs-doh-engine/)
    -   If you want to see the queries on the wire that Firefox is sending you need to export the master key secrets and then import them into wireshark.
        -   Documentation on the key format is here: [https://developer.mozilla.org/en-US/docs/Mozilla/Projects/NSS/Key\_Log\_Format](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/NSS/Key_Log_Format)
        -   See this [Sharkfest presentation](https://lekensteyn.nl/files/wireshark-tls-debugging-sharkfest19us.pdf) for more details (note Wireshark 3.0 supports DoH)
-   **Bromite** ([https://www.bromite.org/](https://www.bromite.org/))
    -   What is Bromite? It is a fork of Chromium ([https://www.chromium.org/](https://www.chromium.org/)): “Bromite is Chromium plus ad blocking and privacy enhancements; take back your browser! Bromite aims at providing a no-clutter browsing experience without privacy-invasive features and with the addition of a fast ad-blocking engine.” Note that at the moment Bromite is only for Android, it currently does not provide builds for desktop.
    -   In release 67.0.3396.88 Bromite has enabled the underlying DoH implementation in Chromium by exposing configuration options (via [chrome://flags](chrome://flags/)). Today the choice is either Google or Cloudflare DoH servers but it is up to the user to choose: [https://github.com/bromite/bromite/wiki/Enabling-DNS-over-HTTPS](https://github.com/bromite/bromite/wiki/Enabling-DNS-over-HTTPS)
-   **Chrome**
    -   Chrome has a full DoH implementation but the configuration for it is not exposed. However if you want to try it out use something like the following example for macOS:
        
        ```
        /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --enable-features="dns-over-https<DoHTrial" --force-fieldtrials="DoHTrial/Group1" --force-fieldtrial-params="DoHTrial.Group1:server/https%3A%2F%2Fcloudflare-dns%2Ecom%2Fdns-query/method/POST"
        ```
        

## DoQ

### Desktop/mobile

[AdGuard’s dnsproxy supports DoQ](https://github.com/AdguardTeam/dnsproxy), also see [AdGuardHome](https://github.com/AdguardTeam/AdGuardHome)
