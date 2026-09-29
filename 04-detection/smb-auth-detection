# Scenario 03 — SMB Authentication Detection

## Objective

Analisar no Splunk os eventos de autenticação gerados pelas tentativas de acesso SMB ao Windows 10 e identificar o padrão de falhas seguidas de um login bem-sucedido.

## Log Source

- **Host:** `P1` (Windows 10 — `192.168.56.50`)
- **Sourcetype:** `WinEventLog:Security`
- **Event Codes:** `4625` (Failed Logon) e `4624` (Successful Logon)

## Authentication Events

Comecei por pesquisar os eventos de autenticação com sucesso e falha no host Windows:

```spl
host="P1" sourcetype="WinEventLog:Security" (EventCode=4625 OR EventCode=4624)
```

A pesquisa devolveu vários eventos, incluindo atividade normal do sistema. Entre eles, foram identificados os três eventos correspondentes às tentativas realizadas a partir do Kali:

- `18:14:42` — `4625` (falha)
- `18:14:48` — `4625` (falha)
- `18:15:05` — `4624` (sucesso)

![SMB Authentication Events](https://raw.githubusercontent.com/rhuan001/cybersecurity-home-lab/main/04-detection/screenshots/6-splunk-smb-auth-events.png)

## Successful Logon Investigation

Para analisar em detalhe o login bem-sucedido, pesquisei o evento `4624` associado à conta `vboxuser`:

```spl
host="P1" sourcetype="WinEventLog:Security" EventCode=4624 Account_Name="vboxuser"
```

Ao expandir o evento correspondente às `18:15:05`, foram confirmados os seguintes campos:

- **Logon Type:** `3` (Network)
- **Account Name:** `vboxuser`
- **Workstation Name:** `KALI`
- **Source Network Address:** `192.168.56.20`
- **Authentication Package:** `NTLM`

Estes campos confirmam que a autenticação teve origem na máquina Kali, através da rede, utilizando o protocolo NTLM.

![SMB Successful Logon Details](https://raw.githubusercontent.com/rhuan001/cybersecurity-home-lab/main/04-detection/screenshots/7-splunk-smb-successful-logon-details.png)

## Analysis

Os eventos confirmaram o padrão esperado: duas tentativas de autenticação falhadas (`4625`) seguidas de um login bem-sucedido (`4624`), todas com origem em `192.168.56.20`.

O campo `Logon Type: 3` permite distinguir autenticações vindas da rede da restante atividade local do sistema, sendo um filtro útil para isolar tentativas de acesso remoto.

Neste laboratório o número de tentativas foi reduzido. Num ambiente real, uma detection query eficaz para este tipo de atividade basear-se-ia na contagem de eventos `4625` por IP de origem num intervalo de tempo, de forma semelhante à deteção utilizada no cenário SSH, permitindo identificar padrões de password spraying ou brute-force contra o serviço SMB.
