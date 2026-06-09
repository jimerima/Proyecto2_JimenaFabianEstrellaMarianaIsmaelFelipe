# Proyecto 2 — Inteligencia de Negocios: TEC Digital

**Curso:** TI6900 - Inteligencia de Negocios  
**Semestre:** I Semestre 2026  
**Docente:** Sánchez Soto Michael

---

## Descripción del problema

El área de soporte del TEC Digital gestiona diariamente tickets relacionados con incidencias técnicas, solicitudes administrativas, problemas de acceso y consultas académicas provenientes de estudiantes, docentes y personal administrativo. Aunque el sistema registra cada caso en Zendesk, no existía una solución que permitiera **consolidar y analizar históricamente** el comportamiento del servicio.

Esta carencia impedía:
- Medir tiempos de resolución y cumplimiento de SLA.
- Identificar servicios con mayor carga de incidentes.
- Evaluar la distribución de trabajo entre agentes.
- Anticipar picos de demanda en períodos académicos críticos (matrícula, exámenes, etc.).

---

## Organización seleccionada

**TEC Digital** — Unidad del Tecnológico de Costa Rica encargada de administrar y brindar soporte a las plataformas digitales institucionales (gestión de cursos, matrículas, evaluaciones, comunicación académica, entre otros). Utiliza **Zendesk** como sistema de ticketing.

---

## Arquitectura de la solución

La solución implementa una arquitectura analítica de tres capas:

```
[Fuente operacional]          [Capa de transformación]       [Capa analítica]
CSV exportado de Zendesk  →   EasyMorph (proceso ETL)   →   Power BI Dashboard
        ↓                              ↓
   MySQL (staging)            MySQL (Data Warehouse)
                               Esquema estrella
```

**Flujo detallado:**

1. **Extracción:** Archivo CSV histórico (1 337 registros desde diciembre 2025) importado a MySQL como almacenamiento temporal.
2. **Transformación (EasyMorph):**
   - Conversión de fechas seriales de Excel a formato estándar `YYYY-MM-DD`.
   - Tratamiento de valores nulos → categoría `No especificado`.
   - Corrección de caracteres UTF-8 rotos en nombres.
   - Generación de atributos derivados: `categoria_servicio`, `tipo_canal`, `tipo_organizacion`, `nivel_prioridad`, `semestre_academico`, `es_periodo_matricula`, `es_fin_semestre`.
   - Cálculo de métricas: `tiempo_resolucion_horas` y `cumple_sla` (umbral ≤ 4 horas).
3. **Carga:** Tablas dimensionales y tabla de hechos cargadas en MySQL (Data Warehouse definitivo).
4. **Visualización:** Power BI conectado directamente al DW; dashboard con 5 pantallas analíticas.

---

## Integrantes del equipo

| Nombre | Carné|
|---|---|
| Rivera Madrigal Jimena | 2023066336|
| Herrera Bermúdez Mariana | 2023800120|
| Martínez Camacho Fabián | 2023154879|
| Corrales Vargas Felipe | 2020035096|
| Argüello Morales Estrella | 2023161710|
| Soto Valdivia Ismael | 2019043735|
---

## Herramientas utilizadas

| Herramienta | Versión / Descripción | Uso en el proyecto |
|---|---|---|
| **EasyMorph** | Desktop | Motor de transformación ETL |
| **MySQL** | 8.x | Almacenamiento staging y Data Warehouse |
| **Power BI Desktop** | Más reciente disponible | Dashboards y visualizaciones analíticas |
| **Zendesk** | SaaS | Sistema de ticketing fuente (solo lectura) |
| **GitHub** | — | Control de versiones y colaboración |
| **Microsoft Teams** | — | Comunicación del equipo |
| **Zoom** | — | Reuniones virtuales |

---

## Instrucciones de ejecución

### Prerrequisitos

- MySQL 8.x instalado y en ejecución.
- EasyMorph Desktop instalado.
- Power BI Desktop instalado.
- Acceso al repositorio y a los archivos del directorio `Datos/`.

### Paso 1 — Configurar la base de datos

```sql
-- Crear la base de datos
CREATE DATABASE tec_digital;
USE tec_digital;

-- Importar el archivo CSV fuente como tabla staging
-- (usar el asistente de importación de MySQL Workbench o el comando LOAD DATA)
LOAD DATA INFILE 'Datos/HistoricoTickets_TecDigital.csv'
INTO TABLE tickets_raw
FIELDS TERMINATED BY ';'
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;
```

### Paso 2 — Ejecutar el proceso ETL en EasyMorph

1. Abrir EasyMorph Desktop.
2. Cargar el proyecto ETL ubicado en `Datos/ETL/`.
3. Configurar la conexión a MySQL apuntando a la base de datos `tec_digital`.
4. Ejecutar el flujo completo. EasyMorph generará automáticamente las siguientes tablas limpias en `Datos/Tablas_Limpias/`:
   - `dim_tiempo.csv`
   - `dim_agente.csv`
   - `dim_servicio.csv`
   - `dim_canal.csv`
   - `dim_tipo.csv`
   - `dim_prioridad.csv`
   - `dim_usuario.csv`
   - `dim_estado.csv`
   - `fact_ticket.csv`
5. Las mismas tablas se cargan automáticamente en MySQL como el Data Warehouse definitivo.

### Paso 3 — Abrir el Dashboard en Power BI

1. Abrir el archivo `Dashboard_Proyecto2_BI.pbix` con Power BI Desktop.
2. Si se solicita actualizar la conexión de datos, apuntar al servidor MySQL local con la base de datos `tec_digital`.
3. Actualizar el modelo (`Inicio → Actualizar`).
4. Navegar entre las 5 páginas analíticas del reporte.

---

## Estructura del repositorio

```
Proyecto2_JimenaFabianEstrellaMarianaIsmaelFelipe/
│
├── Datos/ # Dimensiones y tabla de hechos procesadas
│   ├── Dimensiones/
│       ├── dim_tiempo.csv
│       ├── dim_agente.csv
│       ├── dim_servicio.csv
│       ├── dim_canal.csv
│       ├── dim_tipo.csv
│       ├── dim_prioridad.csv
│       ├── dim_usuario.csv
│       ├── dim_estado.csv
│    ├── Hechos/
│        └── fact_table.csv      
│   └── ETL_BI.morph  # Proyecto EasyMorph con el flujo completo           
│      
│
├── Diagramas/
│   ├── Diagrama estrella.png      
│   └── Diagrama modelo operacional.drawio.png       
│
├── ReporteDepartamento/ # Archivos originales proporcionados por TEC Digital
│                                
├── Dashboard_Proyecto2_BI.pbix       # Dashboard interactivo en Power BI
├── Exposicion_BI.pdf                 # Presentación del proyecto
├── InformeProyecto_TecDigital.pdf    # Informe técnico completo
├── TD-60-2026_Aval para uso de inform... # Aval institucional del TEC Digital
└── README.md                         # Este archivo
```

---

## Recursos adicionales

- **Grabación de la presentación (Google Drive):** https://drive.google.com/file/d/1CNe6JmRbf7cHf_faVX7zqfyKEzWSsJaS/view?usp=drivesdk
- **Grabación de la presentación (OneDrive):** https://estudianteccrmy.sharepoint.com/:v:/g/personal/mar_rivera_estudiantec_cr/IQAYLuIZ7i7USpeIjxhiMA2AAcGeYwLJYMtvQGMTosbW5xk?e=j94AEg
