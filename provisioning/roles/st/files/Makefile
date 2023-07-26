#
#
# OpenSAND is an emulation testbed aiming to represent in a cost effective way a
# satellite telecommunication system for research and engineering activities.
#
#
# Copyright © 2020 TAS
#
#
# This file is part of the OpenSAND testbed.
#
#
# OpenSAND is free software : you can redistribute it and/or modify it under the
# terms of the GNU General Public License as published by the Free Software
# Foundation, either version 3 of the License, or (at your option) any later
# version.
#
# This program is distributed in the hope that it will be useful, but WITHOUT
# ANY WARRANTY, without even the implied warranty of MERCHANTABILITY or FITNESS
# FOR A PARTICULAR PURPOSE. See the GNU General Public License for more
# details.
#
# You should have received a copy of the GNU General Public License along with
# this program. If not, see http://www.gnu.org/licenses/.
#
#

# Author: Franklin SIMO <armelfrancklin.simotegueu@viveris.fr>
#         Bastien TAURAN <bastien.tauran@viveris.fr>

# Reviewer: Mattia QUADRINI <quadrini@romars.tech>
#         RomARS s.r.l. <romars.tech>

# ID of the GW in opensand
GW_ID=0
# ID of the ST in opensand
ST_ID=1
# Interface used by OpenSAND, towards SAT
EMU_IFACE=eth0
# Emulation IP of the GW
EMU_IP_GW=192.168.0.3
# Emulation IP of the SAT
EMU_IP_SAT=192.168.0.1
# Emulation IP of the ST
EMU_IP_ST=192.168.0.2
# The first two Bytes of the subnet behind Workstations (-->1st.2nd<--.3rd.4th)
LAN_WS=10.20
# The third Byte of the subnet behind Workstations (1st.2nd.-->3rd<--.4th)
LAN_UNTAG_WS=0
# The last byte identifying the WS-ST (1st.2nd.3rd.-->4th<--)
LAN_IP_WS_ST=1
# The last byte identifying the WS-GW (1st.2nd.3rd.-->4th<--)
LAN_IP_WS_GW=2
# Physical interface towards the Workstation ST
LAN_IFACE=eth1
# Name of the TAP interface on this machine
TAP_IFACE=opensand_tap
# MAC address of the GW TAP interface 
TAP_MAC_GW=00:00:00:00:00:01
# MAC address of the ST TAP interface
TAP_MAC_ST=00:00:00:00:00:02
# MAC address of the WS-GW
WS_GW_MAC=00:00:00:00:00:11
# MAC address of the WS-ST
WS_ST_MAC=00:00:00:00:00:22
# Name of the bridge on this machine
BR_IFACE=opensand_br
# IP Address of the bridge on this machine
BR_IFACE_IP=192.168.63.15
# Subnet mask
NET_DIGITS=24
# Collector?
ENABLE_COLLECTOR=false
# IP address of the collector
COLLECTOR_IP=10.10.0.142
# Logs?
ENABLE_LOCAL_LOGS=false
# Path of the folder containing logs
LOG_FOLDER=/var/log/opensand
# List of the vlan ID used. This is the third Byte of the Subnets using VLAN
VLAN_USED=10 20
# First two Bytes of the subnet using the VLAN, usually are the same of the LAN_WS
VLAN_BRIDGE_IP_BASE=10.20
# Last Byte identifying the IP of the bridge using VLAN
VLAN_BRIDGE_IP_TERM=15
# GThe default VLAN ID
VLAN_ID_DEFAULT=2

all: clean network

clean: stop
	ip link del $(TAP_IFACE) || true
	ip link del $(BR_IFACE) || true
	ip link set $(EMU_IFACE) down
	ip link set $(LAN_IFACE) down
	iptables -t mangle -F

