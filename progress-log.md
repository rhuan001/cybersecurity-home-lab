# Progress Log

## Dia 1 — 25/07
- Criei e configurei a rede host-only (192.168.56.0/24), DHCP desativado
- Defini que cada VM vai ter 2 adaptadores: NAT (internet) + host-only (comunicação entre VMs)
- Criei a VM Ubuntu-Server-SIEM (2048MB RAM, 2 CPUs, 20GB disco)
## Dia 2 — 28/07
- Configurei IP fixo (192.168.56.10) na interface host-only via Netplan
- Configurei acesso SSH à VM, para facilitar transferências e gestão remota
- Transferi o instalador do Splunk para a VM via SCP
- Instalei o Splunk (.deb)
- Criei utilizador dedicado (siem) para correr o Splunk sem privilégios root
- Splunk a correr, interface web acessível e confirmada via browser
## Dia — 25/08/2026
- Configurei o Metasploitable2 como máquina alvo vulnerável do laboratório.
- Mantive apenas o Adapter Host-only ativo, isolando a VM da Internet.
- Configurei o IP estático `192.168.56.30/24` na interface `eth0`.
- Confirmei o Kali Linux com o IP `192.168.56.20/24` na rede Host-only.
- Testei a conectividade Kali → Metasploitable2 através de `ping`.
- Resultado: 4 packets transmitted, 4 received, 0% packet loss.
- Instalei e configurei a VM Windows 10.
- Configurei o Adapter 1 como NAT e o Adapter 2 como Host-only.
- Configurei o IP estático `192.168.56.50/24` na interface Host-only.
- Adicionei uma regra ICMPv4 no Windows Firewall para permitir testes de `ping`.
- Confirmei a comunicação entre Kali Linux e Windows 10 através da rede Host-only.
## Dia — 27/08/2026
- Configurei o Splunk Enterprise para receber dados de forwarders na porta TCP `9997`.
- Instalei o Splunk Universal Forwarder no Windows 10.
- Configurei o Receiving Indexer para `192.168.56.10:9997`.
- Criei o ficheiro `inputs.conf` para recolher os logs `Application`, `Security` e `System`.
- Confirmei o serviço `SplunkForwarder` em estado `RUNNING`.
- Validei a comunicação entre o Windows 10 e o Splunk Enterprise.
- Confirmei a ingestão dos Windows Event Logs no Splunk Web através da pesquisa `index=* sourcetype="WinEventLog:*"`.
- Configurei a recolha de logs Linux no Splunk.
- Adicionei o utilizador `siem` ao grupo `adm` para permitir acesso aos logs do sistema.
- Configurei a ingestão de `/var/log/auth.log` e `/var/log/syslog`.
- Validei com sucesso os sourcetypes `linux_secure` e `syslog` no Splunk Web.
