# MITRE ATT&CK Mapping

Mapeamento das técnicas simuladas neste laboratório para o framework **MITRE ATT&CK**, com a evidence de cada uma e a detection associada.

**Reference:** MITRE ATT&CK v19 (Enterprise), consultado em attack.mitre.org a 30/09/2026. Os IDs, nomes e tactics abaixo seguem a nomenclatura dessa versão.

---

## 1. Mapping Criteria

- Foram mapeadas **apenas** as técnicas efetivamente executadas ou observadas neste laboratório. Técnicas típicas deste tipo de ataque, mas não executadas, **não** estão na tabela principal (ver secção 4).
- Cada técnica aponta para a evidence que a sustenta (documentos e screenshots do repositório).
- O ambiente é um laboratório isolado e todos os ataques foram simulados pelo próprio. O mapeamento descreve as técnicas emuladas e não atividade real.
- Quando o mapeamento é aproximado, isso está indicado na secção 6.

---

## 2. Executed Techniques

| Phase | Technique | ID | Tactic | Tool Used | Evidence |
|---|---|---|---|---|---|
| Reconnaissance | Remote System Discovery | `T1018` | Discovery | `nmap -sn`, Wireshark | [`02-reconnaissance/commands.md`](02-reconnaissance/commands.md) — Host Discovery (screenshots 11 and 12) |
| Reconnaissance | Network Service Discovery | `T1046` | Discovery | `nmap -sS`, `nmap -sV`, `nmap -sC -sV` | [`02-reconnaissance/commands.md`](02-reconnaissance/commands.md) — SYN Scan, Service Detection and Detailed Enumeration (screenshots 13 to 15) |
| Reconnaissance | Account Discovery: Local Account | `T1087.001` | Discovery | `nmap --script smb-enum-users` | [`02-reconnaissance/commands.md`](02-reconnaissance/commands.md) — SMB Enumeration (screenshot 18) |
| Reconnaissance | Network Share Discovery | `T1135` | Discovery | `nmap --script smb-enum-shares`, `smbclient -N -L` | [`02-reconnaissance/commands.md`](02-reconnaissance/commands.md) — SMB Enumeration and Manual SMB Validation (screenshots 19 and 21) |
| Scenario 1 — SSH | Brute Force: Password Guessing | `T1110.001` | Credential Access | Medusa (Hydra failed: MAC algorithm incompatibility) | [`03-exploitation/ssh-dictionary-attack.md`](03-exploitation/ssh-dictionary-attack.md); 7 `Failed password` events in [`04-detection/ssh-detection.md`](04-detection/ssh-detection.md) |
| Scenario 1 — SSH | Valid Accounts: Local Accounts | `T1078.003` | Initial Access | Medusa (`msfadmin` credential found) | `Accepted password` event for `msfadmin` at 22:25:39 in [`04-detection/ssh-detection.md`](04-detection/ssh-detection.md) |
| Scenario 2 — DVWA | Exploit Public-Facing Application | `T1190` | Initial Access | Burp Suite (Proxy and Repeater) | [`03-exploitation/dvwa-sql-injection.md`](03-exploitation/dvwa-sql-injection.md): 5 users returned; 4660-byte response in [`04-detection/dvwa-sql-injection-detection.md`](04-detection/dvwa-sql-injection-detection.md) |
| Scenario 3 — SMB | Valid Accounts: Local Accounts | `T1078.003` | Initial Access | `smbclient` | `4624` event (account `vboxuser`, `Logon Type: 3`, NTLM) in [`04-detection/smb-auth-detection.md`](04-detection/smb-auth-detection.md) |
| Scenario 3 — SMB | Network Share Discovery | `T1135` | Discovery | `smbclient -L` | [`03-exploitation/smb-auth-attack.md`](03-exploitation/smb-auth-attack.md): shares `ADMIN$`, `C$` and `IPC$` listed (screenshot `3-smb-auth-attempts-kali.png`) |

---

## 3. Coverage by Tactic