network: clean
	ip link set $(EMU_IFACE) up
	ip link set $(LAN_IFACE) up
	sleep 0.1
	ip address flush dev $(EMU_IFACE)
	ip address flush dev $(LAN_IFACE)
	ip address replace $(EMU_IP_ST)/$(NET_DIGITS) dev $(EMU_IFACE)
	ip tuntap add mode tap $(TAP_IFACE)
	ip link set dev $(TAP_IFACE) address $(TAP_MAC_ST)
	ip link add name $(BR_IFACE) type bridge
	ip address add $(BR_IFACE_IP)/$(NET_DIGITS) dev $(BR_IFACE)
	ip link set dev $(TAP_IFACE) master $(BR_IFACE)
	ip link set dev $(LAN_IFACE) master $(BR_IFACE)
	ip link set $(BR_IFACE) type bridge vlan_filtering 1 ageing_time 0
	bridge vlan add dev $(TAP_IFACE) vid $(VLAN_ID_DEFAULT) pvid tagged master
	bridge vlan add dev $(BR_IFACE) vid $(VLAN_ID_DEFAULT) pvid untagged self
	bridge vlan add dev $(LAN_IFACE) vid $(VLAN_ID_DEFAULT) pvid untagged master
	@for VLAN in $(VLAN_USED) ; do \
		bridge vlan add dev $(TAP_IFACE) vid $$VLAN tagged master ; \
		ip link add link $(BR_IFACE) name $(BR_IFACE).$$VLAN type vlan id $$VLAN ; \
		ip address add $(VLAN_BRIDGE_IP_BASE).$$VLAN.$(VLAN_BRIDGE_IP_TERM)/$(NET_DIGITS) dev $(BR_IFACE).$$VLAN ; \
		bridge vlan add dev $(BR_IFACE) vid $$VLAN tagged self ; \
		bridge vlan add dev $(LAN_IFACE) vid $$VLAN tagged master ; \
	done
	ip link set $(BR_IFACE) up
	ip link set $(TAP_IFACE) up
	@for VLAN in $(VLAN_USED) ; do \
		ip link set $(BR_IFACE).$$VLAN up ; \
		ip neigh add $(LAN_WS).$$VLAN.$(LAN_IP_WS_ST) dev $(BR_IFACE).$$VLAN lladdr $(WS_ST_MAC); \
		ip neigh add $(LAN_WS).$$VLAN.$(LAN_IP_WS_GW) dev $(BR_IFACE).$$VLAN lladdr $(WS_GW_MAC); \
	done
	# From ST To GW
	ip neigh replace $(LAN_WS).$(LAN_UNTAG_WS).$(LAN_IP_WS_GW) dev $(BR_IFACE) lladdr $(WS_GW_MAC)
	# From GW To ST
	ip neigh replace $(LAN_WS).$(LAN_UNTAG_WS).$(LAN_IP_WS_ST) dev $(BR_IFACE) lladdr $(WS_ST_MAC)
	# Routing
	ip route add $(LAN_WS).$(LAN_UNTAG_WS).0/$(NET_DIGITS) dev $(BR_IFACE)

generate-xml:
	cp infrastructure.xml infrastructure_updated.xml
	sed -i "s|GW_ID|$(GW_ID)|g" infrastructure_updated.xml
	sed -i "s|ST_ID|$(ST_ID)|g" infrastructure_updated.xml
	sed -i "s|SAT_ID|$(SAT_ID)|g" infrastructure_updated.xml
	sed -i "s|EMU_IP_GW|$(EMU_IP_GW)|g" infrastructure_updated.xml
	sed -i "s|EMU_IP_SAT|$(EMU_IP_SAT)|g" infrastructure_updated.xml
	sed -i "s|EMU_IP_ST|$(EMU_IP_ST)|g" infrastructure_updated.xml
	sed -i "s|TAP_IFACE|$(TAP_IFACE)|g" infrastructure_updated.xml
	sed -i "s|TAP_MAC_GW|$(TAP_MAC_GW)|g" infrastructure_updated.xml
	sed -i "s|TAP_MAC_ST|$(TAP_MAC_ST)|g" infrastructure_updated.xml
	sed -i "s|WS_GW_MAC|$(WS_GW_MAC)|g" infrastructure_updated.xml
	sed -i "s|WS_ST_MAC|$(WS_ST_MAC)|g" infrastructure_updated.xml
	sed -i "s|ENABLE_COLLECTOR|$(ENABLE_COLLECTOR)|g" infrastructure_updated.xml
	sed -i "s|COLLECTOR_IP|$(COLLECTOR_IP)|g" infrastructure_updated.xml
	sed -i "s|ENABLE_LOCAL_LOGS|$(ENABLE_LOCAL_LOGS)|g" infrastructure_updated.xml
	sed -i "s|LOG_FOLDER|$(LOG_FOLDER)|g" infrastructure_updated.xml
	cp topology.xml topology_updated.xml
	sed -i "s|GW_ID|$(GW_ID)|g" topology_updated.xml
	sed -i "s|SAT_ID|$(SAT_ID)|g" topology_updated.xml

run: generate-xml
	opensand -i infrastructure_updated.xml -t topology_updated.xml -p profile.xml &

stop:
	killall -q opensand || true


.PHONY: all run network clean stop generate-xml
