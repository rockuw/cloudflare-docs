---
pcx-content-type: concept
title: Private hostnames and IPs
weight: 5
---

# Private hostnames and IPs

{{<Aside type="note" header="Traffic handling">}}

When the old replica is stopped, it will drop long-lived HTTP requests (for example, Websocket) and TCP connections (for example, SSH). UDP flows will also be dropped, as they are modeled based on timeouts. When the new replica connects, it will handle all new traffic, including new HTTP requests, TCP connections, and UDP flows.

{{</Aside>}}

Building out a private network has two primary components: the infrastructure side and the client side.

The infrastructure side is powered by Cloudflare Tunnel, which connects your infrastructure to Cloudflare — whether that be a singular application, many applications, or an entire network segment. This is made possible by running `cloudflared` in your environment to establish multiple secure, outbound-only, load-balanced links to Cloudflare.

On the client side, your end users need to be able to easily connect to Cloudflare and, more importantly, your network. This connection is handled by Cloudflare WARP. This client can be rolled out to your entire organization in just a few minutes using your in-house MDM tooling and it establishes a secure connection from your users’ devices to the Cloudflare network.

![Network diagram](/cloudflare-one/static/documentation/connections/private-ips-diagram.png)

Follow the steps below to define your internal DNS resolver with Cloudflare Zero Trust and to resolve requests to your private network using Cloudflare Tunnel.

## Prerequisites

- Cloudflare Tunnel must be properly [configured](/cloudflare-one/connections/connect-apps/configuration/) to route traffic to a private IP space.
- `cloudflared` must be connected to Cloudflare from your target private network.
- Cloudflare WARP must be installed on end-user devices to connect your users to Cloudflare.

## Enable UDP support

1.  On the [Zero Trust dashboard](https://dash.teams.cloudflare.com), navigate to **Settings** > **Network**.
1.  Scroll down to Firewall settings.
1.  Ensure the Proxy is enabled and both TCP and UDP are selected.

    ![Enable UDP](/cloudflare-one/static/secure-origin-connections/warp-to-tunnel-internal-dns/enable-udp.png)

## Create a Local Domain Fallback entry

Next, we need to create a [Local Domain Fallback](/cloudflare-one/connections/connect-devices/warp/exclude-traffic/local-domains/) entry.

1.  Remain in **Network Settings** and scroll further down to **Local Domain Fallback**.

    ![Manage Local Domains](/cloudflare-one/static/secure-origin-connections/warp-to-tunnel-internal-dns/manage-local-domain-fallback.png)

1.  Click **Manage**.

1.  Create a new Local Domain Fallback entry pointing to the internal DNS resolver. The rule in the following example instructs the WARP client to resolve all requests for `myorg.privatecorp` through an internal resolver at `10.0.0.25` rather than attempting to resolve this publicly.

![Create Local Domains](/cloudflare-one/static/secure-origin-connections/warp-to-tunnel-internal-dns/create-local-domain-fallback.png)

{{<Aside type="note">}}

While on the Network Settings page, ensure that **Split Tunnels** are configured to include traffic to private IPs and hostnames in the traffic sent by WARP to Cloudflare. For guidance on how to do that, refer to [these instructions](/cloudflare-one/connections/connect-apps/private-net/#optional-ensure-that-traffic-can-reach-your-network).

{{</Aside>}}

## Update `cloudflared`

Next, update your Cloudflare Tunnel configuration to ensure it is using QUIC as the default transport protocol. To do this, you can either set the `protocol: quic` property in your [configuration file](/cloudflare-one/connections/connect-apps/configuration/local-management/configuration-file/) or [pass the `–-protocol quic` flag](/cloudflare-one/connections/connect-apps/configuration/arguments/) directly through your CLI.

Finally, update to the latest available version (2021.12.3 as of the time of writing) of cloudflared running on your target private network.

![Update Cloudflared](/cloudflare-one/static/secure-origin-connections/warp-to-tunnel-internal-dns/update-cfd.png)

You can now resolve requests through the internal DNS server you set up in your private network.

## Test the setup

For testing, run a `dig` command for the internal DNS service:

```sh
$ dig AAAA www.myorg.privatecorp
```

The `dig` will work because `myorg.privatecorp` was configured above as a fallback domain. If you skip that step, you can still force `dig` to use your private DNS resolver:

```sh
$ dig @10.0.0.25 AAAA www.myorg.privatecorp
```

Both `dig` commands will fail if the WARP client is disabled in your end user's device.

## Troubleshooting

Use the following troubleshooting strategies if you are running into issues while configuring your private network with Cloudflare Tunnel.

- Ensure that `cloudflared` is connected to Cloudflare by visiting Access > Tunnels in the Zero Trust dashboard.

- Ensure that `cloudflared` is running with `quic` protocol (search for `Initial protocol quic` in its logs).

- Ensure that the machine where `cloudflared` is running is allowed to egress via UDP to port 7844 to talk out to Cloudflare.

- Ensure that end-user devices are enrolled into WARP by visiting https://help.teams.cloudflare.com

- Double-check the precedence of your application policies in the Gateway Network policies tab. Ensure that a more global Block or Allow policy will not supersede the application policies.

- Check the Gateway Audit Logs Network tab to see whether your UDP DNS resolutions are being allowed or blocked.

- Ensure that your Private DNS resolver is available over a routable private IP address. You can check that by trying the `dig` commands on your machine running `cloudflared`.

- Check your set up by using `dig ... +tcp` to force the DNS resolution to use TCP instead of UDP.
