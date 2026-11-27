# DOCUMENTO MAESTRO DEL PROYECTO

**Título:** Desarrollo de un modelo para optimizar el diseño de sistemas de riego presurizado, basado en parámetros hidráulicos, costos de inversión y operación: un estudio de caso.

**Estudiante:** Martín Alfredo Boudoire Troncoso
**Profesor Guía:** Julio Haberland Arellano
**Programa:** AFE — Magíster en Manejo de Suelos y Aguas, Universidad de Chile
**Horizonte:** 2026 – 2027

---

## 1. Propósito del documento maestro

Este documento traduce la pauta guía (`Optimización de Riego Localizado.docx` / `Proyecto MBT.md`) en una **hoja de ruta de desarrollo**: define el alcance técnico, la arquitectura del software, los módulos a programar, los datos de entrada y salida, las ecuaciones a implementar y los hitos de avance. Es el documento de referencia transversal para todo el ciclo de vida del proyecto (desarrollo, validación, estudio de caso y redacción del informe final).

## 2. Resumen operativo

Se desarrollará una herramienta computacional de código abierto, implementada en Python 3.12, que resuelve el dimensionamiento óptimo (diámetros comerciales) de redes de riego localizado de alta frecuencia (RLAF) de topología ramificada telescópica. El optimizador minimiza el **Valor Actual Neto (VAN) de costos** sobre horizontes de 1, 10 y 20 años, considerando CAPEX (tuberías) y OPEX (energía eléctrica de bombeo bajo tarifa AT-4.3). El modelo se acopla a **EPANET 2.3** mediante **EPyT** (EPANET-Python Toolkit) para la verificación hidráulica, lee la topología desde **QGIS** vía GeoPackage (`.gpkg`) y se valida en un predio real.

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

