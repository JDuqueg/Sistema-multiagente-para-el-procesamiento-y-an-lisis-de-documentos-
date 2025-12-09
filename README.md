# Sistema Agentic AI Multi-Agente para Procesamiento de Documentos

**Universidad Nacional de Colombia - Sede Medellín**  
**Procesamiento del Lenguaje Natural (PLN)**  
**Práctica 2: Agentic AI con LLMs y LangChain 1.0**

## 📋 Descripción

Sistema multi-agente basado en LangChain 1.0 que implementa RAG (Retrieval Augmented Generation) para análisis inteligente de documentos. El sistema integra 5 agentes especializados utilizando los LLMs de Groq (familia Llama) de forma diferenciada según las necesidades de cada componente.

**Dominio de aplicación:** Agricultura - análisis de documentos sobre técnicas agrícolas, manejo de cultivos, sostenibilidad, y prácticas de producción.

## 🏗️ Arquitectura del Sistema

### Agentes Implementados

1. **DocumentIndexer (Agente de Indexación)**
   - Carga documentos (PDF/TXT/HTML)
   - Limpieza, chunking y generación de embeddings
   - Indexación en FAISS

2. **QueryClassifier (Agente Clasificador)**
   - **LLM:** Groq (Llama 3.3 70B Versatile)
   - **Justificación:** Llama 3.3 70B ofrece excelente capacidad de interpretación del lenguaje natural y comprensión contextual profunda con velocidad superior, esencial para clasificar intenciones con precisión
   - Clasifica consultas en 4 categorías: búsqueda, resumen, comparación, general

3. **SemanticRetriever (Agente Recuperador)**
   - **LLM:** Groq (Llama 3.1 8B Instant)
   - **Justificación:** Llama 3.1 8B Instant proporciona velocidad de inferencia excepcional, crítica para recuperación ultrarrápida de documentos
   - Búsqueda semántica en FAISS

4. **RAGResponseGenerator (Agente Generador)**
   - **LLM:** Groq (Llama 3.3 70B Versatile)
   - **Justificación:** Llama 3.3 70B ofrece generación ultrarrápida con alta calidad, ideal para respuestas contextuales complejas en tiempo real
   - Genera respuestas justificadas con citas

5. **ResponseVerifier (Agente Verificador)**
   - **LLM:** Groq (Llama 3.3 70B Versatile)
   - **Justificación:** Llama 3.3 70B sobresale en razonamiento complejo y validación lógica, necesario para detectar alucinaciones y verificar coherencia
   - Valida coherencia y evita alucinaciones

6. **Orchestrator (Agente Orquestador)**
   - **LLM:** Groq (Llama 3.1 8B Instant)
   - **Justificación:** Llama 3.1 8B garantiza decisiones de enrutamiento instantáneas, manteniendo el flujo del sistema fluido
   - Coordina el flujo entre todos los agentes

## 🔄 Flujo del Sistema

```
Usuario → Orchestrator → QueryClassifier (Gemini)
                              ↓
                         [Clasifica intención]
                              ↓
                    ┌─────────┴─────────┐
                    ↓                   ↓
              [general]           [búsqueda/resumen/comparación]
                    ↓                   ↓
            Respuesta directa    SemanticRetriever (Groq)
            con Gemini                  ↓
                              RAGResponseGenerator (Groq)
                                        ↓
                              ResponseVerifier (Gemini)
                                        ↓
                                [¿Válida?]
                              ↙          ↘
                            Sí           No → Regenerar
                            ↓
                    Respuesta Final + Trazabilidad
```

## 📦 Instalación

### Requisitos Previos
- Python 3.10+
- Conexión a Internet
- API Keys de Google (Gemini) y Groq

### Opción 1: Instalación Automática (Recomendada)

```bash
# 1. Navega al directorio del proyecto
cd ruta/a/tu/proyecto

# 2. Ejecuta el configurador
python setup_project.py
```

Este script:
- Crea la estructura de carpetas necesaria (`data/`, `faiss_index/`, `results/`)
- Opcionalmente copia tus documentos desde la ubicación antigua
- Crea el archivo `.gitignore`
- Verifica la estructura

### Opción 2: Instalación Manual

```bash
# 1. Navega al directorio del proyecto
cd ruta/a/tu/proyecto

# 2. Crea la estructura de carpetas
mkdir data
mkdir faiss_index
mkdir results

# 3. Coloca tus documentos en la carpeta data/
# (arrastra o copia tus PDFs/TXT/HTML)
```

### Paso 2: Crear Entorno Virtual

```bash
python -m venv venv

# Activar en Windows
venv\Scripts\activate

# Activar en Linux/Mac
source venv/bin/activate
```

