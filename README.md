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
Entrada de Datos            Base Central SQLite              Salida a Capis
──────────────────────────────────────────────────────────────────────────
    QR escanea        ┌─────────────────────┐
    (celular)    ────►│                     │
                      │    safety.db        │
    Checklist    ────►│  (trazabilidad      │◄────────────┐
    (formulario)      │   integrada)        │             │
                      │                     │     Reportes (pandas)
    CV snapshot  ────►│                     │  
    (cámara fija)     │                     │             │
                      └─────────────────────┘         Genera un reporte a 
                                                      los capis 
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
- Python 3.11+ 
- SQLite3 (incluido en Python)
- Colab GPU para entrenamiento CV

**Recursos:**
- [SQLite Tutorial](https://sqlitetutorial.net)
- [DB Browser for SQLite](https://sqlitebrowser.org) — herramienta gráfica (INSTALA ESTO PRIMERO)
- [SQLBolt](https://sqlbolt.com) — ejercicios interactivos
- [Python sqlite3 docs](https://docs.python.org/3/library/sqlite3.html)

---

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

## 🎬 Flujo operativo (día a día en el paddock)

```
MAÑANA:
 ├─ 08:00 — Init: python src/init_db.py (si es primer día)
 ├─ 08:15 — Briefing: todos firman checklist_epp
 └─ 09:00 — CV: snapshot inicial del panel de herramientas

DURANTE EVENTO:
 ├─ QR: cada préstamo/devolución escaneado
 ├─ Checklist: entrada/salida de zona de herramientas (guantes + lentes + validador)
  CV: snapshot cada 2h para detectar cambios

CIERRE DE TURNO:
 ├─ Reconciliación: QR vs CV (¿hay discrepancias?)
 └─ Generador reportes
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

## 📞 Contribuciones

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

**Last updated:** 2026-07-05
