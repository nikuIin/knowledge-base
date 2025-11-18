#python 

[[Чистота кода]]
[[Python.canvas|Python canvas]]
[[https://thecode.media/guard-clauses/]]

Паттерн `guards` помогает повысить читаемость кода при большой вложенности проверок. 
Большая вложенность понижает читаемость кода, а иногда даже прикодится скролить горизонтально, чтобы понять, что вообще происходит. Не дело. 
Выход: **инвертация проверок**

Было:

```python
if wifi:
	if login:
		if admin:
			...
		else:
			raise AdminError()
	else:
		raise LoginError()
else:
	raise WifiError()
```

Стало:

```python
if not !wifi:
	raise WifiError()
if not !login:
	raise LoginError()
if not admin:
	raise AdminError()
```
