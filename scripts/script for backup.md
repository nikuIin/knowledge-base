
```bash
#!/bin/bash
while IFS= read -r line; do
  URL=$line && curl -X GET $URL >> $(echo $URL | tr / _ | sed 's/https:__insperia.ru_//' | sed 's/$/\.html/')
done < $1
```