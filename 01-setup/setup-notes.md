# Setup Notes

## Network Configuration
- Rede escolhida: VirtualBox Host-Only Ethernet Adapter
- Range de IP: 192.168.56.0/24
- DHCP Server: desativado (IPs atribuídos manualmente a cada VM)

Cada VM vai ter 2 adaptadores de rede:
- Adapter 1: NAT (acesso à internet, para updates e downloads)
- Adapter 2: Host-only (192.168.56.0/24) — comunicação entre VMs e simulação de ataques

![host-only network](screenshots/01-hostonly-network-created.png)
## Ubuntu Server SIEM — Installation
- VM criada: Ubuntu-Server-SIEM
- Ubuntu 26.04 LTS
- RAM: 2048 MB | CPUs: 2 | Disco: 20GB
- Guest Additions: não instaladas (script de instalação automática falhou com erro em vboxpostinstall.sh; reinstalação sem essa opção resolveu)
- Rede: Adapter 1 (NAT) + Adapter 2 (Host-only, 192.168.56.0/24)
- Login confirmado com sucesso, IP atribuído via NAT: 10.0.2.15 (Adapter 1)

![ubuntu server login success](screenshots/02-ubuntu-server-login-success.png)

## Static IP Configuration (Host-only Interface)
- Interface configurada: enp0s8
- IP fixo atribuído: 192.168.56.10/24
- Configuração feita via Netplan (/etc/netplan/50-cloud-init.yaml)
- Permissões do ficheiro netplan ajustadas com `chmod 600` (recomendação do próprio Netplan, ficheiro não deve ser acessível por outros)
- Convenção de IPs definida para o lab: .10 = Ubuntu Server (SIEM), .20 = Kali, .30 = Metasploitable2, .40 = Windows 10

![static ip configured](screenshots/03-ubuntu-static-ip-configured.png)
