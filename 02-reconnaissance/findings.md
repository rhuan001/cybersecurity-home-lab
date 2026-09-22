# Reconnaissance Findings

## Finding 01 — Anonymous FTP Access

**Target:** `192.168.56.30`  
**Port:** `21/TCP`  
**Service:** `vsFTPd 2.3.4`  
**Severity:** Medium

### Description

O serviço FTP permite anonymous authentication, sem exigir uma conta pessoal válida.

Identifiquei esta configuração através do Nmap e confirmei-a manualmente ao estabelecer uma ligação FTP com o utilizador `anonymous`.

O servidor respondeu:

`230 Login successful.`

Depois, executei um directory listing com o comando:

`ls -la`

Durante o teste, não foram encontrados ficheiros expostos no diretório acessível ao utilizador anónimo.

Também identifiquei que a ligação FTP utiliza plain text, ou seja, os dados de autenticação e os ficheiros transferidos não são encriptados.

### Impact

O anonymous FTP access aumenta a attack surface do servidor, permitindo que utilizadores sem uma conta pessoal válida interajam com o serviço.

Embora não tenha encontrado ficheiros expostos durante o teste, esta configuração pode originar riscos como:

- Unauthorized access a ficheiros, caso sejam disponibilizados no diretório FTP
- Information disclosure
- Upload ou modificação não autorizada de ficheiros, caso existam write permissions
- Interceção de credenciais FTP e dados transferidos devido à ausência de encriptação

### Evidence

![FTP Enumeration](screenshots/16-ftp-enumeration.png)

![FTP Anonymous Login](screenshots/17-ftp-anonymous-login.png)

### Recommendation

- Desativar anonymous FTP access, caso não seja necessário.
- Restringir o acesso FTP a utilizadores autorizados.
- Substituir FTP por um protocolo com encriptação, como SFTP ou FTPS.
- Rever as permissões dos diretórios FTP.
- Manter o serviço atualizado ou removê-lo caso não seja necessário.

### Status

**Confirmed**

---

## Finding 02 — Insecure SMB Configuration and Anonymous Access

**Target:** `192.168.56.30`  
**Ports:** `139/TCP`, `445/TCP`  
**Service:** `Samba 3.0.20-Debian`  
**Severity:** High

### Description

Durante a SMB enumeration, identifiquei várias configurações inseguras no serviço.

O Nmap revelou que SMBv1 está ativo e que o SMB message signing está desativado.

Também identifiquei que o serviço permite anonymous/guest authentication, possibilitando a enumeração de SMB shares e de contas de utilizadores locais.

A share `tmp` foi identificada pelo Nmap como permitindo anonymous `READ/WRITE` access.

Para validar os resultados, utilizei o `smbclient` e confirmei manualmente que era possível:

- Estabelecer uma ligação SMB sem password
- Enumerar as shares disponíveis
- Aceder à share `tmp`
- Listar o conteúdo da share

A permissão de escrita na share `tmp` foi reportada pelo Nmap, mas não foi confirmada manualmente.

### Impact

Estas configurações aumentam a attack surface do sistema e permitem que um utilizador não autenticado recolha informações ou interaja com recursos partilhados.

Os principais riscos incluem:

- Enumeração de usernames válidos
- Enumeração de SMB shares
- Acesso não autorizado a recursos partilhados
- Possível exposição de ficheiros em shares acessíveis anonimamente
- Riscos associados à utilização do protocolo legacy SMBv1
- Maior exposição a determinados ataques de SMB relay ou Man-in-the-Middle quando o SMB signing está desativado

### Evidence

![SMB User Enumeration](screenshots/18-smb-enumeration-users.png)

![SMB Share Enumeration](screenshots/19-smb-enumeration-shares.png)

![SMB Security Settings](screenshots/20-smb-security-settings.png)

![SMB Anonymous Shares](screenshots/21-smb-anonymous-shares.png)

![SMB TMP Listing](screenshots/22-smb-tmp-listing.png)

### Recommendation

- Desativar SMBv1 e utilizar versões modernas do protocolo SMB.
- Ativar SMB message signing quando aplicável.
- Desativar anonymous e guest SMB access, caso não sejam necessários.
- Restringir o acesso às SMB shares através de authentication e permissions adequadas.
- Remover shares desnecessárias.
- Limitar o acesso SMB a hosts e network segments autorizados.
- Atualizar o Samba para uma versão suportada.

### Status

**Confirmed**

---

## Finding 03 — Exposed PHP Information Page

**Target:** `192.168.56.30`  
**Port:** `80/TCP`  
**Service:** Apache HTTP Server / PHP  
**Severity:** Low

### Description

Durante a HTTP enumeration, identifiquei uma página de informação PHP acessível sem autenticação:

`http://192.168.56.30/phpinfo.php`

Esta página expõe informações detalhadas sobre a configuração PHP e o sistema onde o serviço está instalado.

Entre as informações disponíveis encontram-se:

- PHP version `5.2.4-2ubuntu5.10`
- Operating system e kernel information
- Server API configuration
- PHP configuration file locations
- Caminho do ficheiro `php.ini`
- PHP extensions e modules carregados
- Internal filesystem paths

O ficheiro foi inicialmente identificado através do script `http-enum` do Nmap e posteriormente validado através do browser.

### Impact

A exposição da página `phpinfo()` permite que um utilizador não autenticado obtenha informações técnicas detalhadas sobre o servidor.

Embora esta exposição não permita, por si só, obter acesso ao sistema, pode ajudar um atacante durante a reconnaissance ao revelar:

- Software versions
- Server configuration
- PHP components instalados
- Internal filesystem paths
- Tecnologias que podem apresentar vulnerabilidades conhecidas

Estas informações podem facilitar a preparação de ataques mais direcionados.

### Evidence

![HTTP Enumeration](screenshots/26-http-enum.png)

![PHPInfo Information Disclosure](screenshots/27-phpinfo-information-disclosure.png)

### Recommendation

- Remover páginas `phpinfo()` acessíveis publicamente em ambientes de produção.
- Restringir o acesso a páginas de diagnóstico e desenvolvimento a administradores autorizados.
- Evitar a exposição desnecessária de software versions e configurações internas.
- Rever o web root para identificar outros ficheiros de teste, desenvolvimento ou diagnóstico.
- Manter o PHP e o web server atualizados e em versões suportadas.

### Status

**Confirmed**
