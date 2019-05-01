# Cross-compiling Python 3.5.5 for ARM-kernel on Linux

## Введение
  В данном проекте последовательно изложены шаги сборки и кросс-компиляции интерпретатора Python 3.5.5 для ядра ARM. Кросс-компиляция проводилась на ОС Ubuntu 12.04, смонитированной на виртуальной машине. Интерпретатор предназначен для работы в ОС Linux на промышленном контроллере Adam-3600, но возможно применение и на других контроллерах (не знаю, не проверял).
## Зависимости
Провести инсталляцию обновлений
```
$ sudo apt-get install update
$ sudo apt-get install upgrade
```
Инсталлировать следующие библиотеки
```
$ sudo apt-get install libssl-dev
$ sudo apt-get install libffi-dev
$ sudo apt-get install gcc-arm-linux-gnueabihf
```
## Кросс-компиляция и сборка
Выполнить последовательно команды
```
$ cd $HOME
$ mkdir PythonSrc
$ cd PythonSrc
$ wget https://www.python.org/ftp/python/3.5.5/Python-3.5.5.tgz
$ tar zxf Python-3.5.5.tgz
$ mv Python-3.5.5 Python-3.5.5-host
$ cd Python-3.5.5-host
$ ./configure --prefix=$HOME/PythonSrc/PythonHost
$ make python Parser/pgen
$ make install
```
