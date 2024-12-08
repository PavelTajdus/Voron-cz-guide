# Aktualizace Raspberry Pi na Debian Bookworm

Tento návod vás provede procesem aktualizace operačního systému Raspberry Pi z Debian Bullseye na Debian Bookworm.

## Doporučený postup: Čistá instalace
Nejbezpečnější způsob aktualizace je provést čistou instalaci pomocí nástroje Raspberry Pi Imager. Tento postup minimalizuje riziko problémů a ztráty dat.

1. Stáhněte si Raspberry Pi Imager
2. Vyberte nejnovější verzi Debian Bookworm
3. Nahrajte obraz na SD kartu
4. Vložte SD kartu do Raspberry Pi a spusťte systém

## Alternativní postup: Ruční aktualizace
Pokud chcete aktualizovat systém bez přeinstalace, postupujte podle následujících kroků. **Doporučujeme zálohovat všechna důležitá data před zahájením procesu.**

### 1. Příprava systému
```bash
sudo apt update && sudo apt upgrade -y
sudo apt dist-upgrade
```

### 2. Úprava repozitářů
```bash
sudo sed -i "s/bullseye/bookworm/g" /etc/apt/sources.list
sudo sed -i "s/bullseye/bookworm/g" /etc/apt/sources.list.d/raspi.list
sudo sed -i 's/non-free/non-free non-free-firmware/g' /etc/apt/sources.list
```

### 3. Provedení aktualizace
```bash
sudo apt update
sudo apt full-upgrade -y
sudo apt clean -y
sudo apt autoremove -y
```

### 4. Restart systému
```bash
sudo reboot
```

## Důležitá upozornění
- Celý proces může trvat přibližně 30 minut
- Během aktualizace budete muset potvrdit několik výzev k nahrazení konfiguračních souborů
