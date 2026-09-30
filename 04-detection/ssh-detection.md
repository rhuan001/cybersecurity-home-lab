# Scenario 01 — SSH Attack Detection

## Objective

Analisar no Splunk os eventos SSH gerados durante o dictionary attack realizado com Medusa e criar uma detection query para identificar múltiplas tentativas de autenticação falhadas.

## Log Source

- **Host:** `Metasploitable2`
- **Sourcetype:** `metasploitable_auth`
- **Source:** `/var/log/metasploitable-auth.log`

As horas apresentadas neste documento estão em UTC, conforme o timestamp syslog (`+00:00`) dos eventos. Os eventos são de 22/09/2026.

## SSH Authentication Events

Comecei por pesquisar os eventos de autenticação SSH, sem filtrar pelo IP do Kali:

```spl
index=* host="Metasploitable2" sourcetype="metasploitable_auth" ("Failed password" OR "Accepted password")
```

A pesquisa devolveu 12 eventos (intervalo de tempo: All time), entre tentativas falhadas e logins bem-sucedidos. Os resultados incluíam também eventos anteriores ao ataque com Medusa.

![SSH Authentication Events](https://raw.githubusercontent.com/rhuan001/cybersecurity-home-lab/main/04-detection/screenshots/1-ssh-auth-events-splunk.png)

## Detailed SSH Log Analysis

Para analisar outras mensagens geradas pelo serviço SSH, utilizei:

```spl
index=* host="Metasploitable2" sourcetype="metasploitable_auth" "sshd"
| sort 0 _time
| table _time _raw
```

A pesquisa apresentou 32 eventos no período selecionado (última hora), incluindo:

- `Failed password`
- `pam_unix(sshd:auth): authentication failure`
- `PAM 3 more authentication failures`
- `PAM service(sshd) ignoring max retries`
- `Accepted password`
- Abertura e encerramento de sessão

Os 32 eventos não representam 32 passwords diferentes, porque uma tentativa de autenticação pode gerar várias mensagens.

Os primeiros eventos desta pesquisa (`21:57:03` e `21:57:05`, incluindo um `Failed password` para `msfadmin` a partir de `192.168.56.20`) são anteriores ao ataque com Medusa, que começa por volta das `22:24`.

![Detailed SSH Logs](https://raw.githubusercontent.com/rhuan001/cybersecurity-home-lab/main/04-detection/screenshots/2-ssh-detailed-auth-logs.png)

## SSH Brute Force Detection

Criei uma SPL query para identificar IPs com cinco ou mais eventos `Failed password` num intervalo de dois minutos:

```spl
index=* host="Metasploitable2" sourcetype="metasploitable_auth" "Failed password"
| rex "from (?<ip>\S+)"
| bin _time span=2m
| stats count by _time ip
| where count>=5
```

A query identifica automaticamente o IP de origem, sem precisar de conhecer antecipadamente o IP do atacante.

A pesquisa (intervalo de tempo: última hora) analisou 8 eventos `Failed password`. O resultado foi:

| _time | ip | count |
|---|---|---|
| 2026-09-22 22:24:00 | 192.168.56.20 | 7 |

Ou seja, **7 eventos `Failed password` provenientes de `192.168.56.20`** no intervalo de dois minutos iniciado às 22:24. O oitavo evento é consistente com a falha das `21:57:05`, anterior ao ataque, que fica isolada no seu intervalo e por isso não atinge o threshold.

![SSH Brute Force Detection](https://raw.githubusercontent.com/rhuan001/cybersecurity-home-lab/main/04-detection/screenshots/3-ssh-brute-force-detection.png)

## Successful Login Investigation

Depois de identificar o IP associado às falhas, pesquisei os logins bem-sucedidos provenientes dessa origem:

```spl
index=* host="Metasploitable2" sourcetype="metasploitable_auth" "Accepted password" "192.168.56.20"
```

O Splunk confirmou um evento `Accepted password` para o utilizador `msfadmin`, com origem em `192.168.56.20`, às 22:25:39:

```text
2026-09-22T22:25:39.794999+00:00 192.168.56.30 sshd[5271]: Accepted password for msfadmin from 192.168.56.20 port 42882 ssh2
```

![SSH Successful Login](https://raw.githubusercontent.com/rhuan001/cybersecurity-home-lab/main/04-detection/screenshots/4-ssh-successful-login.png)

## Analysis

A detection query identificou múltiplas falhas de autenticação provenientes do mesmo IP. A investigação posterior confirmou um login bem-sucedido dessa origem.

Na primeira pesquisa (captura 1) vê-se que o evento `Accepted password` (`22:25:39.794`) foi precedido por um evento `Failed password` (`22:25:39.792`) no mesmo processo `sshd[5271]` e com a mesma porta de origem (`42882`), ou seja, na mesma ligação.

O Medusa reportou 26 verificações, mas o número de eventos `Failed password` registados no Splunk não corresponde necessariamente ao número de passwords testadas.

O threshold de cinco falhas em dois minutos foi definido para este laboratório. Num ambiente real, seria necessário ajustá-lo ao comportamento normal dos utilizadores e analisar possíveis falsos positivos.
