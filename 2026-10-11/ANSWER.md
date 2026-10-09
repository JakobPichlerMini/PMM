# Antwort: Welche Inhalte der 3. Klasse lassen sich mit R effizienter lösen?

*Antwort auf [`PROMPT.md`](PROMPT.md) – erzeugt mit OpenCode.*

## Kurzfassung

Praktisch der **gesamte Statistikstoff der 3. Klasse (KM5 + KM6)** lässt sich
mit R effizienter lösen als „per Hand“. Der Grund ist immer derselbe:

- **Wahrscheinlichkeitstafeln und Formeln werden überflüssig** – die
  `d/p/q/r`-Funktionsfamilie liefert Dichte, Verteilung, Quantile und
  Zufallszahlen für jede Verteilung sofort.
- **Rechenfehler entfallen** – statt mehrstufiger Handrechnung schreibt man
  eine Zeile Code, die reproduzierbar und prüfbar ist.
- **Daten statt Spielbeispiele** – echte Datensätze (z. B.
  `assets/betriebsdaten.csv`) lassen sich direkt auswerten.
- **Visualisierung und Simulation** schaffen didaktischen Mehrwert, der mit
  Taschenrechner und Tafel gar nicht möglich ist.
- **Reproduzierbarkeit** – ein R-Skript/Quarto-Dokument dokumentiert den
  gesamten Rechenweg und ist jederzeit erneut ausführbar.

## Vergleichstabelle

| Inhalt (KM) | Klassische Hand-Methode | R-Lösung | Gewinn |
|---|---|---|---|
| Zufallsvariablen, f(x) vs. F(x) (KM5) | Tafelwerk, Rechnen von Summen | `dbinom`, `pnorm`, Plot f vs. F | Rechenerleichterung + Anschauung |
| Diskrete Verteilungen (KM5) | Binomialkoeffizient, Tabellen | `dbinom`, `phyper`, `dpois` | Zeitgewinn, exakte „mindestens/maximal“-Werte |
| Normalverteilung & Standardisierung (KM5) | z-Tabelle, Interpolation | `pnorm`, `qnorm` | tabellefrei, beliebige μ/σ |
| Exponentialverteilung (KM5) | Integral von Hand | `pexp`, `dexp`, `rexp` | sofort, inkl. „Gedächtnislosigkeit“ prüfbar |
| Lage- und Streumaße (KM5) | Summen von Hand, mühsam bei n>10 | `mean`, `median`, `sd`, `quantile` | skaliert mit beliebigem n |
| Parameter vs. Schätzwerte (KM5) | Bessel-Formel, Gefühl für n | `var`/`sd` (n−1), Simulation | Gesetz der großen Zahlen sichtbar |
| Vertrauensbereiche (KM6) | Tafeln für z/t/χ², Fehlerquelle | `qnorm`, `qt`, `qchisq`, `t.test` | automatisch, reproduzierbar |
| Prüfergebnisse darstellen (KM6) | Excel-Diagramm, manuell | `ggplot2`, Quarto | großer Vorteil, reproduzierbar |
| Kennzahlen & Ausreißer (KM6) | 1,5·IQR-Regel per Hand | `IQR`, `quantile`, Filterlogik | Skript statt Einzelfall |
| Lebensdauer (KM6) | Kurvenpapier, Probierlösung | `fitdistrplus`, `pweibull`, `pexp` | Verteilungsfit statt Schätzen |

## KM5 – Verteilungen & Schätzer

### 1. Zufallsvariablen und Wahrscheinlichkeit (f(x) vs. F(x))

Mit R lässt sich der Unterschied zwischen **Dichte/PMF** `d…()` und
**Verteilung** `p…()` unmittelbar zeigen – statt abstrakt an der Tafel:

```r
# Binomial: PMF vs. CDF
dbinom(3, size = 10, prob = 0.4)   # P(X = 3)
pbinom(3, size = 10, prob = 0.4)   # P(X <= 3)

# f(x) und F(x) plotten
x <- 0:10
plot(x, dbinom(x, 10, 0.4), type = "h", lwd = 9,
     main = "Dichte f(x)", xlab = "x", ylab = "P(X = x)")
plot(x, pbinom(x, 10, 0.4), type = "s",
     main = "Verteilung F(x)", xlab = "x", ylab = "P(X <= x)")
```

**Effizienter**, weil das mühsame Summieren von Einzelwahrscheinlichkeiten zu
einer einzeiligen Operation wird – und der qualitative Unterschied f vs. F
durch zwei Plots sofort „klick“ macht.

