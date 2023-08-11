# Startovní kontroly
Veškerou kabeláž máme hotovou, máme flešnutou desku, nastavený `printer.cfg` a klipper nám komunikuje s deskou. Skvěle. Tak teď musíme postupně otestovat, že nám vše funguje a správně se hýbe.

## Endstopy
Není nutné dělat všechny kroky v přesném pořadí, nicméně Endstopy byste z bezpečnostních důvodů měli otestovat dříve, než se pokusíte pohybovat s motory.
Jak ověřit funkčnost a správné zapojení endstopů? Nejjednodušší způsob je v Settings -> Endstops. Zde máte vypsané všechny endstopy a pomocí tlačítka obnovit můžete zjistit jejich aktuální stav. Případně můžete do konzole napsat příkaz `QUERY_ENDSTOPS`, který vám stav endstopů vypíše také.
- Přesuňte tiskovou hlavu doprostřed podložky (aby nebyly endstopy sepnuté) a klikněte na tlačítko obnovit. Měli byste u všech endstopů vidět stav `open`.
- Pokud nemáte žádný endstop sepnutý a vidíte všude `open`, zkuste ručně endstop sepnout (a držet). Při tom klikněte na tlačítko obnovit. U sepnutého endstopu by se mělo objevit `triggered`. Takto je to v pořádku a pokračujte s kontrolou na další endstopy.
- Pokud endstop sepnutý není, ale přesto vidíte `triggered` a po zmáčknutí endstopu vidíte `open`, je obrácená logika. Přejděte do `printer.cfg`, najděte daný endstop a buďto přidejte, nebo odeberte `!`. Například najde v configu tento zápis `endstop_pin: ^!ar3`. Pro obrácení logiky endstopu tedy upravíte na `endstop_pin: ^ar3`.
- Pokud endstop nereaguje na sepnutí, může být špatně zapojený kabel (do jiného konektoru) nebo máte v `printer.cfg` špatně zadaný Pin.
- Většina dnešních desek používá Pullup PIN (to se označuje '^' před názvem pinu), proto zkontrolujte, zdali máte stříšku před názvem pinu uvedenou.
- pokud ani přesto endstop nereaguje, a všechny předchozí body máte správně, vyměňte spínač.

## Nahřívání, termistory, větráky
Jedná se o tři odlišné, avšak vzájemně propojené věci, které proto otestujeme současně.
V Mainsailu v dashboardu vidíte na pravé straně grafy teplot. Jsou tam v čase zobrazeny jak teploty hotendu, tak teploty podložky. Těch teplot tam můžete vidtě více (například teplota Raspberry, či teplota desky), ale pro fukčnost jsou klíčové teploty hotendu a desky.

![Mainsail a nastavení teplot](/images/mainsail_temp_graph.png)

Pokud by nefungovaly termistory, klipper by vyhodil chybu už dříve, takže v tomto kroku víte, že termistor je zapojený a ukazuje nějaké teploty. Měli byste tam vidět přibližně pokojové teploty.

- Nyní nad grafem teplot do okýkna zadejte teplotu hotend 50 stupňů a zmáčkněte enter nebo klikněte někam bokem. 
    - Na hotendu by se vám měl začít točit větrák pro ofuk hotendu. Na hlavě Stealthburner i Afterburner je to ten spodní.
    - Měla by vám začít stoupat teplota (sledujte ji v grafu nebo vedle nastavení teplot)
    - Pokud se vám roztočil větrák pro ofuk tisku, místo větráku u hotendu, máte přehozené kabely. Zkontrolujte zapojení.
    - U termistoru byste měli mít nastavený správný typ termistoru, jinak vám při vyšších teplotách může vznikat velká odchylka od reálných teplot

- Stejný postup zopakujeme pro podložku. Zde není žádný ventilátor, takže sledujeme jen teploty
- Pokud se nám teploty nemění, zkontrolujeme že máme vše správně zapojeno ve správných terminálech či konektorech na desce, a že máme správně v `printer.cfg` zadány správné piny.
- Posledním krokem této kontroly je spuštění ventilátoru pro ofuk tisku. V dashboardu je slider pro ovládání ventilátoru, otestujeme že funguje jak na 100 %, tak i při nižších rychlostech okolo 20 - 30 %. Tentokrát se jedná o ten horní ventilátor.

## Motory
Zkontrolujte, že po zapnutí tiskárny můžete hýbat se všemi motory volně. Pokud by s některým z motorů hýbat nešlo (je aktivní), pak je zřejmě obrácena logika pro `enable_pin` nastavení. Stejně jako u endstopů, přidejte nebo odeberte vykřičník. Například `enable_pin: !ar38` přepište na `enable_pin: ar38`. Restartuje firmware a zkuste, jestli se dá s motem hýbat.

### Stepper Buzz
Nyní se pustíme do kontroly motorů. Klipper má jednoduchý příkaz, kterým rozpohybuje specifikovaný motor.
Do konzole zadejte příkaz `STEPPER_BUZZ STEPPER=stepper_x` a sledujte, co dělá tiskárna. Tento příkaz posune motor o +10mm a zpět. A 10x opakuje.
Tímto způsobem otestujte všechny motory. 

