# Scenario 02 — DVWA SQL Injection Detection

## Objective

Analisar no Splunk os eventos gerados pelos pedidos HTTP com SQL Injection submetidos ao DVWA e identificar nos logs do Apache os pedidos associados à exploração.

## Log Source

- **Host:** `Metasploitable2`
- **Sourcetype:** `metasploitable_apache_live`
- **Source:** `/var/log/metasploitable-apache-live.log`

## Apache Request Events

Pesquisei os pedidos HTTP que continham aspas simples codificadas em URL (`%27`), sem filtrar ainda por IP:

```spl
host="Metasploitable2" sourcetype="metasploitable_apache_live" "%27"
```

A pesquisa devolveu 6 eventos. Quatro são pedidos cujo URL contém o payload e dois são pedidos de navegação normal cujo campo *referrer* continha o payload (a captura assinala os quatro primeiros com uma caixa vermelha).

![Splunk SQLi Detection Query](https://raw.githubusercontent.com/rhuan001/cybersecurity-home-lab/main/04-detection/screenshots/5-splunk-sqli-detection-query.png)

| Hora (Apache, -0400) | Pedido | Resposta | Observação |
|---|---|---|---|
| 17:40:48 | `GET /dvwa/vulnerabilities/sqli/?id=3%27+OR+%271%27%3D%271&Submit=Submit` | 4660 bytes | Payload no URL |
| 17:40:43 | `GET /dvwa/vulnerabilities/sqli/` | 4333 bytes | Payload apenas no *referrer* |
| 17:40:34 | `GET /dvwa/vulnerabilities/sqli/?id=3%27+OR+%271%27%3D%271&Submit=Submit` | 4660 bytes | Payload no URL |
| 17:40:27 | `GET /dvwa/security.php` | 4106 bytes | Payload apenas no *referrer* |
| 17:39:34 | `GET /dvwa/vulnerabilities/sqli/?id=3%27+OR+%271%27%3D%271&Submit=Submit` | 4336 bytes | Payload no URL |
| 17:38:06 | `GET /dvwa/vulnerabilities/sqli/?id=id%3D3%27+OR+%271%27%3D%271&Submit=Submit` | 4336 bytes | Payload no URL (parâmetro `id` com valor `id=3' OR ...`) |

Todos os pedidos têm origem em `192.168.56.20`.

## Successful Exploitation Identification

Entre os eventos com o payload no URL, os pedidos correspondentes à exploração bem-sucedida foram identificados pelo tamanho de resposta superior (`4660 bytes`), presente em dois eventos (17:40:34 e 17:40:48):

```text
192.168.56.20 - - [28/Sep/2026:17:40:34 -0400] "GET /dvwa/vulnerabilities/sqli/?id=3%27+OR+%271%27%3D%271&Submit=Submit HTTP/1.1" 200 4660
```

O valor `4660 bytes` é superior aos `4387 bytes` de uma resposta normal com um único utilizador, o que é consistente com a devolução de múltiplos registos, confirmada na aplicação (5 utilizadores).

Os dois pedidos com `4336 bytes` (17:38:06 e 17:39:34) tiveram uma resposta inferior. Isto é consistente com as tentativas iniciais sem resultados registadas no `progress-log.md`, quando o Security Level do DVWA estava, sem intenção, em `High`.

**Nota sobre timestamps:** o Apache regista a hora com o fuso `-0400` (por exemplo `17:40:34`), enquanto o Splunk apresenta a hora do evento syslog em UTC (por exemplo `21:40:35`). Correspondem ao mesmo momento.

## Analysis

A pesquisa por `%27` identificou os pedidos HTTP contendo aspas simples codificadas em URL provenientes da origem `192.168.56.20`, incluindo os pedidos associados à exploração observada.

Ao contrário do cenário SSH, aqui não foi utilizado um threshold baseado em contagem de eventos — a pesquisa baseou-se na presença de um carácter associado a SQL Injection no próprio pedido HTTP. A distinção entre tentativas e exploração bem-sucedida foi feita através do tamanho da resposta.

Esta pesquisa é pouco específica: devolveu também pedidos de navegação normal cujo *referrer* continha o payload (2 dos 6 eventos), e teria falsos positivos com pesquisas legítimas que incluam aspas. Num ambiente real, seria necessário refinar a pesquisa, por exemplo combinando a aspa codificada com palavras-chave SQL como `OR` ou `UNION`, limitando-a a parâmetros específicos da aplicação e ignorando o campo *referrer*, e considerar a integração de uma ferramenta como um WAF para uma deteção mais robusta.
