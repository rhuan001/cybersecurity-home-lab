# Scenario 03 — Incident Response: SMB Authentication Attack

## 1. Incident Summary

Foram detetados dois eventos de falha de autenticação (`4625`) seguidos de um evento de autenticação bem-sucedida (`4624`) no Windows 10 (`192.168.56.50`, hostname `P1`), com origem no host `192.168.56.20`, através do serviço SMB. A autenticação bem-sucedida foi realizada com a conta `vboxuser`.

O padrão observado é consistente com tentativas de autenticação remota contra o serviço SMB. Segundo `03-exploitation/smb-auth-attack.md`, as duas falhas corresponderam a uma tentativa com password incorreta e outra com username incorreto.

---

## 2. Timeline

| Time | Event |
|------|-------|
| 29/09/2026 18:14:42 | `4625` event (Failed Logon) from `192.168.56.20` |
| 29/09/2026 18:14:48 | `4625` event (Failed Logon) from `192.168.56.20` |
| 29/09/2026 18:15:05 | `4624` event (Successful Logon) for account `vboxuser`, `Logon Type: 3` (Network), from `192.168.56.20` |

> Horas apresentadas no Splunk. O evento bruto regista a hora local do `P1` (por exemplo `07:15:05 PM` para o evento `4624`), uma hora depois da hora apresentada.

---

## 3. Affected Systems and Accounts

- **Target system:** Windows 10 — `192.168.56.50` (hostname `P1`)
- **Service:** SMB — `445/TCP`
- **Account involved:** `vboxuser`
- **Attack source:** `192.168.56.20` (workstation `KALI`)
- **Authentication protocol:** NTLM

---

## 4. Evidence

- **Authentication events:** pesquisa dos eventos `4625` e `4624` no host Windows, sourcetype `WinEventLog:Security`

```spl
host="P1" sourcetype="WinEventLog:Security" (EventCode=4625 OR EventCode=4624)
```

Resultado: a pesquisa devolveu 142 eventos nas últimas 24 horas. Entre a atividade normal do sistema, três eventos correspondem às tentativas realizadas a partir do Kali (`18:14:42` — `4625`, `18:14:48` — `4625`, `18:15:05` — `4624`).

![SMB Authentication Events](../04-detection/screenshots/6-splunk-smb-auth-events.png)

- **Successful logon investigation:** pesquisa do evento `4624` associado à conta `vboxuser`

```spl
host="P1" sourcetype="WinEventLog:Security" EventCode=4624 Account_Name="vboxuser"
```

Campos confirmados no evento das `18:15:05`:

```text
Logon Type:               3 (Network)
Account Name:             vboxuser
Account Domain:           P1
Workstation Name:         KALI
Source Network Address:   192.168.56.20
Logon Process:            NtLmSsp
Authentication Package:   NTLM
Package Name (NTLM only): NTLM V2
```

![SMB Successful Logon Details](../04-detection/screenshots/7-splunk-smb-successful-logon-details.png)

Evidence flow:

```text
  2 x 4625 events (192.168.56.20, 18:14:42 and 18:14:48)
              │
              ▼
  1 x 4624 event (vboxuser, Logon Type 3, NTLM V2, 18:15:05)
```

Evidence documented in `03-exploitation/smb-auth-attack.md` and `04-detection/smb-auth-detection.md`.

---

## 5. Impact

A evidência observada mostra que, após duas falhas de autenticação, ocorreu uma autenticação SMB bem-sucedida com a conta `vboxuser` a partir de `192.168.56.20`, através da rede e com NTLM.

Durante o teste foi confirmada a autenticação e a enumeração das partilhas disponíveis (`ADMIN$`, `C$`, `IPC$`), mas não foi verificado o acesso ao conteúdo das partilhas. Também não foi testado se existe account lockout ou rate limiting, dado o número reduzido de tentativas.

Num ambiente real, uma autenticação SMB bem-sucedida deste tipo poderia permitir acesso a partilhas de ficheiros e, dependendo dos privilégios da conta, acesso a recursos administrativos do sistema. O uso de NTLM está associado, em ambientes que dependem deste protocolo, a técnicas como pass-the-hash e NTLM relay; estas técnicas não foram executadas neste cenário e são referidas apenas como contexto.

---

## 6. Containment (Recommended Actions)

As seguintes ações de contenção seriam aplicáveis a este tipo de incidente. Não foram executadas neste laboratório e são listadas como resposta recomendada:

- Bloquear o IP de origem (`192.168.56.20`) ao nível da firewall
- Desativar temporariamente a conta envolvida (`vboxuser`) ou repor a sua password
- Terminar as sessões SMB ativas provenientes da origem suspeita
- Rever os eventos `4624` e `4625` do host para identificar outras autenticações a partir da mesma origem

---

## 7. Remediation (Recommended Actions)

- Impor passwords fortes em todas as contas locais com acesso remoto
- Implementar uma account lockout policy após múltiplas falhas de autenticação
- Restringir o acesso à porta `445/TCP` por firewall, permitindo apenas origens necessárias
- Desativar File and Printer Sharing e Network Discovery nos sistemas onde não sejam necessários
- Restringir ou desativar o uso de NTLM, preferindo Kerberos, e ativar SMB signing
- Monitorizar eventos `4625` por IP de origem num intervalo de tempo

---

## 8. Lessons Learned

Os Windows Security Event Logs permitiram identificar o padrão de falhas seguidas de um login bem-sucedido. O campo `Logon Type: 3` mostrou-se útil para isolar autenticações vindas da rede da restante atividade local do sistema, e os campos `Workstation Name` e `Source Network Address` permitiram atribuir a autenticação ao host de origem.

Neste cenário a identificação foi feita por pesquisa manual dos eventos `4625` e `4624`, dado o número reduzido de tentativas. Num ambiente de produção, uma detection query eficaz basear-se-ia na contagem de eventos `4625` por IP de origem num intervalo de tempo, de forma semelhante à deteção utilizada no cenário SSH, permitindo identificar padrões de password spraying ou brute-force contra o serviço SMB.
