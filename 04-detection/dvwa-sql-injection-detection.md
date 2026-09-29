# Scenario 02 — DVWA SQL Injection Detection

## Objective

Analisar no Splunk os eventos gerados pelo pedido HTTP com SQL Injection submetido ao DVWA e criar uma detection query para identificar padrões de SQL Injection nos logs do Apache.

## Log Source

- **Host:** `Metasploitable2`
- **Sourcetype:** `metasploitable_apache_live`
- **Source:** `/var/log/metasploitable-apache-live.log`

## Apache Request Events

Comecei por pesquisar os pedidos HTTP que continham aspas simples codificadas em URL (`%27`), sem filtrar ainda por IP:

```spl
host="Metasploitable2" sourcetype="metasploitable_apache_live" "%27"
```

A pesquisa devolveu 6 eventos, incluindo pedidos de navegação normal cujo campo *referrer* continha o payload, e os pedidos diretamente associados à exploração.

![Splunk SQLi Detection Query](https://raw.githubusercontent.com/rhuan001/cybersecurity-home-lab/main/04-detection/screenshots/5-splunk-sqli-detection-query.png)

## SQL Injection Detection Query

Refinei a pesquisa para identificar especificamente pedidos que combinam uma aspa codificada com a palavra-chave `OR`, uma assinatura mais próxima de uma tentativa de SQL Injection:

```spl
host="Metasploitable2" sourcetype="metasploitable_apache_live" "%27" "OR"
| table _time, host, _raw
| sort - _time
```

A query identificou os pedidos associados às tentativas de SQL Injection, incluindo os dois pedidos que resultaram em exploração bem-sucedida (`Content-Length: 4660`).

## Successful Exploitation Identification

Dentro dos eventos devolvidos, os pedidos correspondentes à exploração bem-sucedida foram identificados pelo tamanho de resposta superior ao esperado:

```text
192.168.56.20 - - [28/Sep/2026:17:40:34 -0400] "GET /dvwa/vulnerabilities/sqli/?id=3%27+OR+%271%27%3D%271&Submit=Submit HTTP/1.1" 200 4660
```

O valor `4660 bytes`, superior aos `4387 bytes` de uma resposta normal com um único utilizador, confirma que a query devolveu múltiplos registos.

## Analysis

A detection query identificou corretamente os pedidos HTTP contendo o padrão de SQL Injection (`%27` + `OR`), a partir da origem `192.168.56.20`.

Ao contrário do cenário SSH, aqui não foi necessário um threshold baseado em contagem de eventos — a deteção baseou-se na presença de caracteres e palavras-chave associadas a SQL Injection no próprio pedido HTTP.

Num ambiente real, esta assinatura simples geraria falsos positivos com pesquisas legítimas que incluam aspas ou a palavra "OR". Seria necessário refinar a query (por exemplo, limitar a parâmetros específicos da aplicação, ou usar uma lista mais ampla de padrões de SQL Injection) e considerar a integração de uma ferramenta como um WAF para deteção mais robusta.
