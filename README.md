# Proyecto: Telecomunicaciones — Identificar operadores ineficaces (CallMeMaybe)

## Objetivo
CallMeMaybe (telefonía virtual) desea una función para supervisión que identifique **operadores menos eficaces**.  
Un operador se considera ineficaz si:
1) Tiene **muchas llamadas entrantes perdidas** (internas y externas)  
2) Presenta **tiempos de espera altos** en llamadas entrantes  
3) Si se espera que haga llamadas salientes, un **volumen bajo de salientes** puede indicar ineficacia

Este proyecto realiza:
- Análisis Exploratorio de Datos (EDA)
- Identificación de operadores ineficaces (métricas + criterios)
- Pruebas de hipótesis estadísticas (validación)
- Presentación de conclusiones en PDF

---

## Dataset
Se utilizan dos archivos CSV:

### `telecom_dataset_us.csv` (llamadas)
Columnas principales:
- `user_id`: ID de cuenta cliente
- `date`: fecha/hora del registro
- `direction`: `in` (entrante) / `out` (saliente)
- `internal`: llamada interna (entre operadores) o externa
- `operator_id`: ID operador
- `is_missed_call`: llamada perdida (True/False)
- `calls_count`: número de llamadas en el registro
- `call_duration`: duración sin espera
- `total_call_duration`: duración con espera

### `telecom_clients_us.csv` (clientes)
- `user_id`
- `tariff_plan`
- `date_start`

> Nota: El dataset de clientes se mantiene como contexto. El análisis principal se centra en desempeño por operador (llamadas).

---

## Tecnologías y herramientas
- **Python**
- **Jupyter Notebook** (en VS Code)
- **pandas / numpy** (procesamiento y agregaciones)
- **matplotlib** (gráficos)
- **scipy** (pruebas de hipótesis)

---

## Metodología (resumen)

1) **Carga y revisión inicial**
- Se cargaron `calls` y `clients`.
- Se revisó estructura (`shape`, `columns`, `info`) y nulos:
  - `operator_id`: 8,172 nulos (~15.2%)
  - `internal`: 117 nulos (~0.22%)
- Decisión: no eliminar el dataset original. Se creó `calls_ops = calls[calls["operator_id"].notna()]` para analizar desempeño por operador sin perder contexto.

2) **EDA (Análisis Exploratorio)**
- Visualizaciones:
  - Histograma de duración promedio por llamada
  - Gráfico circular (internas vs externas)
  - Conteo por dirección (`in` vs `out`) y análisis por separado
- Hallazgos: predominan llamadas externas (~88%) y la duración está sesgada a la derecha (muchas cortas, pocas muy largas).

3) **Métricas por operador**
- Se construyó `operators_kpi` (1 fila por operador) con:
  - Entrantes: `incoming_calls`, `incoming_missed_calls`, `missed_rate_in`
  - Espera entrante: `avg_wait_time_in = (total_call_duration/calls_count) - (call_duration/calls_count)` (recortado a mínimo 0)
  - Salientes: `outgoing_calls`

4) **Criterio de “ineficaz” (percentiles)**
- Reglas:
  - `missed_rate_in` ≥ p90 (muchas perdidas)
  - `avg_wait_time_in` ≥ p90 (espera alta)
  - `outgoing_calls` ≤ p10 (pocas salientes)
- Se marca `ineffective=True` si cumple **al menos una** condición.
- Resultado: 754 operadores analizados; 311 ineficaces; 443 no ineficaces.

5) **Pruebas de hipótesis (validación)**
- Comparación de grupos (`ineffective=True` vs `False`) con **Mann–Whitney U**:
  - H1 (perdidas): p≈0.000248; promedio `missed_rate_in` ineficaces≈0.036 vs no ineficaces≈0.0037 → se rechaza H0
  - H2 (espera): p≈3.48e-22; promedio espera ineficaces≈23.0s vs no ineficaces≈13.6s → se rechaza H0

## Conclusiones (resumen ejecutivo)
- Se definieron métricas operativas por operador (entrantes y salientes) y se identificaron **311** operadores con señales de ineficacia.
- Las pruebas estadísticas confirman que el grupo ineficaz presenta **mayor tasa de llamadas perdidas** (significativo) y **mayor tiempo de espera entrante** (altamente significativo).


