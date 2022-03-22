# Stručný návod na zprovoznění tiskárny Voron
Není cílem vytvořit podrobný návod, ale jen základní postup s odkazy kde najít další informace. Ne vše je v klipperu přehledně popsáno, některé věci jsou rozstřílené různě po webu, a tak jsem si řekl že si to pro sebe hodím na jedno místo. A třeba to využijete také.

## Instalace klipperu
Možností instalace je více. Já vybírám tu zřejmě nejjednodušší. Pomocí Raspberry Pi Imageru si vytvoříme SD kartu pro naše Raspberry Pi.

### Raspberry Pi Imager
https://www.raspberrypi.com/software/

Výhoda RPI Imageru je v tom, že si při vytváření SD karty nastavíte rovnou všechny potřebné údaje. Nemusíte nic dalšího stahovat, v Imageru si vyberete image, a ten se vám na pozadí stáhne. Přednastavíte si wifi, sítový název a další věci.

Podrobný návod zde:
https://docs.mainsail.xyz/setup/mainsailos/pi-imager

## Propojení klipperu s deskou tiskárny
Tohle je podle mě nejkomplikovanější část celého klipperu. Hlavně proto, že každá deska pro 3D tiskárnu to má jinak.

### Připojení k RPI přes SSh
Nejříve se potřebujeme připojit do RPI přes terminál, Putty, či jiný software. Na linuxu nebo Macu máte terminál přímo integrovaný, na windows doporučuji stáhnout si Putty. 
https://www.putty.org/

Po spuštění Putty stačí do Host name napsat síťové jméno vaši tiskárny (napři voron.local) nebo její IP adresu a kliknout na tlačítko Open. Budete vyzváni abyste zadali uživatelské jméno (výchozí je pi) a heslo (výchozí je raspberry). Ty jste si mohli nastavit v RPI Imageru. Záleží co jste tam zadali.

### Vytváříme firmware pro desku
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

### Jak se dostat k firmware.bin souboru?
Způsobů je opět více, můžete se připojit přes FTP, ale asi nejjednodušší je zkopírovat si soubor s firmwarem do složky, ze které si pak tento soubor můžete stáhnout přímo z webového rozhraní mainsailu.

Soubor klipper.bin si přesuneme do složky (a rovnou přejmenujeme), do které se dostaneme z mainsailu:
```
cp ~/klipper/out/klipper.bin ~/klipper_config/firmware.bin
```

### ID desky pro použití v printer.cfg
Poslední věc, kterou budeme potřebovat je cesta k desce, pomocí které dokáže klipper s deskou komunikovat.
```
ls /dev/serial/by-id/*
```
Po zadání tohoto příkazu do konzole by na vás mělo vypadnout něco jako `/dev/serial/by-id/usb-1a86_USB2.0-Serial-if00-port0
`. Toto si zkopírujeme, budeme to potřebovat dále.
Pokud se vám nic takového neobjevilo, znamená to že se RPI vaši desku z nějakého důvodu nevidí.