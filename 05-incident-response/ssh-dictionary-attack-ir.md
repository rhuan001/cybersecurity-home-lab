# Scenario 01 — Incident Response: SSH Dictionary Attack

## 1. Incident Summary

Foi detetada uma série de eventos `Failed password` no serviço SSH do Metasploitable2 (`192.168.56.30`), com origem no host `192.168.56.20`, seguida de um evento `Accepted password` para a conta `msfadmin` a partir da mesma origem.

O padrão observado é consistente com um dictionary attack contra o serviço SSH.

---

## 2. Timeline

| Time (UTC) | Event |
|------------|-------|
| 22/09/2026 22:24 (2-minute window) | 7 `Failed password` events from `192.168.56.20`, identified by the detection query in Splunk |
| 22/09/2026 22:25:39 | `Accepted password` event for account `msfadmin` from `192.168.56.20` (port `42882`) |

> Horas em UTC, conforme o timestamp syslog (`+00:00`) dos eventos no Splunk.

---

## 3. Affected Systems and Accounts

- **Target system:** Metasploitable2 — `192.168.56.30`
- **Service:** SSH — `22/TCP`
- **Account involved:** `msfadmin`
- **Attack source:** `192.168.56.20`

---

## 4. Evidence

- **Authentication failures:** detection query sobre o sourcetype `metasploitable_auth` (5 ou mais eventos `Failed password` por IP num intervalo de 2 minutos)

```spl
index=* host="Metasploitable2" sourcetype="metasploitable_auth" "Failed password"
| rex "from (?<ip>\S+)"
| bin _time span=2m
| stats count by _time ip
| where count>=5
```

Resultado: 8 eventos `Failed password` na última hora, dos quais 7 provenientes de `192.168.56.20` no intervalo de 2 minutos iniciado às 22:24.

![SSH Brute Force Detection](../04-detection/screenshots/3-ssh-brute-force-detection.png)

- **Successful authentication:** pesquisa dos eventos `Accepted password` a partir do IP identificado

```spl
index=* host="Metasploitable2" sourcetype="metasploitable_auth" "Accepted password" "192.168.56.20"
```

Resultado: 1 evento `Accepted password` para o utilizador `msfadmin`, com origem em `192.168.56.20`, às 22:25:39:

```text
2026-09-22T22:25:39.794999+00:00 192.168.56.30 sshd[5271]: Accepted password for msfadmin from 192.168.56.20 port 42882 ssh2
```

![SSH Successful Login](../04-detection/screenshots/4-ssh-successful-login.png)

Evidence flow:

```text
  7 "Failed password" events (192.168.56.20, 2-minute window starting at 22:24)
              │
              ▼
  1 "Accepted password" event (msfadmin, 192.168.56.20, 22:25:39)
```

Evidence documented in `03-exploitation/ssh-dictionary-attack.md` and `04-detection/ssh-detection.md`.

---

## 5. Impact

A evidência observada mostra que, após múltiplas falhas de autenticação, ocorreu uma autenticação SSH bem-sucedida com a conta `msfadmin` a partir de `192.168.56.20`.

Neste cenário não foi analisada a atividade realizada durante a sessão. Num ambiente real, uma autenticação bem-sucedida deste tipo poderia permitir acesso remoto ao sistema e execução de comandos e, dependendo dos privilégios da conta, movimentação lateral ou escalada de privilégios.

---

## 6. Containment (Recommended Actions)

As seguintes ações de contenção seriam aplicáveis a este tipo de incidente. Não foram executadas neste laboratório e são listadas como resposta recomendada:

- Bloquear o IP de origem (`192.168.56.20`) ao nível da firewall
- Desativar temporariamente a conta envolvida (`msfadmin`)
- Terminar sessões SSH ativas provenientes da origem suspeita

---

## 7. Remediation (Recommended Actions)

- Impor uma política de passwords fortes
- Implementar uma account lockout policy após múltiplas falhas de autenticação
- Restringir o acesso SSH por IP (allowlist) ou através de VPN
- Preferir autenticação por chave em vez de password
- Considerar a utilização de ferramentas como o fail2ban para bloquear automaticamente IPs com múltiplas falhas

---

## 8. Lessons Learned

A monitorização centralizada dos logs de autenticação permitiu identificar o padrão de múltiplas falhas seguidas de uma autenticação bem-sucedida.

Uma detection query baseada na contagem de falhas por IP num intervalo de tempo mostrou-se eficaz para identificar este tipo de atividade, sem ser necessário conhecer antecipadamente o IP do atacante. O número de eventos `Failed password` no Splunk (7) não corresponde ao número de verificações reportadas pelo Medusa (26), pelo que a contagem de eventos não deve ser interpretada como o número de passwords testadas.

Num ambiente de produção, o threshold utilizado (5 falhas em 2 minutos) deveria ser ajustado ao comportamento normal dos utilizadores para reduzir falsos positivos.