### 2. Diskrete Verteilungen (Binomial, Hypergeometrisch, Poisson)

Klassisch braucht man Binomialkoeffizienten und Tabellen; in R liefert jede
Verteilung ihre `d/p/q/r`-Familie:

```r
# Binomial: höchstens / mindestens
dbinom(2, 20, 0.1)          # genau 2 Treffer
pbinom(2, 20, 0.1)          # höchstens 2
1 - pbinom(1, 20, 0.1)      # mindestens 2

# Hypergeometrisch (Ziehen ohne Zurücklegen)
phyper(q = 1, m = 5, n = 15, k = 4)   # P(X <= 1)

# Poisson: Ereignisse pro Einheit
dpois(3, lambda = 2)        # P(X = 3) bei λ = 2
ppois(3, lambda = 2)        # P(X <= 3)
```

**Effizienter**, weil „höchstens“, „mindestens“, „genau“ und der Wechsel
zwischen den Verteilungen nur ein anderes Funktionspräfix sind – genau die
Unterscheidungen, die per Hand die meisten Fehler produzieren.

### 3. Normalverteilung und Standardisierung

z-Tabellen sind der klassische Zeitfresser. R rechnet direkt mit jeder
Kombination aus μ und σ:

```r
# Standardisierung vs. direkt
z <- (85 - 100) / 15
pnorm(z)                     # über z
pnorm(85, mean = 100, sd = 15)   # direkt, gleiches Ergebnis

qnorm(0.975)                 # z-Wert 1,96 ohne Tabelle

# 68-95-99,7-Regel prüfen
pnorm(1) - pnorm(-1)         # 0.6827
pnorm(2) - pnorm(-2)         # 0.9545
pnorm(3) - pnorm(-3)         # 0.9973
```

**Effizienter**, weil keine Tabelle mehr nötig ist und man die Regel sofort
nachrechnen kann – Verständnis der z-Transformation kommt mit dazu.

### 4. Exponentialverteilung

```r
pexp(5, rate = 0.2)          # P(X <= 5) bei λ = 0,2
qexp(0.5, rate = 0.2)        # Median
mean(rexp(10000, rate = 0.2)) # Simulation → ~ 1/λ = 5
```

**Effizienter**, weil das Integral von Hand entfällt und die
**Gedächtnislosigkeit** per Simulation überprüfbar ist:
`pexp(5, 0.2) == pexp(10, 0.2) - pexp(5, 0.2)` – nur Stichproben bzw.
Rechenweg statt Theoriebeweis.

### 5. Lage- und Streumaße

```r
x <- c(12, 15, 18, 22, 25, 29)
mean(x); median(x); sd(x); var(x); range(x); IQR(x)
quantile(x, probs = c(0.25, 0.5, 0.75))
```

**Effizienter**, weil die Handrechnung mit der Anzahl der Werte wächst, R aber
mit jedem n gleich schnell bleibt – und `sd()` gleich die richtige
(Bessel-)Definition verwendet.

### 6. Parameter vs. Schätzwerte (Bessel n−1, Gesetz der großen Zahlen)

```r
# Stichprobenvarianz mit n−1 vs. n
x <- c(2, 4, 4, 4, 5, 5, 7, 9)
var(x)                                  # n−1 (Bessel) = 4.571
mean((x - mean(x))^2)                   # n     = 4.0

# Gesetz der großen Zahlen sichtbar machen
set.seed(1)
mw <- cumsum(rnorm(2000)) / 1:2000
plot(mw, type = "l", ylim = c(-1, 1),
     main = "Gesetz der großen Zahlen", xlab = "n", ylab = "Mittelwert")
abline(h = 0, col = "red")
```

**Effizienter + didaktischer Mehrwert**: Die Bessel-Korrektur wird nicht nur
„behauptet“, sondern der Unterschied n vs. n−1 ist nachrechenbar; das Gesetz
der großen Zahlen wird als Kurve erlebbar.

## KM6 – Konfidenz, Darstellung & Lebensdauer

### 7. Vertrauensbereiche (SE, z-, t-, χ²-Verteilung)

