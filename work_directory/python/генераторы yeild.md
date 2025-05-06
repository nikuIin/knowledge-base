
```python
with open("path\log.txt", "r") as log_file:
     err_gen = (st for st in log_file if "error" in st)
     for item in err_gen:
         <обработка строки item>
```

У нас создается тут генератор. Но он начинает искать в строке слово ERROR только в момент итерации. То есть при инициализации генератора по факту он ничего не содержит. 