### Paso 3: Instalar Dependencias

```bash
pip install -r requirements.txt
```

### Paso 4: Configurar Variables de Entorno

Edita el archivo `.env`:

```env
# API Keys
GOOGLE_API_KEY=AIzaSyCkBEhiNP85HAvgDlTpPcUJl3UkCm3Aruc
GROQ_API_KEY=tu_api_key_aqui  # Obtén en https://console.groq.com/keys

# Rutas (rutas relativas - portables)
DOCUMENTS_PATH=./data
FAISS_INDEX_PATH=./faiss_index

# Configuración
CHUNK_SIZE=1000
CHUNK_OVERLAP=200
MAX_RETRIEVAL_DOCS=5
```

### Paso 5: Verificar Documentos

Asegúrate de tener al menos 100 documentos en `./data/`

Para verificar:
```bash
# Windows
dir /s data\*.pdf data\*.txt data\*.html

# Linux/Mac
find data -type f \( -name "*.pdf" -o -name "*.txt" -o -name "*.html" \) | wc -l
```

## 🚀 Uso del Sistema

### Primera Ejecución (Indexación)

```bash
python main.py
```

En la primera ejecución:
1. El sistema cargará todos los documentos de la carpeta `data`
2. Generará embeddings y creará el índice FAISS
3. Guardará el índice en `faiss_index/` para ejecuciones futuras

**Nota:** La indexación puede tardar varios minutos dependiendo del número de documentos.

### Ejecuciones Posteriores

```bash
python main.py
```

El sistema cargará el índice existente y estará listo inmediatamente.

### Modo Interactivo

Una vez iniciado, puedes hacer consultas sobre agricultura:

```
👤 Ingresa tu consulta: ¿Qué técnicas de riego se mencionan en los documentos?
```

Tipos de consultas soportadas:
- **Búsqueda:** "¿Qué información hay sobre fertilización orgánica en los documentos?"
- **Resumen:** "Resume el contenido sobre control de plagas"
- **Comparación:** "Compara agricultura orgánica con convencional según los documentos"
- **General:** "¿Qué es la rotación de cultivos?" (respuestas sin necesidad de documentos)

## 🧪 Casos de Prueba

Para ejecutar la suite de casos de prueba:

```bash
python test_cases.py
```

Esto ejecutará 12+ casos de prueba predefinidos y generará:
- Reporte JSON con resultados detallados (`test_results.json`)
- Tabla resumen en consola
- Estadísticas de rendimiento

## 📊 Trazabilidad

El sistema mantiene trazabilidad completa de cada consulta:
- Timestamp de cada evento
- Agente que ejecutó la acción
- Detalles de las decisiones
- Documentos utilizados
- Resultados de verificación

Para ver la trazabilidad, responde 'S' cuando se te pregunte después de cada consulta.

## 🔑 Justificación de Selección de LLMs

### Groq con Modelos Llama
**Usado en todos los agentes**

**Modelos utilizados:**
- **Llama 3.3 70B Versatile:** Para tareas de clasificación, generación y verificación
- **Llama 3.1 8B Instant:** Para tareas de orquestación y recuperación rápida

**Justificación General:**
- **Velocidad extrema:** Groq ofrece la inferencia más rápida del mercado (hasta 800 tokens/segundo)
- **Bajo costo:** API gratuita con límites generosos para desarrollo y pruebas
- **Alta calidad:** Los modelos Llama 3.x son de código abierto y muy confiables
- **Disponibilidad:** No requiere lista de espera ni aprobaciones
- **Consistencia:** Usar una sola familia de modelos simplifica la arquitectura

**Estrategia de Selección por Agente:**

1. **QueryClassifier (Llama 3.3 70B):**
   - Requiere comprensión profunda del lenguaje natural
   - El modelo 70B ofrece mejor precisión en clasificación de intenciones
   - Velocidad suficiente para no afectar experiencia de usuario

2. **SemanticRetriever (Llama 3.1 8B Instant):**
   - Operación de alta frecuencia que requiere máxima velocidad
   - 8B Instant es el más rápido disponible
   - La recuperación es principalmente basada en embeddings, el LLM es auxiliar

3. **RAGResponseGenerator (Llama 3.3 70B):**
   - Generación de texto requiere el modelo más capaz
   - 70B ofrece respuestas más coherentes y contextuales
   - Balance óptimo entre calidad y velocidad

4. **ResponseVerifier (Llama 3.3 70B):**
   - Razonamiento crítico requiere modelo más grande
   - Detección de alucinaciones necesita capacidad analítica profunda
   - 70B ofrece mejor validación lógica

