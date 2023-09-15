# Startovní gcode
Před prvním tiskem je potřeba nastavit `PRINT_START` makro a upravit startovní gcode v Průša Sliceru (případně ve sliceru vaší volby). Tento postup je určený pro Průša Slicer, potažmo Super Slicer.
Ještě doplním, že toto je mnou používaný gcode pro Voron 2.4 300mm. Pokud máte jinou tiskárnu, některé části budete muset upravit, viz níže v komentářích. Samozřejmě toto není žádné dogma. Svoje makra si můžete nastavit a upravit dle svých potřeb, možností a samozřejmě parametrů tiskárny. Toto berte jako inspiraci a vysvětlení jednotlivých položek pro lepší pochopení.

V Průša Sliceru na záložce `Nastavení tiskárny` přejděte do sekce `Vlastní G-code`. A do `Začátek G-code` vložte:
```
M104 S0
M140 S0
PRINT_START BED_TEMP=[first_layer_bed_temperature] EXTRUDER_TEMP=[first_layer_temperature]
M221 S{if layer_height<0.075}100{else}95{endif}
```

Pro OrcaSlicer vložte:
```
M190 S0
M109 S0
PRINT_START EXTRUDER_TEMP=[nozzle_temperature_initial_layer] BED_TEMP=[bed_temperature_initial_layer_single]
```

#### Co a proč tam vlastně vkládám?
Dobrá otázka, nevkládejte do tiskárny slepě kód, kterému nerozumíte. Startovní gcode si rozebereme postupně:
- `M104 S0` - nastaví nulovou hodnotu teploty pro hotend
- `M140 S0` - nastaví nulovou hodnotu teploty pro bed

A teď si asi říkáte, proč nastavujeme nulovou teplotu? Je to proto, že Průša Slicer má různé ochrany. Pokud nevidí ve startovním gcode příkaz `M104` a `M140`, automaticky jej sám doplní. Aby nám to nedělalo bordel, nastavíme na začátku gcode nuly, a v našem makru si pak teploty nastavíme podle předaných parametrů, viz níže.

- `PRINT_START BED_TEMP=[first_layer_bed_temperature] EXTRUDER_TEMP=[first_layer_temperature]`

`PRINT_START` je naše makro, které následně vložíme do `printer.cfg` v naší tiskárně s klipper firmware. `BED_TEMP` a `EXTRUDER_TEMP` jsou proměnné, které doplníme hodnotami ze sliceru, a předáme makru. S těmito proměnnými pak pracujeme dále.
- `M221 S{if layer_height<0.075}100{else}95{endif}` - Úprava flow, protože Průša Slicer přeextrudovává. Je to chybka, kterou řeší tento řádek kódu. Prostě nastavíte flow na 95 % a máte vyřešeno. 

Více o tomto problému v komentáři od Josefa Průši: https://forum.prusa3d.com/forum/original-prusa-i3-mk3s-mk3-others-archive/updated-slic3r-pe-over-extrusion-and-cooling-solved/

### Pokračujeme dále

V Průša Sliceru na záložce `Nastavení tiskárny` přejděte do sekce `Vlastní G-code`. A do `Konec G-code` vložte:
```
PRINT_END
```

PRINT_END je další makro, které si rozebereme níže.

Poslední úprava ve Sliceru:
V Průša Sliceru na záložce `Nastavení tiskárny` přejděte do sekce `Vlastní G-code`. A do `G-code před změnou vrstvy` vložte:
```
;BEFORE_LAYER_CHANGE
G92 E0.0
;[layer_z]
```
- `G92 E0.0` Tento gcode nám resetuje vzdálenost extruderu. Čili před každou vrstvou začínáme s čistým štítem :-)

Toto je vše, co je potřeba nastavit na straně Průša Sliceru. Nyní nás čekají makra.

# Makra