Například názvy motorů u Voron 2.4 jsou takto:
- stepper_x
- stepper_y
- stepper_z
- stepper_z1
- stepper_z2
- stepper_z3
- extruder

Pokud se hýbe jiný motor než by měl, zkontrolujte že je zapojen ve správném konektoru na desce, případně pin v `printer.cfg`. Motor by se měl hýbat v plusových hodnotách a zpět. Čili například u Z motorů u Voron 2.4 by měl motor pohnout gantry směrem nahorů (a pak hned dolů).

Pokud tomu tak není a motor jede opačným směrem, opět upravte logiku u `dir_pin` přidáním nebo odebráním vykřičníku. Například `dir_pin: PC1` změňte na `dir_pin: !PC1`.

Takto otestujte všechny motory. A to včetně extruderu.

### První homování
Pokud jsme zkontrolovali endstopy, a motory nám chodí tak jak chceme, můžeme zkusit první homování. Doporučuji začít nejdříve osou X, pak Y, a pak Z. Pokud máme Voron 2.4, musíme ještě nastavit v `printer.cfg` kde je umístět Z endstop. Takže na to pozor.

**Před samotným homováním si připravte buďto prst na vypínač tiskárny, nebo v mainsailu v pravém horním rohu je tlačítko pro okamžité vypnutí tiskárny.**

**Tlačítko v mainsailu funguje dobře, takže doporučuji používat to (a hlavně nemusíte pak čekat než naběhne Raspberry), ale přeci jen kdyby se něco posralo, buďte připraveni skočit po hlavním vypínači.**

Při homování by měl jet motor směrem k endstopu. Pokud nejede, máme buď špatně nastavený směr otáčení motoru (`dir_pin`), nebo máme špatně specifikované umístění endstopu. Nicméně tyto věci jsou v připraveném configu pro danou desku nebo tiskárnu již většinou správně. Proto to nebudu tady rozepisovat.

U CoreXY je někdy náročné nastavit směr pohybu motorů. Voron má krásně zpracované obrázky jak by se motory měly hýbat, a hlavně který je který. Sledujte směr pohybu motorů (hlavně na x a y ose) a podle obrázku vyhledejte jestli je to ok, nebo jestli máte něco otočit. Případně můžete mít zaměněný Motor A a B.

https://docs.vorondesign.com/build/startup/#motor-configuration-guides

## PID kalibrace
Před prvním tiskem silně doporučuji udělat PID kalibraci. Je to jednorázová akce na pár minut, a máte jistotu že se hotend i podložka budou chovat tak jak mají. PID kalibraci doporučuji udělat i pokud měníte hotend, dáváte jinou trysku, silikonovou ponožku nebo třeba jiný termistor.

Kalibrace je jednoduchá. Do konzole zadejte příkaz:
```
PID_CALIBRATE HEATER=extruder TARGET=210
```
*Tento příkaz říká, že chcete kalibrovat "extruder" při teplotě 210 stupňů.*

Pro kalibraci vyhřívané podložky použijte příkaz:
```
PID_CALIBRATE HEATER=heater_bed TARGET=60
```
Teploty nastavujte podle materiálu, který budete tisknout nejvíce. Pokud tisknete nejvíce ABS, nastavte vyšší teploty v `TARGET`. 

Pozor, pokud máte například podložku pojmenovanou v configu jinak, do `HEATER` zadejte váš název!

## Kalibrace Z endstopu u Voron 2.4 a Voron Trident
Ačkoliv to vypadá jako kravina, pár mých zničených PEI podložek by mohlo vyprávět o opaku. Toto je velmi důležitá kalibrace, která zajistí že se vám nebude měnit výška Z. Pokud neprovedete tuto kalibraci (u tiskáren se Z endstopem), budete mít problémy s výškou Z.

Rozhodněte se, jestli uděláte kalibraci za studena, či za tepla. Pokud za tepla, musíte se ujistit že máte očištěnou trysku od zbytku filamentu.

Před samotnou kalibrací musíte mít tiskárnu zahoumovanou `G28` a vymažte původní hodnoty MESHe pomocí `BED_MESH_CLEAR` a proveďte vyrovnání podložky (QGL nebo Z TILT u Tridenta).

Znovu proveďte `G28`

Najeďte do bodu, kde budete měřit vzdálenost trysky od podložky

Spusťte v konzoli příkaz `Z_ENDSTOP_CALIBRATE`. Jak Mainsail, tak Fluidd mají na kalibraci grafickou nádstavbu, takže následné úpravy vzdálenosti trysky od podložky jsou už jednoduché. Pomocí klasické papírové metody (papír pod tryskou musí jemně drhnout) a snižováním nebo zvyšováním výšky Z najděte správnou polohu. Následně klikněte na Accept a pomocí tlačítka SAVE CONFIG naměřené hodnoty uložte a restartujte klipper.

Nezapomeňte si v configu u `[bed_mesh]` nastavit `relative_reference_index`. Relative reference index nastavte co nejblíže k bodu, kde jste měřili Z offset.
Více info jak nastavit relative reference index najdete v záznamu mého streamu: https://youtu.be/Vi1-iHbne04
