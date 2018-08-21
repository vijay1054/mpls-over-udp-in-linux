# mpls-over-udp-in-linux
Steps to configure mpls over udp tunnels in linux

Lets assume two systems A and B are connected back to back
and are assigned 192.168.1.1 and 192.168.1.2.
In this example MPLS lable 100 is used for traffic from A to B
and 200 is used for traffic from B to A.

Make sure the kernel is compiled with IPIP, MPLS and FOU support
and iproute2 uitls has support to configure MPLS and FOU (foo over UDP).

On system A:

    $ ifconfig eth0 192.168.1.1/24
    $ ip fou add port 6635 ipproto 137
    $ ip link add name fou-tun type ipip remote 192.168.1.2 local 192.168.1.1 ttl 225 encap fou encap-sport auto encap-dport 6635

    $ ifconfig fou-tun 10.1.1.1/24
    $ ip route change 10.1.1.0/24 encap mpls 100 dev fou-tun
    $ ip -f mpls route add 200 dev lo

    $ sysctl -w net.mpls.conf.fou-ipip.input=1

On System B:

    $ ifconfig eth0 192.168.1.2/24 /* eth0 is tunnel endpoint */
    $ ip fou add port 6635 ipproto 137
    $ ip link add name fou-tun type ipip remote 192.168.1.1 local 192.168.1.2 ttl 225 encap fou encap-sport auto encap-dport 6635

    $ ifconfig fou-tun 10.1.1.2/24
    $ ip route change 10.1.1.0/24 encap mpls 200 dev fou-tun
    $ ip -f mpls route add 100 dev lo

    $ sysctl -w net.mpls.conf.fou-ipip.input=1
