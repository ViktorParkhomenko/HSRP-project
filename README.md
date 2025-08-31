# Hot Standby Router Protocol (HSRP)

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
- interface g0/2
- standby 10 ip 10.1.1.1
- standby 10 preempt
- standby 10 priority 110
<img width="786" height="195" alt="image" src="https://github.com/user-attachments/assets/66c7d53f-41a7-4e88-b8cb-ba4248a0141c" />


On R2 (Standby Router)
- interface g0/2
- standby 10 ip 10.1.1.1
- standby 10 preempt
 ! default priority = 100
<img width="797" height="254" alt="image" src="https://github.com/user-attachments/assets/d0f4ca2a-bcc6-4887-b5e6-f3ff4222c229" />


### Verification

- show standby brief → quick status (active/standby, priority, virtual IP).
<img width="930" height="151" alt="image" src="https://github.com/user-attachments/assets/3d0e56f1-3640-4206-b107-12c8f0a2edd6" />
<img width="919" height="147" alt="image" src="https://github.com/user-attachments/assets/72a830a5-b946-4dd1-b8da-aa9019876271" />

- show standby → detailed info (timers, MAC, authentication).
<img width="830" height="343" alt="image" src="https://github.com/user-attachments/assets/352bed16-4ce3-49b7-887f-0c0ab1acad50" />


Advanced Features
Subsecond Timers (HSRP v2)
- standby version 2
- standby timers msec 100 msec 300
<img width="637" height="108" alt="image" src="https://github.com/user-attachments/assets/a77d4f77-df54-4ac2-b7ba-9f346d1284ef" />
<img width="666" height="127" alt="image" src="https://github.com/user-attachments/assets/651194a8-f76d-4728-b6c9-4926279d08df" />


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

### Testing

Continuous pings to 1.1.1.1 showed only a few drops during failover.
<img width="889" height="217" alt="image" src="https://github.com/user-attachments/assets/e25d01b0-5d3f-47ca-8a22-cdc869622ae3" />

With interface shutdown, R2 took over as active after hold time expired.

With resign messages, failover was immediate.

Subsecond timers (v2) improved convergence speed.
<img width="864" height="83" alt="image" src="https://github.com/user-attachments/assets/8cbd3c62-ac83-4e4c-91b6-ce0d8b4fff99" />

### Best Practices

Make the HSRP active router also the STP root bridge for the VLAN.

If routers handle multiple VLANs, load balance HSRP groups (e.g., R1 active for VLANs 10–20, R2 active for VLANs 30–40).

### Conclusion

HSRP provides seamless gateway redundancy in Cisco environments. With advanced features like preemption, object tracking, authentication, and subsecond timers, it ensures high availability and quick failover for enterprise networks.
