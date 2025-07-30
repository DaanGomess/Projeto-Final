
# Relatório Completo - Mapeamento de Rede Interna
**Trilha de Formação em Cybersecurity – Módulo 1**

---

##  Desafio
##  Feito por:  
**Analista: Danilo Gomes** 
**Data: 27/07/2025**
**Versão: Ubuntu 24.04** 

---

##  1. Sub-redes identificadas

### **Corp Net**
- Faixa: 10.10.10.0/24
- Hosts encontrados:
  - 10.10.10.1
  - 10.10.10.10 - WS_001.projeto_final_opcao_1_corp_net
  - 10.10.10.101 - WS_002.projeto_final_opcao_1_corp_net
  - 10.10.10.127 - WS_003.projeto_final_opcao_1_corp_net
  - 10.10.10.222 - WS_004.projeto_final_opcao_1_corp_net
  - 10.10.10.2

### **Guest Net**
- Faixa: 10.10.50.0/24
- Hosts encontrados:
  - 10.10.50.1
  - 10.10.50.2 - notebook-carlos
  - 10.10.50.3 - laptop-luiz
  - 10.10.50.4 - laptop-vastro
  - 10.10.50.5 - macbook-aline
  - 10.10.50.6

### **Infra Net**
- Faixa: 10.10.30.0/24
- Hosts encontrados:
  - 10.10.30.1
  - 10.10.30.10 - ftp-server
  - 10.10.30.11 - mysql-server
  - 10.10.30.15 - samba-server
  - 10.10.30.17 - opendap (LDAP)
  - 10.10.30.117 - zabbix-server
  - 10.10.30.227 - legacy-server
  - 10.10.30.2

---

##  2. Inventário Técnico Detalhado

| IP           | Nome do Host                 | Função / Serviço              | Porta(s) | Observações |
|--------------|------------------------------|-------------------------------|----------|--------------|
| 10.10.10.1   | -                            | Workstation                   | -        | Corp Net |
| 10.10.10.10  | WS_001                       | Workstation                   | -        | Corp Net |
| 10.10.10.101 | WS_002                       | Workstation                   | -        | Corp Net |
| 10.10.10.127 | WS_003                       | Workstation                   | -        | Corp Net |
| 10.10.10.222 | WS_004                       | Workstation                   | -        | Corp Net |
| 10.10.50.1   | -                            | Notebook Carlos               | -        | Guest Net |
| 10.10.50.2   | notebook-carlos              | Notebook                      | -        | Guest Net |
| 10.10.50.3   | laptop-luiz                  | Laptop                        | -        | Guest Net |
| 10.10.50.4   | laptop-vastro                | Laptop                        | -        | Guest Net |
| 10.10.50.5   | macbook-aline                | MacBook                       | -        | Guest Net |
| 10.10.50.6   | -                            | -                             | -        | Guest Net |
| 10.10.30.1   | -                            | Provável Gateway              | -        | Infra Net |
| 10.10.30.10  | ftp-server                   | FTP Server                    | 21/tcp   | Nmap detectou FTP aberto |
| 10.10.30.11  | mysql-server                 | MySQL Server 8.0.42           | 3306/tcp | Salt exposto, Auth Plugin: caching_sha2_password |
| 10.10.30.15  | samba-server                 | Samba SMB Server              | 445/tcp  | SMB shares detectados |
| 10.10.30.17  | opendap                      | LDAP Server                   | 389/tcp  | Naming Context: dc=example,dc=org |
| 10.10.30.117 | zabbix-server                | Zabbix Web (nginx)            | 80/tcp   | Página login visível |
| 10.10.30.227 | legacy-server                | Servidor legado               | ?        | Detalhes não explorados |
| 10.10.30.2   | -                            | -                             | -        | Infra Net |

---

##  3. Evidências Coletadas

**Comandos principais:**  
- `nmap -sn -T4 10.10.10.0/24 -oG -` para descobrir hosts vivos no Corp Net.  
- `nmap -sn -T4 10.10.50.0/24 -oG -` para descobrir hosts vivos no Guest Net.  
- `nmap -sn -T4 10.10.30.0/24 -oG -` para descobrir hosts vivos no Infra Net.  
- `nmap -p 21,3306,389,445,80 --script <scripts>` para serviços específicos.

**Serviços mapeados:**  
- SMB enum shares e OS discovery: `smb-os-discovery,smb-enum-shares`  
- LDAP rootDSE info: `ldap-rootdse`  
- MySQL info: `mysql-info`  
- Web server (Zabbix) testado com `curl`

**HTML exposto:**  
- Zabbix Web exibe login, metadados do painel, arquivos CSS, favicon e scripts inline.

**LDAP rootDSE:**  
- Contexto: `dc=example,dc=org`  
- Mecanismos SASL: SCRAM, GSSAPI, NTLM, DIGEST-MD5.

**MySQL:**  
- Versão: 8.0.42  
- Plugin: `caching_sha2_password`  
- Salt visível no handshake.

**Arquivos gerados:**  
- `infra_net_servico_smb.txt`
- `infra_net_servico_zabbix.txt`
- `infra_net_servico_ldap-rootdse.txt`
- `infra_net_servico_mysql-info.txt`
- `infra_net_ips.txt` e `infra_net_ips_hosts.txt`
- `corp_net_ips.txt` e `corp_net_ips_hosts.txt`
- `guest_net_ips.txt` e `guest_net_ips_hosts.txt`

---

##  4. Diagnóstico

- **Rede segmentada corretamente:** Corp, Guest, Infra.
- **Múltiplos serviços críticos expostos:** FTP, SMB, LDAP, MySQL, Zabbix Web.
- **Vulnerabilidades potenciais:** Credenciais podem ser brute-forçadas, banner info leakage, interfaces sem TLS.
- **Monitoramento exposto:** Zabbix acessível externamente.
- **Rede Guest com dispositivos pessoais nominais:** risco de vazamento de identidade e pivotamento.

---

##  5. Recomendações de Segurança

 **Curto Prazo:**  
- Restringir acesso ao Zabbix via firewall ou VPN.
- Ativar TLS/SSL em LDAP, SMB, MySQL.
- Revisar permissões SMB shares e auditoria.
- Desativar banners e ajustes `skip-show-database` no MySQL.
- Validar senhas fortes em LDAP.

 **Médio Prazo:**  
- Habilitar autenticação 2FA LDAP / Zabbix.
- Migrar servidor legado.
- Implantar honeypots e regras IDS.
- Segmentar Guest Net com VLAN isolada.

---

##  6. Plano de Ação 80/20

| Ação                                   | Impacto | Facilidade | Prioridade |
|----------------------------------------|---------|------------|------------|
| Restringir Zabbix por IP / VPN         | Alto    | Fácil      | Alta  |
| Forçar TLS em LDAP / MySQL / SMB       | Alto    | Médio      | Alta  |
| Revisar e endurecer SMB shares         | Médio   | Fácil      | Alta  |
| Isolar Guest Net via VLAN              | Médio   | Fácil      | Alta  |
| Habilitar 2FA LDAP / Zabbix            | Alto    | Médio      | Média |
| Migrar servidor legado                 | Alto    | Difícil    | Média |
 

