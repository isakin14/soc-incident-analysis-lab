# SOC Cheat Sheet + Anotações referente ao log estudado

| O que olhar                          | Red flag (suspeito)                   | O que pode ser                  |
| ------------------------------------ | ------------------------------------- | ------------------------------- |
| **1. Caminho do executável (Image)** | `C:\Users\Public\`, `Temp`, `AppData` | Malware disfarçado              |
| **2. Nome do processo**              | `svchost.exe` fora do System32        | Masquerading                    |
| **3. CommandLine**                   | `-enc`, `-nop`, `hidden`              | Execução ofuscada               |
| **4. Processo pai (ParentImage)**    | `winword.exe → cmd.exe`               | Macro maliciosa                 |
| **5. PowerShell**                    | Execução silenciosa ou codificada     | Ataque “fileless”               |
| **6. Criação de arquivos**           | `.ps1`, `.vbs`, `.bat` em `Public`    | Dropper / script malicioso      |
| **7. Registro (persistência)**       | `HKCU\...\Run`                        | Malware iniciando com o sistema |
| **8. Conexão de rede**               | IP externo desconhecido               | C2 (Command & Control)          |
| **9. Horário**                       | Atividade 2h–4h da manhã              | Execução automatizada           |
| **10. Cadeia de eventos**            | Processo → rede → arquivo → registro  | Ataque completo                 |

## Quais eventos são suspeitos? (cite linhas ou trechos)?

Time: 2026-04-20 09:18:11 | EventID: 1 | ProcessCreate  
Image: C:\Windows\System32\winword.exe  
CommandLine: winword.exe C:\Users\hr.user\Downloads\invoice_april.docm  
ParentImage: explorer.exe

Time: 2026-04-20 09:17:30 | EventID: 1 | ProcessCreate  
Image: C:\Windows\System32\cmd.exe  
CommandLine: cmd.exe /c dir  
ParentImage: explorer.exe

Time: 2026-04-20 09:18:11 | EventID: 1 | ProcessCreate  
Image: C:\Windows\System32\winword.exe  
CommandLine: winword.exe C:\Users\hr.user\Downloads\invoice_april.docm  
ParentImage: explorer.exe

Time: 2026-04-20 09:18:15 | EventID: 1 | ProcessCreate  
Image: C:\Windows\System32\cmd.exe  
CommandLine: cmd.exe /c powershell -nop -w hidden -enc SQBFAFgAIAAoAE4AZQB3A...
ParentImage: winword.exe

Time: 2026-04-20 09:18:16 | EventID: 3 | NetworkConnect  
Image: powershell.exe  
DestinationIp: 51.128.19.24  
Port: 80

Time: 2026-04-20 09:18:18 | EventID: 11 | FileCreate  
TargetFilename: C:\Users\Public\sysupdate.ps1 
  
Time: 2026-04-20 09:18:21 | EventID: 3 | NetworkConnect  
Image: powershell.exe  
DestinationIp: 185.77.216.55  
Port: 443

Time: 2026-04-20 09:19:30 | EventID: 13 | RegistryValueSet  
TargetObject: HKCU\Software\Microsoft\Windows\CurrentVersion\Run\SysUpdate  
Details: C:\Users\Public\sysupdate.ps1

## Qual é a cadeia de ataque?

- Usuário abre:
    invoice_april.docm
    Documento com macro
    
- Microsoft Word executa:
    winword.exe → cmd.exe
    
- `cmd.exe` chama:
    `powershell -nop -w hidden -enc ...`
    
- PowerShell:
    - decodifica Base64
    - baixa conteúdo de:
        
        `51.128.19.24`
        
- Cria payload:
    `C:\Users\Public\sysupdate.ps1`
    
- Executa:
    `powershell -ExecutionPolicy Bypass`
    
- Nova conexão externa:
    `185.77.216.55`
    
- Persistência:
    `HKCU\...\Run\SysUpdate`

 ## O que é ruído (comportamento normal)?

Time: 2026-04-20 09:14:02 | EventID: 1 | ProcessCreate  
Image: C:\Windows\System32\chrome.exe  
ParentImage: C:\Windows\explorer.exe

Time: 2026-04-20 09:15:10 | EventID: 3 | NetworkConnect  
Image: chrome.exe  
DestinationIp: 142.250.78.14  
Port: 443

Time: 2026-04-20 09:17:02 | EventID: 1 | ProcessCreate  
Image: C:\Windows\System32\Teams.exe  
ParentImage: explorer.exe

Time: 2026-04-20 09:18:19 | EventID: 1 | ProcessCreate  
Image: C:\Windows\System32\powershell.exe  
CommandLine: powershell -ExecutionPolicy Bypass -File C:\Users\Public\sysupdate.ps1  
ParentImage: cmd.exe 

Time: 2026-04-20 09:19:05 | EventID: 1 | ProcessCreate  
Image: C:\Windows\System32\outlook.exe  
ParentImage: explorer.exe

Time: 2026-04-20 09:20:02 | EventID: 1 | ProcessCreate  
Image: C:\Windows\System32\notepad.exe  
ParentImage: explorer.exe

## Classificação do incidente
 - Severidade: CRÍTICA
 - Tipo: Malware (Initial Access + Execution + Persistence)

## Plano de resposta (nível SOC)

Fase 1 — Contenção imediata
- Isolar máquina da rede
- Bloquear IPs:
    - `51.128.19.24`
    - `185.77.216.55`
- Suspender conta do usuário (temporário)

Fase 2 — Investigação
- Coletar:
    - `invoice_april.docm`
    - `sysupdate.ps1`
- Decodificar Base64 do PowerShell
- Verificar:
    - outros hosts com mesmo comportamento
    - logs de proxy / firewall

Fase 3 — Erradicação
- Remover:
	- `sysupdate.ps1`
    - chave de registro `Run`
- Rodar antivírus / EDR
- Validar integridade do sistema

Fase 4 — Pós-incidente
- Reset de credenciais do usuário
- Treinamento de phishing
- Criar regra no SIEM:
`winword.exe → powershell.exe`
