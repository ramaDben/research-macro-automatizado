# Sistema de Research Macro Automatizado

> **Caso de estudio.** Este repositorio documenta la arquitectura y los resultados
> de un sistema en producción. **No contiene el código fuente**: la implementación
> es privada. Lo que encontrarás aquí es el problema que resuelve, cómo está
> construido, y qué produce.

---

## El problema

Producir análisis de mercado diario es un trabajo que no escala bien a mano.
Cada jornada exige lo mismo: recolectar precios de varios activos, calcular
niveles técnicos, cruzarlos con datos macroeconómicos oficiales, revisar qué dijo
la prensa financiera, y sintetizar todo en un informe legible antes de que abra
el mercado.

Hecho manualmente son entre dos y tres horas diarias. Y tiene tres defectos que
importan más que el tiempo:

1. **No es reproducible.** Dos analistas con los mismos datos llegan a informes
   distintos, y ninguno puede reconstruir cómo llegó al de ayer.
2. **No es verificable.** Cuando una cifra sale mal, no hay forma de rastrear de
   dónde salió.
3. **No es escalable.** Cubrir más activos significa proporcionalmente más horas.

## La solución

Un pipeline que convierte ese proceso en una operación determinista y trazable.

```text
   FUENTES DE DATOS              PROCESAMIENTO              SALIDA
   ─────────────────             ─────────────              ──────

   Terminal de mercado  ─┐
   (precios OHLC)        │
                         ├──►  Cálculo técnico    ─┐
   API Banco Central    ─┤     (EMA, ATR, niveles  │
   de Chile              │      dinámicos)         │
   (tasas, IPC, macro)   │                         ├──►  Informe diario
                         │                         │     verificado
   Calendario macro     ─┤     Orquestación de    ─┤     (HTML + PNG)
   (eventos y sorpresas) │     agentes de IA       │
                         │     (síntesis y         │     + mensaje de
   Prensa financiera    ─┘      verificación       │       distribución
   (fuentes acotadas)          cruzada)           ─┘
```

### Decisiones de diseño que vale la pena explicar

**Separación entre datos y narrativa.** Los números nunca los produce un modelo
de lenguaje. Los precios y los indicadores vienen del terminal de mercado y de la
API del Banco Central; los agentes solo redactan sobre datos ya calculados. Esto
elimina de raíz el riesgo de que el sistema invente una cifra.

**Verificación cruzada antes de publicar.** Cada afirmación con contenido
factual se contrasta contra la fuente antes de entrar al informe. Lo que no se
puede verificar, no se publica.

**Contrato de error explícito.** Si una fuente no responde, el sistema devuelve
un error identificable, nunca un valor vacío ni un silencio. Un informe que no se
genera es un problema menor; un informe con un dato inventado es un problema
grave.

**Salida pensada para el canal.** El informe se renderiza a resolución retina
para que sea legible en pantalla de teléfono, porque ahí es donde efectivamente
se lee.

## Resultados

| | Antes | Después |
|---|---|---|
| Tiempo por informe | 2–3 horas | Minutos |
| Reproducibilidad | Ninguna | Total: mismo input, mismo output |
| Trazabilidad de cifras | Manual | Cada dato apunta a su fuente |
| Activos cubiertos | Limitado por horas | Limitado por cobertura de datos |

## Stack

**Python** para el pipeline y la lógica de cálculo · **APIs REST** para ingesta de
datos oficiales · **Orquestación de agentes de IA** para síntesis y verificación ·
**Renderizado HTML a imagen** para la salida.

## Ejemplos de output

_En preparación._

---

## Por qué el código es privado

Este sistema es metodología aplicada, no una librería de propósito general. La
arquitectura la comparto con gusto — de hecho está arriba. La implementación no,
por la misma razón por la que un fondo publica su tesis pero no su modelo.

Si estás evaluando mi trabajo y necesitas ver más, escríbeme y lo conversamos:
puedo mostrar el sistema funcionando en una llamada.

**Benjamín Bravo Soza** — [LinkedIn](https://linkedin.com/in/benjaminbravosoza) ·
benjamin.bravo@redieb.cl
