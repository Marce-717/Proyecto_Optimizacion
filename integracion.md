# EPANET con Python mediante EPyT: análisis técnico

## Resumen

EPyT (*EPANET Python Toolkit*) permite manejar desde Python modelos de redes de distribución creados para EPANET: abrir un archivo `.inp`, consultar o modificar datos de la red, ejecutar simulaciones y extraer resultados. La resolución hidráulica sigue a cargo del motor nativo de EPANET; EPyT ofrece una interfaz Python a ese motor. Por eso se integra bien con el análisis de datos, la automatización de escenarios y herramientas como NumPy, pandas y Matplotlib. [Repositorio y README de EPyT](https://github.com/KIOS-Research/EPyT), [artículo técnico de EPyT](https://doi.org/10.21105/joss.05947)

## Arquitectura

El flujo de trabajo puede entenderse en cuatro capas:

1. **Modelo de entrada:** el archivo `.inp` contiene la topología —nudos, tuberías, bombas y válvulas—, las demandas, los patrones horarios, las curvas, los controles y las opciones de simulación.
2. **EPyT:** la clase `epanet` abre ese modelo y ofrece métodos para consultar y modificar sus elementos.
3. **Motor EPANET nativo:** EPyT llama a las bibliotecas del Toolkit de EPANET para resolver las ecuaciones hidráulicas y de calidad.
4. **Resultados en Python:** las series de tiempo y propiedades se devuelven como arreglos/objetos que se pueden analizar o graficar.

El artículo describe el objeto EPyT como una representación del modelo y sus funciones, y muestra que las propiedades y resultados se actualizan cuando se modifica la red. [Artículo técnico](https://joss.theoj.org/papers/10.21105/joss.05947.pdf)

## Instalación y compatibilidad

El README del fork [KIOS-Research/EPyT](https://github.com/KIOS-Research/EPyT) indica Python 3.9 o superior, Windows/macOS/Linux y EPANET 2.2. Propone instalar el paquete publicado con:

```bash
python -m pip install epyt
```

La documentación del repositorio también muestra que se puede indicar una biblioteca nativa específica mediante `customlib`; esto puede ser útil cuando el sistema no encuentra automáticamente la biblioteca o se necesita controlar qué versión se carga.

**Punto de versión:** a la fecha de esta revisión, el README de KIOS sigue mencionando EPANET 2.2, pero los lanzamientos del repositorio de mantenimiento [OpenWaterAnalytics/EPyT](https://github.com/OpenWaterAnalytics/EPyT/releases) llegan a EPyT 2.3.5.2. Las notas de la serie 2.3 describen compatibilidad con nuevas funciones de EPANET 2.3, entre ellas análisis de demanda dependiente de presión (PDA), fugas y tipos adicionales de válvulas y curvas. No conviene asumir que el README y todas las versiones instaladas documentan la misma combinación. Registra las versiones de EPyT, EPANET y Python y confirma la biblioteca cargada, especialmente para funciones recientes.

## Carga y revisión del modelo

La forma básica de abrir una red es:

```python
from epyt import epanet

d = epanet(r"datos/Net1.inp")

print("Nudos:", d.getNodeCount())
print("Tuberías y otros enlaces:", d.getLinkCount())
print("Elevaciones:", d.getNodeElevations())
```

La ruta debe apuntar al archivo `.inp` que se desea analizar. Conviene revisar al inicio los conteos, unidades, elevaciones, diámetros, rugosidades, demandas y configuración de tiempo. EPyT expone funciones para consultar esos datos y otras para modificarlos, como `getLinkDiameter`, `setLinkDiameter`, `getNodeBaseDemands` y `setNodeBaseDemands`. El índice de los elementos y su ID textual no son lo mismo; conserva una relación entre ambos al interpretar resultados. [README y funciones del repositorio](https://github.com/KIOS-Research/EPyT)

## Ejecución y extracción de resultados

Para un análisis hidráulico completo, EPyT ofrece `getComputedHydraulicTimeSeries()`. Entre los resultados documentados están tiempo, presión, demanda, carga hidráulica, volumen de tanque, caudal, velocidad, pérdida de carga, estado y energía. Las propiedades se consultan como campos del objeto devuelto.

```python
from epyt import epanet

d = epanet(r"datos/Net1.inp")

H = d.getComputedHydraulicTimeSeries(
    ["Time", "Pressure", "Head", "Flow", "Velocity"]
)

horas = H.Time / 3600
presion = H.Pressure
caudal = H.Flow

print("Tiempos (h):", horas)
print("Presión por nudo:", presion.shape)
print("Caudal por enlace:", caudal.shape)

d.unload()
```

La API también permite avanzar paso a paso, útil si necesitas observar o intervenir en estados intermedios:

```python
d.openHydraulicAnalysis()
d.initializeHydraulicAnalysis()

tstep = 1
while tstep > 0:
    t = d.runHydraulicAnalysis()
    presiones = d.getNodePressure()
    caudales = d.getLinkFlows()
    tstep = d.nextHydraulicAnalysisStep()

d.closeHydraulicAnalysis()
d.unload()
```

Para un análisis de calidad del agua, se usa `getComputedQualityTimeSeries()`. **Los intervalos de salida hidráulica y de calidad pueden ser distintos**; utiliza la serie de tiempo de cada resultado para alinear y comparar datos. La documentación muestra resultados de calidad en nudos y enlaces, además de herramientas de graficación. [Ejemplos y API de EPyT](https://wateranalytics.org/EPyT/examples.html), [artículo técnico](https://joss.theoj.org/papers/10.21105/joss.05947.pdf)

## Aplicaciones hidráulicas

EPyT sirve como base para automatizar tareas de análisis, por ejemplo:

- **Escenarios de demanda:** modificar consumos o patrones y comparar presiones y caudales.
- **Evaluación de operación:** estudiar estados de bombas, válvulas, tanques y controles a lo largo del período simulado.
- **Detección de problemas operativos:** localizar presiones bajas, velocidades fuera del rango objetivo o enlaces con flujo nulo o invertido.
- **Calidad del agua:** analizar concentración, edad o propagación de sustancias según el modelo y las funciones empleadas.
- **Calibración y sensibilidad:** ejecutar escenarios que varían rugosidades, demandas o parámetros de bombas y comparar el efecto en observaciones.
- **Integración con datos:** procesar salidas junto con mediciones de presión, caudal o consumo mediante el ecosistema científico de Python.

La documentación también destaca funciones para cambios de red, controles, patrones, matrices de conectividad y gráficas. Las capacidades exactas dependen de la versión de EPyT y de la biblioteca EPANET asociada.

## Límites y recomendaciones

- EPyT no valida por sí solo que los datos de entrada representen correctamente el sistema real. Una simulación convergente puede seguir usando demandas, cotas, rugosidades o controles incorrectos.
- Verifica las **unidades** de caudal, presión, longitud y diámetros en el `.inp` y en los resultados. Las unidades pueden afectar la interpretación y conversiones.
- Comprueba estados de bombas y válvulas, curvas, niveles iniciales y controles en distintos tiempos, no solo en el estado inicial.
- En redes con presión insuficiente, el modelo tradicional de demanda fija puede no reflejar la demanda realmente entregada. Considera PDA si tu versión del motor y EPyT lo admiten y tienes parámetros defendibles para configurarlo.
- Distingue entre resultados de un análisis de período único y una simulación extendida; patrones horarios, almacenamiento y controles importan en esta última.
- Conserva una copia del `.inp` original antes de modificar parámetros y documenta cada escenario.
- Cierra el objeto con `unload()` cuando termines, en particular al procesar muchos archivos o escenarios.
- Si los resultados son inesperados, revisa primero mensajes de error, opciones de simulación y unidades; luego contrasta un caso pequeño con EPANET y confirma la versión de la biblioteca cargada. El README del proyecto advierte que también pueden existir problemas en las bibliotecas EPANET subyacentes.

**Conclusión:** EPyT conecta Python con el Toolkit nativo de EPANET mediante una interfaz orientada a objetos. Para empezar basta instalar el paquete, abrir un `.inp` y llamar a los métodos de consulta y simulación. El principal cuidado técnico es controlar las versiones y revisar el modelo y sus unidades antes de confiar en las conclusiones hidráulicas.
