## 🛡️ Microsoft Sentinel & Defender XDR: Laboratório SOC Completo

Este repositório demonstra a utilização do Microsoft Sentinel e seus recursos integrados em um lab interativo. 

O objetivo é demonstrar a transformação de dados brutos (logs de um *endpoint*) em inteligência acionável e automação de resposta, utilizando táticas reais mapeadas pelo MITRE ATT&CK.

## 🏗️ Arquitetura do Laboratório

| Componente | Função no Projeto |
| :--- | :--- |
| **Máquina Virtual** | Windows Server/10 (Azure VM) configurada para auditoria local. |
| **Agente de Coleta** | Azure Monitor Agent (AMA) para roteamento de telemetria. |
| **SIEM & XDR** | Microsoft Sentinel unificado ao portal do Microsoft Defender XDR. |
| **Simulação** | Atomic Red Team (PowerShell) para emulação de ataques. |
| **Automação (SOAR)** | Azure Logic Apps (Playbooks) para resposta automatizada. |

---

## 1. Coleta e Governança de Telemetria

Para garantir a visibilidade das ameaças, a política de auditoria do sistema operacional foi configurada para registrar eventos detalhados de execução de processos.

1. Configuração da *Group Policy Object* (GPO) no Windows para auditar a Criação de Processos, gerando o **Event ID 4688**.
2. Habilitação da auditoria de *Process Command Line* para capturar os argumentos exatos digitados pelo atacante.
3. Instalação do Azure Monitor Agent (AMA) via extensão da nuvem, conectando a VM ao *Log Analytics Workspace*.
---

## 2. Unificação da Plataforma de Operações
Atualmente, o *workspace* do Microsoft Sentinel foi integrado ao portal unificado de segurança da Microsoft, integrando diversas funções de segurança no portal.

1. Acesso ao portal `security.microsoft.com`.
2. Conexão do *Log Analytics Workspace* ao Microsoft Defender XDR.
---

## 3. Simulação de Ameaças
Utilizou-se o *framework* "Atomic Red Team" para gerar logs maliciosos e validar o *pipeline* de dados. O foco foi a técnica de reconhecimento de usuários, muito utilizada por atacantes.

* **Tática MITRE ATT&CK:** Discovery
* **Técnica MITRE ATT&CK:** T1033 (System Owner/User Discovery)

**Comandos Executados:**
```powershell
Import-Module Invoke-AtomicRedTeam
Invoke-AtomicTest T1033 -PathToAtomicsFolder "C:\AtomicRedTeam\atomics"
```
Basicamente, rodou-se os comandos nativos de reconhecimento (whoami, quser, wmic useraccount), populando o  SIEM com possíveis indicadores de ataque.

---
## 4. Analytics - Detecção 
Para haver detecção e alerta pelo SIEM, foi desenvolvida uma regra analítica para monitoramento contínuo da telemetria bruta, gerando automaticamente de incidente.

1. Criação de regra *Scheduled Query* utilizando *Kusto Query Language* (KQL). Foi agendada para rodar a cada 5 minutos.
2. **Lógica de Detecção:**
   ```kusto
   SecurityEvent
   | where EventID == 4688
   | where CommandLine has_any ("whoami", "wmic useraccount", "quser", "query user", "cmdkey")
   | project TimeGenerated, Computer, Account, ProcessName, CommandLine
---
## 5. Orquestração e Resposta Automatizada (SOAR)
Implementação de um mecanismo de resposta automática para redução do tempo de resposta.

1. Criação de um *Playbook* no Azure Logic Apps.
2. Concessão de privilégios via RBAC (*Microsoft Sentinel Automation Contributor*) ao *Resource Group* do laboratório.
3. Configuração do fluxo lógico para extrair variáveis dinâmicas do incidente (Título, URL, Severidade) em formato JSON.
4. Acoplamento do *Playbook* à regra KQL (Passo 4) para disparo imediato de notificações por e-mail à equipe do SOC.

Agora, assim que a query ocorrer, disparará um alerta via e-mail para o usuário cadastrado.

---
## 6. Inteligência Analítica
A fase final consolidou os dados analisados em ferramentas de gestão visual e inteligência preditiva.

1. **Dashboards (Workbooks):** Construção de painéis visuais com KQL direto na base `SecurityEvent`, mapeando a frequência das táticas do MITRE ATT&CK enfrentadas pela infraestrutura.
2. **Análise Comportamental (UEBA):** Ativação do motor de *Machine Learning* do Microsoft Defender (*Entity Behavior Analytics*).
---

## 📌 Conclusões do Laboratório

Essa é só uma pequena amostra do que os SIEMs podem fazer. A análise dos logs automaticamente pelo SOAR ajuda o *Analista SOC* à diminuir o tempo de resposta à incidentes, e, a criação desses playbooks pode acelerar ainda mais a detecção de possíveis ameaças.

A unificação do SIEM e XDR também traz ganhos operacionais imediatos ao reduzir a alternância de contexto do analista durante o *Threat Hunting*. Além disso, a combinação de automação com o aprendizado de máquina (UEBA) mostra que podemos deixar o SIEM ser cada vez mais autônomo e independente.
