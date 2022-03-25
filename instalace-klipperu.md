# Instalace klipperu
Možností instalace je více. Já vybírám tu zřejmě nejjednodušší. Pomocí Raspberry Pi Imageru si vytvoříme SD kartu pro naše Raspberry Pi.

## Raspberry Pi Imager
<a href="https://www.raspberrypi.com/software/" target="_blank">https://www.raspberrypi.com/software/</a>

Výhoda RPI Imageru je v tom, že si při vytváření SD karty nastavíte rovnou všechny potřebné údaje. Nemusíte nic dalšího stahovat, v Imageru si vyberete image, a ten se vám na pozadí stáhne. Přednastavíte si wifi, sítový název a další věci.

Podrobný návod zde:
https://docs.mainsail.xyz/setup/mainsailos/pi-imager

# Propojení klipperu s deskou tiskárny
Tohle je podle mě nejkomplikovanější část celého klipperu. Hlavně proto, že každá deska pro 3D tiskárnu to má jinak.

## Připojení k RPI přes SSh
Nejříve se potřebujeme připojit do RPI přes terminál, Putty, či jiný software. Na linuxu nebo Macu máte terminál přímo integrovaný, na windows doporučuji stáhnout si Putty. 
https://www.putty.org/

Po spuštění Putty stačí do Host name napsat síťové jméno vaši tiskárny (napři voron.local) nebo její IP adresu a kliknout na tlačítko Open. Budete vyzváni abyste zadali uživatelské jméno (výchozí je pi) a heslo (výchozí je raspberry). Ty jste si mohli nastavit v RPI Imageru. Záleží co jste tam zadali.

## Vytváříme firmware pro desku
Nyní postupujeme podle návodu Klipperu.
https://www.klipper3d.org/Installation.html#building-and-flashing-the-micro-controller

Nejprve se přesuneme do složky /klipper a spustíme konfigurační program
```
cd ~/klipper/
make menuconfig
```
Zde nakonfigurujeme parametry podle vaší desky. Doporučuji si najít vaši desku v seznamu klipper configů:
https://github.com/Klipper3d/klipper/tree/master/config

Rozklikněte si vaší desku, a v úvodu souboru budete mít instrukce jak firmware připravit a jak jej nahrát do desky. jak jsem psal výše, toto se liší pro každou desku.

Jakmile máme vše nastaveno, pomocí klávesy ESC ukončíme menuconfig, a změny uložíme.

Nyní vytvoříme soubor s firmware pomocí příkazu:
```
make
```

Nyní máme vytvořený soubor `~/klipper/out/klipper.bin`. Tento soubor ve většině případů přejmenujeme na `firmware.bin`, nahrajeme na SD kartu a přesuneme do desky, a po restartu desky se nám firmware nahraje do desky. Každopádně tento postup se může lišit, a proto postupujte podle informací k vaší desce.

## Jak se dostat k firmware.bin souboru?
Způsobů je opět více, ale asi nejjednodušší je zkopírovat si soubor s firmwarem do složky, ze které si pak tento soubor můžete stáhnout přímo z webového rozhraní mainsailu.

Soubor klipper.bin si přesuneme do složky (a rovnou přejmenujeme), do které se dostaneme z mainsailu:
```
cp ~/klipper/out/klipper.bin ~/klipper_config/firmware.bin
```

## ID desky pro použití v printer.cfg
Poslední věc, kterou budeme potřebovat je cesta k desce, pomocí které dokáže klipper s deskou komunikovat.
```
ls /dev/serial/by-id/*
```
Po zadání tohoto příkazu do konzole by na vás mělo vypadnout něco jako `/dev/serial/by-id/usb-1a86_USB2.0-Serial-if00-port0`. Toto si zkopírujeme, budeme to potřebovat dále. Pokud se vám nic takového neobjevilo, znamená to že se RPI vaši desku z nějakého důvodu nevidí.

## Nahrání firmware do desky
Firmware jsme si připravili, a přesunuli do do složky. Nyní se přesuneme do webového rozhraní Mainsailu. Do prohlížeče zadejte IP adresu vašeho RPI, nebo síťový název (voron.local, mainsailos.local atd.).

V menu Configuration (tam kde máme mainsail.cfg, moonraker.cfg a další soubory s nastavením) bychom měli vidět soubor firmware.bin. Ten si stáhneme, zkopírujeme na SD kartu a vložíme do desky. Po restartu desky by měl být firmware nainstalovaný. Pro kontrolu můžeme SD kartu vyjmout z desky, vložit do počítače a mělibychom tam vidět například `firmware.cur`. To znamená že se nám FW úspěšně nainstaloval.

# printer.cfg a nastavení naší tiskárny
Poslední věc pro propojení Klipperu a tiskárny je vytvoření `printer.cfg` a doplnení výše uvedene cesty. Pak by nám vše mělo začít komunikovat. Ale postupně. Výše jsem psal o seznamu configů pro různé desky. Najdeme si config pro naši desku, stáhneme jej a nahrajeme do mainsailu. Případě si tento soubor vytvoříme ručně, a obsah ukázkového souboru do něj zkopírujeme.

Otevřeme si `printer.cfg` a najdeme v něm `[mcu]`. Sem doplníme správnou cestu k desce, kterou jsme zjistili pomocí výše uvedenené příkazu v terminálu (nebo putty). Výsledek by měl vypadat nějak takto:
```
[mcu]
serial: /dev/serial/by-id/usb-1a86_USB2.0-Serial-if00-port0
```

`printer.cfg` si nyní uložíme a zároveň restartujeme desku tlačítkem `Save and Restart`. Pokud jsme vše udělali správně, na hlavní stránce v Dashboardu bychom měli vidět teploty hotendu a desky v grafu.

V dashboardu můžete vidět ještě oranžové upozornění ohledně chybějících věcí. Přejdeme opět do `printer.cfg` a hned na první řádek doplníme `[include mainsail.cfg]`. Uložíme a restartujeme. Nyní by už mělo být vše v pořádku a můžeme postoupit dále.

## Co dále?
[Startovní kontroly před spuštěním tiskárny](startovni-kontroly.md)