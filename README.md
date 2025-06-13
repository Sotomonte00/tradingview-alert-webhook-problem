# Problema: Alertas de Webhook de Estrategia Pine Script v5 NO se Disparan

Este repositorio contiene un Ejemplo Mínimo, Completo y Verificable (EMCV) de un script de estrategia de Pine Script v5 para TradingView.

El objetivo es solicitar ayuda a la comunidad para resolver un problema persistente donde las **alertas de webhook no se disparan** cuando las órdenes de la estrategia se ejecutan en el gráfico de TradingView.

---

## Descripción Detallada del Problema

Mi estrategia (`Francotirador MNQ - Estrategia Semi-Automática (Personalizada) Limit Gemeni`) **funciona correctamente en el gráfico**:
* Ejecuta `strategy.entry()` a mercado o con órdenes límite.
* Gestiona `strategy.exit()` para Stop Loss y Take Profit.
* Las órdenes se registran como "Filled" (Llenadas) en el "Inspector de Estrategia" (pestañas 'Lista de Órdenes' y 'Operaciones').

Sin embargo, **NUNCA recibo NINGUNA alerta de webhook** en mi URL, a pesar de haber configurado todo de manera exhaustiva.

### Pasos de Depuración que he Realizado:

1.  **Código Pine Script (v5):**
    * Confirmado `//@version=5` al inicio y declaración `strategy(...)` correcta.
    * Probé **dos métodos diferentes** para generar alertas:
        * **Con `alert()` en Pine Script:** Usando `alert(miJsonPersonalizado)` y configurando la alarma en TV con `{{alert.message}}`.
        * **Con marcadores de posición directos:** Simplificando el Pine Script (sin `alert()` ni `alert_message`) y colocando un JSON con `{{strategy.order.action}}`, `{{strategy.order.price}}`, etc., directamente en el campo "Mensaje de Webhook" de la alerta de TradingView.
2.  **Configuración de la Alarma en TradingView (Minuciosa):**
    * Siempre borro y recreo la alarma.
    * Refresco TradingView completamente.
    * La "Condición" de la alarma selecciona mi estrategia.
    * La "Opción de Alerta" es siempre **"Cualquier orden ejecutada o modificada"**.
    * El "Webhook URL" está marcada y la URL funciona (probada manualmente).
    * El "Mensaje de Webhook" se configura **EXACTAMENTE** según el método de prueba.
3.  **Observación del Comportamiento:**
    * Las órdenes de la estrategia se ejecutan en el gráfico.
    * **Nunca** se recibe ninguna alerta en mi webhook para ninguna operación.

Estoy muy frustrado ya que he agotado todas las variables que puedo controlar desde mi lado. Esto sugiere un problema en el sistema de alertas interno de TradingView.

---

## Código EMCV (Ejemplo Mínimo, Completo y Verificable)

Aquí les proporciono una versión simplificada de mi estrategia para que puedan revisarla y, si es posible, intentar reproducir el comportamiento. **Este código no incluye toda la lógica compleja ni los dibujos de la estrategia original, solo lo esencial para la ejecución de órdenes.**

