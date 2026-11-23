# 📐 TutorMates DUA — Cuaderno Privado de Gestión Docente de Matemáticas (3º ESO)

Aplicación web privada diseñada exclusivamente para **profesores de Matemáticas de Educación Secundaria**, concebida para automatizar la burocracia administrativa, planificar secuenciaciones de aula realistas, gestionar sustituciones imprevistas, crear dinámicas cooperativas y gamificadas, y generar material adaptado al **Diseño Universal para el Aprendizaje (DUA)**, garantizando un cumplimiento riguroso del **Reglamento General de Protección de Datos (RGPD)**.

---

## 🛡️ 1. Arquitectura de Privacidad y Cumplimiento RGPD (GDPR)

La protección del menor y de los datos académicos sensibles en el entorno educativo es un imperativo legal y ético. Esta aplicación implementa el principio de **Privacidad desde el Diseño y por Defecto** (*Privacy by Design and by Default*, Art. 25 del RGPD UE 2016/679 y Ley Orgánica 3/2018 LOPDGDD):

1. **Anonimización Radical por Códigos Estancos (`ALU_01` a `ALU_15`):**
   - El sistema opera únicamente con identificadores numéricos ficticios.
   - En ningún momento se almacenan, leen ni transmiten nombres, apellidos, DNI, correos electrónicos, fechas de nacimiento ni direcciones del alumnado.
2. **Cero Exposición de Datos Personales (PII) a Modelos de Lenguaje:**
   - Al interactuar con la API de Google Gemini (`google-genai`), los prompts y contextos contienen exclusivamente códigos alfanuméricos y observaciones pedagógicas objetivas.
   - Las empresas proveedoras de IA nunca reciben datos de carácter personal que permitan identificar o reidentificar a un estudiante.
   - **Detección Dinámica de Modelos:** La aplicación consulta dinámicamente `client.models.list()` para seleccionar de forma automática un modelo *Flash* vigente y compatible con `generateContent` (ej. `gemini-2.5-flash`), evitando errores 404 por modelos descatalogados.
3. **Custodia de la Matriz de Reidentificación:**
   - La correspondencia entre el código anónimo (`ALU_02`) y la identidad del estudiante permanece exclusivamente en el cuaderno de calificaciones oficial del docente o en los sistemas de gestión académica institucional del centro (e.g., Séneca, Raíces, Ítaca), desconectada por completo de esta herramienta en la nube.
4. **Borradores Administrativos Seguros:**
   - Las notificaciones oficiales de absentismo, citaciones y comunicaciones con familias generadas por IA emplean marcadores y códigos anónimos, permitiendo al docente realizar la inserción final del nombre del alumno en su cliente de correo interno o procesador de textos local.

---

## 🔒 2. Mecanismo de Seguridad: Control de Acceso Exclusivo Docente

Para evitar de manera estricta que el alumnado o terceros ajenos al equipo docente puedan visualizar las calificaciones, notas pedagógicas o pruebas de evaluación:

- **Bloqueo del Lado del Servidor (*Server-side Gate*):** Antes de renderizar cualquier pestaña, componente o dato del aula, la aplicación valida la credencial contra `st.secrets["TEACHER_PASSWORD"]`.
- **Detención de Flujo Inmediata (`st.stop()`):** Si la contraseña no es introducida o no coincide, el script de Streamlit se detiene inmediatamente. No se carga el dataset en memoria ni se ejecuta ninguna llamada a la API.
- **Gestión Segura de Credenciales:** La contraseña docente y la clave de la API de Gemini se administran mediante el gestor de *Secrets* de Streamlit (nunca en el código fuente ni en repositorios públicos).
- **Cierre de Sesión:** Dispone de un botón específico en la barra lateral para cerrar la sesión en ordenadores de aulas compartidas o salas de profesorado.

---

## 🚀 3. Funcionalidades de Automatización Docente

La aplicación organiza el trabajo en 5 pestañas principales y especializadas mediante sub-pestañas integradas:

### 📋 Pestaña 1: Pasar Lista y Burocracia
Organizada en dos sub-pestañas:

