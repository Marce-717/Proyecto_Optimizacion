# DOCUMENTO MAESTRO DEL PROYECTO

**Título:** Desarrollo de un modelo para optimizar el diseño de sistemas de riego presurizado, basado en parámetros hidráulicos, costos de inversión y operación: un estudio de caso.

**Estudiante:** Martín Alfredo Boudoire Troncoso \
**Profesor Guía:** Julio Haberland Arellano \
**Programa:** AFE — Magíster en Manejo de Suelos y Aguas, Universidad de Chile \
**Horizonte:** 2026 – 2027

## 1. Propósito del documento maestro

Este documento traduce la pauta guía (`Optimización de Riego Localizado.docx` / `Proyecto MBT.md`) en una **hoja de ruta de desarrollo**: define el alcance técnico, la arquitectura del software, los módulos a programar, los datos de entrada y salida, las ecuaciones a implementar y los hitos de avance. Es el documento de referencia transversal para todo el ciclo de vida del proyecto (desarrollo, validación, estudio de caso y redacción del informe final).

## 2. Resumen operativo

Se desarrollará una herramienta computacional de código abierto, implementada en Python 3.12, que resuelve el dimensionamiento óptimo (diámetros comerciales) de redes de riego localizado de alta frecuencia (RLAF) de topología ramificada telescópica. El optimizador minimiza el **Valor Actual Neto (VAN) de costos** sobre horizontes de 1, 10 y 20 años, considerando CAPEX (tuberías) y OPEX (energía eléctrica de bombeo bajo tarifa AT-4.3). El modelo se acopla a **EPANET 2.2** mediante **WNTR** para la verificación hidráulica, lee la topología desde **QGIS** vía GeoPackage (`.gpkg`) y se valida en un predio real.

## 3. Pregunta de investigación y objetivos

**Pregunta:** ¿Es posible optimizar y mejorar la eficiencia técnico-económica del diseño de redes de riego localizado mediante un modelo automatizado de código abierto, en comparación con las metodologías convencionales?

**Objetivo general:** Desarrollar un modelo de optimización de código abierto que maximice la eficiencia hidráulico-económica del diseño de RLAF sobre un diseño existente.

**Objetivos específicos:**
1. Formular el modelo de optimización, estableciendo funciones de costo y restricciones hidráulico-físicas.
2. Validar la consistencia hidráulica del modelo con EPANET.
3. Evaluar la eficiencia técnico-económica mediante un estudio de caso, cuantificando diferencias en CAPEX y OPEX frente a un diseño convencional.

## 4. Alcance y supuestos

- Redes **ramificadas telescópicas** (diámetros decrecientes aguas abajo), no malladas.
- Régimen de **flujo permanente** en el escenario operativo crítico (máxima ETc).
- Catálogo comercial discreto **Hoffens PVC**: PN6, PN8, PN10, PN12,5, PN16; ~16 diámetros nominales (32–400 mm).
- Tarifa eléctrica chilena regulada **AT-4.3** (energía, potencia máxima, potencia en punta configurable).
- Tasa de descuento anual constante del **10 %**.
- Horizontes de evaluación: **1, 10 y 20 años**.
- Velocidad admisible: **0,5 ≤ v ≤ 2,0 m/s**.
- Presión mínima en el nodo más desfavorable ≥ **presión nominal del emisor**.

  ## 5. Arquitectura de la herramienta computacional

```json
┌─────────────────────────────────────────────────────────────┐
│                     CAPA DE ENTRADA (GIS)                   │
│  QGIS 3.34 → red.gpkg (nodos, tramos, cotas, demandas)      │
└──────────────────────────────┬──────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────┐
│                  CAPA DE LÓGICA (Python 3.12)               │
│  ┌─────────────┐  ┌─────────────┐  ┌────────────────────┐   │
│  │  io_gis     │→ │ hidraulica  │→ │  optimizador       │   │
│  │ (lectura)   │  │ (HW, DT, v) │  │ (búsqueda          │   │
│  └─────────────┘  └─────────────┘  │  exhaustiva        │   │
│  ┌─────────────┐  ┌─────────────┐  │  telescópica)      │   │
│  │ catalogo    │  │ costos      │  └─────────┬──────────┘   │
│  │ (Hoffens)   │  │ (CAPEX/VAN) │            │              │
│  └─────────────┘  └─────────────┘            │              │
│  ┌─────────────┐  ┌─────────────┐            │              │
│  │ tarifa_AT43 │  │ energia     │            │              │
│  │ (OPEX)      │  │ (kWh/anual) │            │              │
│  └─────────────┘  └─────────────┘            │              │
└──────────────────────────────────────────────┼──────────────┘
                                               │
┌──────────────────────────────────────────────▼──────────────┐
│          CAPA DE VERIFICACIÓN FÍSICA (EPANET 2.2)           │
│                    WNTR API ← red.inp                       │
└──────────────────────────────┬──────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────┐
│               CAPA DE SALIDA Y REPORTE                      │
│  resultados.csv · reporte.html · figuras.png · red_opt.inp  │
└─────────────────────────────────────────────────────────────┘
```

### 5.1 Stack tecnológico

| Componente | Herramienta | Versión | Rol |
|---|---|---|---|
| GIS | QGIS | 3.34 | Trazado de red, cotas, exportación `.gpkg` |
| Lenguaje | Python | 3.12 | Lógica del optimizador |
| IDE | Spyder 3 | — | Edición, debug, control de versiones |
| Simulador hidráulico | EPANET | 2.2 | Verificación física |
| API acoplamiento | WNTR | ≥ 1.1 | Lectura/escritura `.inp` desde Python |
| Librerías numéricas | NumPy, pandas, SciPy | — | Álgebra, I/O tabular |
| Geoespacial | GeoPandas, Shapely, Fiona | — | Lectura `.gpkg` |
| Visualización | matplotlib, plotly | — | Figuras y reportes |
| Asistencia IA | Claude (Anthropic) | — | Soporte a programación y verificación sintáctica |
| Control versiones | Git | — | Historial y respaldo |
