# PaddockGuard
### Safety Management System for the Safety Leadership Award

![Badge](https://img.shields.io/badge/Python-3.11%2B-blue?style=flat-square)
![Badge](https://img.shields.io/badge/SQLite-3-green?style=flat-square)
![Badge](https://img.shields.io/badge/YOLO11-GPU%20Ready-orange?style=flat-square)
![Badge](https://img.shields.io/badge/Status-Active%20Development-brightgreen?style=flat-square)

---

## 📋 Descripción General

**PaddockGuard** es una plataforma de gestión integral de seguridad para el paddock de Shell Eco-Marathon 2026. Integra tres subsistemas en una sola base de datos para maximizar el cumplimiento de normas HSSE (Sección 2 del reglamento) y generar evidencia documentada que demuestre liderazgo en seguridad.

La propuesta busca ganar el **Safety Leadership Award (Art. 242)** mediante:
- **Trazabilidad QR** de préstamo/devolución de herramientas y EPP
- **Checklist digital** de uso de EPP con validación cruzada (firma de segundo miembro)
- **Visión por computadora** que verifica orden del panel de herramientas en tiempo real
- **Reportes automáticos** que evidencian cumplimiento contra los cuatro subcriterios del premio

---

## 🎯 Enfoque: Sección 2 (Safety) del Reglamento

| Criterio del Premio | Cómo PaddockGuard lo evidencia |
|---|---|
| **i. Paddock condition, planning and management** | Reportes de préstamos inter-equipos + CV verifica orden del panel |
| **ii. Safety leadership** | Historial de checklist EPP con firma cruzada (role modelling) |
| **iii. Compliance** | Query diaria de cumplimiento de guantes/lentes por persona/día |
| **iv. Interacción proactiva con Safety Team** | Log de incidentes con acciones resueltas + reportes ofrecidos diariamente |

---

## 🏗️ Arquitectura

```
Entrada de Datos            Base Central SQLite              Salida para Jurado
──────────────────────────────────────────────────────────────────────────
    QR escanea        ┌─────────────────────┐
    (celular)    ────►│                     │
                      │    safety.db        │
    Checklist    ────►│  (trazabilidad      │◄────────────┐
    (formulario)      │   integrada)        │             │
                      │                     │     Reportes (pandas)
    CV snapshot  ────►│                     │  
    (cámara fija)     │                     │             │
                      └─────────────────────┘         Envía al
                                                      Safety Team
                                                      (diariamente)
```

### Modelo de datos central

```sql
items ──────┬─────► prestamos (QR trazable)
            ├─────► checklist_epp (validación con firma)
            ├─────► estado_panel (CV verifica orden)
            └─────► incidentes (feedback resuelto)
personal ───┘
```

---

## 🚀 Quick Start

### Requisitos
- Python 3.11+ (fijado para evitar incompatibilidad PyTorch-CUDA con 3.14+)
- SQLite3 (incluido en Python)
- GPU recomendada para entrenamiento CV (Colab gratis)

### Instalación

```bash
# Clonar repositorio
git clone https://github.com/tu-equipo/paddock-guard.git
cd paddock-guard

# Crear virtual environment
py -3.11 -m venv .venv_safety
.venv_safety\Scripts\activate

# Instalar dependencias
pip install -r requirements.txt

# Inicializar base de datos
python src/init_db.py

# Generar QR (un QR por herramienta)
python src/qr_generar.py
```

Archivos generados:
- `safety.db` — base de datos principal
- `qr_codes/` — PNGs de códigos QR listos para imprimir

---

## 📦 Módulos principales

### 1️⃣ **QR Trazable** (`src/qr_*.py`)

Registra préstamo/devolución de herramientas y EPP entre equipos.

```python
from src.qr_prestamo import procesar_scan
# Al escanear: ITEM:5 → registra préstamo o devolución automáticamente
procesar_scan("ITEM:5", equipo="Equipo Alfa", responsable="Juan")
```

**Resultado en BD:** columna `equipo_solicitante` permite filtrar por equipo externo → reporte "ayudamos a X equipos con Y artículos".

### 2️⃣ **Checklist EPP + Firma Cruzada** (`src/checklist.py`)

Validación obligatoria: guantes + lentes con segundo miembro como validador.

```python
from src.checklist import registrar_checklist
registrar_checklist(
    persona_id=1,
    guantes=True, lentes=True, zapato=True,
    zona='paddock', validado_por='María'
)
```

**Regla de negocio:** sin `validado_por`, el INSERT falla. Esto eleva la autodeclaración a evidencia auditable.

### 3️⃣ **CV: Verificador de Orden de Panel** (`src/cv_orden.py`)

Visión automática que verifica que las herramientas más usadas estén en su slot en el shadow board.

```python
from src.cv_orden import snapshot_panel
import cv2

cap = cv2.VideoCapture(0)  # cámara fija apuntando al panel
ret, frame = cap.read()
snapshot_panel(frame)  # ejecuta YOLO11 y registra resultado en estado_panel
```

**Innovación clave:** cruza datos — si QR dice "devuelto" pero CV no lo ve, genera `discrepancia=1` → flag para investigar.

### 4️⃣ **Reportes** (`src/reportes.py`)

Genera cuatro reportes diarios que mapean directo a los subcriterios Art. 242.

```python
from src.reportes import generar_reporte_diario
generar_reporte_diario()  # imprime tablas y guarda CSV
```

**Salida típica:**

```
PRÉSTAMOS POR EQUIPO (Art. 242.c.i):
Equipo Alfa:     12 préstamos, 2 pendientes
Equipo Beta:     8 préstamos,  0 pendientes

CUMPLIMIENTO EPP POR DÍA (Art. 242.c.iii):
2026-09-15:  8 con guantes, 8 con lentes, 8 registros
2026-09-16:  9 con guantes, 9 con lentes, 9 registros

DISCREPANCIAS CV (Innovación):
2026-09-15:  1 discrepancia (llave destornillador no en panel pero QR dice devuelta)

INCIDENTES RESUELTOS (Art. 242.c.iii directo):
[Observación] Cables sueltos en zona herramientas → Acción: asegurados con velcro
```

---

## 📊 Entrenamiento CV (Colab GPU)

Para la capa de visión (opcional pero recomendada):

1. Sube `notebooks/entrenar_yolo.ipynb` a Google Colab
2. Dataset: ~150–200 fotos del panel con/sin herramientas
3. Clases: 4–6 herramientas principales (llave, destornillador, multímetro, etc.)
4. Descarga peso entrenado a `models/best.pt`

```python
# Colab, GPU
from ultralytics import YOLO
model = YOLO('yolo11n.pt')
model.train(data='herramientas.yaml', epochs=100, imgsz=640, device=0)
```

> **Nota:** Dataset privado en rama `training/` (no en main). Aquí solo documentación de flujo.

---

## 📚 Aprender SQLite (plan enfocado)

No dominas SQL entero — solo estas piezas:

1. **Fundamentos (2–3h)** — `CREATE TABLE`, `INSERT`, `SELECT WHERE`
2. **Consultas útiles (2–3h)** — `GROUP BY`, `COUNT`, `CASE WHEN`
3. **Relaciones (3–4h)** ← **LO MÁS IMPORTANTE** — `JOIN`, `FOREIGN KEY`
4. **Modificación (2h)** — `UPDATE`, `NULL`, estado de préstamo
5. **SQLite + Python (2–3h)** — módulo `sqlite3`, consultas parametrizadas
6. **Buenas prácticas (1h)** — índices, transacciones, backup

**Recursos:**
- [SQLite Tutorial](https://sqlitetutorial.net)
- [DB Browser for SQLite](https://sqlitebrowser.org) — herramienta gráfica (INSTALA ESTO PRIMERO)
- [SQLBolt](https://sqlbolt.com) — ejercicios interactivos
- [Python sqlite3 docs](https://docs.python.org/3/library/sqlite3.html)

---

## 🎬 Flujo operativo (día a día en el paddock)

```
MAÑANA:
 ├─ 08:00 — Init: python src/init_db.py (si es primer día)
 ├─ 08:15 — Briefing: todos firman checklist_epp
 └─ 09:00 — CV: snapshot inicial del panel de herramientas

DURANTE EVENTO:
 ├─ QR: cada préstamo/devolución escaneado
 ├─ Checklist: entrada/salida de zona de herramientas (guantes + lentes + validador)
 └─ CV: snapshot cada 2h para detectar cambios

CIERRE DE TURNO:
 ├─ Reconciliación: QR vs CV (¿hay discrepancias?)
 ├─ Generador reportes
 └─ Ofrecimiento proactivo al Safety Team (Art. 242.d)
```

---

## 📁 Estructura del repositorio

```
paddock-guard/
├── README.md                      # Este archivo
├── requirements.txt               # Dependencias
├── safety.db                      # Base (generada al instalar)
├── db/
│   └── schema.sql                 # DDL completo
├── src/
│   ├── init_db.py                 # Crea tablas
│   ├── qr_generar.py              # Genera PNGs de QR
│   ├── qr_prestamo.py             # Procesa escaneo
│   ├── checklist.py               # Registra EPP con firma
│   ├── cv_orden.py                # YOLO11 + escritura SQLite
│   └── reportes.py                # Genera reportes para jurado
├── models/
│   └── best.pt                    # Peso YOLO (descargado de Colab)
├── notebooks/
│   └── entrenar_yolo.ipynb        # Colab GPU (rama training/)
├── qr_codes/                      # Códigos QR (generados)
└── tests/
    └── test_db.py                 # Validación básica
```

---

## ⚙️ Configuración

### Variables de entorno (opcional)

Crea `.env` si necesitas configs dinámicas:

```env
DB_PATH=safety.db
CV_CAMERA=0  # índice de cámara (0 = por defecto)
CV_MODEL=models/best.pt
REPORT_DAILY_HOUR=16  # generar reporte a las 4 PM
```

### Personalización de items

Edita `db/schema.sql` e `INSERT INTO items`:

```sql
INSERT INTO items (nombre, categoria, subtipo, cantidad_total, en_panel_cv, slot_panel)
VALUES ('Llave Torque', 'herramienta', 'destornillador', 1, 1, 'A1');
```

---

## 🔍 Validación / Testing

```bash
# Test básico: verifica que la BD se crea sin errores
python tests/test_db.py

# Simula un ciclo completo de préstamo/devolución
python tests/test_qr_flow.py

# Verifica que CV genera detecciones (requiere imagen de prueba)
python tests/test_cv_inference.py
```

---

## 📈 Resultados esperados para el jurado

| Métrica | Rango | Esfuerzo |
|---|---|---|
| Préstamos documentados (Equipo X a Y) | 10–30 por día | Bajo |
| Cumplimiento EPP (%) | 85–100% | Bajo |
| Discrepancias detectadas por CV | 1–5 por día | Medio |
| Incidentes resueltos | 100% en <24h | Bajo |
| **Veredicto jurado (Art. 242)** | ✅ Oro / Plata / Bronce | **Alto impacto** |

---

## 🛠️ Stack técnico

| Componente | Tecnología | Por qué |
|---|---|---|
| Base de datos | SQLite | Portátil (1 archivo), relacional, sin servidor |
| Lenguaje | Python 3.11+ | Mismo que tu SCARA, evita bloqueo CUDA |
| Visión (opcional) | YOLO11n | Ligero, rápido en GPU Colab, detección acotada |
| Entrenam. CV | Colab (GPU) | Gratis, sin config local |
| Reportes | Pandas | Query SQL → tabla limpia en segundos |
| Escaneo | QRCode (pyzbar) | Celular + cualquier app de QR estándar |
| Checklist | Google Forms + CSV | Sincronizable a SQLite, cero fricción |

---

## 💡 Notas de implementación

### Sin CV (opción A — Fases 1–3, 100% confiable)
QR + Checklist ya son suficientes para demostrar **paddock management** y **safety leadership**. Implementa esto primero.

### Con CV (opción B — Fase 4, innovación verificable)
Agrega visión para diferenciarte. **Pero corre snapshots bajo demanda, no stream continuo** — más robusto ante iluminación variable del paddock.

### Recomendación
Implementa la ruta **A** antes del evento (1–2 semanas). Si sobra tiempo + GPU disponible, entrena CV en Colab y agrega **B** en los últimos días.

---

## 📞 Contacto / Contribuciones

**Responsables del proyecto:**
- Team Manager: [-]
- Desarrollo DB: [Paulina R.]
- Desarrollo CV: [Paulina R.]


## 🎯 Roadmap

- [x] Diseño de arquitectura
- [ ] Esquema SQLite
- [ ] Módulo QR (semana 1)
- [ ] Módulo Checklist (semana 1)
- [ ] Entrenamiento CV en Colab (semana 2)
- [ ] Módulo CV + integración (semana 2)
- [ ] Reportes automáticos (semana 2)
- [ ] Testing completo (semana 3)
- [ ] Deploy en paddock (día del evento)

---

**Last updated:** 2026-07-04 | **Status:** Ready for implementation
