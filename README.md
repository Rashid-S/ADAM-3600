# Cross-compiling Python 3.5.5 for ARM-kernel on Linux

## Введение
  В данном проекте последовательно изложены шаги сборки и кросс-компиляции интерпретатора Python 3.5.5 для ядра ARM. Кросс-компиляция проводилась на ОС Ubuntu 12.04, смонитированной на виртуальной машине. Интерпретатор предназначен для работы в ОС Linux на промышленном контроллере Adam-3600 (далее - PLC), но возможно применение и на других контроллерах (не знаю, не проверял).
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
```
$ cd $HOME
$ wget https://www.python.org/ftp/python/3.5.5/Python-3.5.5.tgz
$ tar zxf Python-3.5.5.tgz
$ cd Python-3.5.5
$ export CC=arm-linux-gnueabihf-gcc
$ export CXX=arm-linux-gnueabihf-g++
$ export AR=arm-linux-gnueabihf-ar
$ export RANLIB=arm-linux-gnueabihf-ranlib
$ export ac_cv_file__dev_ptmx=no
$ export ac_cv_file__dev_ptc=no
$ export ac_cv_have_long_long_format=yes
$ export PATH=/home/USER/PythonSrc/PythonHost/bin:$PATH
$ ./configure --host=arm-linux-gnueabihf --target=arm-linux-gnueabihf --build=x86_64-linux-gnu --prefix=$HOME/PythonSrc/PythonTarget --disable-ipv6 --enable-shared
$ export HOSTPYTHON=$HOME/PythonSrc/Python-3.5.5-host/python3
$ export HOSTPGEN=$HOME/Python-3.5.5-host/Parser/pgen
$ export BLDSHARED="arm-linux-gnueabihf-gcc -shared"
$ export CROSS_COMPILE=arm-linux-gnueabihf-
$ export CROSS_COMPILE_TARGET=yes
$ export HOSTARCH=arm-linux
$ export BUILDARCH=arm-linux-gnueabihf
$ make
$ make install
```
Скомпилированный проект должен появиться в директории `$HOME/PythonSrc/PythonTarget/`
## Запуск интерпретатора на Adam-3600
В директории `$HOME/PythonSrc/PythonTarget/` размещены четыре каталога - `/bin, /include, /lib, /share`. Я скопировал содержимое каждой директории в одноименные директории на PLC - `/usr/bin, /usr/include, /usr/lib, /usr/share` соотвественно.

Войти в директорий `/usr/bin`. Выполнить в командной строке
```
chmod +x python3.5
```
Далее
```
python3.5
```
В ответ в командной строке должно появиться приглашение интерпретатора
```
Python 3.5.5 (default, Apr 28 2019, 20:19:45)
[GCC 4.6.3] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>

```
