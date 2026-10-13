# UIXSS — Indicador de Smart Money Concepts

Indicador de TradingView (Pine Script v5) que marca **en vivo** los conceptos del sistema (basado en los PDFs *Glosario* y *FVGs y Order Blocks*) y genera **señales Long/Short** por confluencia.

- **Archivos:** `UIXSS.pine` (indicador) · `UIXSS_BACKTEST.pine` (strategy para backtesting)
- **Nombre en TradingView:** `UIXSS` (aplicado en tu gráfico y guardado en tu nube)
- **Versión:** v5
- **Probado en:** Bybit BTCUSDT.P

---

## Cómo instalarlo / actualizarlo

Ya está aplicado. Para reinstalar: Editor Pine → pegar `UIXSS.pine` → **Adicionar ao gráfico** → **Ctrl+S** (guardar).

> TradingView Desktop se auto-actualiza y a veces se reinicia. El script en la nube **no se pierde**.

---

## Qué marca (concepto por concepto)

| Concepto (PDFs) | Estado | En el gráfico |
|---|---|---|
| **Order Blocks multi-TF** (4H/12H/1D/1W/1M) | ✅ | Líneas horizontales fijas, color por TF, etiquetas escalonadas |
| **OB del TF actual + Delta** | ✅ | Caja (contorno) con "OB Δ +/−" (compra vs venta) |
| **FVG / Imbalance** (3 velas) | ✅ | Cajas verdes/rojas "FVG" (ventana reciente) |
| **ChoCh** (cambio de carácter) | ✅ | Etiqueta "ChoCh" (ventana reciente) |
| **BOS / Ruptura** | ✅ | Etiqueta "BOS" (off por defecto) |
| **Premium / Discount / Equilibrium** | ✅ | Líneas punteadas (roja arriba, verde abajo, media) |
| **Liquidez EQH / EQL** | ✅ | Líneas punteadas "EQH" / "EQL" |
| **Barridos de liquidez** (liq. long/short) | ✅ | Triángulos ▲ (sell-side) / ▽ (buy-side) |
| **Weak High / Low** | ✅ | Marcas chicas ▵ / ▿ (swing roto) |
| **Strong High / Low** | ✅ | Línea roja/lima del swing que aguantó y causó la ruptura |
| **Auto-ajuste** | ✅ | Las etiquetas siguen el borde derecho al hacer zoom/scroll |
| **Pullback** | ✅ | Línea **azul** de retroceso |
| **Señales LONG / SHORT** | ✅ | Etiquetas verde/rojo + alertas |
| **Reversal** | ⚠️ | Lo cubre el ChoCh |
| **Liquidaciones Coinglass** | ❌ | Externo a TradingView |

---

## Señales LONG / SHORT (confluencia)

Es lo que convierte el análisis en un gatillo concreto:

- **LONG** = barrido *sell-side* (toma liquidez bajo un mínimo y cierra arriba) **estando en zona Discount** (bajo el Equilibrium) **+ delta a favor** (compra).
- **SHORT** = barrido *buy-side* **estando en zona Premium** **+ delta a favor** (venta).

Extras:
- **Cooldown** (`Barras mínimas entre señales`, default 6) para que no se apilen.
- **Alertas** listas: creá una alerta en TradingView y elegí "UIXSS LONG" o "UIXSS SHORT".
- `Exigir delta a favor` (on/off) para filtrar más o menos.

> Es un modelo de **confluencia**, no una bola de cristal. Siempre confirmá con tu propio criterio y gestioná el riesgo (SL/TP).

---

## En qué temporalidad usarlo (y por qué)

Regla general de este tipo de sistema (barridos + estructura):

| TF | Señales | Ruido | Uso |
|----|---------|-------|-----|
| 1–5m | Muchísimas | Muy alto | Scalping; muchos barridos son ruido. No recomendado solo. |
| **15m–1h** | Medias | Medio | **Intradía**. Buen balance: la estructura ya es significativa. |
| **4h** | Pocas | Bajo | **Swing**. Señales más limpias y de mayor probabilidad. Sweet spot para BTC. |
| 1D+ | Muy pocas | Muy bajo | Posición / sesgo de fondo. |

**Por qué:** cuanto más subís de temporalidad, **menos señales pero más limpias** (la huella institucional se ve mejor y hay menos ruido retail). En TFs bajos hay muchos barridos que son solo ruido.

**Forma recomendada de usarlo (multi-timeframe):**
1. **4h / 1D** → sesgo y Order Blocks de referencia (las líneas de color ya te los muestran en cualquier TF).
2. **15m / 1h** → afinar la entrada y ver ChoCh/FVG recientes.
3. Tomar la señal cuando **coincide** con un OB de TF alto (confluencia).

**¿La más rentable? — medido con el backtest** (`UIXSS_BACKTEST.pine`, BTCUSDT.P, ene-2024 → sep-2026, capital 1000, RR 2:1, delta a favor).

