# 📏 Reglas de trabajo — APE3: Simulación M/M/1

> Guía rápida para todo el equipo. Léela antes de tocar el repositorio.
> La división de tareas por integrante se envía por separado.

---

## 1. Stack (dependencias AUTORIZADAS)

Solo estas 4 dependencias directas. **No instalar ninguna otra sin acordarlo con el equipo.**

| Dependencia | Uso |
|---|---|
| `simpy` | Simulación de eventos discretos (el motor del M/M/1) |
| `numpy` | Generación de variables aleatorias (`numpy.random.default_rng(seed)`) |
| `matplotlib` | Las 4 gráficas obligatorias + la animación de la cola |
| `streamlit` | Interfaz web local, en español |

- **Prohibido:** Plotly, pandas, scipy, kaleido, pytest u otras librerías.
- **Pruebas:** con `unittest` de la biblioteca estándar (sin instalar nada).
- `requirements.txt` es obligatorio y solo debe contener las dependencias reales del proyecto.

---

## 2. Estructura inicial (arquitectura MVC)

```text
simulacion_mm1/
├── app.py                          # Punto de entrada (solo conecta componentes)
├── model/
│   ├── __init__.py
│   ├── queue_model.py              # Simulación SimPy + registros de clientes
│   └── metrics.py                  # Métricas teóricas, simuladas y estadística
├── controller/
│   ├── __init__.py
│   └── simulation_controller.py    # Valida parámetros, coordina, prepara resultados
├── view/
│   ├── __init__.py
│   ├── dashboard.py                # Interfaz: paneles, tarjetas, tablas, textos
│   ├── graficas.py                 # Las 4 gráficas obligatorias con Matplotlib
│   └── animation.py                # Animación de la cola (FuncAnimation → GIF)
├── tests/
│   ├── test_metrics.py             # Pruebas de las fórmulas teóricas
│   └── test_simulation.py          # Pruebas del modelo SimPy
├── requirements.txt
├── .gitignore
└── README.md
```

### Regla de separación MVC (obligatoria)

- **model/** no importa Streamlit ni nada de la vista. Solo calcula y devuelve datos.
- **controller/** valida y coordina. **No dibuja** nada.
- **view/** solo presenta (controles, tablas, gráficas). No implementa fórmulas ni la simulación.
- **app.py** debe quedar corto (~10 líneas): iniciar la aplicación y conectar componentes.

---

## 3. Contratos de función (NO se cambian sin avisar al grupo)

Estas firmas las usan varias personas. Si necesitas modificar una, avisa **antes** en el grupo.

```python
# model/queue_model.py
def run_simulation(lambda_rate, mu_rate, num_customers, seed) -> dict:
    # Devuelve:
    # {
    #   "customers": [ {id, arrival, service_start, wait,
    #                   service_duration, departure, system_time}, ... ],
    #   "area_cola": float,            # área bajo la curva de tamaño de cola
    #   "tiempo_observacion": float,   # 1ª llegada → última salida
    #   "servidor_ocupado": float,     # tiempo total ocupado
    #   ... métricas simuladas básicas
    # }

# model/metrics.py
def metricas_teoricas(lambda_rate, mu_rate) -> dict:   # rho, L, Lq, W, Wq (valida λ < μ)
def metricas_simuladas(resultado) -> dict:             # Wq, Lq, W, L, atendidos, utilización
def tabla_comparativa(teorico, simulado, escenario) -> list[dict]

# controller/simulation_controller.py
def validar_parametros(lambda_rate, mu_rate, num_customers, seed) -> None  # errores en español
def ejecutar_escenario(lambda_rate, mu_rate, num_customers, seed) -> dict
def ejecutar_todos_los_escenarios(...) -> list[dict]
```

---

## 4. Reglas de la simulación (decisiones ya tomadas)

1. **Cantidad de clientes, no tiempo fijo:** `num_customers` define la corrida y **todos los clientes terminan de ser atendidos** antes de calcular métricas.
2. **Semilla obligatoria:** usar `np.random.default_rng(seed)` (nunca `np.random.seed()` global). La semilla visible en la interfaz se muestra siempre.
3. **Fórmulas teóricas exactas del prompt:**
   - `rho = lambda_rate / mu_rate` (es λ/μ, **no** μ/λ — la guía del docente tiene un error tipográfico)
   - `L = λ / (μ − λ)` · `Lq = λ² / (μ(μ − λ))` · `W = 1 / (μ − λ)` · `Wq = λ / (μ(μ − λ))`
   - Si `λ ≥ μ` o tasas ≤ 0: **no calcular métricas**, mostrar error en español.
4. **Lq simulado por integración temporal:** área acumulada bajo la curva de cola ÷ tiempo de observación (no promedio de observaciones instantáneas).
5. **Coherencia obligatoria:** espera ≥ 0 · `system_time = wait + service_duration` · el servidor nunca atiende a 2 clientes a la vez.
6. **Escenarios fijos (no se alteran):** `(2.0, 3.0)`, `(2.5, 3.0)`, `(2.9, 3.0)`.
7. **Sin datos ficticios:** todo resultado que se muestre sale de una ejecución real del modelo.

---

## 5. Reglas Git

1. Trabajamos en ramas: `feature/nombre_integrante` → se integra a `main`.
2. **Nadie hace `push` directo a `main`.**
3. Formato de commit: `A: descripción` (inicial de quien commitea).
4. Antes de empezar: `git pull` / `git fetch`.
5. **Nunca subir:** carpetas de entorno virtual (`.venv/`, `venv/`), `__pycache__/`, cachés, archivos del editor ni resultados generados (eso ya lo cubre `.gitignore`).
6. Sí se suben: código, tests, `README.md`, `requirements.txt`, `.gitignore`.
7. No hacer commits con nombres de otros integrantes ni mensajes inventados.

---

## 6. Cómo ejecutar (referencia rápida)

```bash
# 1. Entorno virtual
python -m venv .venv
source .venv/bin/activate          # Linux/macOS
# .venv\Scripts\activate           # Windows

# 2. Dependencias
pip install -r requirements.txt

# 3. Pruebas (deben pasar todas)
python -m unittest discover -s tests -v

# 4. Aplicación
streamlit run app.py
```

---

## 7. Criterios de calidad (al escribir código)

- Nombres de variables y funciones descriptivos; mensajes de error en español.
- Funciones pequeñas; comentarios solo donde explican una decisión o un cálculo.
- Sin TODO, `pass` ni implementaciones ficticias; sin excepciones ocultas.
- Sin dependencias, bases de datos, APIs ni patrones extra: **MVC y nada más**.
- Cada archivo debe poder explicarse solo: es material de defensa en clase.

---

## 8. Flujo de trabajo

1. Reunión breve: confirmar quién hace qué (la división viene por separado).
2. Cada quien implementa su archivo respetando los contratos de la sección 3.
3. Ejecuta las pruebas antes de commitear: `python -m unittest discover -s tests -v`.
4. Commitea en tu rama con tu prefijo (`A:`, `B:`, `C:`, `D:`).
5. Revisión cruzada: nadie integra su archivo sin que otro lo haya leído.
6. En la defensa, cada integrante debe poder explicar **todo** el modelo, no solo su parte.

---

*Documento vivo: si una regla cambia, se actualiza aquí y se avisa al grupo.*
