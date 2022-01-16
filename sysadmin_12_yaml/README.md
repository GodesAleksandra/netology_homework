
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

file = 'servers.json'
servers = ['drive.google.com', 'mail.google.com', 'google.com']
error = ''
data_json = ''

with open(file, 'r') as outfile:
        file_json = json.load(outfile)

for sv in servers:
        ip_cur = s.gethostbyname(sv)
        ip_prev = file_json[sv]
        data_json = data_json + '"' + sv + '":"' + ip_cur + '",'
        if ip_prev != ip_cur:
                error = error + ', ' + sv + ' IP mismatch: ' + ip_prev + ' ' + ip_cur

if error:
        with open(file, "w") as write_file:
                write_file.write('{'+data_json[:-1]+'}')
        print('[ERROR]'+error[1:])
```

### Вывод скрипта при запуске при тестировании:
```
???
```

### json-файл(ы), который(е) записал ваш скрипт:
```json
???
```

### yml-файл(ы), который(е) записал ваш скрипт:
```yaml
???
```
