# PIC12F508 Touch RGB Controller

Firmware sviluppato in **MPLAB IDE / XC8** per una piccola scheda di controllo con **interfaccia touch capacitiva** e illuminazione RGB.

Il progetto utilizza un **Microchip PIC12F508** e implementa direttamente via firmware il rilevamento dell'interazione touch, la gestione del cambio colore e la dissolvenza dei LED tramite **PWM software**.

## Funzionalità

* Rilevamento dell'interazione tramite ingresso touch
* Selezione ciclica dei tre colori:

  * Red
  * Green
  * Blue
* Cambio colore ad ogni nuova interazione
* Debounce software dell'ingresso touch
* Fade-in / fade-out della luminosità
* PWM realizzato interamente via software
* Gestione dello stato tramite singoli bit di una variabile di controllo
* Utilizzo diretto dei registri GPIO del PIC

## Hardware

### Microcontrollore

**Microchip PIC12F508**

Configurazione utilizzata:

* Oscillatore RC interno
* Frequenza di clock: 4 MHz
* Watchdog disabilitato
* Code Protection disabilitato
* Pin MCLR disabilitato

Il microcontrollore è stato scelto per le dimensioni e le risorse estremamente ridotte, rendendo necessario implementare direttamente nel firmware la gestione temporale e il PWM.

## Logica di funzionamento

Il touch viene utilizzato come comando per cambiare il colore dell'illuminazione.

```text
             TOUCH
               │
               ▼
        ┌──────────────┐
        │  PIC12F508   │
        │              │
        │ Debounce     │
        │ State        │
        │ Fade         │
        │ Software PWM │
        └──────┬───────┘
               │
        ┌──────┼──────┐
        ▼      ▼      ▼
        R      G      B
```

Ad ogni nuova attivazione del touch viene selezionato il colore successivo:

```text
RED → GREEN → BLUE → RED → ...
```

Durante l'interazione viene inoltre modificato il valore di `fade`, ottenendo una variazione progressiva della luminosità.

## PWM software

Il controllo della luminosità viene realizzato senza utilizzare un modulo PWM hardware dedicato.

Il firmware genera manualmente il periodo PWM:

```cpp
for (unsigned char i = 0; i < 128; i++)
{
    if (i < fade)
        GPIO = rgbBox[rgbControl];
    else
        GPIO = 0;

    __delay_us(100);
}
```

Il valore `fade` determina quindi la porzione del periodo durante la quale il colore selezionato rimane attivo.

La gestione utilizza **128 livelli di duty cycle**, con una temporizzazione basata sul clock interno del microcontrollore.

## Gestione dello stato

Una singola variabile viene utilizzata per memorizzare più informazioni tramite bit:

```cpp
unsigned char ctrl = 0b11;
```

* **Bit 0** — direzione della variazione di `fade`
* **Bit 1** — stato precedente dell'ingresso touch

Il secondo bit viene utilizzato per distinguere una nuova attivazione da un ingresso che rimane continuamente attivo, realizzando così un semplice **debounce software / edge detection**.

La direzione del fade viene invece invertita quando il valore raggiunge i limiti:

```text
fade = 0
   │
   │  fade +
   ▼
fade = 128
   │
   │  fade -
   ▼
fade = 0
```

## Mappatura dei colori

I colori vengono rappresentati direttamente come pattern di bit da scrivere sul registro `GPIO`:

```cpp
unsigned char rgbBox[] = {
    0b010000,   // Red
    0b100000,   // Green
    0b000100    // Blue
};
```

Questo permette di associare ogni colore alla relativa uscita senza introdurre strutture o librerie aggiuntive.

## Tecnologie

* **Microchip PIC12F508**
* **MPLAB IDE**
* **XC8**
* C embedded
* GPIO register-level programming
* Software PWM
* Software debounce
* Internal RC oscillator

## Struttura

```text
.
├── main.c
└── README.md
```

## Note sul progetto

Questo firmware rappresenta un esempio di sviluppo embedded su un microcontrollore con risorse molto limitate.

La logica di controllo è stata mantenuta volutamente semplice e vicina all'hardware, utilizzando direttamente i registri del PIC e implementando via software le funzioni necessarie alla gestione del touch e dell'illuminazione.

Il progetto è stato sviluppato per un dispositivo reale e non come semplice esercizio didattico.

---

## Autore

**Fabio Crivellaro**

Progetto sviluppato in ambito professionale.
