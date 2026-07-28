# Progress Log

## Dia 1 — 25/07
- Criei e configurei a rede host-only (192.168.56.0/24), DHCP desativado
- Defini que cada VM vai ter 2 adaptadores: NAT (internet) + host-only (comunicação entre VMs)
- Criei a VM Ubuntu-Server-SIEM (2048MB RAM, 2 CPUs, 20GB disco)
## Dia 2 — 28/07
- Configurei IP fixo (192.168.56.10) na interface host-only via Netplan
- Transferi o instalador do Splunk para a VM via SCP
- Instalei o Splunk (.deb)
- Criei utilizador dedicado (siem) para correr o Splunk sem privilégios root
- Configurei acesso SSH à VM, para facilitar transferências e gestão remota
- Splunk a correr, interface web acessível e confirmada via browser