| Tactic | Techniques |
|---|---|
| Discovery | `T1018`, `T1046`, `T1087.001`, `T1135` |
| Credential Access | `T1110.001` |
| Initial Access | `T1190`, `T1078.003` |

---

## 4. Non-Executed Techniques (Context Only)

Estas técnicas são referidas nos documentos do laboratório como risco associado, mas **não foram executadas** e por isso não fazem parte do mapeamento principal.

| Technique | ID | Referenced In | Why Not Mapped |
|---|---|---|---|
| Pass the Hash | `T1550.002` | Scenario 3 IR and exploitation docs (NTLM context) | Not executed |
| Adversary-in-the-Middle: Name Resolution Poisoning and SMB Relay | `T1557.001` | Reconnaissance Finding 02 (SMB signing disabled) and Scenario 3 IR | Not executed |
| Remote Services: SMB/Windows Admin Shares | `T1021.002` | — | A autenticação SMB com conta válida é compatível com esta técnica, mas só foram listadas as partilhas, sem ações como o utilizador autenticado nem acesso ao conteúdo |

Uma injeção do tipo UNION-based no DVWA, que não foi testada, continuaria a corresponder a `T1190`.

---

## 5. Associated Detection

| Technique | Splunk Detection | Log Source | Document |
|---|---|---|---|
| `T1110.001` (SSH) | Count of `Failed password` per IP (5 or more in 2 minutes): 7 events from `192.168.56.20` | `metasploitable_auth` | [`04-detection/ssh-detection.md`](04-detection/ssh-detection.md) |
| `T1078.003` (SSH) | Search for `Accepted password` from the identified IP | `metasploitable_auth` | [`04-detection/ssh-detection.md`](04-detection/ssh-detection.md) |
| `T1190` | Search for `%27` (6 events), with exploitation identified by response size | `metasploitable_apache_live` | [`04-detection/dvwa-sql-injection-detection.md`](04-detection/dvwa-sql-injection-detection.md) |
| `T1078.003` (SMB) | Manual search for `4624` (success) events, analysed together with the `4625` (failure) events from the same source | `WinEventLog:Security` | [`04-detection/smb-auth-detection.md`](04-detection/smb-auth-detection.md) |

**Techniques without a Splunk detection:** `T1018`, `T1046`, `T1087.001` e `T1135`. O tráfego ARP do host discovery foi observado no Wireshark, mas não foi criada nenhuma detection para a atividade de discovery. Esta é uma lacuna identificada para evolução do laboratório.

---

## 6. Notes and Limitations

- **`T1190` (approximate mapping):** a técnica descreve a exploração de sistemas expostos à Internet. O DVWA está numa rede interna isolada, pelo que este é o mapeamento mais próximo e não um mapeamento exato para a exploração de uma aplicação web vulnerável.
- **`T1110.001` in Scenario 1:** a wordlist tinha 25 passwords da RockYou e a credencial válida conhecida do laboratório na última posição. A técnica foi simulada, não descoberta por tentativa cega.
- **Scenario 3 without `T1110.001`:** foram feitas apenas 3 tentativas (2 falhadas e 1 bem-sucedida), o que representa validação de credenciais e não um ataque de brute force. Por isso as duas falhas (eventos `4625`) não foram mapeadas como password guessing e só se mapeia o que foi demonstrado: a autenticação com uma conta válida e a listagem das partilhas.
- **`T1087.001`:** a descrição da técnica refere comandos executados no próprio sistema. Aqui a enumeração foi feita remotamente com um script do Nmap através do SMB, pelo que o mapeamento é aproximado.
- **`T1078.003`:** a técnica aparece em várias tactics no framework (Initial Access, Persistence, Privilege Escalation e Stealth). Neste laboratório só é relevante a tactic **Initial Access**, já que a atividade após o login não foi analisada.
- **Out of scope:** não foi analisada a atividade dentro das sessões autenticadas, por isso não há técnicas mapeadas nas tactics Execution, Lateral Movement ou Privilege Escalation.
