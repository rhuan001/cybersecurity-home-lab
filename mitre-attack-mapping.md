# MITRE ATT&CK Mapping

Mapeamento das técnicas simuladas neste laboratório para o framework **MITRE ATT&CK**, com a evidência de cada uma e a deteção associada.

**Referência:** MITRE ATT&CK v19 (Enterprise), consultado em attack.mitre.org a 30/09/2026. Os IDs, nomes e táticas abaixo seguem a nomenclatura dessa versão.

---

## 1. Critério do Mapeamento

- Foram mapeadas **apenas** as técnicas efetivamente executadas ou observadas neste laboratório. Técnicas típicas deste tipo de ataque, mas não executadas, **não** estão na tabela principal (ver secção 4).
- Cada técnica aponta para a evidência que a sustenta (documentos e capturas do repositório).
- O ambiente é um laboratório isolado e todos os ataques foram simulados pelo próprio. O mapeamento descreve as técnicas emuladas e não atividade real.
- Quando o mapeamento é aproximado, isso está indicado na secção 6.

---

## 2. Técnicas Executadas

| Fase | Técnica | ID | Tática | Ferramenta usada | Evidência |
|---|---|---|---|---|---|
| Reconnaissance | Remote System Discovery | `T1018` | Discovery | `nmap -sn`, Wireshark | [`02-reconnaissance/commands.md`](02-reconnaissance/commands.md) — Host Discovery (capturas 11 e 12) |
| Reconnaissance | Network Service Discovery | `T1046` | Discovery | `nmap -sS`, `nmap -sV`, `nmap -sC -sV` | [`02-reconnaissance/commands.md`](02-reconnaissance/commands.md) — SYN Scan, Service Detection e Detailed Enumeration (capturas 13 a 15) |
| Reconnaissance | Account Discovery: Local Account | `T1087.001` | Discovery | `nmap --script smb-enum-users` | [`02-reconnaissance/commands.md`](02-reconnaissance/commands.md) — SMB Enumeration (captura 18) |
| Reconnaissance | Network Share Discovery | `T1135` | Discovery | `nmap --script smb-enum-shares`, `smbclient -N -L` | [`02-reconnaissance/commands.md`](02-reconnaissance/commands.md) — SMB Enumeration e Manual SMB Validation (capturas 19 e 21) |
| Scenario 1 — SSH | Brute Force: Password Guessing | `T1110.001` | Credential Access | Medusa (o Hydra falhou por incompatibilidade de algoritmos MAC) | [`03-exploitation/ssh-dictionary-attack.md`](03-exploitation/ssh-dictionary-attack.md); 7 eventos `Failed password` em [`04-detection/ssh-detection.md`](04-detection/ssh-detection.md) |
| Scenario 1 — SSH | Valid Accounts: Local Accounts | `T1078.003` | Initial Access | Medusa (credencial `msfadmin` encontrada) | Evento `Accepted password` para `msfadmin` às 22:25:39 em [`04-detection/ssh-detection.md`](04-detection/ssh-detection.md) |
| Scenario 2 — DVWA | Exploit Public-Facing Application | `T1190` | Initial Access | Burp Suite (Proxy e Repeater) | [`03-exploitation/dvwa-sql-injection.md`](03-exploitation/dvwa-sql-injection.md): 5 utilizadores devolvidos; pedido com resposta de 4660 bytes em [`04-detection/dvwa-sql-injection-detection.md`](04-detection/dvwa-sql-injection-detection.md) |
| Scenario 3 — SMB | Valid Accounts: Local Accounts | `T1078.003` | Initial Access | `smbclient` | Evento `4624` (conta `vboxuser`, `Logon Type: 3`, NTLM) em [`04-detection/smb-auth-detection.md`](04-detection/smb-auth-detection.md) |
| Scenario 3 — SMB | Network Share Discovery | `T1135` | Discovery | `smbclient -L` | [`03-exploitation/smb-auth-attack.md`](03-exploitation/smb-auth-attack.md): partilhas `ADMIN$`, `C$` e `IPC$` listadas (captura `3-smb-auth-attempts-kali.png`) |

---

## 3. Cobertura por Tática

