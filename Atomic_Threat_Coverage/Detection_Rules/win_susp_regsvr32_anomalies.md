| Title                    | Regsvr32 Anomaly       |
|:-------------------------|:------------------|
| **Description**          | Detects various anomalies in relation to regsvr32.exe |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li><li>[TA0002: Execution](https://attack.mitre.org/tactics/TA0002)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1117: Regsvr32](https://attack.mitre.org/techniques/T1117)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1117: Regsvr32](../Triggers/T1117.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Unknown</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://subt0x10.blogspot.de/2017/04/bypass-application-whitelisting-script.html](https://subt0x10.blogspot.de/2017/04/bypass-application-whitelisting-script.html)</li></ul>  |
| **Author**               | Florian Roth |
| Other Tags           | <ul><li>car.2019-04-002</li><li>car.2019-04-003</li></ul> | 

## Detection Rules

### Sigma rule

```
title: Regsvr32 Anomaly
id: 8e2b24c9-4add-46a0-b4bb-0057b4e6187d
status: experimental
description: Detects various anomalies in relation to regsvr32.exe
author: Florian Roth
date: 2019/01/16
references:
    - https://subt0x10.blogspot.de/2017/04/bypass-application-whitelisting-script.html
tags:
    - attack.t1117
    - attack.defense_evasion
    - attack.execution
    - car.2019-04-002
    - car.2019-04-003
logsource:
    category: process_creation
    product: windows
detection:
    selection1:
        Image: '*\regsvr32.exe'
        CommandLine: '*\Temp\\*'
    selection2:
        Image: '*\regsvr32.exe'
        ParentImage: '*\powershell.exe'
    selection3:
        Image: '*\regsvr32.exe'
        ParentImage: '*\cmd.exe'
    selection4:
        Image: '*\regsvr32.exe'
        CommandLine:
            - '*/i:http* scrobj.dll'
            - '*/i:ftp* scrobj.dll'
    selection5:
        Image: '*\wscript.exe'
        ParentImage: '*\regsvr32.exe'
    selection6:
        Image: '*\EXCEL.EXE'
        CommandLine: '*..\..\..\Windows\System32\regsvr32.exe *'
    condition: 1 of them
fields:
    - CommandLine
    - ParentCommandLine
falsepositives:
    - Unknown
level: high

```





### powershell
    
```
Get-WinEvent | where {(($_.message -match "Image.*.*\\regsvr32.exe" -and $_.message -match "CommandLine.*.*\\Temp\\.*") -or ($_.message -match "Image.*.*\\regsvr32.exe" -and $_.message -match "ParentImage.*.*\\powershell.exe") -or ($_.message -match "Image.*.*\\regsvr32.exe" -and $_.message -match "ParentImage.*.*\\cmd.exe") -or ($_.message -match "Image.*.*\\regsvr32.exe" -and ($_.message -match "CommandLine.*.*/i:http.* scrobj.dll" -or $_.message -match "CommandLine.*.*/i:ftp.* scrobj.dll")) -or ($_.message -match "Image.*.*\\wscript.exe" -and $_.message -match "ParentImage.*.*\\regsvr32.exe") -or ($_.message -match "Image.*.*\\EXCEL.EXE" -and $_.message -match "CommandLine.*.*..\\..\\..\\Windows\\System32\\regsvr32.exe .*")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
((winlog.event_data.Image.keyword:*\\regsvr32.exe AND winlog.event_data.CommandLine.keyword:*\\Temp\\*) OR (winlog.event_data.Image.keyword:*\\regsvr32.exe AND winlog.event_data.ParentImage.keyword:*\\powershell.exe) OR (winlog.event_data.Image.keyword:*\\regsvr32.exe AND winlog.event_data.ParentImage.keyword:*\\cmd.exe) OR (winlog.event_data.Image.keyword:*\\regsvr32.exe AND winlog.event_data.CommandLine.keyword:(*\/i\:http*\ scrobj.dll OR *\/i\:ftp*\ scrobj.dll)) OR (winlog.event_data.Image.keyword:*\\wscript.exe AND winlog.event_data.ParentImage.keyword:*\\regsvr32.exe) OR (winlog.event_data.Image.keyword:*\\EXCEL.EXE AND winlog.event_data.CommandLine.keyword:*..\\..\\..\\Windows\\System32\\regsvr32.exe\ *))
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/8e2b24c9-4add-46a0-b4bb-0057b4e6187d <<EOF
{
  "metadata": {
    "title": "Regsvr32 Anomaly",
    "description": "Detects various anomalies in relation to regsvr32.exe",
    "tags": [
      "attack.t1117",
      "attack.defense_evasion",
      "attack.execution",
      "car.2019-04-002",
      "car.2019-04-003"
    ],
    "query": "((winlog.event_data.Image.keyword:*\\\\regsvr32.exe AND winlog.event_data.CommandLine.keyword:*\\\\Temp\\\\*) OR (winlog.event_data.Image.keyword:*\\\\regsvr32.exe AND winlog.event_data.ParentImage.keyword:*\\\\powershell.exe) OR (winlog.event_data.Image.keyword:*\\\\regsvr32.exe AND winlog.event_data.ParentImage.keyword:*\\\\cmd.exe) OR (winlog.event_data.Image.keyword:*\\\\regsvr32.exe AND winlog.event_data.CommandLine.keyword:(*\\/i\\:http*\\ scrobj.dll OR *\\/i\\:ftp*\\ scrobj.dll)) OR (winlog.event_data.Image.keyword:*\\\\wscript.exe AND winlog.event_data.ParentImage.keyword:*\\\\regsvr32.exe) OR (winlog.event_data.Image.keyword:*\\\\EXCEL.EXE AND winlog.event_data.CommandLine.keyword:*..\\\\..\\\\..\\\\Windows\\\\System32\\\\regsvr32.exe\\ *))"
  },
  "trigger": {
    "schedule": {
      "interval": "30m"
    }
  },
  "input": {
    "search": {
      "request": {
        "body": {
          "size": 0,
          "query": {
            "bool": {
              "must": [
                {
                  "query_string": {
                    "query": "((winlog.event_data.Image.keyword:*\\\\regsvr32.exe AND winlog.event_data.CommandLine.keyword:*\\\\Temp\\\\*) OR (winlog.event_data.Image.keyword:*\\\\regsvr32.exe AND winlog.event_data.ParentImage.keyword:*\\\\powershell.exe) OR (winlog.event_data.Image.keyword:*\\\\regsvr32.exe AND winlog.event_data.ParentImage.keyword:*\\\\cmd.exe) OR (winlog.event_data.Image.keyword:*\\\\regsvr32.exe AND winlog.event_data.CommandLine.keyword:(*\\/i\\:http*\\ scrobj.dll OR *\\/i\\:ftp*\\ scrobj.dll)) OR (winlog.event_data.Image.keyword:*\\\\wscript.exe AND winlog.event_data.ParentImage.keyword:*\\\\regsvr32.exe) OR (winlog.event_data.Image.keyword:*\\\\EXCEL.EXE AND winlog.event_data.CommandLine.keyword:*..\\\\..\\\\..\\\\Windows\\\\System32\\\\regsvr32.exe\\ *))",
                    "analyze_wildcard": true
                  }
                }
              ],
              "filter": {
                "range": {
                  "timestamp": {
                    "gte": "now-30m/m"
                  }
                }
              }
            }
          }
        },
        "indices": [
          "winlogbeat-*"
        ]
      }
    }
  },
  "condition": {
    "compare": {
      "ctx.payload.hits.total": {
        "not_eq": 0
      }
    }
  },
  "actions": {
    "send_email": {
      "email": {
        "to": "root@localhost",
        "subject": "Sigma Rule 'Regsvr32 Anomaly'",
        "body": "Hits:\n{{#ctx.payload.hits.hits}}Hit on {{_source.@timestamp}}:\n      CommandLine = {{_source.CommandLine}}\nParentCommandLine = {{_source.ParentCommandLine}}================================================================================\n{{/ctx.payload.hits.hits}}",
        "attachments": {
          "data.json": {
            "data": {
              "format": "json"
            }
          }
        }
      }
    }
  }
}
EOF

```


### graylog
    
```
((Image.keyword:*\\regsvr32.exe AND CommandLine.keyword:*\\Temp\\*) OR (Image.keyword:*\\regsvr32.exe AND ParentImage.keyword:*\\powershell.exe) OR (Image.keyword:*\\regsvr32.exe AND ParentImage.keyword:*\\cmd.exe) OR (Image.keyword:*\\regsvr32.exe AND CommandLine.keyword:(*\/i\:http* scrobj.dll *\/i\:ftp* scrobj.dll)) OR (Image.keyword:*\\wscript.exe AND ParentImage.keyword:*\\regsvr32.exe) OR (Image.keyword:*\\EXCEL.EXE AND CommandLine.keyword:*..\\..\\..\\Windows\\System32\\regsvr32.exe *))
```


### splunk
    
```
((Image="*\\regsvr32.exe" CommandLine="*\\Temp\\*") OR (Image="*\\regsvr32.exe" ParentImage="*\\powershell.exe") OR (Image="*\\regsvr32.exe" ParentImage="*\\cmd.exe") OR (Image="*\\regsvr32.exe" (CommandLine="*/i:http* scrobj.dll" OR CommandLine="*/i:ftp* scrobj.dll")) OR (Image="*\\wscript.exe" ParentImage="*\\regsvr32.exe") OR (Image="*\\EXCEL.EXE" CommandLine="*..\\..\\..\\Windows\\System32\\regsvr32.exe *")) | table CommandLine,ParentCommandLine
```


### logpoint
    
```
((Image="*\\regsvr32.exe" CommandLine="*\\Temp\\*") OR (Image="*\\regsvr32.exe" ParentImage="*\\powershell.exe") OR (Image="*\\regsvr32.exe" ParentImage="*\\cmd.exe") OR (Image="*\\regsvr32.exe" CommandLine IN ["*/i:http* scrobj.dll", "*/i:ftp* scrobj.dll"]) OR (Image="*\\wscript.exe" ParentImage="*\\regsvr32.exe") OR (Image="*\\EXCEL.EXE" CommandLine="*..\\..\\..\\Windows\\System32\\regsvr32.exe *"))
```


### grep
    
```
grep -P '^(?:.*(?:.*(?:.*(?=.*.*\regsvr32\.exe)(?=.*.*\Temp\\.*))|.*(?:.*(?=.*.*\regsvr32\.exe)(?=.*.*\powershell\.exe))|.*(?:.*(?=.*.*\regsvr32\.exe)(?=.*.*\cmd\.exe))|.*(?:.*(?=.*.*\regsvr32\.exe)(?=.*(?:.*.*/i:http.* scrobj\.dll|.*.*/i:ftp.* scrobj\.dll)))|.*(?:.*(?=.*.*\wscript\.exe)(?=.*.*\regsvr32\.exe))|.*(?:.*(?=.*.*\EXCEL\.EXE)(?=.*.*\.\.\\.\.\\.\.\Windows\System32\regsvr32\.exe .*))))'
```