- **Sub-pestaña '📋 Control de Asistencia y Burocracia':**
  - **Mini-Dashboard Estadístico del Aula:**
    - *Nota Media del Grupo:* Rendimiento global del aula con comparativa de aprobado general.
    - *% Alumnos en Riesgo:* Conteo exacto de estudiantes con Nota Media < 5.0 o más de 4 faltas acumuladas.
    - *Total Faltas Acumuladas:* Sumatorio de ausencias con media individualizada.
  - **Analítica Visual Nativa de Aula:**
    - *Gráfico de Dispersión (Scatter Chart):* Correlación directa entre Faltas Acumuladas y Nota Media clasificada por Perfil DUA.
    - *Gráfico de Barras (Bar Chart):* Desglose de faltas individuales por código `ALU_XX` frente al umbral crítico de absentismo (>4 faltas).
  - **Pase de Lista Interactivo:** Tabla editable (`st.data_editor`) para marcar la asistencia de hoy, ajustar faltas y redactar observaciones pedagógicas.
  - **Exportación en CSV:** Botón para descargar el registro actualizado de la sesión con un solo clic.
  - **Redacción Automatizada de Partes Oficiales con IA:**
    - Generación de partes para **Jefatura de Estudios / Comisión de Absentismo**.
    - Citaciones formales para **Familias y Tutores Legales**, cumpliendo con la normativa de absentismo de ESO.
  - **Ficha Preparatoria para Entrevistas con Familias (Tutoría Oficial):**
    - Informe conciso en 4 bloques: diagnóstico general, guía de explicaciones cercana para padres, compromisos compartidos familia-centro y preguntas estratégicas.
    - Exportación en **Microsoft Word (`.docx`)** con cabecera oficial y en **texto plano (`.txt`)**.

- **Sub-pestaña '🕒 Horario de Atención a Familias':**
  - **Registro Centralizado del Equipo Docente:** Carga los horarios de atención a padres de las 11 materias de 3º de ESO desde `profesores_tutorias.csv` (Matemáticas, Lengua, Inglés, Física y Química, Biología y Geología, Geografía e Historia, Tecnología, Educación Física, Música, Plástica y Orientación).
  - **Filtros Dinámicos:** Selector para filtrar instantáneamente por materia o por día de la semana (Lunes a Viernes).
  - **Asistente de Citación Oficial con IA:**
    - Selecciona el profesor/materia y el alumno por código anónimo (`ALU_XX`).
    - Gemini redacta un mensaje formal, empático y cordial convocando a la familia a la reunión, citando explícitamente el día exacto, la franja horaria, la ubicación (despacho/aula) y el canal de confirmación institucional según el registro oficial.
    - Botón para descargar el borrador en archivo de texto (`.txt`).

---

### 📅 Pestaña 2: Planificación de Aula
Organizada en cuatro sub-pestañas:

- **Sub-pestaña '📅 Distribución de Horas y Temario':**
  - Cálculo automático de sesiones netas descontando festivos y días no lectivos.
  - Secuenciación didáctica sesión por sesión (estructura 10-30-10 min) con adaptaciones DUA.
  - Inclusión de sesiones de dudas, simulacro cooperativo, examen y retroalimentación post-evaluación.

- **Sub-pestaña '🚨 Modo Guardia Express':**
  - Generador de fichas de guardia autocontenidas de **50 minutos** listas para usar ante una sustitución imprevista.
  - Incluye:
    1. Instrucciones organizativas rápidas para el profesor/a de guardia (5 min).
    2. Actividad de activación o calentamiento en pizarra (5-10 min).
    3. Cuatro ejercicios graduados en dificultad (básico, comprensión, problema aplicado y reto).
    4. **Hoja de soluciones explicadas paso a paso en lenguaje accesible**, redactada para que cualquier docente no especialista (Historia, Inglés, EF, etc.) pueda escribir y corregir los ejercicios en la pizarra con total tranquilidad.
  - Descarga en **Word (`.docx`)** con cabecera formal institucional.

- **Sub-pestaña '🎞️ Diapositivas de Sesión':**
  - Generador de presentaciones visuales estructuradas de **6 a 8 diapositivas** para proyectar en el aula según el tema, curso, duración y enfoque pedagógico (teoría guiada, práctica o repaso).
  - Cada diapositiva incluye: Título impactante, desarrollo didáctico con fórmulas en LaTeX nítidas y un bloque de **Notas del Profesor** con pautas de dinamización oral y preguntas para la clase.
  - **Vista Previa en Tarjetas:** Renderizado visual inmediato dentro de Streamlit.
  - **Descarga en HTML Interactivo Autocontenido (Reveal.js + KaTeX CDN):**
    - Archivo `.html` único sin dependencias locales.
    - Navegación con flechas del teclado y modo **pantalla completa** pulsando la tecla **F**.
    - Renderizado matemático perfecto con KaTeX.