| Tática | Técnicas |
|---|---|
| Discovery | `T1018`, `T1046`, `T1087.001`, `T1135` |
| Credential Access | `T1110.001` |
| Initial Access | `T1190`, `T1078.003` |

---

## 4. Técnicas Não Executadas (Apenas Contexto)

Estas técnicas são referidas nos documentos do laboratório como risco associado, mas **não foram executadas** e por isso não fazem parte do mapeamento principal.

| Técnica | ID | Onde é referida | Porque não está mapeada |
|---|---|---|---|
| Pass the Hash | `T1550.002` | IR e exploração do Scenario 3, associada ao uso de NTLM | Não foi executada |
| Adversary-in-the-Middle: Name Resolution Poisoning and SMB Relay | `T1557.001` | Finding 02 de reconnaissance (SMB signing desativado) e IR do Scenario 3 | Não foi executada |
| Remote Services: SMB/Windows Admin Shares | `T1021.002` | — | A autenticação SMB com conta válida é compatível com esta técnica, mas só foram listadas as partilhas, sem ações como o utilizador autenticado nem acesso ao conteúdo |

Uma injeção do tipo UNION-based no DVWA, que não foi testada, continuaria a corresponder a `T1190`.

---

## 5. Deteção Associada

| Técnica | Deteção no Splunk | Fonte de logs | Documento |
|---|---|---|---|
| `T1110.001` (SSH) | Contagem de `Failed password` por IP (5 ou mais em 2 minutos): 7 eventos de `192.168.56.20` | `metasploitable_auth` | [`04-detection/ssh-detection.md`](04-detection/ssh-detection.md) |
| `T1078.003` (SSH) | Pesquisa de `Accepted password` a partir do IP identificado | `metasploitable_auth` | [`04-detection/ssh-detection.md`](04-detection/ssh-detection.md) |
| `T1190` | Pesquisa por `%27` (6 eventos), com identificação da exploração pelo tamanho da resposta | `metasploitable_apache_live` | [`04-detection/dvwa-sql-injection-detection.md`](04-detection/dvwa-sql-injection-detection.md) |
| `T1078.003` (SMB) | Pesquisa manual de eventos `4624` (sucesso), analisados em conjunto com os eventos `4625` (falha) da mesma origem | `WinEventLog:Security` | [`04-detection/smb-auth-detection.md`](04-detection/smb-auth-detection.md) |

**Técnicas sem deteção criada no Splunk:** `T1018`, `T1046`, `T1087.001` e `T1135`. O tráfego ARP do host discovery foi observado no Wireshark, mas não foi criada nenhuma deteção para a atividade de discovery. Esta é uma lacuna identificada para evolução do laboratório.

---

## 6. Notas e Limitações

- **`T1190` (mapeamento aproximado):** a técnica descreve a exploração de sistemas expostos à Internet. O DVWA está numa rede interna isolada, pelo que este é o mapeamento mais próximo e não um mapeamento exato para a exploração de uma aplicação web vulnerável.
- **`T1110.001` no Scenario 1:** a wordlist tinha 25 passwords da RockYou e a credencial válida conhecida do laboratório na última posição. A técnica foi simulada, não descoberta por tentativa cega.
- **Scenario 3 sem `T1110.001`:** foram feitas apenas 3 tentativas (2 falhadas e 1 bem-sucedida), o que representa validação de credenciais e não um ataque de brute force. Por isso as duas falhas (eventos `4625`) não foram mapeadas como password guessing e só se mapeia o que foi demonstrado: a autenticação com uma conta válida e a listagem das partilhas.
- **`T1087.001`:** a descrição da técnica refere comandos executados no próprio sistema. Aqui a enumeração foi feita remotamente com um script do Nmap através do SMB, pelo que o mapeamento é aproximado.
- **`T1078.003`:** a técnica aparece em várias táticas no framework (Initial Access, Persistence, Privilege Escalation e Stealth). Neste laboratório só é relevante a tática **Initial Access**, já que a atividade após o login não foi analisada.
- **Fora do âmbito:** não foi analisada a atividade dentro das sessões autenticadas, por isso não há técnicas mapeadas nas táticas Execution, Lateral Movement ou Privilege Escalation.