5. **Orchestrator (Llama 3.1 8B Instant):**
   - Decisiones de enrutamiento son simples y binarias
   - Velocidad es crítica para mantener flujo fluido
   - 8B Instant es perfecto para esta tarea

### Alternativa Considerada: Gemini

**Por qué NO se usó Gemini:**
- Requiere configuración adicional de Google Cloud
- Límites de API más restrictivos en plan gratuito
- Mayor latencia comparado con Groq
- Complejidad adicional al manejar dos proveedores diferentes

**Cuándo considerar Gemini:**
- Proyectos con presupuesto para APIs premium
- Casos que requieren capacidades multimodales (imágenes, video)
- Cuando se necesita integración con ecosistema Google

### Embeddings: Sentence Transformers Local

**Modelo:** all-MiniLM-L6-v2

**Justificación:**
- **Gratuito y sin límites:** No consume cuota de API
- **Rápido:** Procesamiento local en GPU/CPU
- **Eficiente:** Solo 80MB de memoria
- **Portable:** Funciona sin conexión a internet (después de primera descarga)
- **Calidad:** Excelente para embeddings en español e inglés

## 📁 Estructura del Proyecto

```
.
├── main.py                 # Sistema principal multi-agente
├── test_cases.py          # Suite de casos de prueba
├── requirements.txt       # Dependencias
├── .env                   # Variables de entorno
├── README.md             # Este archivo
├── data/                 # Documentos (100+)
│   ├── *.pdf
│   ├── *.txt
│   └── *.html
├── faiss_index/          # Índice FAISS (generado)
│   ├── index.faiss
│   └── index.pkl
└── test_results.json     # Resultados de pruebas (generado)
```

## 🛠️ Herramientas (Tools) Implementadas

Las herramientas están integradas dentro de los agentes:

1. **Document Loaders** (PyPDFLoader, TextLoader, UnstructuredHTMLLoader)
2. **Text Splitter** (RecursiveCharacterTextSplitter)
3. **Embeddings Generator** (GoogleGenerativeAIEmbeddings)
4. **Vector Store** (FAISS)
5. **LLM Chains** (ChatPromptTemplate + LLM + OutputParser)
6. **Similarity Search** (FAISS similarity_search)
7. **Traceability Logger** (Sistema de logging personalizado)

## 📝 Generación de Informe Técnico

Para documentar los casos de uso:

1. Ejecuta `test_cases.py`
2. Revisa `test_results.json`
3. Captura pantallas del sistema en ejecución
4. Documenta:
   - Descripción del dominio seleccionado
   - Justificación de LLMs por agente
   - Resultados de los 10+ casos de uso
   - Trazabilidad de ejecuciones
   - Análisis de rendimiento

## 🎥 Video de Sustentación

Contenido sugerido para el pitch (máx. 5 minutos):

1. **Introducción (30s)**
   - Presentación del equipo
   - Dominio seleccionado

2. **Arquitectura (1m)**
   - Diagrama de agentes
   - Flujo del sistema
   - Justificación de LLMs

3. **Demostración (2m 30s)**
   - Caso de búsqueda
   - Caso de resumen
   - Caso de comparación
   - Caso general
   - Visualización de trazabilidad

4. **Aspectos Técnicos (1m)**
   - Integración LangChain
   - RAG con FAISS
   - Manejo de errores y verificación

5. **Conclusiones (30s)**
   - Logros alcanzados
   - Posibles mejoras

## 🐛 Solución de Problemas

### Error: "GROQ_API_KEY no configurada"
- Obtén tu API key en https://console.groq.com/keys
- Añádela al archivo `.env`

### Error al cargar documentos
- Verifica que la ruta sea correcta
- Asegura que los archivos tengan permisos de lectura
- Verifica que los PDFs no estén encriptados

### Error de memoria al indexar
- Reduce `chunk_size` en el código
- Procesa documentos en lotes más pequeños
- Aumenta la memoria disponible para Python

### Respuestas lentas
- Verifica tu conexión a Internet
- Los LLMs requieren conexión para inferencia
- Primera ejecución siempre es más lenta (embeddings)

## 📚 Referencias

- [LangChain Documentation](https://python.langchain.com/)
- [Google Gemini API](https://ai.google.dev/)
- [Groq API](https://console.groq.com/docs)
- [FAISS Documentation](https://github.com/facebookresearch/faiss)

## 👥 Equipo

[Añade aquí los nombres de los integrantes del grupo]

## 📄 Licencia

Este proyecto es parte de un trabajo académico para la Universidad Nacional de Colombia.

---

**Fecha de Entrega:** Miércoles 10 de diciembre de 2025, 12:00 meridiano  
**Profesor:** Jaime Alberto Guzmán Luna