**Base (sin filtros):**

| TF | Neto | Win rate | Profit Factor | DD máx | Trades |
|----|------|----------|---------------|--------|--------|
| 15m | −2,01% | 33,0% | 0,51 ❌ | 2,15% | 212 |
| 1h | −1,77% | 36,6% | 0,78 ❌ | 2,68% | 186 |
| **4h** | +1,35% | 37,4% | 1,12 | 1,87% | 131 |
| 1D | +0,18% | 36,4% | 1,01 | 5,31% | 55 |

**Optimizado (filtro de tendencia EMA200 de 4h + momentum):**

| TF | Neto | Win rate | Profit Factor | DD máx | Trades |
|----|------|----------|---------------|--------|--------|
| 15m | −1,37% | 25,5% | 0,38 ❌ | 1,48% | 98 |
| 1h | −4,22% | 11,1% | 0,20 ❌ | 5,54% | 9 |
| **4h** | **+2,76%** | 41,3% | **1,76 ✅** | **0,86%** | 46 |
| 1D | +6,92% | 100% | — ⚠️ | 0,53% | 3 (muestra chica) |

**Conclusión:** **4h es la temporalidad ganadora.** La optimización subió el profit factor de 1,12 → **1,76**, duplicó el retorno y bajó el drawdown a la mitad, con menos trades pero de mejor calidad (46). El 1D marca +6,92% pero con solo 3 trades = no confiable. 15m/1h pierden. Los filtros están calibrados para 4h. *Esto no es asesoramiento financiero.*

---

## Ajustes principales (engranaje del indicador)

- **Order Blocks Multi-TF:** on/off por TF + color · `Distancia máxima al precio (%)` (filtro, default 12) · `Atenuar OB mitigado` · `Zona completa`
- **ChoCh/FVG:** `Ventana en horas` (default 24 — se ven mejor en 15m/1h) · `Fuerza swing`
- **Zonas/Liquidez:** Premium/Discount, EQH/EQL, Barridos, Pullback, Panel Delta
- **OB del TF actual + Delta:** cantidad en pantalla, colores
- **Señales:** on/off · `Filtro de Tendencia` (EMA TF Mayor / Estructura Local ChoCh / Desactivado) · `Exigir desplazamiento institucional` · `Exigir momentum / reclaim` · `Barras mínimas entre señales` · `Mostrar SL / TP / BE` (RR 2:1)
- **Absorción y Confluencia HTF (Astra v2):** `Exigir mecha de rechazo prominente` (filtra rupturas sin absorción) · `Exigir volumen relativo (RVOL)` (volumen institucional > SMA20) · `Exigir confluencia con Order Block HTF` (ancla la entrada a OB de 4H/1D) · `Mover SL a Break-Even en +1R` (blindaje automático de operaciones en ganancia)

---

## Limitaciones (honesto)

1. **Delta** = aproximado por posición del cierre en el rango (no order-flow real por zona). El panel superior sí usa volumen intrabar de 1m de la vela actual.
2. **FVG/ChoCh** casi no se ven en 4h (ventana de 24h ≈ 6 velas). Mejor en 15m/1h.
3. Sin **Coinglass** (mapa de liquidaciones): es externo.

---

## Solución de problemas

**El eje de precios se ve raro / muy alejado (ej: de 155 a 5.8M):** es un estado de la *vista* del gráfico (escala en manual o logarítmica), NO del indicador. Se arregla con **doble-click sobre el eje de precios** (la columna de números a la derecha), o click en el botón **"A"** (auto) abajo a la derecha del eje. El indicador no dibuja nada en valores extremos (todo queda entre ~44k y ~107k).

## Roadmap

- [x] ~~Backtesting real (strategy)~~ → `UIXSS_BACKTEST.pine` (hecho; ver tabla arriba)
- [x] ~~Strong High/Low~~ (hecho en v5)
- [x] ~~Optimizar el strategy~~ → filtro de tendencia (EMA) + momentum: PF 1,12 → **1,76** en 4h (v2)
- [x] ~~Exigir que la señal coincida con un **OB de TF alto**~~ (incorporado en SMC v2 - Astra)
- [x] ~~Absorción de volumen institucional (Mecha de rechazo + RVOL)~~ (incorporado en SMC v2)
- [x] ~~Gestión de riesgo con Break-Even automático a +1R~~ (incorporado en SMC v2)
- [ ] Delta real por order block (order flow con footprint).

## Cómo correr el backtest

1. Editor Pine → pegar `UIXSS_BACKTEST.pine` → **Adicionar ao gráfico**.
2. Se abre el **Strategy Tester** abajo con L&P, win rate, profit factor, drawdown.
3. Cambiá de temporalidad (15m/1h/4h/1D) y compará. Ajustá RR, delta y cooldown en la config del strategy.

---

*Generado con Claude Code. No es asesoramiento financiero.*
