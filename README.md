"# Cybersecuritychallenge" 

#For Apache, enable it in firewall. If you do not do this, Apacha will not get through firewall.
#We all know the firewall shenanigans during our studies - easy fix right here.
#check first
sudo ufw app list
#Then allow
sudo ufw allow 'Apache (full / source / ...)'

#Deny ANY zone transfer, configured in named.conf.options in Bind9 (DNS resolving)
#allow-transfer { none; };

#Enables FTP connection from anywhere to server. NO ALLOW = BLOCKED BY FIREWALL = No ftp :(.
#Also enable SSH. I noticed ubuntu host is more forgiving than ubuntu server in this case - same for Apache.
iptables -I INPUT -m tcp -p tcp --dport 21 -j ACCEPT 
iptables -I INPUT -m tcp -p tcp --dport 22 -j ACCEPT


# Limit the number of incoming tcp connections
# Interface 0 incoming syn-flood protection
iptables -N syn_flood
iptables -A INPUT -p tcp --syn -j syn_flood
iptables -A syn_flood -m limit --limit 1/s --limit-burst 3 -j RETURN
iptables -A syn_flood -j DROP

#Limiting the incoming icmp ping request:
#Max 5 echo requests per minute
# iptables -t filter -A INPUT -p icmp --icmp-type echo-request -m limit --limit 5/minute --limit-burst 1 -j ACCEPT
# iptables -t filter -A INPUT -p icmp -j DROP
# iptables -t filter -A OUTPUT -p icmp --icmp-type echo-reply -j ACCEPT

#Block ALL but (security updates) - sudo apt update
iptables -A OUTPUT -m state --state NEW,RELATED,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp --dport 80 -m state --state NEW -j ACCEPT
iptables -A OUTPUT -p tcp --dport 53 -m state --state NEW -j ACCEPT
iptables -A OUTPUT -p udp --dport 53 -m state --state NEW -j ACCEPT
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A INPUT -j LOG
iptables -A OUTPUT -j LOG
iptables -P OUTPUT DROP
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -L

#Allow your own connections (loopback - localhost)
#The first rule single handedly allows all outgoing connections.
iptables -A INPUT -i lo -j ACCEPT
#The others are just there for support.
iptables -A FORWARD -o lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

#Unfortunately i cannot seem to get the last rule in place correctly.
#The rule underneath has to be the key part in order to only allow security updates - since it now allows all.
#iptables -A INPUT -i lo -j ACCEPT
#Even when combining the block rules (dport, p, state,... specified) i cannot find a work around.
#Mission failed on the third rule.