Ve vaší tiskárně si otevřete soubor `printer.cfg` a někam k ostatním makrům si vložte tento kód a uložte config (vysvětlivky k jednotlivým částem jsou v komentáři přimo v makrech):
```
[gcode_macro PRINT_START]
gcode:
    {% set BED_TEMP = params.BED_TEMP|default(60)|float %}
    {% set EXTRUDER_TEMP = params.EXTRUDER_TEMP|default(190)|float %}
    # Předchozí dva řádky nám vytvořily proměnné BED_TEMP a EXTRUDER_TEMP, a doplnily je hodnotami ze Sliceru
    # V případě že slicer hodnoty nepředá, nastaví se defaultně 190 pro hotend, a 60 pro bed

    # Nastavení teplot
    # M104 nám nastaví teplotu na 150 stupňů. Je to teplota, kdy se dá očistit tryska a filament "neslintá"
    M104 S150
    # M190 nám nastaví teplotu bedu na hodnotu, kterou do makra předal slicer. A čekáme než se nahřeje bed
    M190 S{BED_TEMP}
    # M109 nastaví teplotu hotendu na hodnotu, kterou nám do makra předal slicer a čeká na její dosažení
    M109 S150 ; Set non dripping hotend temperature
    # UG90 nám přepne na absolutní koordináty
    G90
    # M83 přepne extruder na relativní vzdálenosti
    M83
    # POZOR - G32 je makro pro voron 2.4 a předpokládá se, že jej máte. Pro pořádek jsem jej přidal za PRINT_END
    # Toto makro nám zařídí homování a vyrovnání gantry u V2.4, pokud máte jinou tiskárnu, následující řádek smažte
    G32
    # BED_MESH_CLEAR nám vymaže předchozí uložené hodnoty meshe, chceme začínat s čistým štítem
    BED_MESH_CLEAR
    # A následující příkaz nám zkalibruje podložku pomocí sensoru (bltouch, indukční sensor, crtouch, klicky a jiné)
    BED_MESH_CALIBRATE

    # G1 je gcode pro pohyb. Nyní přesuneme trysku do levého spodního rohu 5mm nad podložku
    G1 X3 Y6 Z5 F5000
    # Nyní sjedeme tryskou 0,3mm nad podložku
    G1 Z0.3 F3000
    
    # Nyní čekáme na nahřátí trysky na hodnotu předanou slicerem
    M109 S{EXTRUDER_TEMP}
    
    # Resetování vzdálenosti extruderu
    G92 E0
    
    # Očištění trysky. Následující gcode nám pomalu posunuje trysku 14cm doprava a extruder vytlačí 30mm filamentu
    # První pohyb je pomalejší a delší (až do 12cm), a následuje rychlé očištění 2 cm na výsledných 14 cm
    G1 X120 E30 F600
    G1 X140 F5000
    G92 E0
    # Následně před samotným tiskem proběhne krátká retrakce a resetování vzdálenosti extruderu
    G1 E-0.2 F600
    G92 E0
    
[gcode_macro PRINT_END]
gcode:
    # Uložíme aktuální stav tiskárny (pozici trysky, stav extruderu a další hodnoty)
    SAVE_GCODE_STATE NAME=STATE_PRINT_END
    # Vypneme nahřívání hotendu a bedu
    TURN_OFF_HEATERS
    # Přepneme na relativní vzdálenosti, a zvedneme trysku o 10mm nahoru
    # POZOR - pokud tisknete vysoké tisky na maximální výšku tiskárny, tento gcode vám může dělat potíže
    # Následně přepneme zpět na absolutní pozicování/vzdálenosti pomocí G90
    G91
    G1 Z10 F3000
    G90
    # Přesuneme toolhead doprava a dopředu
    # Zde záleží, kam chcete umístit toolhead/tiskovou hlavu po skončení tisku. Není dobré ji nechat viset na tiskem
    # nastavte podle vaší tiskárny a podle toho kde chcete tiskovou hlavu mít.
    # v tomto případě jde tisková hlava dopředu (Y20) a doprava (X300)
    G1 Y20
    G1 X300
    
    # Počkáme na vyčištění bufferu
    M400
    # Vynulujeme vzdálenosti extruderu
    G92 E0
    # Zatáhneme filament 20mm dovnitř hotendu
    # tento řádek nám pak umožní vytáhnout filament z tiskárny i za studena a je kompenzován 
    # v PRINT_START (proto tlačíme 30mm filamentu před tiskem)
    G1 E-20.0 F3000
    # Vypneme ventilátor
    M106 S0
    # Vypneme motory
    M84
    # Vymažeme naměřený bed mesh
    BED_MESH_CLEAR
    # Obnovíme hodnoty původního stavu tiskárny
    RESTORE_GCODE_STATE NAME=STATE_PRINT_END
    
[gcode_macro G32]
gcode:
    # Uložíme aktuální stav tiskárny (pozici trysky, stav extruderu a další hodnoty)
    SAVE_GCODE_STATE NAME=STATE_G32
    # Přepneme na absolutní pozicování/vzdálenosti
    G90
    # zde je podmínka - pokud není tiskárna "vyhoumovaná" provede se příkaz home - G28
    {% if printer.toolhead.homed_axes != "xyz" %}
        G28
    {% endif %}
    # Tento řádek spustí levlování Gantry pro Voron 2.4
    QUAD_GANTRY_LEVEL
    # Následuje opět home, ale nyní jen pro osu Z
    G28 Z
    # Obnovíme hodnoty původního stavu tiskárny
    RESTORE_GCODE_STATE NAME=STATE_G32
```
