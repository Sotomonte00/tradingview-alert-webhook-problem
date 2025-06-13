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
// Pega aquí el código EMCV de tu estrategia
// Asegurate de que sea la versión simplificada que preparamos:
// - Con strategy() y //@version=5
// - Con variables fijas en lugar de inputs
// - Con lógica de entrada/salida simplificada
// - SIN ninguna función `alert()`
// - SIN `alert_message` en strategy.entry/exit
// - SIN ningún código de dibujo (plot, label, table, etc.)