```r
x <- c(101, 99, 104, 98, 102, 100, 97, 103)

# KI für μ bei unbekanntem σ über die t-Verteilung
t.test(x, conf.level = 0.95)$conf.int

# Manuell: SE = s/√n, t-Quantil
n <- length(x); se <- sd(x) / sqrt(n)
mean(x) + c(-1, 1) * qt(0.975, df = n - 1) * se

# z-Wert ohne Tabelle, χ²-Quantil für σ²
qnorm(0.975)          # 1.96
qchisq(0.975, df = 7)
```

**Effizienter**, weil die (fehleranfällige) Arbeit mit mehreren Tafeln
(z, t, χ²) durch jeweils ein Quantil ersetzt wird – und `t.test()` den
Standardfall in einer Zeile erledigt.

### 8. Auswertung und Darstellung von Prüfergebnissen

Das ist der Inhalt mit dem **größten Effizienzvorteil**, weil er von Natur aus
grafisch ist:

```r
library(ggplot2)

ggplot(daten, aes(x = ergebnis)) +
  geom_histogram(bins = 15, fill = "steelblue", colour = "white")

ggplot(daten, aes(y = ergebnis)) +
  geom_boxplot(fill = "lightgreen")

ggplot(daten, aes(sample = ergebnis)) +
  geom_qq() + geom_qq_line(colour = "red")   # Normalitätscheck

ggplot(daten, aes(x = herkunft, y = ergebnis)) +
  geom_point()                               # Streudiagramm
```

**Effizienter**, weil in Excel jede Grafik manuell neu geklickt werden muss,
während in R einmal geschriebener Code beliebig viele Datensätze gleich
darstellt – und per Quarto direkt in einen Bericht fließt.

### 9. Kennzahlen und Ausreißer (IQR, 1,5·IQR-Regel)

```r
q1 <- quantile(x, 0.25); q3 <- quantile(x, 0.75)
iqr <- IQR(x)
grenzen <- c(q1 - 1.5 * iqr, q3 + 1.5 * iqr)
x[x < grenzen[1] | x > grenzen[2]]          # Ausreißer
```

**Effizienter**, weil die Regel als Filter in einer Zeile steckt statt als
Einzelprüfung pro Wert.

### 10. Lebensdauerverteilungen (Weibull/Exponential, MTBF/MTTF)

```r
library(fitdistrplus)

# Weibull an Ausfalldaten anpassen
fit <- fitdist(ausfalldaten, "weibull")
shape <- fit$estimate["shape"]   # β
scale <- fit$estimate["scale"]   # η
1 / shape                        # 63,2-%-Faustregel-Hinweis

# MTBF als Erwartungswert
MTBF <- fit$estimate["scale"] * gamma(1 + 1 / shape)

# Ausfallwahrscheinlichkeit zu einem Zeitpunkt
pweibull(1000, shape = shape, scale = scale)
```

**Effizienter**, weil das Anpassen einer Verteilung (früher Kurvenpapier und
Probieren) nun automatisch geschieht und MTBF/MTTF direkt berechenbar sind.

## Grenzen – wo R keinen Vorteil bringt

1. **Verständnis der Konzepte:** R liefert das Ergebnis, aber die Bedeutung
   von Erwartungstreue, Standardfehler oder KI-Breite muss man trotzdem
   verstehen. `t.test()` „erklärt“ keine Inferenz.
2. **Modellwahl:** Welche Verteilung passt? Diese Entscheidung trifft R nicht
   automatisch – sie erfordert Fachurteil (und oft Plots).
3. **Formale Herleitungen:** Beweise (z. B. Varianz der Binomialverteilung)
   lernt man per Hand; das Ergebnis aus R allein ist kein Beweis.
4. **Triviale Einzelfälle:** Ein Mittelwert aus fünf Zahlen ist per Hand
   schneller als ein Skript (Einrichtungsaufwand).
5. **Interpretation und Kontext:** Wirtschaftliche/organisatorische
   Schlussfolgerungen aus Zahlen bleiben menschliche Arbeit.

## Fazit

In der 3. Klasse ist R besonders dort **effizienter**, wo viele gleichartige
Rechenschritte oder Tafel-Lookups nötig sind (Verteilungen, Quantile,
Konfidenzintervalle) und überall dort, wo Ergebnisse **visualisiert** oder
**simuliert** werden (Prüfergebnisse, Gesetz der großen Zahlen,
Lebensdauer). Der größte Gewinn ist aber nicht nur Geschwindigkeit, sondern
**Reproduzierbarkeit**: Ein einziges R-Skript dokumentiert den gesamten
Rechenweg und lässt sich in Quarto direkt als Bericht ausgeben.
