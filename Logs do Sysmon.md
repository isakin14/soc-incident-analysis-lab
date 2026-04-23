Como ler Logs do sistemas (Sysmon)

![[Event1Snippingtool.png]]
## Isso é um Event Viewer mostrando um evento do Sysmon
O programa no qual o Log registra o processo foi o SnippingTool (ferramenta de captura de tela do win11), como pode ver é um Evento 1 (no canto superior da tela), esse número denomina oque ocorreu nesse evento, o numero 1 significa que houve um processo criado (no qual foi uma printscreen).

## A maior parte dessas informações são totalmente descartada
Vc não necessariamente precisa ler tudo, vc filtra mentalmente (oque esta marcado em vermelho é oque te interessa)

1. Horário:
`UtcTime: 2026-04-21 16:21:49.056`
- Aqui você consegue averiguar quando foi realizado esse evento, caso esteja fora de um horário padrão de uso, é de suspeitar.

2. Processo Executado:
`Image: C:\Program Files\WindowsApps\Microsoft.ScreenSketch_11.2601.12.0_x64_8wekyb3d8bbwe\SnippingTool\SnippingTool.exe`
- O programa executado foi a ferramenta de recorte do Windows.

3. Como ele foi executado:
`CommandLine: "C:\Program Files\WindowsApps\Microsoft.ScreenSketch_11.2601.12.0_x64_8wekyb3d8bbwe\SnippingTool\SnippingTool.exe"`
- Aqui a execução define como foi executado dentro do terminal do dispositivo, caso houver parâmetros estranhos e flags suspeitas0, é para ficar alerta.

4. Caminho do arquivo:
`CurrentDirectory: C:\WINDOWS\System32\`
- Isso é MUITO importante pois é aonde o caminho do evento percorreu, aqui está um diretório legitimo do Windows, pois o arquivo utilizado foi o SnippingTool.exe oficial da Microsoft.

4. Quem executou:
`User: Isaki\isaki`
- Aqui é o usuário logado que executou o programa.

5. Integrity Level:
`IntegrityLevel: Medium`
- Aqui define a integridade donde foi rodado, nesse caso foi um usuário normal.

6. Hash do arquivo:
`Hashes: SHA256=00A2A0A0167BC8EF945E86349BBB70899B05A5E3CE7ACC7E9C9BBB9E33A58CD2`
- Isso serve para verificar a integridade do arquivo, sites como VirusTotal consegue comparar o Hash utilizado pelo arquivo para a verificação do malware, através de uma database de vírus existente.

7. Quem chamou:
`ParentImage: C:\Windows\explorer.exe`
- Aqui define de onde o arquivo foi chamado.

8. Categoria da tarefa: Process Create (rule: ProcessCreate)
- Existe diversas categoria de tarefas, cada uma com a sua função, aqui foi um processo todo criado.