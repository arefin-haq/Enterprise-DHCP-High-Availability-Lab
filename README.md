```text

           ENTERPRISE DHCP HIGH AVAILABILITY (FAILOVER) LAB

![Network Diagram](https://raw.githubusercontent.com/arefin-haq/Enterprise-DHCP-High-Availability-Lab/main/diagrams/Enterprise_DHCP.png)
OVERVIEW
This laboratory project focuses on building a robust and resilient 
DHCP environment using RHEL 9.8. By deploying a pair of DHCP 
servers configured in a failover relationship, we ensure continuous 
IP address availability for network clients. This architecture 
effectively eliminates single points of failure, providing automated 
load balancing and seamless state synchronization between peer nodes.

OBJECTIVE
- Implement DHCP Failover protocol (Primary/Secondary roles).
- Achieve network redundancy to prevent service outages.
- Distribute DHCP pools efficiently across two servers.
- Automate state synchronisation and client lease management.
- Validate system reliability through simulated failure testing.

1. GLOBAL SETTINGS (Applied to both servers)
   ddns-update-style none;
   authoritative;
   default-lease-time 600;
   max-lease-time 7200;

2. SYSTEM TIME SYNCHRONIZATION
   Time accuracy is critical for server state consistency.

   # Primary Server Status:
   Reference ID    : 70D233D0 (112.210.51.208.pldt.net)
   System time     : 0.001366893 seconds slow
   Leap status     : Normal

   # Secondary Server Status:
   Reference ID    : 70D233D0 (112.210.51.208.pldt.net)
   System time     : 0.001103954 seconds slow
   Leap status     : Normal

3. DHCP SERVER CONFIGURATIONS
   --- PRIMARY SERVER ---
   failover peer "dhcp-failover" {
     primary;
     address 192.168.100.242;
     port 647;
     peer address 192.168.100.243;
     peer port 647;
     max-response-delay 60;
     max-unacked-updates 10;
     mclt 3600;
     split 128;
     load balance max seconds 3;
   }

   --- SECONDARY SERVER ---
   failover peer "dhcp-failover" {
     secondary;
     address 192.168.100.243;
     port 647;
     peer address 192.168.100.242;
     peer port 647;
     max-response-delay 60;
     max-unacked-updates 10;
     load balance max seconds 3;
   }

4. SUBNET SETUP (Common)
   subnet 192.168.100.0 netmask 255.255.255.0 {
     pool {
       failover peer "dhcp-failover";
       range 192.168.100.100 192.168.100.200;
       option routers 192.168.100.1;
       option domain-name-servers 1.1.1.3,1.0.0.3;
     }
   }

5. FIREWALL CONFIGURATION
   - Commands:
     firewall-cmd --permanent --add-service=dhcp
     firewall-cmd --permanent --add-port=647/tcp
     firewall-cmd --reload

6. LOG MONITORING & TROUBLESHOOTING
   - Real-time logs: journalctl -u dhcpd -f
   - Check Syntax: dhcpd -t -cf /etc/dhcp/dhcpd.conf
   - Check Status: systemctl status dhcpd
   - Test Connection:
     telnet [PEER_IP] 647 (or)
     nc -zv [PEER_IP] 647
   - View Leases: cat /var/lib/dhcpd/dhcpd.leases

7. TESTING & VALIDATION
   - Failover Test: Secondary correctly takes over when Primary is down.
   - Recovery: Automatic synchronisation returns to 'Both servers normal'.
   
   * Note: The architecture design presented in this lab is an original 
     work, conceptualised and implemented independently, without any 
     external assistance.
======================================================================
