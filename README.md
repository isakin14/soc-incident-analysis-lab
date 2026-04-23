
# SOC Incident Analysis Lab

<img width="582" height="768" alt="Screenshot 2026-04-22 213927" src="https://github.com/user-attachments/assets/304ac264-0d9d-4e71-a667-43f275f9a1ff" /> <img width="584" height="766" alt="Screenshot 2026-04-22 213958" src="https://github.com/user-attachments/assets/4ee1f75b-b9cf-4e8d-8fce-7b69e2c3620c" />

## Overview

Este projeto simula a análise de um incidente de segurança em um ambiente corporativo, utilizando logs do Sysmon para identificar atividades maliciosas, reconstruir a cadeia de ataque e propor ações de resposta.

O objetivo é demonstrar habilidades práticas de:

- Análise de logs
- Detecção de comportamento malicioso
- Resposta a incidentes

---

## Scenario

Um usuário do setor de RH (`hr.user`) executa arquivos baixados da internet.  
Durante a análise dos logs, foi identificado um possível comprometimento após a execução de um arquivo com macro.

---

## Log Source

- Ferramenta: Sysmon
- Eventos analisados:
    - Process Creation (Event ID 1)
    - Network Connection (Event ID 3)
    - File Creation (Event ID 11)
    - Registry Modification (Event ID 13)

---

## Key Findings

### Suspicious Activity Identified

- Execução de arquivo com macro:
    
    `invoice_april.docm`
    
- Cadeia de execução suspeita:
    
    `winword.exe → cmd.exe → powershell.exe`
    
- Uso de Windows PowerShell com parâmetros maliciosos:
    - `-nop`
    - `-w hidden`
    - `-enc` (comando ofuscado em Base64)
- Conexões externas identificadas:
    - `51.128.19.24:80`
    - `185.77.216.55:443`
- Criação de script malicioso:
    
    `C:\Users\Public\sysupdate.ps1`
    
- Persistência via registro:
    
    `HKCU\Software\Microsoft\Windows\CurrentVersion\Run\SysUpdate`
    

---

## Attack Chain

A cadeia de ataque observada segue o padrão:

1. Usuário executa arquivo `.docm` malicioso
2. Macro inicia execução de comandos via `cmd.exe`
3. Execução de PowerShell ofuscado (Base64)
4. Download de payload remoto
5. Criação de script `.ps1` no sistema
6. Execução com bypass de política de segurança
7. Comunicação com servidor externo (C2)
8. Estabelecimento de persistência no sistema

---

## Incident Classification

- **Severity:** Critical
- **Type:** Malware Infection (Macro-based Execution + Persistence)
- **Technique Highlights:**
    - Macro Execution
    - Living-off-the-Land (PowerShell)
    - Persistence via Registry
    - External C2 Communication

---

## Response Actions

### Containment

- Isolamento da máquina afetada
- Bloqueio de IPs maliciosos em firewall

### Investigation

- Análise do script `sysupdate.ps1`
- Decodificação do comando PowerShell em Base64
- Verificação de outros hosts comprometidos

### Eradication

- Remoção de arquivos maliciosos
- Remoção da persistência no registro
- Varredura completa com antivírus/EDR

### Post-Incident

- Reset de credenciais do usuário
- Treinamento de conscientização (phishing)
- Criação de regras de detecção para comportamento similar

---

## Skills Demonstrated

- Log Analysis (Sysmon)
- Threat Detection
- Incident Response
- Malware Behavior Analysis
- Windows Internals (Processes, Registry, Execution Flow)

---

## Notes

Este projeto foi desenvolvido com fins educacionais, simulando um cenário realista de atuação em um SOC (Security Operations Center).
