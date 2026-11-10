# Casio F-91W Digital Watch Emulator (ZYBO)

Projekt 1 i 02321 HW/SW programming. Forfattere: Ludvig og Frederik

Emulering af Casio F-91W på ZYBO-boardet. Urets display vises som ASCII-grafik over den serielle port. Knapper, switche og LEDs bruges som urets knapper og indikatorer.

## Funktioner

- Interrupts til knapper, switche og timer (10 ms tick)
- ASCII-grafik med eget 5x7-font, stabilt billede (cursor-hjem i stedet for at rydde skærmen)
- Fire modes: Time, Alarm, StopWatch, Setting
- Indstilling af tid med hurtig fremrykning (hold A i mere end 2 s)
- Speedup via switche: 10x / 20x / 30x / 60x
- Stopur med net time measurement
- Daglig alarm (LEDs blinker) samt dag og dato

## IO-mapping

| ZYBO | Funktion |
|---|---|
| BTN0 | A: øg værdi / start-stop stopur |
| BTN2 | C: skift mode |
| BTN3 | L: skift felt / net time / nulstil |
| SW3..SW0 | Speedup: `1---`=60x, `01--`=30x, `001-`=20x, `0001`=10x, `0000`=1x |
| LD3..LD0 | Mode: `1000` Time, `0100` Alarm, `0010` StopWatch, `0001` Setting |

## Brug

Modes skiftes med C: Time → Alarm → StopWatch → Setting → Time.

- **Setting:** L vælger sekunder, minutter eller timer. A øger værdien. Hold A i mere end 2 s for hurtig fremrykning.
- **Alarm:** L vælger time, minut eller on/off. A øger eller skifter. Et vilkårligt tryk stopper en ringende alarm.
- **StopWatch:** A starter og stopper. L under kørsel fryser eller frigiver visningen (net time). L når stoppet viser tiden og nulstiller derefter.

## Design

Interrupt-handlerne sætter kun flag og tæller ticks. Al udskrivning sker i main-loop, som behandler alle akkumulerede ticks (`handled_ticks != tick_count`), så ingen ticks mistes. Speedup kalder `Clock_Update()` og `Alarm_Update()` flere gange pr. tick.

| Fil | Indhold |
|---|---|
| `Casio_main.c` | Hardware, interrupts, modes, main-loop |
| `clock.c` | Tid, dato, ugedag, skudår, indstilling |
| `alarm.c` | Alarmtid og ring-logik |
| `stopwatch.c` | Stopur |
| `print.c` | ASCII-font og udskrift |

## Opbygning

1. Opret hardware i Vivado efter tutorial 2: AXI GPIO for knapper, LEDs og switche, samt AXI Timer, alle med interrupts til PS.
2. Opret SDK-projekt og tilføj kildefilerne.
3. Åbn en serielterminal med ANSI-understøttelse (115200 baud, 8N1).
4. Programmér FPGA og kør.

## Kendte begrænsninger

- Dato og ugedag kan ikke indstilles.
- Uret står stille i Setting-mode.
- Stopuret går fra 59:59,99 tilbage til 00:00,00.
- Ingen software-debounce af knapper.

## Test

**TODO:** Udfyld testtabel med forventet og faktisk resultat.

| Testcase | Forventet | Faktisk |
|---|---|---|
| Opstart | Time-mode, LD=`1000` | Virker |
| C cykler modes | Rækkefølge og LEDs som ovenfor | Virker |
| Speedup | 10x / 20x / 30x / 60x | Virker |
| Setting + hold A | Hurtig fremrykning efter 2 s | Virker |
| Stopur net time | Visning fryser, tiden løber | Virker |
| Alarm | LEDs blinker, stopper ved tryk | Virker |
