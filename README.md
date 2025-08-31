# Hot Standby Router Protocol (HSRP)
- Configure HSRP for servicing the virtual IP address 10.1.1.1
- R1 = Active Router, R2 = Standby Router
- Configure routers to reclaim roles in case of a failure
- Test failover configu


In this project, I reviewed and configured HSRP (Hot Standby Router Protocol).
HSRP is a first-hop redundancy protocol (FHRP) that allows multiple routers to provide a single virtual gateway for hosts. This ensures redundancy and avoids a single point of failure in the default gateway.

<img width="907" height="752" alt="image" src="https://github.com/user-attachments/assets/d29dc77b-e96d-4830-95d4-08942f5cbfe4" />

### How HSRP Works

The PC’s default gateway points to a virtual IP address (e.g., 10.1.1.1), not tied to a physical router.

R1 (active router) forwards traffic for that virtual IP.

R2 (standby router) monitors R1 via hello messages (default every 3 sec).

If R1 fails, R2 takes over after the hold timer (default 10 sec).

Timers

Hello: 3 sec

Hold: 10 sec (must be ≥ 3 × hello)

### HSRP Characteristics

- Cisco proprietary, defined in RFC 2281.

- Two versions exist:

Version 1 Virtual MAC format: 0000.0C07.ACxx (xx = group number). Uses multicast 224.0.0.2.

Version 2 Supports more groups and subsecond timers (e.g., 300 ms failover). Uses multicast 224.0.0.102.

- Active vs Standby router terminology.

- Preempt Option (disabled by default) allows a router with higher priority to retake the active role when it recovers.
- Default Hello Interval: 3 sec.
- Default Holdtime: 10 sec.
- Version 1 Multicast address: 224.0.0.2
- Version 2 Multicast address: 224.0.0.102
- Cannot Use Interface IP Address as Virtual IP Address 


### Lab Topology:

PC default gateway: 10.1.1.1 (virtual IP).

R1 and R2 both connected to the same subnet (Gigabit 0/1).

Internet simulated as 1.1.1.1.

Configuration
On R1 (Active Router)
interface g0/1
 standby 10 ip 10.1.1.1
 standby 10 preempt
 standby 10 priority 110

On R2 (Standby Router)
interface g0/1
 standby 10 ip 10.1.1.1
 standby 10 preempt
 ! default priority = 100


Verification

show standby brief → quick status (active/standby, priority, virtual IP).

show standby → detailed info (timers, MAC, authentication).

Advanced Features
Subsecond Timers (HSRP v2)
standby version 2
standby 10 timers msec 100 msec 300

Interface Tracking (Enhanced Object Tracking)

Decrease priority if WAN interface goes down:

track 1 interface g0/2 line-protocol
interface g0/1
 standby 10 track 1 decrement 20

Route Tracking

Monitor a route in routing table:

ip route 2.2.2.0 255.255.255.0 null0
track 2 ip route 2.2.2.0/24 reachability
interface g0/1
 standby 10 track 2 decrement 20

Authentication

Prevent rogue HSRP routers:

interface g0/1
 standby 10 authentication md5 key-string $3Cr3T

Testing

Continuous pings to 1.1.1.1 showed only a few drops during failover.

With interface shutdown, R2 took over as active after hold time expired.

With resign messages, failover was immediate.

Subsecond timers (v2) improved convergence speed.

Best Practices

Make the HSRP active router also the STP root bridge for the VLAN.

If routers handle multiple VLANs, load balance HSRP groups (e.g., R1 active for VLANs 10–20, R2 active for VLANs 30–40).

Conclusion

HSRP provides seamless gateway redundancy in Cisco environments. With advanced features like preemption, object tracking, authentication, and subsecond timers, it ensures high availability and quick failover for enterprise networks.
