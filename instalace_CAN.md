# Návod na instalaci CAN desky od Fysetc a UCAN
- https://www.hotend.cz/upgrade-kity/587-voron-stealthburner-can-pcb-v11-kit-kabelaz.html
- https://www.hotend.cz/zakladni-desky/588-ucan-v10-usb-to-can.html

## Pozor na verzi desky! 
Tento návod platí pro desky ve verzi 1.1.  
**Pro aktuální verze 1.3 postupujte podle [návodu výrobce](https://wiki.fysetc.com/SB%20CAN%20ToolHead/?sscid=c1k7_12jhvq&).**

## Instalace potřebných balíčků
Do příkazové řádky zadej následující příkazy:
```bash
sudo apt-get install gcc-arm-none-eabi cmake dfu-util -y
```

# Instalace UCAN
Do příkazové řádky zadej následující příkazy:
```bash
git clone https://github.com/bigtreetech/candleLight_fw
cd candleLight_fw
mkdir build
cd build
cmake .. -DCMAKE_TOOLCHAIN_FILE=../cmake/gcc-arm-none-eabi-8-2019-q3-update.cmake
make
```

## Připojení UCAN v DFU modu
> ⚠️ Bezpečnostní upozornění: Během DFU režimu je BOOT pin nastaven na HIGH. Tento pin je sdílen s topný tělískem hotendu a způsobí jeho ohřev - odpojte napájení 24V při vstupu do DFU režimu.

1. Propoj 3V3 a B0
2. Připoj USB do UCAN

Do příkazové řádky zadej následující příkaz:
```bash
lsusb
```
Očekávaný výstup:
```
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 004: ID 1d50:614e OpenMoko, Inc. stm32f446xx
Bus 001 Device 013: ID 0483:df11 STMicroelectronics STM Device in DFU Mode ## UCAN v DFU MODU##
Bus 001 Device 002: ID 2109:3431 VIA Labs, Inc. Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

Do příkazové řádky zadej následující příkaz:
```bash
dfu-util -l
```
Očekávaný výstup:
```
dfu-util 0.9

Copyright 2005-2009 Weston Schmidt, Harald Welte and OpenMoko Inc.
Copyright 2010-2016 Tormod Volden and Stefan Schmidt
This program is Free Software and has ABSOLUTELY NO WARRANTY
Please report bugs to http://sourceforge.net/p/dfu-util/tickets/

Found DFU: [0483:df11] ver=2200, devnum=13, cfg=1, intf=0, path="1-1.1", alt=1, name="@Option Bytes  /0x1FFFF800/01*016 e", serial="FFFFFFFEFFFF"
Found DFU: [0483:df11] ver=2200, devnum=13, cfg=1, intf=0, path="1-1.1", alt=0, name="@Internal Flash  /0x08000000/064*0002Kg", serial="FFFFFFFEFFFF"
```

Do příkazové řádky zadej následující příkaz:
```bash
make flash-candleLight_fw
```

1. Odpoj USB z UCAN
2. Odstraň propojku 3V3 a B0
3. Připoj USB do UCAN

# Instalace Komunikace
Pro více informací navštiv [tento odkaz](https://maz0r.github.io/klipper_canbus/controller/canable.html)

Do příkazové řádky zadej následující příkazy:
```bash
cd ~
sudo nano /etc/network/interfaces.d/can0
```

Zadej tohle:
```
allow-hotplug can0
iface can0 can static
 bitrate 500000
 up ifconfig $IFACE txqueuelen 256
 pre-up ip link set can0 type can bitrate 500000
 pre-up ip link set can0 txqueuelen 256
```

Použij `CTRL+X` pro uložení.

Poté restartuj systém pomocí následujícího příkazu:
```bash
sudo reboot
```

# Instalace FYSTEC SB CAN TH
1. Propoj CAN desku a Raspberry
2. Připoj Napájecí kabel do CAN desky (+24V)

Do příkazové řádky zadej následující příkaz:
```bash
lsusb
```
Očekávaný výstup:
```
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 006: ID 0483:df11 STMicroelectronics STM Device in DFU Mode #### SB CAN TH ####
Bus 001 Device 005: ID 1d50:614e OpenMoko, Inc. stm32f446xx
Bus 001 Device 002: ID 2109:3431 VIA Labs, Inc. Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

Do příkazové řádky zadej následující příkazy:
```bash
cd ~/klipper/
make menuconfig
```
Ujisti se, že tvoje konfigurace odpovídá obrázku FYSTEC SB CAN TH Menu config

![image](/images/SB_CAN_ToolHEAD_menuconfig.png?raw=true)

Ukonči použitím ESC nebo Q, potvrď ano (Y).

Pokračuj zadáním následujících příkazů:
```bash
make
```

```bash
dfu-util -R -a 0 -s 0x08000000:leave -D out/klipper.bin
```

## Instalace CANBOOT
Pokračuj dále v instalaci podle následujícího postupu. Originál tohoto návodu najdeš na [tomto odkaze](https://maz0r.github.io/klipper_canbus/toolhead/sb_can_v1.1.html), za což děkuji jeho autorovi.

### Generování souboru firmware CANboot 
Klonuj repozitář CanBoot do svého Raspberry Pi:

```bash
cd ~/
git clone https://github.com/Arksine/CanBoot
cd CanBoot
make menuconfig
```

Nakonfiguruj svůj makefile pro FYSETC SB CAN TH s STM32F072

![image](/images/sb_can_v1.1_canboot_make.png)
  
Ukonči použitím ESC nebo Q, potvrď ano (Y).

### Sestavení firmware
```bash
make clean
make
```

### Připojení desky pro nahrání
Vypni desku SB-CAN-TH na alespoň 5 sekund odpojením kabelu CANBUS.

Připoj své zařízení k Raspberry Pi přes USB.

Připoj znovu napájecí zdroj / vstup CAN signálu.

Ověř, zda je zařízení v bootloader módu pomocí lsusb.
Měl bys vidět něco jako:
```bash
Bus 001 Device 005: ID 0483:df11 STMicroelectronics STM Device in DFU Mode
```
Nahraj bootloader canboot na desku. Tvoje DeviceID (0483:df11) může být jiné, ZKONTROLUJ TO! (viz krok 2)

#### VYMAZÁNÍ A NAHRÁNÍ FIRMWARE CANBOOT

```bash
sudo dfu-util -a 0 -D ~/CanBoot/out/canboot.bin --dfuse-address 0x08000000:force:mass-erase:leave -d 0483:df11
```

![Obrazek canboot_bootloader_flash.png](https://maz0r.github.io/klipper_canbus/images/canboot_bootloader_flash.png)

Poznámka: Pokud vidíš chybu po provedení předchozího kroku, neboj se, vše je v pořádku, pokud vidíš text "File Downloaded Successfully".

Vypni desku SB-CAN-TH na alespoň 5 sekund odpojením kabelu CANBUS.

Odpoj USB kabel.

Nyní můžeš připojit SB CAN desku k UCAN.

Počkej, až se zařízení spustí, a ujisti se, že tvá CAN0 síť je spuštěna a vidíš zařízení

```bash
~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0
```
nebo
```bash
~/CanBoot/scripts/flash_can.py -i can0 -q
```
Měl bys vidět něco jako:
```bash
"Found canbus_uuid=XXXXXXXXXX, Application: CanBoot"
```
Předpokládám, že výše uvedený krok ti poskytl UUID, a tak nyní můžeš nahrát Klipper na svou desku přes CanBoot...
(pokud ne, viz sekce pro řešení problémů [zde](https://maz0r.github.io/klipper_canbus/controller/canable.html))

```bash
cd ~/klipper
make menuconfig
```
Povol nízkoúrovňovou konfiguraci nastavením následujících položek.

![Obrazek sbcan_v1.1_klipper_make.png](https://maz0r.github.io/klipper_canbus/images/sb_can_v1.1_klipper_make.png)

Stiskni Q pro ukončení a Y pro uložení změn.

```bash
make clean
make
```
Nyní můžeš nahrát firmware na desku

```bash
python3 ~/CanBoot/scripts/flash_can.py -i can0 -f ~/klipper/out/klipper.bin -u MYUUID
```
![Obrazek canboot_flash.png](https://maz0r.github.io/klipper_canbus/images/canboot_flash.png)

Pokud vše proběhlo v pořádku, měl bys mít na své CAN desce nainstalovaný klipper.

Pro ověření toho můžeš dotazovat canbus uuid s

```bash
~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0
```
Měl bys vidět něco jako
```bash
"Found canbus_uuid=XXXXXXXXXX, Application: Klipper"
```
Ukázkový config pro inspiraci najdeš <a href="https://maz0r.github.io/klipper_canbus/toolhead/example_configs/toolhead_fysect_sbcan_1_1.cfg" target="_blank">zde</a>.

# ADXL
Nezapomeň na další kroky, které jsou k dispozici na [této stránce](https://github.com/Klipper3d/klipper/blob/master/docs/RPi_microcontroller.md).
