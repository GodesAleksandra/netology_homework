
## Обязательная задача 1
Мы выгрузили JSON, который получили через API запрос к нашему сервису:
```
    { "info" : "Sample JSON output from our service\t",
        "elements" :[
            { "name" : "first",
            "type" : "server",
            "ip" : 7175 
            }
            { "name" : "second",
            "type" : "proxy",
            "ip : 71.78.22.43
            }
        ]
    }
```
  Нужно найти и исправить все ошибки, которые допускает наш сервис
  
Правильно:
```
    { "info" : "Sample JSON output from our service\t",
        "elements" :[
            { "name" : "first",
            "type" : "server",
            "ip" : 7175 
            },
            { "name" : "second",
            "type" : "proxy",
            "ip" : "71.78.22.43"
            }
        ]
    }
```

## Обязательная задача 2
В прошлый рабочий день мы создавали скрипт, позволяющий опрашивать веб-сервисы и получать их IP. К уже реализованному функционалу нам нужно добавить возможность записи JSON и YAML файлов, описывающих наши сервисы. Формат записи JSON по одному сервису: `{ "имя сервиса" : "его IP"}`. Формат записи YAML по одному сервису: `- имя сервиса: его IP`. Если в момент исполнения скрипта меняется IP у сервиса - он должен так же поменяться в yml и json файле.

### Ваш скрипт:
```python
#!/usr/bin/env python3

import socket as s
import json
import yaml

file_json = 'servers.json'
file_yml = 'servers.yml'
servers = ['drive.google.com', 'mail.google.com', 'google.com']
error = ''
data = {}

with open(file_json, 'r') as outfile:
        file = json.load(outfile)

for sv in servers:
        ip_cur = s.gethostbyname(sv)
        ip_prev = file[sv]
        data[sv] = ip_cur
        if ip_prev != ip_cur:
                error = error + ', ' + sv + ' IP mismatch: ' + ip_prev + ' ' + ip_cur

if error:
        with open(file_yml, "w") as write_file:
                yaml.dump(data, write_file)
        with open(file_json, "w") as write_file:
                json.dump(data, write_file)
        print('[ERROR]'+error[1:])
```

### Вывод скрипта при запуске при тестировании:
```
[ERROR] drive.google.com IP mismatch: 216.58.212.142 142.250.186.110, mail.google.com IP mismatch: 142.250.74.197 142.250.186.133, google.com IP mismatch: 142.250.186.110 172.217.16.142
```

### json-файл(ы), который(е) записал ваш скрипт:
```json
{"drive.google.com": "142.250.186.110", "mail.google.com": "142.250.186.133", "google.com": "172.217.16.142"}
```

### yml-файл(ы), который(е) записал ваш скрипт:
```yaml
drive.google.com: 142.250.186.110
google.com: 172.217.16.142
mail.google.com: 142.250.186.133
```
