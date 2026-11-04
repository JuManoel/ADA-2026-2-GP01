# Proyecto Oráculo · ADAG1 · Grupo G01

Análisis y Diseño de Algoritmos · 2026-B · Universidad de Caldas

**Integrantes:** Andrés Santiago Restrepo Duque · Keyla Daniela Cartagena Díaz
**Subproyecto:** ADAG1 · Proyecto Oráculo
**Versión entregada:** rama `main`, primera entrega (solución base)

## De qué trata

Buscamos una configuración de prompt que haga que un modelo de lenguaje cumpla restricciones verificables. Una configuración tiene un índice por ranura (`rol`, `estrategia`, `formato`, `estilo`, `cierre`, con 8 opciones cada una) y una temperatura. Con la temperatura fija en 0.0 el espacio tiene 8⁵ = 32 768 configuraciones. La precisión de una configuración es la fracción de instancias que cumplen **todas** sus restricciones.

La solución base es un **ascenso por coordenadas con reinicios y poda guiada por trazas de fallos**: se cambia una ranura a la vez, y las opciones que, según las trazas, rompen instancias que otra opción resuelve se descartan de las pasadas siguientes. El resultado es la mejor configuración observada, no un óptimo demostrado.

## Contenido

| Ruta | Qué es |
|---|---|
| `notebooks/01_busqueda.ipynb` | Notebook de búsqueda: método, registro de configuraciones y precisiones, análisis de trazas, poda y validación con `oraculo.validar`. Genera `entrega.json`. |
| `notebooks/02_evaluacion.ipynb` | Notebook de evaluación: carga `entrega.json` y la evalúa sobre `datos_prueba.json` sin editar código. |
| `entrega.json` | Configuración entregada (grupo, semana, índices y temperatura). Se genera al ejecutar el notebook de búsqueda. |
| `resultados/` | Evidencia de la ejecución: `evaluaciones.csv`, `registro_busqueda.json`, `poda.json`, `curva_busqueda.png`, `cache_oraculo.json.gz`. |
| `docs/manual_tecnico.md` | Manual técnico de la solución base. |
| `oraculo.py`, `ayudas.py`, `datos_visibles.json` | Archivos del curso, sin modificar ([repo original](https://github.com/DanielMelo404/Proyecto-Oraculo-AyD)). |
| `proyecto_oraculo.ipynb` | Notebook original del curso, como referencia. |

## Requisitos

- Google Colab con GPU **T4** (el modelo `Qwen/Qwen3-1.7B` necesita CUDA).
- Python 3.10 o superior (el de Colab).
- Dependencias en `requirements.txt`; los notebooks las instalan solos. Además clonan [allenai/open-instruct](https://github.com/allenai/open-instruct), que trae los verificadores de restricciones.
- El repositorio debe ser **público** para que Colab lo pueda clonar.

## Cómo ejecutar

### Búsqueda (reproduce `entrega.json`)

1. Abrir `notebooks/01_busqueda.ipynb` en Colab (*Archivo ▸ Abrir notebook ▸ GitHub* y pegar la URL de este repositorio).
2. Elegir la GPU T4, ejecutar la sección 1 y reiniciar la sesión.
3. Ejecutar el resto en orden. La caché se guarda en Google Drive (`MyDrive/ADA_Oraculo_G01/`), así que una desconexión no pierde lo generado.
4. Al final se descarga `resultados_busqueda.zip` con `entrega.json` y la carpeta `resultados/`.

Los parámetros (semilla, pasadas, reinicios, umbral de poda) están en la sección 2. Con la misma semilla y temperatura 0.0 los resultados son los mismos.

### Evaluación

1. Abrir `notebooks/02_evaluacion.ipynb` en Colab con GPU T4.
2. Opcional: subir el archivo de prueba como `/content/datos_prueba.json`. Si no existe, se evalúa sobre la partición de validación.
3. Ejecutar la sección 1, reiniciar y ejecutar el resto. No hay que editar nada.

### Ejemplo pequeño (prueba de corrección)

Para comprobar que todo funciona sin gastar mucho tiempo de GPU, se puede evaluar una sola configuración sobre 5 instancias después de la sección 2 del notebook de búsqueda:

```python
r = oraculo.evaluar({"rol": 0, "estrategia": 0, "formato": 0, "estilo": 0, "cierre": 0,
                     "temperatura": 0.0}, busqueda[:5], semilla=1)
print(r)   # Resultado(precision=..., fallos=.../5)
```

La salida respeta las restricciones del problema si la precisión está entre 0 y 1, `r.n == 5` y cada traza tiene `id`, `violo` y `salida`.

## Formato de `entrega.json`

```json
{
 "grupo": "G01",
 "config": {"rol": 0, "estrategia": 0, "formato": 0, "estilo": 0, "cierre": 0, "temperatura": 0.0},
 "semana": 5
}
```

Cada índice va de 0 a 7 y `temperatura` es 0.0, 0.3 o 0.7. Los valores de arriba son solo un ejemplo del formato: el archivo real lo genera el notebook de búsqueda.
