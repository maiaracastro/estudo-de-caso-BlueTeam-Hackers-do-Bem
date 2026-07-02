# Estudo de Caso — Defesa e Monitoramento (Blue Team) | Hackers do Bem

Este repositório contém a documentação técnica, topologia e arquivos de configuração desenvolvidos para o estudo de caso final da extensão em Blue Team do programa Hackers do Bem. O projeto simula a estruturação de uma infraestrutura corporativa segura, aplicando conceitos de segmentação de rede, gerenciamento de identidades, automação com Ansible e centralização de logs para auditoria.

---

## 💻 Visão Geral do Projeto

A arquitetura foi projetada para garantir resiliência, redundância de serviços e rastreabilidade total de incidentes em um ambiente empresarial fictício (`wheels.intnet`), segmentado de forma inteligente para mitigar superfícies de ataque.

### 🛠️ Tecnologias e Protocolos Utilizados
* **Design de Redes:** Draw.io (Topologia Lógica e Endereçamento CIDR)
* **Serviços de Infraestrutura:** NSD (DNS Primário) & Unbound (DNS Recorrente/Validador)
* **Diretório e Autenticação:** OpenLDAP (`nslcd`) e ferramentas de gerenciamento (`ldapscripts`)
* **Automação de TI:** Ansible (Gerenciamento de Roles, Playbooks e Sudoers)
* **SIEM / Gestão de Logs:** Graylog Open e protocolo Syslog (UDP/5140)
* **Sistemas Operacionais:** GNU/Linux (Debian/Ubuntu-based)

---

## 🏢 1. Arquitetura e Segmentação de Rede (VLANs)

A rede interna foi dividida em sub-redes lógicas dedicadas para isolar privilégios e tráfego sensível, organizadas conforme o mapeamento a seguir:

| Nome da Rede | Tag VLAN | Descrição |
| :--- | :---: | :--- |
| **Rede Default** | 1 | Gerenciamento nativo de switches e ativos de rede. |
| **Telefones IP** | 10 | Segmentação de tráfego de voz (VoIP) com prioridade de QoS. |
| **Rede sem fio - Corporativa** | 20 | Acesso Wi-Fi exclusivo para dispositivos e colaboradores internos. |
| **Rede sem fio - BYOD** | 30 | Dispositivos pessoais de funcionários isolados da rede crítica. |
| **Rede sem fio - Visitante** | 40 | Acesso restrito à internet para clientes externos. |
| **Servidores** | 50 | Zona restrita para infraestrutura (AD, LDAP, DNS, Graylog). |
| **Impressoras** | 60 | Segmentação de periféricos de impressão e recursos compartilhados. |

> 📌 **Nota de Implementação:** As estações de trabalho dos departamentos (Administrativo, Financeiro, Logística, Pessoal e Manutenção) foram alocadas em blocos CIDR específicos no barramento principal para controle granular de acessos.

---

## 🛡️ 2. Serviços de Redes e Segurança Implementados

### 🔍 2.1. Resolução de Nomes Confiável (DNS)
Implementação de uma estrutura dividida na VM **LinServer** para otimizar desempenho e segurança:
* **NSD:** Atua estritamente como servidor autoritativo para a zona interna local (`wheels.intnet`).
* **Unbound:** Atua como o resolvedor recursivo que atende às requisições dos clientes e repassa as consultas da zona local para o NSD na porta `8053`, garantindo proteção contra envenenamento de cache.

### 👤 2.2. Autenticação Centralizada (LDAP)
* Integração via `nslcd.conf` e automação de contas com o `ldapscripts.conf`.
* Mapeamento direto de usuários (`ou=People`) e grupos (`ou=Groups`), unificando o controle de credenciais das estações.

### 🤖 2.3. Automação e Hardening com Ansible
Desenvolvimento de uma Role focada em governança para propagar políticas restritivas do arquivo `sudoers`:
* Criação de aliases para administradores e grupos de usuários do LDAP.
* Restrição estrita de comandos executáveis via comandos autorizados (`LDAPCMDS`), aplicando o princípio do privilégio mínimo.
* Orquestração centralizada gerenciando o inventário dos hosts da infraestrutura.

---

## 📊 3. Centralização de Logs e Auditoria (SIEM com Graylog)

Para garantir monitoramento contínuo, as máquinas virtuais e servidores enviam logs do sistema em tempo real para um servidor centralizado **Graylog (DMZ)** através do fluxo **Syslog (UDP/5140)**.

O painel foi estruturado para exibir:
1. O somatório de falhas e tentativas de autenticação inválidas.
2. Gráficos de barras diários para análise de tendência de acessos.
3. Listagem detalhada em tempo real com os últimos logs coletados das VMs (ex: `LinClient`), ideal para triagem rápida e resposta a incidentes.

---

## 📁 Estrutura deste Repositório

```text
├── .github/
├── configs/
│   ├── dns/               # nsd.conf e unbound.conf
│   ├── ldap/              # nslcd.conf e ldapscripts.conf
│   └── ansible/           # ansible.cfg, hosts e srv.yml
├── roles/
│   └── sudoers/
│       ├── tasks/main.yml # Task de propagação
│       └── files/sudoers  # Arquivo de regras de privilégios
└── README.md