```
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
│          CAPA DE VERIFICACIÓN FÍSICA (EPANET 2.3)           │
│                    EPyT API ← red.inp                       │
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
| IDE | VS Code | — | Edición, debug, control de versiones |
| Simulador hidráulico | EPANET | 2.3 | Verificación física |
| API acoplamiento | EPyT | ≥ 2.3.5 | Interfaz Python al toolkit de EPANET (`.inp`) |
| Librerías numéricas | NumPy, pandas, SciPy | — | Álgebra, I/O tabular |
| Geoespacial | GeoPandas, Shapely, Fiona | — | Lectura `.gpkg` |
| Visualización | matplotlib, plotly | — | Figuras y reportes |
| Asistencia IA | Claude (Anthropic) | — | Soporte a programación y verificación sintáctica |
| Control versiones | Git | — | Historial y respaldo |

### 5.2 Estructura de carpetas propuesta

```
Tesis_Martin_bouduarie_riego/
├── DOCUMENTO_MAESTRO.md                ← este archivo
├── Proyecto MBT.md                     ← pauta guía (AFE)
├── Optimización de Riego Localizado.docx
├── Proyecto_tesis.ipynb                ← notebook exploratorio
├── src/
│   ├── __init__.py
│   ├── io_gis.py                       ← lectura .gpkg → grafo
│   ├── catalogo.py                     ← catálogo Hoffens PVC
│   ├── hidraulica.py                   ← Hazen-Williams / Darcy, velocidades
│   ├── tarifa.py                       ← modelo AT-4.3
│   ├── energia.py                      ← kWh anuales de bombeo
│   ├── costos.py                       ← CAPEX, OPEX, VAN
│   ├── optimizador.py                  ← búsqueda exhaustiva telescópica
│   ├── epanet_bridge.py                ← EPyT: export .inp y verificación
│   └── reporte.py                      ← tablas, figuras, HTML
├── data/
│   ├── gis/                            ← redes .gpkg del predio
│   ├── catalogo_hoffens.csv            ← diámetros, PN, costos
│   ├── tarifas_at43.csv                ← cargos vigentes
│   └── cultivo_etc.csv                 ← demanda crítica
├── outputs/
│   ├── resultados/                     ← CSV por corrida
│   ├── figuras/                        ← PNG/SVG
│   └── reportes/                       ← HTML/PDF
├── tests/
│   ├── test_hidraulica.py
│   ├── test_costos.py
│   └── test_optimizador.py
├── docs/
│   └── (memoria técnica, manual de uso)
├── pyproject.toml / requirements.txt
└── README.md
```

## 6. Modelo matemático

### 6.1 Función objetivo

$$
\min_{\mathbf{D} \in \mathcal{C}^n}\; VAN(\mathbf{D}) = CAPEX(\mathbf{D}) + \sum_{t=1}^{T} \frac{OPEX_t(\mathbf{D})}{(1+r)^t}
$$

donde:
- $\mathbf{D} = (D_1, D_2, \dots, D_n)$ vector de diámetros nominales asignados a los $n$ tramos de la red.
- $\mathcal{C}$ catálogo discreto de diámetros comerciales (Hoffens PVC).
- $T \in \{1, 10, 20\}$ años.
- $r = 0{,}10$ tasa de descuento anual.

### 6.2 CAPEX

$$CAPEX(\mathbf{D}) = \sum_{i=1}^{n} L_i \cdot c(D_i, PN_i)$$

$L_i$ longitud del tramo $i$ (m); $c$ costo unitario ($/m) del catálogo según diámetro y clase de presión.

### 6.3 Hidráulica — pérdida de carga (Hazen-Williams, por defecto)

$$h_{f,i} = 10{,}674 \cdot \frac{Q_i^{1{,}852}}{C^{1{,}852} \cdot D_i^{4{,}871}} \cdot L_i$$

con $C = 150$ para PVC. Alternativa: Darcy-Weisbach para contraste.

**Velocidad por tramo:** $v_i = \dfrac{4 Q_i}{\pi D_i^{2}}$; se exige $0{,}5 \le v_i \le 2{,}0$ m/s.

**Carga dinámica total (CDT):** suma de la cota geométrica entre fuente y nodo crítico, pérdidas por fricción, pérdidas locales (≈ 10 % de fricción) y presión mínima requerida en emisor.

### 6.4 OPEX anual — energía de bombeo

$$P_{\text{bomba}} = \frac{\gamma \cdot Q \cdot H}{\eta_b \cdot \eta_m}\quad[\text{kW}]$$

$$E_{\text{anual}} = P_{\text{bomba}} \cdot h_{\text{riego/anuales}}\quad[\text{kWh}]$$

**Costo anual con AT-4.3:**
$$OPEX_t = c_E \cdot E_{\text{anual}} + c_{P,\max} \cdot 12 \cdot P_{\max} + c_{P,\text{punta}} \cdot n_{\text{meses punta}} \cdot P_{\text{punta}}$$

con $c_{P,\text{punta}}$ **parámetro configurable** (puede desactivarse en contratos planos).

### 6.5 Restricciones

1. $p_{\text{nodo crítico}} \ge p_{\text{emisor}}^{\text{nominal}}$.
2. $0{,}5 \le v_i \le 2{,}0$ m/s para todo $i$.
3. Regla telescópica: $D_i \ge D_{i+1}$ a lo largo de cualquier rama desde el cabezal.
4. $PN_i \ge p_{\max,i}$ (presión estática máxima en el tramo).
5. $D_i \in \mathcal{C}$ (discreto).

## 7. Algoritmo de optimización

**Enfoque:** búsqueda exhaustiva **discreta** con **podas** por telescopía y por restricciones hidráulicas.

**Esqueleto (pseudocódigo):**

```
función optimizar(red, catálogo, T, r):
    mejor_VAN ← ∞
    mejor_D   ← None
    orden ← ordenar_tramos_depth_first_desde_cabezal(red)
    para cada combinación D en producto_telescópico(catálogo, orden):
        si no cumple_PN(D, red): continuar
        v, h_f ← hidraulica(D, red)
        si no (0.5 ≤ v ≤ 2.0 en todo tramo): continuar
        p_crit ← presion_nodo_critico(red, h_f)
        si p_crit < p_emisor_nominal: continuar
        CDT ← calcular_CDT(red, h_f)
        OPEX_t ← costo_energia_AT43(CDT, Q, horas)
        VAN ← CAPEX(D) + Σ OPEX_t / (1+r)^t  para t=1..T
        si VAN < mejor_VAN:
            mejor_VAN, mejor_D ← VAN, D
    retornar mejor_D, mejor_VAN
