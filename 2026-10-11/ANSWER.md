# Antwort: Welche Inhalte der 3. Klasse lassen sich mit R effizienter lösen?

*Antwort auf [`PROMPT.md`](PROMPT.md) – erzeugt mit OpenCode.*

Fast der gesamte Statistikstoff der 3. Klasse (KM5 + KM6) ist mit R
effizienter lösbar als per Hand: Die `d/p/q/r`-Familie ersetzt Tafeln und
Formeln, Code ist reproduzierbar, und Plots/Simulationen schaffen einen
didaktischen Mehrwert, den Taschenrechner und Excel nicht bieten.

## Vergleichstabelle

| Inhalt | Hand-Methode | R-Lösung |
|---|---|---|
| Zufallsvariablen f(x)/F(x) (KM5) | Tafeln, Summen | `dbinom`, `pnorm` + Plot |
| Diskrete Verteilungen (KM5) | Binomialkoeffizient, Tabellen | `dbinom`, `phyper`, `dpois` |
| Normalverteilung & Standardisierung (KM5) | z-Tabelle, Interpolation | `pnorm`, `qnorm` |
| Exponentialverteilung (KM5) | Integral von Hand | `pexp`, `dexp`, `rexp` |
| Lage-/Streumaße (KM5) | Rechnen von Hand | `mean`, `median`, `sd`, `IQR` |
| Parameter vs. Schätzwerte (KM5) | Bessel-Formel | `var`/`sd` (n−1), Simulation |
| Vertrauensbereiche (KM6) | z/t/χ²-Tafeln | `qnorm`, `qt`, `qchisq`, `t.test` |
| Prüfergebnisse darstellen (KM6) | Excel-Diagramme | `ggplot2`, Quarto |
| Kennzahlen & Ausreißer (KM6) | 1,5·IQR-Regel per Hand | `IQR`, `quantile` |
| Lebensdauer (KM6) | Kurvenpapier, Probieren | `fitdistrplus`, `pweibull` |

## Wichtigste Beispiele

```r
# Verteilungen: d = Dichte/PMF, p = Verteilung, q = Quantil, r = Zufall
dbinom(3, 10, 0.4);  pbinom(3, 10, 0.4)   # genau / höchstens
pnorm(85, 100, 15);  qnorm(0.975)         # Normalverteilung, z = 1,96

# Lage- und Streumaße
mean(x); median(x); sd(x); IQR(x)         # sd nutzt korrekt Bessel n−1

# Konfidenzintervall für μ
t.test(x, conf.level = 0.95)$conf.int

# Darstellung
library(ggplot2)
ggplot(daten, aes(x = ergebnis)) + geom_histogram(bins = 15)
```

## Grenzen

- R liefert Ergebnisse, **erklärt aber keine Konzepte** (Erwartungstreue,
  KI-Breite …).
- **Modell-/Verteilungswahl** bleibt eine fachliche Entscheidung.
- **Formale Beweise** und das Verständnis der Herleitung ersetzt R nicht.
- Bei **kleinen Einzelfällen** ist die Handrechnung oft schneller als ein
  Skript.

## Fazit

R ist vor allem dort effizienter, wo viele gleichartige Rechenschritte oder
Tafel-Lookups nötig sind (Verteilungen, Quantile, Konfidenzintervalle) und wo
Ergebnisse **visualisiert oder simuliert** werden (Prüfergebnisse, Gesetz der
großen Zahlen, Lebensdauer). Der größte Gewinn neben der Zeit ist die
**Reproduzierbarkeit**: ein Skript dokumentiert den ganzen Rechenweg.
