# Cross-compiling Python 3.5.5 for ARM-kernel on Linux

## 1. Введение
  В данном проекте последовательно изложены шаги сборки и кросс-компиляции интерпретатора Python 3.5.5 для ядра ARM. Кросс-компиляция проводилась на ОС Ubuntu 12.04, смонитированной на виртуальной машине. Интерпретатор предназначен для работы в ОС Linux на промышленном контроллере Adam-3600 (далее - PLC), но возможно применение и на других контроллерах (не знаю, не проверял).
## 2. Зависимости
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
$ sudo apt-get install dff
$ reboot
```
Проверить версию arm-linux-gnueabihf-gcc командой `arm-linux-gnueabihf-gcc --version`. Версия должна быть 4.6.3.
## 3. Кросс-компиляция OpenSSL 1.0.1t
```
$ cd $HOME
wget https://www.openssl.org/source/old/1.0.1/openssl-1.0.1t.tar.gz
tar xvzf openssl-1.0.1t.tar.gz
$ cd openssl-1.0.1t
$ ./Configure linux-generic32 shared --cross-compile-prefix=arm-linux-gnueabihf-
$ make
$ sudo make install
$ mkdir lib
$ cp ./*.{so,so.1.0.0,a,pc} ./lib
```
## 4. Кросс-компиляция и сборка Python-3.5.5
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
$ export PATH=/home/rashid/PythonSrc/PythonHost/bin:$PATH
$ export OPENSSL_ROOT=/home/rashid/openssl-1.0.1t
$ ./configure --host=arm-linux-gnueabihf --target=arm-linux-gnueabihf --build=x86_64-linux-gnu --prefix=$HOME/PythonSrc/PythonTarget --disable-ipv6 --enable-shared
$ export HOSTPYTHON=$HOME/PythonSrc/Python-3.5.5-host/python3
$ export HOSTPGEN=$HOME/Python-3.5.5-host/Parser/pgen
$ export BLDSHARED="arm-linux-gnueabihf-gcc -shared"
$ export CROSS_COMPILE=arm-linux-gnueabihf-
$ export CROSS_COMPILE_TARGET=yes
$ export HOSTARCH=arm-linux
$ export BUILDARCH=arm-linux-gnueabihf
```
Откорректировать файлы `/usr/rashid/Python-3.5.5/Modules/Setup.dist` и `/usr/rashid/Python-3.5.5/setup.py`.

Далее
```
$ make
$ make install
```
Скомпилированный проект должен появиться в директории `$HOME/PythonSrc/PythonTarget/`
## 5. Запуск интерпретатора на Adam-3600
В директории `$HOME/PythonSrc/PythonTarget/` размещены четыре каталога - `/bin, /include, /lib, /share`. Я скопировал содержимое каждой директории в одноименные директории на PLC - `/usr/bin, /usr/include, /usr/lib, /usr/share` соотвественно, за исключением каталога `/lib/python3.5/test`, ввиду того что на PLC не так много места, да и надобности этого каталога нету.

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
## 6. Дополнительные опции
### 6.1 Обмен данными по последовательному интерфейсу 232/485
В PLC предусмотрены 3 последовательных порта (один 232 и два 485). Для работы с этими портами необходимы библиотеки `pyserial, future, iso8601, yaml`.
### 6.2 Обмен данными по протоколу MODBUS-RTU
Для работы по протоколу MODBUS-RTU я использовал библиотеку `minimalmodbus`. Необходимо учитывать что должны быть установлены библиотеки из предыдущего пункта.