```

**Podas clave** para hacer tractable la búsqueda exhaustiva:
- Enumerar solo combinaciones telescópicamente válidas (reduce el espacio en órdenes de magnitud).
- Cortar ramas en cuanto se viole velocidad máxima (los diámetros menores en ese tramo quedan descartados aguas abajo).
- Cota inferior por tramo: $D_i \ge D_{\min}(Q_i, v_{\max})$.

## 8. Datos de entrada

### 8.1 Desde QGIS (`red.gpkg`)

| Capa | Atributos requeridos |
|---|---|
| `nodos` | `id`, `tipo` (fuente/emisor/juntura), `x`, `y`, `cota_m`, `q_demanda_lps`, `p_min_bar` |
| `tramos` | `id`, `nodo_i`, `nodo_j`, `longitud_m`, `material` |

### 8.2 Catálogo Hoffens (`catalogo_hoffens.csv`)

Columnas: `dn_mm`, `di_mm` (diámetro interno), `pn` (6/8/10/12.5/16), `costo_clp_m`, `peso_kg_m` (opcional).

### 8.3 Tarifa AT-4.3 (`tarifas_at43.csv`)

Columnas: `cargo_energia_clp_kwh`, `cargo_potencia_max_clp_kw_mes`, `cargo_potencia_punta_clp_kw_mes`, `meses_punta`, `horario_punta_ini`, `horario_punta_fin`.

### 8.4 Operación

`horas_riego_dia`, `dias_temporada`, `eta_bomba`, `eta_motor`, `presion_emisor_bar`.

## 9. Salidas

- `red_opt_{1a,10a,20a}.inp` — archivo EPANET optimizado por horizonte.
- `resultados.csv` — tabla con: tramo, D_asignado, PN, v, h_f, costo_tramo.
- `resumen.csv` — por escenario: CAPEX, OPEX_anual, VAN, CDT, P_bomba.
- `comparacion.csv` — diseño convencional vs. optimizado (brecha %).
- `reporte.html` — figuras: isobaras, histogramas de velocidad, curva VAN(T).

## 10. Validación con EPANET (OE2)

1. Exportar la red óptima a `.inp` vía EPyT.
2. Ejecutar simulación hidráulica permanente.
3. Contrastar presiones y velocidades del motor interno vs. EPANET.
4. **Criterio de aceptación:** error relativo < 1 % en presiones y < 2 % en caudales/velocidades.

## 11. Estudio de caso (OE3)

1. Levantar la red existente del predio en QGIS.
2. Caracterizar la línea base: CAPEX real pagado, OPEX anual facturado.
3. Correr el optimizador para T = 1, 10 y 20 años.
4. Obtener tres diseños óptimos (uno por horizonte).
5. Análisis multitemporal de brecha:
   - **Corto plazo (1 año):** % ahorro CAPEX + OPEX_1 vs. convencional.
   - **Mediano plazo (10 años):** VAN convencional vs. VAN óptimo_10.
   - **Largo plazo (20 años):** VAN convencional vs. VAN óptimo_20.
6. Reportar reducciones absolutas y porcentuales y sensibilidad a tasa de descuento y a tarifa.

## 12. Plan de desarrollo (hitos y entregables)

Alineado con la Carta Gantt del AFE (2026–2027):

| Fase | Período | Entregable |
|---|---|---|
| F0. Setup repositorio, estructura de carpetas, entorno Python | Nov–Dic 2026 | Repo Git + `requirements.txt` funcionando |
| F1. Módulos hidráulicos y de costos (Act. 1.1) | Ene–Feb 2027 | `hidraulica.py`, `costos.py`, `tarifa.py` + tests unitarios |
| F2. Lector GIS automatizado (Act. 1.2) | Feb–Mar 2027 | `io_gis.py` leyendo `.gpkg` → grafo |
| F3. Algoritmo de optimización telescópica (Act. 1.3) | Mar–Abr 2027 | `optimizador.py` con podas |
| F4. Acoplamiento EPANET vía EPyT (Act. 2.1–2.2) | Abr–May 2027 | `epanet_bridge.py` + prueba de consistencia |
| F5. Estudio de caso: levantamiento del predio (Act. 3.1) | May 2027 | `red_predio.gpkg` |
| F6. Análisis de costos existentes (Act. 3.2) | May–Jun 2027 | `linea_base.csv` |
| F7. Comparación multitemporal (Act. 3.3) | Jun 2027 | `comparacion.csv` + figuras |
| F8. Informe de Avance 1 | **31-05-2027** | Documento escrito |
| F9. Informe de Avance 2 | **30-06-2027** | Documento escrito |
| F10. Entrega para calificación | **31-07-2027** | AFE final |

## 13. Riesgos y mitigaciones

| Riesgo | Mitigación |
|---|---|
| Explosión combinatoria en búsqueda exhaustiva | Podas por telescopía y por velocidad; precomputar $D_{\min}$ por tramo |
| Discrepancia entre motor propio y EPANET | Usar EPANET como oráculo; calibrar factor de pérdidas locales |
| Catálogo Hoffens incompleto o con precios desactualizados | Fuente única `data/catalogo_hoffens.csv` versionada; nota con fecha de cotización |
| Cambios en tarifa AT-4.3 | Parametrizar en CSV; análisis de sensibilidad |
| Topología no estrictamente telescópica en el predio real | Documentar excepciones y aplicar relajación controlada o separar subunidades |

## 14. Criterios de calidad del código

- PEP 8 + type hints.
- Cada módulo con docstring y al menos un test unitario.
- Semillas fijas donde aplique (aunque el algoritmo es determinista).
- Reproducibilidad: `requirements.txt` o `pyproject.toml` con versiones pinneadas.
- Datos de entrada bajo `data/` con `README` describiendo origen y fecha.

## 15. Próximos pasos inmediatos

1. Crear estructura de carpetas `src/`, `data/`, `tests/`, `outputs/`.
2. Inicializar repositorio Git y `requirements.txt` con: `epyt`, `geopandas`, `pandas`, `numpy`, `scipy`, `matplotlib`.
3. Cargar `catalogo_hoffens.csv` con datos reales (diámetros, PN, precios de cotización).
4. Prototipar `hidraulica.py` con Hazen-Williams y un test de un tramo contra EPANET.
5. Trazar una red de juguete (3–5 nodos) en QGIS y validar el pipeline completo end-to-end antes de cargar el predio real.

---

*Documento maestro v0.1 — base de trabajo. Actualizar al cierre de cada fase.*
