# Jednoduchý návod na kalibraci extruderu (Rotation Distance)

Tato kalibrace je klíčová pro přesný 3D tisk. Zajišťuje, že když tiskárna chce vytlačit 100 mm filamentu, skutečně vytlačí 100 mm materiálu. Na tiskárnách s firmwarem Klipper (což je standard pro Voron) se tato hodnota nazývá `rotation_distance`.

## Co budete potřebovat?
*   Filament (ideálně světlé barvy, aby na něm byla vidět značka).
*   Digitální posuvné měřítko (šupléru).
*   Tenký fix pro vytvoření značky.
*   Přístup k webovému rozhraní tiskárny (Mainsail nebo Fluidd).

## Krok 1: Příprava tiskárny
1.  Zapněte tiskárnu a otevřete její webové rozhraní (nejčastěji Mainsail).
2.  Nahřejte hotend na tiskovou teplotu pro filament, který používáte. Pro PLA to bývá kolem 220 °C. Pro ABS nebo ASA 260 °C.
3.  V rozhraní jděte do sekce Machine a otevřete soubor `printer.cfg`. Najděte v něm sekci `[extruder]` a poznamenejte si svoji současnou hodnotu u parametru `rotation_distance`. Budete ji potřebovat pro výpočet.

## Krok 2: Měření filamentu
1.  Zaveďte filament do extruderu.
2.  Jako referenční bod použijte tělo extruderu v místě, kde do něj filament vchází.
3.  Od tohoto bodu odměřte na filamentu přesně 120 mm a udělejte na něm tenkou, ale zřetelnou značku fixem.

## Krok 3: Vytlačení (extruze) filamentu
1.  Vraťte se do webového rozhraní a otevřete konzoli.
2.  Nejprve zadejte příkaz `M83` a potvrďte. Tím přepnete extruder do relativního módu.
3.  Poté zadejte příkaz `G1 E100 F60` a potvrďte. Tiskárna se pomalu pokusí vytlačit 100 mm filamentu. Pomalá rychlost (F60) je důležitá pro přesnost.
4.  Počkejte, dokud se extruder nezastaví.

## Krok 4: Kontrola a výpočet
1.  Nyní znovu změřte vzdálenost od stejného referenčního bodu na extruderu po značku, kterou jste udělali na filamentu.
2.  Pokud je extruder perfektně zkalibrovaný, měla by být tato vzdálenost přesně 20 mm (protože 120 mm - 100 mm = 20 mm). Většinou to tak ale poprvé nevyjde.
3.  Zapište si, kolik jste naměřili. Z této hodnoty vypočítáte, kolik filamentu se reálně vytlačilo podle vzorce:
    ```
    Skutečně vytlačená délka = 120 - naměřená vzdálenost po extruzi
    ```
4.  Příklad: Pokud jste po extruzi naměřili 25 mm, reálně se vytlačilo `120 - 25 = 95 mm` filamentu.

## Krok 5: Nastavení nové hodnoty
1.  Otevřete si naši kalkulačku: https://kalkulacka.hotend.cz/
2.  Do kalkulačky zadejte tyto hodnoty:
    *   **Původní hodnota rotation_distance:** Číslo, které jste si zapsali na začátku ze souboru `printer.cfg`.
    *   **Požadovaná délka extrudovaného filamentu:** Zadejte `100`.
    *   **Reálně extrudovaná délka filamentu:** Zadejte hodnotu, kterou jste spočítali v předchozím kroku.
3.  Kalkulačka Vám okamžitě vypočítá novou hodnotu `rotation_distance`.
4.  Vraťte se do souboru `printer.cfg`, najděte opět sekci `[extruder]` a původní hodnotu `rotation_distance` přepište na tu novou z kalkulačky.
5.  Klikněte na tlačítko `SAVE & RESTART` (Uložit a restartovat). Tím se změna trvale uloží a firmware se restartuje s novou hodnotou.

A je to! Tímto máte extruder přesně zkalibrovaný. Pro stoprocentní jistotu můžete celý proces zopakovat. Tentokrát by Vám po změření měla vyjít vzdálenost velmi blízko 20 mm. 