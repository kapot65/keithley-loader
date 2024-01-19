попытка подключить Keihtley KUSB-488 к компьютеру

## Установка
1. Драйвер
```shell
cd linux-gpib-4.3.6/linux-gpib-kernel-4.3.6
make 
sudo make install
```
1. библиотека
```shell
cd linux-gpib-4.3.6/linux-gpib-user-4.3.6
./configure
make 
sudo make install
```

## запуск
1. загрузка firmware в Keihtley KUSB-488
	1. ручная установка
		```shell
		# установить если не установлен
		sudo apt install fxload

		cd linux_gpib_firmware/ni_gpib_usb_b
		# адрес устройства можно узнать командой lsusb
		sudo fxload -D /dev/bus/usb/001/006 -I niusbb_firmware.hex -s niusbb_loader.hex
		```
		после этого дожна загорется зеленая лампочка READY на Keihtley KUSB-488
	2. автоматическая установка
		1. переместить файлы прошивки в `/usr/local/share/usb/ni_usb_gpib/`
			```shell
			sudo cp linux_gpib_firmware/ni_gpib_usb_b/niusbb_firmware.hex /usr/local/share/usb/ni_usb_gpib/
			sudo cp linux_gpib_firmware/ni_gpib_usb_b/niusbb_loader.hex /usr/local/share/usb/ni_usb_gpib/
			```
		2. скопировать rules в `/etc/udev/rules.d/`
			```shell
			sudo cp /usr/local/etc/udev/rules.d/99-ni_usb_gpib.rules /etc/udev/rules.d/
			```
		После этого устройство должно автоматически загружаться при подключении к компьютеру

2. конфигурация linux-gpib (/usr/local/etc/gpib.conf)
	```c
	interface {
		minor = 0	/* board index, minor = 0 uses /dev/gpib0, minor = 1 uses /dev/gpib1, etc. */
		board_type = "ni_usb_b"	/* type of interface board being used */
		name = "violet"	/* optional name, allows you to get a board descriptor using ibfind() */
		pad = 0	/* primary address of interface             */
		sad = 0	/* secondary address of interface           */
		timeout = T3s	/* timeout for commands */

		eos = 0x0a	/* EOS Byte, 0xa is newline and 0xd is carriage return */
		set-reos = yes	/* Terminate read if EOS */
		set-bin = no	/* Compare EOS 8-bit */
		set-xeos = no	/* Assert EOI whenever EOS byte is sent */
		set-eot = yes	/* Assert EOI with last byte on writes */

	/* settings for boards that lack plug-n-play capability */
		base = 0	/* Base io ADDRESS                  */
		irq  = 0	/* Interrupt request level */
		dma  = 0	/* DMA channel (zero disables)      */

	/* pci_bus and pci_slot can be used to distinguish two pci boards supported by the same driver */
	/*	pci_bus = 0 */
	/*	pci_slot = 7 */

		master = yes	/* interface board is system controller */
	}

	device {
		minor = 0	/* minor number for interface board this device is connected to */
		name = "voltmeter"	/* device mnemonic */
		pad = 20	/* The Primary Address */
		sad = 0	/* Secondary Address */

		eos = 0xa	/* EOS Byte */
		set-reos = no /* Terminate read if EOS */
		set-bin = no /* Compare EOS 8-bit */
	}
	```
3. настройка прав доступа
	```shell
	# создание группы gpib и добавление в нее пользователя
	sudo groupadd gpib
	sudo usermod -aG gpib $USER
	# создание файла правил
	sudo nano /etc/udev/rules.d/99-gpib.rules
	# > KERNEL=="gpib[0-9]*", GROUP="gpib", MODE="0660"
	sudo reboot
	```
4. запуск конфигурации
	```shell
	# по какой-то причине gpib_config работает только с sudo
	sudo LD_LIBRARY_PATH=/usr/local/lib/ gpib_config
	```


## тест работы
```shell
cd linux-gpib-4.3.6/linux-gpib-user-4.3.6/examples
./ibtest
# подключиться к устройству (device) с адресом 20
# ввести команду (w) *IDN?
# считать ответ (r)
```