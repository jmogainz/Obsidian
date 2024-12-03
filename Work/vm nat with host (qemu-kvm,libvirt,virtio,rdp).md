- make sure rp_filter is not 2 or 1 for the interfaces
- make sure that ipv4 forwarding for all interfaces is set
#### PRIMARY PROBLEM:
- make sure ip_tables has a postrouting ruleset so that nat can take place
	- make sure ip address of guest is correct
	- make sure interface is spelled correctly
	- RESTART iptables daemon if no connection still and iptables are correct
	- sudo iptables -t nat -L POSTROUTING -n -v
  
  Helpful tutorial (be sure to include tpm though for windows11)
  https://www.youtube.com/watch?v=Zei8i9CpAn0

- Once the network was configured properly with iptables and sysctl settings, and virtio drivers were installed (shift+f10 and then hdwwiz.exe (use the mounted disks to get the netkvm virtio driver)), the setup worked. Then it was about making the graphics works smoothly.
  
  ### Redistribution reqs
  - Just take the qcow2 file with you (qemu disk image)
  - And the full vm config xml dump (one file)
    
    # Shared folder
    - Virtiofs -preview on windows but realtime performance

## Graphics improvement with Remmina and RDP
- install remmina
- allow on windows remote access from anywhere
- configure remmina with username, password, audio, and ip address of guest
- make sure username is actual username (jlmoo)
