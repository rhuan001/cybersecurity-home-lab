# Scenario 01 — SSH Attack Detection

## Objective

Analisar no Splunk os eventos SSH gerados durante o dictionary attack realizado com Medusa e criar uma detection query para identificar múltiplas tentativas de autenticação falhadas.

## Log Source

- **Host:** `Metasploitable2`
- **Sourcetype:** `metasploitable_auth`
- **Source:** `/var/log/metasploitable-auth.log`

## SSH Authentication Events

Comecei por pesquisar os eventos de autenticação SSH, sem filtrar pelo IP do Kali:

```spl
index=* host="Metasploitable2" sourcetype="metasploitable_auth" ("Failed password" OR "Accepted password")
```

A pesquisa mostrou tentativas falhadas e logins bem-sucedidos. Os resultados incluíam também eventos anteriores ao ataque com Medusa.

![SSH Authentication Events](https://raw.githubusercontent.com/rhuan001/cybersecurity-home-lab/main/04-detection/screenshots/34-ssh-auth-events-splunk.png)

## Detailed SSH Log Analysis

Para analisar outras mensagens geradas pelo serviço SSH, utilizei:

```spl
index=* host="Metasploitable2" sourcetype="metasploitable_auth" "sshd"
| sort 0 _time
| table _time _raw
```

A pesquisa apresentou 32 eventos no período selecionado, incluindo:

- `Failed password`
- `pam_unix(sshd:auth): authentication failure`
- `PAM 3 more authentication failures`
- `PAM service(sshd) ignoring max retries`
- `Accepted password`
- Abertura e encerramento de sessão

Os 32 eventos não representam 32 passwords diferentes, porque uma tentativa de autenticação pode gerar várias mensagens.

![Detailed SSH Logs](https://raw.githubusercontent.com/rhuan001/cybersecurity-home-lab/main/04-detection/screenshots/35-ssh-detailed-auth-logs.png)

## SSH Brute Force Detection

Criei uma SPL query para identificar IPs com cinco ou mais eventos `Failed password` num intervalo de dois minutos:

```spl
index=* host="Metasploitable2" sourcetype="metasploitable_auth" "Failed password"
| rex field=_raw "from (?<src_ip>\d{1,3}(?:\.\d{1,3}){3})"
| bin _time span=2m
| stats count AS failed_attempts BY _time src_ip
| where failed_attempts >= 5
| sort - _time
```

A query identifica automaticamente o IP de origem, sem precisar de conhecer antecipadamente o IP do atacante.

O resultado mostrou **7 eventos `Failed password` provenientes de `192.168.56.20`** no intervalo de dois minutos iniciado às 22:24.

![SSH Brute Force Detection](https://raw.githubusercontent.com/rhuan001/cybersecurity-home-lab/main/04-detection/screenshots/36-ssh-brute-force-detection.png)

## Successful Login Investigation

Depois de identificar o IP associado às falhas, pesquisei os logins bem-sucedidos provenientes dessa origem:

```spl
index=* host="Metasploitable2" sourcetype="metasploitable_auth" "Accepted password" "192.168.56.20"
```

O Splunk confirmou um evento `Accepted password` para o utilizador `msfadmin`, com origem em `192.168.56.20`, às 22:25:39.

![SSH Successful Login](https://raw.githubusercontent.com/rhuan001/cybersecurity-home-lab/main/04-detection/screenshots/37-ssh-successful-login.png)

## Analysis

A detection query identificou múltiplas falhas de autenticação provenientes do mesmo IP. A investigação posterior confirmou um login bem-sucedido dessa origem.

O Medusa reportou 26 verificações, mas o número de eventos `Failed password` registados no Splunk não corresponde necessariamente ao número de passwords testadas.

O threshold de cinco falhas em dois minutos foi definido para este laboratório. Num ambiente real, seria necessário ajustá-lo ao comportamento normal dos utilizadores e analisar possíveis falsos positivos.
