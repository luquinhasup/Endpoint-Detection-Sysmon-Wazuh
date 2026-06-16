# 🛡️ Endpoint Security Lab: Integração Sysmon e Wazuh SIEM

## 🎯 Objetivo do Projeto
Laboratório prático focado em monitoramento avançado de endpoints (EDR). O objetivo foi implantar o Microsoft Sysmon em um ambiente Windows, otimizar a captura de eventos de segurança e integrar os logs ao SIEM (Wazuh) para detectar táticas de persistência, escalonamento de privilégios e evasão de defesas.

## 🛠️ Arquitetura e Configuração
* **Endpoint Monitorado:** Windows 11 Pro.
* **Telemetria (Sysmon):** Instalação do Sysmon utilizando a configuração otimizada de mercado **SwiftOnSecurity**, filtrando o ruído do sistema e alertando apenas para eventos de Severidade Nível 3 ou superior.
* **Ingestão de Logs (Wazuh Agent):** Configuração do arquivo `ossec.conf` para capturar o canal `Microsoft-Windows-Sysmon/Operational` no formato `eventchannel`.

  <img width="1444" height="1105" alt="Captura de Tela 2026-06-16 às 03 04 45" src="https://github.com/user-attachments/assets/9abac22d-2f3b-4291-9f2e-b136c2cf8ea0" />


## ⚔️ Simulação de Ataque (Red Team)
Para validar as regras de detecção, simulei táticas reais utilizadas por atacantes após o comprometimento inicial do host:
1. **Persistência e Escalonamento de Privilégios:** Criação de um usuário backdoor oculto (`net user hacker /add`) e inserção direta no grupo de Administradores locais (`net localgroup Administradores hacker /add`).
2. **Evasão de Defesas (Limpeza de Rastros):** Exclusão do usuário malicioso logo após a execução da tarefa (`net user hacker /delete`) e limpeza dos registros do Visualizador de Eventos (`wevtutil cl Application`).
3. **Bypass de Execução:** Tentativa de execução de comandos em PowerShell burlando a política restritiva nativa (`ExecutionPolicy Bypass`).

## 🔎 Análise e Estratégia de Defesa (Blue Team)
O SIEM capturou os eventos críticos com sucesso (ex: Event ID 1 do Sysmon e Eventos 4722/104 do Windows). Com base na análise do comportamento, propus as seguintes melhorias para a maturidade do SOC:

<img width="1710" height="1107" alt="Captura de Tela 2026-06-16 às 03 05 22" src="https://github.com/user-attachments/assets/39cc017e-63a2-4e0a-94a0-7025bd3259b5" />


* **Correlação de Eventos (Event Correlation):** Criação de uma regra customizada no Wazuh para elevar a severidade do alerta (ex: Nível 14) caso os eventos de criação, elevação de privilégio e exclusão de usuário ocorram em uma janela inferior a 10 minutos.
* **Automação de Resposta (Active Response):** Configuração do SIEM para isolar a máquina da rede automaticamente e encerrar a sessão ativa caso a regra de limpeza de logs (`wevtutil cl`) seja disparada.