```pine
//@version=5

// Primero se definen las constantes o valores fijos que se usarán en strategy()
var float default_qty_value_for_strategy = 1000.0 // O un valor que haga sentido, como 100.0, etc.

strategy("Francotirador - [MODO ]", overlay=true, pyramiding=0, default_qty_type=strategy.cash, default_qty_value=default_qty_value_for_strategy)

// === NUEVA VARIABLE PARA CONTROLAR EL INICIO ===
var bool tradeAllowed = false

// === LÓGICA PARA ACTIVAR ESTRATEGIA SOLO EN VELAS FUTURAS ===
if barstate.islastconfirmedhistory
    tradeAllowed := true  // Solo se activa después de la última vela histórica

// === INPUTS BÁSICOS ===
maxRiskUSD = input.float(120, "Riesgo Máximo USD", minval=1)
minRiskUSD = input.float(58, "Riesgo Mínimo USD", minval=1)
valuePerTick = 0.50   // MNQ = $0.50 por tick
ticksPerPoint = 4     // MNQ = 4 ticks por punto

// === VERSIÓN (CONTRATO FIJO) ===
dynamicQtyContratos = 1.0 // <- SE REEMPLAZÓ LA LÓGICA DINÁMICA POR VALOR FIJO

// Personalización del cartel
labelColor = input.color(color.new(color.white, 20), "Color de fondo")
textColor = input.color(color.black, "Color de texto")
textSize = input.string("large", "Tamaño de letra", options=["tiny", "small", "normal", "large", "huge"])
labelPosition = input.string("right", "Posición del cartel", options=["top", "bottom", "left", "right"])

// Función para tamaño de texto
getSize(size) =>
    switch size
        "tiny" => size.tiny
        "small" => size.small
        "large" => size.large
        "huge" => size.huge
        => size.normal

// === NIVELES CLICABLES ===
nivelActivacion = input.price(0.0, "Nivel de Activación (Haz clic en el gráfico)")
stopLoss = input.price(0.0, "Stop Loss (Haz clic en el gráfico)")

// === DIRECCIÓN DEL TRADE ===
var string tradeDirection = "Indefinida"
if nivelActivacion > 0 and stopLoss > 0
    if stopLoss < nivelActivacion
        tradeDirection := "Largo"
    else if stopLoss > nivelActivacion
        tradeDirection := "Corto"
    else
        tradeDirection := "Indefinida"

// === CÁLCULOS BÁSICOS  ===
var float currentStopDistance = na
if tradeDirection == "Largo"
    currentStopDistance := math.abs(close - stopLoss)
else if tradeDirection == "Corto"
    currentStopDistance := math.abs(close - stopLoss)
else
    currentStopDistance := 0.0

currentStopTicks = currentStopDistance * ticksPerPoint
riskPorContratoCalculated = currentStopTicks * valuePerTick

// === VALIDACIÓN  ===
esValido = dynamicQtyContratos > 0 and tradeDirection != "Indefinida"

// === CÁLCULO DEL TAKE PROFIT  ===
var float takeProfitLevel = na
var float entryPriceReal = na

if strategy.position_size != 0
    if na(entryPriceReal)
        entryPriceReal := strategy.position_avg_price
        takeProfitLevel := tradeDirection == "Largo" ? 
             entryPriceReal + math.abs(entryPriceReal - stopLoss) : 
             entryPriceReal - math.abs(entryPriceReal - stopLoss)
        takeProfitLevel := math.round(takeProfitLevel/syminfo.mintick)*syminfo.mintick
else
    entryPriceReal := na
    takeProfitLevel := na

// === ÓRDENES LÍMITE ===
var bool limitOrderPlaced = false 

// Resetear bandera si se abre posición
if strategy.position_size != 0
    limitOrderPlaced := false 

// === LÓGICA DE SEÑAL DE ACTIVACIÓN ===
bool triggerSignal = false

if barstate.isconfirmed and nivelActivacion > 0 and stopLoss > 0 and not limitOrderPlaced and esValido and strategy.position_size == 0 and tradeAllowed
    // Entrada Largo
    if tradeDirection == "Largo" and close > nivelActivacion and open <= nivelActivacion
        triggerSignal := true
    // Entrada Corto
    else if tradeDirection == "Corto" and close < nivelActivacion and open >= nivelActivacion
        triggerSignal := true


// === LÓGICA DE ÓRDENES LÍMITE ===
if triggerSignal
    string orderId = tradeDirection == "Largo" ? "BuyLimit" : "SellLimit"
    entryPrice = tradeDirection == "Largo" ? close + syminfo.mintick : close - syminfo.mintick
    
    strategy.order(id=orderId, 
         direction=tradeDirection == "Largo" ? strategy.long : strategy.short, 
         qty=1,  // <- Valor fijo (reemplaza dynamicQtyContratos)
         limit=entryPrice, 
         comment="Orden límite comunitaria")
    limitOrderPlaced := true

// --- SALIDAS  ---
if strategy.position_size > 0
    strategy.exit("Exit Long", "BuyLimit", stop=stopLoss, limit=takeProfitLevel)
else if strategy.position_size < 0
    strategy.exit("Exit Short", "SellLimit", stop=stopLoss, limit=takeProfitLevel)



// === DIBUJADO OPTIMIZADO ===
plot(nivelActivacion, "Nivel Activación", color=color.new(color.green, 0), linewidth=2)
plot(stopLoss, "Stop Loss", color=color.new(color.red, 0), linewidth=2)
plot(takeProfitLevel > 0 ? takeProfitLevel : na, "Take Profit (1:1)", color.blue, 2, plot.style_linebr)

// ===== ACTIVACIÓN EN TIEMPO REAL =====
if barstate.islastconfirmedhistory
    tradeAllowed := true

// Gestión de etiquetas
var label infoLabel = na
if not na(infoLabel)
    label.delete(infoLabel)
// Crear nueva etiqueta cuando hay valores
if nivelActivacion > 0 and stopLoss > 0
    // Configuración de posición
    yPos = switch labelPosition
        "top" => nivelActivacion * 1.0005
        "bottom" => nivelActivacion * 0.9995
        => nivelActivacion
    
    xOffset = labelPosition == "left" ? -10 : labelPosition == "right" ? 10 : 0
    xPos = bar_index + xOffset
    
    // Configuración de estilo
    labelStyle = switch labelPosition
        "left" => label.style_label_right
        "right" => label.style_label_left
        => label.style_label_center


    // Crear etiqueta 
    labelText = "Dirección: " + tradeDirection + 
          "\nPrecio Entrada: " + (na(entryPriceReal) ? "N/A" : str.tostring(entryPriceReal, format.mintick)) + 
          "\nEstado: " + (esValido ? "✅ ACTIVO" : "❌ INACTIVO")

    infoLabel := label.new(
         x=xPos,
         y=yPos,
         text=labelText,
         style=labelStyle,
         color=labelColor,
         textcolor=textColor,
         size=getSize(textSize),
         textalign=text.align_center)