- **Sub-pestaña '🏆 Reto Matemático Semanal (Gamificación)':**
  - Desafíos lúdicos semanales, enigmas y dinámicas tipo *Escape Room* de aula para dinamizar las clases de 3º de ESO y atender a *Altas Capacidades*.
  - Enigmas con ambientaciones narrativas (detectives, candados, misiones espaciales), sistema de 3 pistas escalonadas y solucionario razonado.
  - Exportación en Word (`.docx`) y Texto (`.txt`).

---

### 🎯 Pestaña 3: Diseñador DUA y Materiales
Organizada en tres sub-pestañas:

- **Sub-pestaña '🎯 Pruebas y Tareas DUA':**
  - **Atención a la Diversidad Multinivel:** Generación simultánea de versiones adaptadas para:
    - *Refuerzo / Nivel Inicial:* Andamiaje paso a paso y coeficientes sencillos.
    - *Estándar / Nivel Medio:* Competencias curriculares de 3º de ESO.
    - *TDAH:* Enunciados concisos, palabras clave en negrita y división en pasos breves sin sobrecarga cognitiva.
    - *Altas Capacidades:* Retos deductivos, pensamiento lateral y generalización algebraica.
    - *Discalculia Leve:* Apoyo visual estructurado sin distractores aritméticos complejos.
  - **Tipografía Matemática LaTeX:** Fórmulas limpias renderizadas con `$ ... $` y `$$ ... $$`.
  - **Exportación a Word (`.docx`) con Ecuaciones Nativas (OMML):**
    - Conversión mediante **Pandoc (`pypandoc`)** con `tex_math_dollars`. Todas las fórmulas LaTeX se convierten en **ecuaciones editables de Microsoft Word (OMML)**.
    - Cabecera formal con Departamento, Curso, Fecha, Nombre y Calificación.

- **Sub-pestaña '👥 Grupos Cooperativos':**
  - **Agrupamientos Pedagógicamente Equilibrados:** Diseña parejas de tutoría entre iguales, tríos o grupos de 4 alumnos a partir de los datos reales del aula (`clase_3eso.csv`).
  - **Asignación de Roles DUA:** En grupos de 4 asigna roles adaptados al perfil del alumno (Coordinador, Secretario, Portavoz, Supervisor de Material).
  - **Gestión de Incompatibilidades:** Campo abierto para indicar restricciones (ej. *'No juntar a ALU_02 con ALU_07'*).
  - **Justificación Pedagógica Individualizada:** Argumenta por qué funciona cada equipo y proporciona pautas de supervisión de mesa.
  - Exportación en **Word (`.docx`)** y **Texto (`.txt`)**.

- **Sub-pestaña '🎮 Kahoots / Quizizz':**
  - **Cuestionarios Gamificados Basados en Errores Típicos:** Genera de 5 a 12 preguntas de 4 opciones donde los 3 distractores evalúan fallos conceptuales y de operativa frecuentes en secundaria (signos negativos, suma directa de denominadores, fallos de distributiva, etc.).
  - **Diagnóstico del Error para el Docente:** Explica en pantalla qué fallo conceptual evidencia cada distractor para trabajarlo activamente en el aula.
  - **Descarga en Plantilla CSV Oficial:** Exporta directamente un archivo `.csv` compatible con la importación automática de **Kahoot, Quizizz y Blooket** (columnas: `Question`, `Answer 1`, `Answer 2`, `Answer 3`, `Answer 4`, `Time limit (sec)`, `Correct answer(s)`).

---

### 🤖 Pestaña 4: Asistente Docente con Contexto del Aula
- **Chatbot Pedagógico Contextualizado:** Acceso en tiempo real al dataset completo de los 15 alumnos anónimos (`ALU_01` a `ALU_15`).
- **Tutoría entre Iguales:** Recomendaciones de agrupamientos cooperativos complementando perfiles fuertes (Altas Capacidades) con alumnos que necesitan apoyo.
- **Detección de Riesgo Académico:** Identificación de alumnos en riesgo de evaluación negativa y planes de rescate.
- **Botones de Consulta Rápida:** Atajos para resolver dudas frecuentes sobre agrupamientos o adaptaciones.

---

