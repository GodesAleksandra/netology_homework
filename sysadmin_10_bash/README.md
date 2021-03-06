## Обязательная задача 1

Есть скрипт:
```bash
a=1
b=2
c=a+b
d=$a+$b
e=$(($a+$b))
```

Какие значения переменным c,d,e будут присвоены? Почему?

| Переменная  | Значение | Обоснование |
| ------------- | ------------- | ------------- |
| `c`  | a+b | значением является строка a+b, т.к. чтобы вместо a и b подставились значения переменных, надо поставить перед ними знак $|
| `d`  | 1+2 | вместо $a и $b подставляются соответствующие переменные, но выражение не вычисляется |
| `e`  | 3 | вместо $a и $b подставляются соответствующие переменные и вычисляется выражение между двумя двойными круглыми скобками |


## Обязательная задача 2
На нашем локальном сервере упал сервис и мы написали скрипт, который постоянно проверяет его доступность, записывая дату проверок до тех пор, пока сервис не станет доступным (после чего скрипт должен завершиться). В скрипте допущена ошибка, из-за которой выполнение не может завершиться, при этом место на Жёстком Диске постоянно уменьшается. Что необходимо сделать, чтобы его исправить:
```bash
while ((1==1)
do
	curl https://localhost:4757
	if (($? != 0))
	then
		date >> curl.log
	fi
done
```

- Не хватает условие выхода из цикла, если сервер доступен;
- Не хватает второй закрывающей круглой скобки в условии while

```bash
while ((1==1))
do
	curl https://localhost:4757
	if (($? != 0))
	then
		date >> curl.log
	else exit
	fi
done
```

Необходимо написать скрипт, который проверяет доступность трёх IP: `192.168.0.1`, `173.194.222.113`, `87.250.250.242` по `80` порту и записывает результат в файл `log`. Проверять доступность необходимо пять раз для каждого узла.

### Ваш скрипт:
```bash
array_address=(192.168.0.1 173.194.222.113 87.250.250.242)
for addr in ${array_address[@]}
do
	n=5
	while (($n > 0))
	do
		curl $addr:80
		echo $addr check_status=$? >> hosts.log
		let "n -= 1"
	done
done
```

## Обязательная задача 3
Необходимо дописать скрипт из предыдущего задания так, чтобы он выполнялся до тех пор, пока один из узлов не окажется недоступным. Если любой из узлов недоступен - IP этого узла пишется в файл error, скрипт прерывается.

### Ваш скрипт:
```bash
array_address=(192.168.0.1 173.194.222.113 87.250.250.242)
status=0
while (($status == 0))
do
	for addr in ${array_address[@]}
	do
		curl $addr:80 >/dev/null
		status=$?
		if (($status != 0))
		then
			echo $addr error >> error.log
		fi
	done
done
```

