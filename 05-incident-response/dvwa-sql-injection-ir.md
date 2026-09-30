# Incident Response — Scenario 2: DVWA SQL Injection

**Incidente:** Exploração de SQL Injection na aplicação DVWA
**Data de deteção:** *(preencher a partir do 04-detection)*
**Analista:** Rhuan
**Origem:** KALI — `192.168.56.20`
**Alvo:** Metasploitable2 (Apache + DVWA, backend MySQL) — `<IP_METASPLOITABLE2>`
**Severidade:** Alta
**Estado:** Contido em ambiente de laboratório (sem impacto em produção)

---

## 1. Resumo do Incidente

Foi identificada a exploração bem-sucedida de uma vulnerabilidade de **SQL Injection** no módulo *SQL Injection* da aplicação **DVWA** (Security Level: Low), alojada no servidor Metasploitable2.

A partir da máquina atacante (KALI), o tráfego HTTP foi intercetado e manipulado com recurso ao **Burp Suite** (Proxy + Repeater). Um pedido legítimo ao parâmetro `id` foi alterado com um payload de injeção que forçou a base de dados a devolver a **totalidade dos registos** da tabela de utilizadores, em vez de um único registo.

A anomalia foi confirmada em três camadas independentes: resposta da aplicação, registo do **Apache access.log** e evento indexado no **Splunk** (`sourcetype=metasploitable_apache_live`).

---

## 2. Cronologia (Timeline)

> Os timestamps exatos devem ser transcritos do `04-detection/` e do Apache access.log.

**Pedido baseline (comportamento legítimo)**

```text
[HH:MM:SS]  GET /dvwa/vulnerabilities/sqli/?id=3  → 1 utilizador devolvido (Content-Length 4387)
```

**Pedido manipulado (exploração)**

```text
[HH:MM:SS]  GET /dvwa/vulnerabilities/sqli/?id=3' OR '1'='1  → 5 utilizadores devolvidos (Content-Length 4660)
```

**Registo e deteção**

```text
[HH:MM:SS]  Pedido malicioso registado em Apache access.log
[HH:MM:SS]  Evento indexado e correlacionado no Splunk (padrão %27 + OR)
```

---

## 3. Sistemas e Contas Afetados

**Sistema afetado**

```text
Metasploitable2 — Apache HTTP Server + DVWA + MySQL
```

**Origem do ataque**

```text
KALI — 192.168.56.20
```

**Dados expostos**

Foram devolvidos os registos dos 5 utilizadores da tabela do DVWA:

```text
admin
Gordon Brown
Hack Me
Pablo Picasso
Bob Smith
```

**Nota de rigor:** o payload utilizado (`OR '1'='1`) expôs os **nomes** dos registos da tabela. A extração de credenciais (hashes de password) exigiria uma injeção do tipo **UNION-based**, que não foi executada neste cenário — fica apenas referida como escalonamento possível.

---

## 4. Evidências

As evidências completas encontram-se em `04-detection/` (Scenario 2). Confirmadas em três níveis:

**Nível 1 — Aplicação**

```text
Baseline (id=3):  1 utilizador
Injeção (3' OR '1'='1):  5 utilizadores
```

**Nível 2 — Apache access.log**

O tamanho da resposta distingue o pedido legítimo do malicioso:

```text
Content-Length (baseline):  4387
Content-Length (injeção):   4660
```

**Nível 3 — Splunk**

Query de deteção (aspa URL-encoded + operador lógico):

```spl
sourcetype="metasploitable_apache_live" "%27" "OR"
```

**Fluxo da evidência:**

```text
  Aplicação (5 utilizadores devolvidos)
              │
              ▼
  Apache access.log (Content-Length 4660 vs 4387)
              │
              ▼
  Splunk (metasploitable_apache_live → padrão %27 + OR)
```

---

## 5. Impacto

A resposta anómala confirma que a aplicação **não valida nem parametriza** a entrada do utilizador, permitindo alterar a lógica da query SQL executada no backend.

Com base na evidência observada, a exposição não autorizada da tabela de utilizadores demonstra a vulnerabilidade. Dependendo da técnica aplicada e das permissões da conta MySQL, uma exploração mais avançada **poderia resultar em** extração de credenciais (via UNION-based injection), leitura de ficheiros do sistema ou, em configurações permissivas, execução de comandos — cenários não testados neste laboratório.

---

## 6. Contenção (Ações Recomendadas)

> As ações abaixo são **recomendações**; não foram executadas neste cenário.

- Bloquear na WAF / proxy inverso os padrões de injeção (aspa simples URL-encoded `%27`, operadores lógicos `OR`/`UNION`).
- Restringir o acesso à aplicação vulnerável através de segmentação de rede.
- Rever os registos do Apache e do Splunk para identificar outros pedidos com o mesmo padrão a partir da origem `192.168.56.20`.

---

## 7. Remediação (Ações Recomendadas)

> Recomendações para eliminar a causa-raiz; não aplicadas neste cenário.

- **Prepared statements / parameterized queries** — separar o código SQL dos dados de entrada (correção primária).
- **Validação e sanitização** da entrada do utilizador (tipo, formato, comprimento).
- **Princípio do menor privilégio** na conta MySQL usada pela aplicação (sem permissões de FILE nem acesso a tabelas do sistema).
- Desativar mensagens de erro detalhadas devolvidas ao cliente.
- Em contexto real, manter o software atualizado e reforçar o nível de segurança da aplicação.

---

## 8. Lições Aprendidas

- A deteção em **três camadas** (aplicação → log web → SIEM) aumenta a confiança na identificação e reduz falsos positivos.
- O **Content-Length** é um indicador simples mas eficaz para distinguir uma resposta legítima de uma resposta manipulada.
- A ingestão dos logs de aplicação (Apache) no Splunk é essencial — um SIEM focado apenas em autenticação não teria detetado este ataque.
- Distinguir o que foi **demonstrado** do que é **teoricamente possível** mantém a documentação rigorosa e defensável.
