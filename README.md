# circadian

Rust daemon to manage the sleep/wake state of remote machines.

## Wake

The basic idea is to send WOL packets to the daemon. The tricky bit is figuring out when to send them. There are 2 options:

* IP Address
* MAC Address

IP Address is not ideal, since it relies on the system ARP cache having an entry. This may be timed out if the remote machine has been asleep for a while.

MAC address should work well. The main issue is that managing MAC addresses is a massive pain. The ideal interface is to provide a host name and automatically figure out the MAC address.

IP addresses are easy to figure out (via DNS). So, the daemon needs to maintain a cache between (IP -> MAC). This can just be periodically merged from the system ARP cache. Making the system ARP cache persistent may also work, but has a larger blast radius of it breaks.

If these are ephemeral devices (e.g. behind tailscale), then DNS -> IP might also not be reliable. So the daemon should maintain a DNS -> IP cache as well.

The DNS -> IP cache can be pruned on a timer. The IP -> MAC entries can be pruned every time a IP is no longer in the DNS -> IP cache.

This gets rid of all problems except seeding the MAC cache, if the first action on a previously unseen IP address is on a sleeping device. This should be rare enough to just spit out a good error message.

## Sleep

Sleep is simpler, but also more annoying. There is no default unauthenticated way to do it. Can take a similar approach to https://github.com/SR-G/sleep-on-lan or provision SSH access for circadian.

The other complication is platform-specifics (if SSH is used). Different OSes will have different commands and decide which to use after SSH.
