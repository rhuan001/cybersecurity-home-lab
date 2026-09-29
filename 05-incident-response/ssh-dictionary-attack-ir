# Scenario 01 — Incident Response: SSH Dictionary Attack

## 1. Resumo do Incidente

Foi detetada uma série de tentativas de autenticação SSH falhadas contra o Metasploitable2 (`192.168.56.30`), com origem no host `192.168.56.20`, seguidas de uma autenticação bem-sucedida com a conta `msfadmin`.

O padrão observado é consistente com um dictionary attack contra o serviço SSH.

---

## 2. Cronologia (Timeline)

| Hora | Evento |
|------|--------|
| ~22:24 | Início de múltiplas tentativas de autenticação SSH falhadas a partir de `192.168.56.20` |
| 22:24 (janela de 2 min) | 7 eventos `Failed password` registados no Splunk |
| 22:25:39 | Evento `Accepted password` para a conta `msfadmin` a partir de `192.168.56.20` |

---

## 3. Sistemas e Contas Afetados

- **Sistema alvo:** Metasploitable2 — `192.168.56.30`
- **Serviço:** SSH — `22/TCP`
- **Conta envolvida:** `msfadmin`
- **Origem do ataque:** `192.168.56.20`

---

## 4. Evidências

- Eventos `Failed password` e `Accepted password` no sourcetype `metasploitable_auth`
- Detection query no Splunk identificou 7 falhas de autenticação num intervalo de 2 minutos a partir do mesmo IP
- Confirmação de um evento `Accepted password` para a conta `msfadmin`, da mesma origem, após as falhas

Evidências documentadas em `04-detection/ssh-detection.md`.

---

## 5. Impacto

A evidência observada mostra que, após múltiplas tentativas falhadas, ocorreu uma autenticação SSH bem-sucedida com a conta `msfadmin` a partir de `192.168.56.20`.

Num ambiente real, uma autenticação bem-sucedida deste tipo poderia permitir acesso remoto ao sistema e execução de comandos e, dependendo dos privilégios da conta, movimentação lateral ou escalada de privilégios.

---

## 6. Contenção (Ações Recomendadas)

As seguintes ações de contenção seriam aplicáveis a este tipo de incidente. Não foram executadas neste laboratório e são listadas como resposta recomendada:

- Bloquear o IP de origem (`192.168.56.20`) ao nível da firewall
- Desativar temporariamente a conta envolvida (`msfadmin`)
- Terminar sessões SSH ativas provenientes da origem suspeita

---

## 7. Remediação (Ações Recomendadas)

- Impor uma política de passwords fortes
- Implementar uma account lockout policy após múltiplas falhas de autenticação
- Restringir o acesso SSH por IP (allowlist) ou através de VPN
- Preferir autenticação por chave em vez de password
- Considerar a utilização de ferramentas como o fail2ban para bloquear automaticamente IPs com múltiplas falhas

---

## 8. Lições Aprendidas

A monitorização centralizada dos logs de autenticação permitiu identificar o padrão de múltiplas falhas seguidas de uma autenticação bem-sucedida.

Uma detection query baseada na contagem de falhas por IP num intervalo de tempo mostrou-se eficaz para identificar este tipo de atividade. Num ambiente de produção, o threshold utilizado deveria ser ajustado ao comportamento normal dos utilizadores para reduzir falsos positivos.