### 📜 Pestaña 5: Normativa y Plan de Convivencia
- **Asistente Oficial Multidocumental Basado en Evidencia:**
  - **Precarga e Indexación Integral de Documentos Oficiales del Centro:**
    - `Normas_organizacion_funcionamento.pdf` (**NOF**, 38 páginas): Régimen de ausencias y justificaciones, puntualidad del alumnado y profesorado, funciones docentes y profesorado de guardia, delegados y junta de delegados, huelgas estudiantiles, copias en exámenes, revisiones y reclamación de calificaciones, comunicación telemática con familias, protocolo de accidentes, fondo solidario de libros y normativa de uso de instalaciones (gimnasio Pompeo y condiciones de cesión del Paraninfo).
    - `Plan Convivencia Xuño 2026.pdf` (**Plan de Convivencia**, 30 páginas): Diagnóstico de convivencia y tipología de conflictos, plan de acogida para nuevos alumnos (1º ESO, inmigrantes), familias y profesorado sustituto, Plan de Acción Tutorial (PAT - programas Convivir, Ser Persoa, Pensar), protocolos rigurosos de prevención, detección e intervención urgente ante acoso escolar y ciberacoso (conforme a la Ley 4/2011 y Decreto 8/2015), tipificación exhaustiva de conductas leves y graves, medidas correctoras reeducadoras (procedimiento conciliado y común), mediación escolar y cuadro competencial sancionador.
  - **Consultas Individuales y Cruzadas con IA:** El chatbot responde tanto a preguntas específicas sobre un documento concreto como a dudas transversales que contrastan o coordinan ambos textos normativos (por ejemplo, regulación y retirada del móvil en las NOF frente a su tipificación como falta leve en el Plan de Convivencia; gestión de faltas y partes frente a guardias).
  - **Selector de Ámbito de Consulta:** Permite enfocar la búsqueda en ambos documentos de forma simultánea (opción predeterminada), filtrar exclusivamente por el *Plan de Convivencia (Xuño 2026)* o por las *NOF*, o consultar documentos PDF complementarios subidos de forma temporal.
  - **Sugerencias Rápidas de Consulta Equilibradas:** 6 accesos directos temáticos sobre acoso escolar, faltas y sanciones, dispositivos móviles, guardias y puntualidad, procedimiento conciliado/mediación, e instalaciones del centro.
  - **Caché Inteligente y Actualización Automática:** Utiliza `@st.cache_data` con cálculo de firma hash (`_signature`) sobre la carpeta de documentos y botón de recarga forzada (`🔄 Recargar documentos`).
  - **Citación Obligatoria de Documento y Página:** Cada respuesta incluye citas verificables con el nombre exacto del archivo y la página de procedencia (`[Nombre_Documento.pdf, pág. X]`).
  - **Garantía Anti-Alucinación:** Si una norma o trámite no consta explícitamente en los documentos cargados, el chatbot responde taxativamente informando de su ausencia sin formular conjeturas.

---

## 📦 4. Estructura del Proyecto

```plaintext
tutor-mates/
├── .gitignore               # Exclusión de credenciales y temporales
├── packages.txt             # Paquetes del sistema operativo para Streamlit Cloud (pandoc)
├── README.md                # Documentación técnica, pedagógica y legal completa
├── requirements.txt         # Dependencias Python (streamlit, google-genai, pandas, pypdf, pypandoc)
├── clase_3eso.csv           # Registro anonimizado de 15 alumnos (perfiles DUA y notas)
├── profesores_tutorias.csv  # Horarios y canales de atención a familias de 11 materias
├── documentos_centro/       # Carpeta para reglamentos y planes de convivencia (PDF)
│   ├── Normas_organizacion_funcionamento.pdf  # NOF (38 págs.): régimen interno, guardias e instalaciones
│   └── Plan Convivencia Xuño 2026.pdf         # Plan Convivencia (30 págs.): acoso, tipificación y sanciones
└── app.py                   # Aplicación principal de Streamlit con sub-pestañas integradas
```

---

## ⚙️ 5. Despliegue en Streamlit Cloud

Para desplegar esta herramienta privada en tu cuenta de **Streamlit Cloud**:

1. Sube estos archivos a tu repositorio en **GitHub**.
2. Entra en [share.streamlit.io](https://share.streamlit.io/) y selecciona tu repositorio.
3. En la sección **App Settings > Secrets**, añade las siguientes variables:

```toml
TEACHER_PASSWORD = "TuClaveSecretaDocente2026"
GEMINI_API_KEY = "AIzaSy..."
```

4. Haz clic en **Deploy**. La aplicación estará lista y protegida para tu uso exclusivo.

---

## 💻 6. Ejecución Local (Opcional)

Si en el futuro deseas ejecutar la aplicación en tu entorno local:

1. Crea el archivo `.streamlit/secrets.toml`:
   ```toml
   TEACHER_PASSWORD = "TuClaveSecretaDocente2026"
   GEMINI_API_KEY = "AIzaSy..."
   ```
2. Instala las dependencias:
   ```bash
   pip install -r requirements.txt
   ```
3. Ejecuta Streamlit:
   ```bash
   streamlit run app.py
   ```
