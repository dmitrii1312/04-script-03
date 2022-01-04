## Обязательная задача 1
Мы выгрузили JSON, который получили через API запрос к нашему сервису:
```
    { "info": "Sample JSON output from our service\t",
        "elements": [
            { "name": "first",
            "type": "server",
            "ip": "71.75.22.43"
            },
            { "name": "second",
            "type": "proxy",
            "ip": "71.78.22.43"
            }
        ]
    }
```
  Нужно найти и исправить все ошибки, которые допускает наш сервис

## Обязательная задача 2
В прошлый рабочий день мы создавали скрипт, позволяющий опрашивать веб-сервисы и получать их IP. К уже реализованному функционалу нам нужно добавить возможность записи JSON и YAML файлов, описывающих наши сервисы. Формат записи JSON по одному сервису: `{ "имя сервиса" : "его IP"}`. Формат записи YAML по одному сервису: `- имя сервиса: его IP`. Если в момент исполнения скрипта меняется IP у сервиса - он должен так же поменяться в yml и json файле.

### Ваш скрипт:
```python
#!/usr/bin/env python3
import socket
import time
import os
import json
import yaml

files = {'drive', 'mail', 'google'}
services = {}
for key in files:
  if os.path.isfile(key+'.yaml'):
    with open(key+'.yaml','r') as ym:
      srv=yaml.safe_load(ym)
      for key, value in srv[0].items():
        services[key]=value
  elif os.path.isfile(key+'.json'):
    with open(key+'.json','r') as js:
      srv=json.load(js)
      for key, value in srv.items():
        services[key]=value

while True:
  for key, value in services.items():
    newip=socket.gethostbyname(key)
    if newip!=value:
      print("[ERROR] {} IP mismatch: {} {}".format(key,value,newip))
      services[key]=newip
      fdict=[]
      fdict.append({key: newip})
      with open(key.split(".")[0]+'.json', 'w') as js:
        json.dump(fdict[0],js)
      with open(key.split(".")[0]+'.yaml', 'w') as ym:
        ym.write(yaml.dump(fdict,explicit_start=True,explicit_end=True))
    else:
      print("{} - {}".format(key,value))
  time.sleep(5)
```

### Вывод скрипта при запуске при тестировании:
```
$ ./task2.py
[ERROR] google.com IP mismatch: 173.14.222.102 64.233.165.113
[ERROR] mail.google.com IP mismatch: 173.14.221.18 173.194.221.18
[ERROR] drive.google.com IP mismatch: 173.194.7.194 173.194.73.194
[ERROR] google.com IP mismatch: 64.233.165.113 64.233.165.101
mail.google.com - 173.194.221.18
drive.google.com - 173.194.73.194
google.com - 64.233.165.101
mail.google.com - 173.194.221.18
drive.google.com - 173.194.73.194
^CTraceback (most recent call last):
  File "./task2.py", line 36, in <module>
    time.sleep(5)
KeyboardInterrupt
```

### json-файл(ы), который(е) записал ваш скрипт:
```json
$ cat drive.json
{"drive.google.com": "173.194.73.194"}
$ cat mail.json
{"mail.google.com": "173.194.221.18"}
$ cat google.json
{"google.com": "64.233.165.101"}
```

### yml-файл(ы), который(е) записал ваш скрипт:
```yaml
$ cat drive.yaml
---
- drive.google.com: 173.194.73.194
...
$ cat mail.yaml
---
- mail.google.com: 173.194.221.18
...
$ cat google.yaml
---
- google.com: 64.233.165.101
...
```
