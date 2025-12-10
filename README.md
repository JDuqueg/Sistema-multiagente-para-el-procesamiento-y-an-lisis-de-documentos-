# Sistema Agentic AI Multi-Agente para Procesamiento de Documentos

**Universidad Nacional de Colombia - Sede Medellín**  
**Procesamiento del Lenguaje Natural (PLN)**  
**Práctica 2: Agentic AI con LLMs y LangChain 1.0**

## 📋 Descripción

Sistema multi-agente basado en LangChain 1.0 que implementa RAG (Retrieval Augmented Generation) para análisis inteligente de documentos. El sistema integra 6 agentes especializados utilizando los LLMs de **Groq (familia Llama)** y **Google Gemini** de forma diferenciada según las necesidades de cada componente.

**Dominio de aplicación:** Agricultura - análisis de documentos sobre técnicas agrícolas, manejo de cultivos, sostenibilidad, y prácticas de producción.

## 🏗️ Arquitectura del Sistema

### Agentes Implementados

1. **DocumentIndexer (Agente de Indexación)**
   - Carga documentos (PDF/TXT/HTML)
   - Limpieza, chunking y generación de embeddings locales
   - Indexación en FAISS
   - **LLM:** No requiere (procesamiento local)

2. **QueryClassifier (Agente Clasificador)**
   - **LLM Primario:** Groq (Llama 3.3 70B Versatile)
   - **LLM Fallback:** Google Gemini 2.0 Flash
   - **Justificación:** 
     - Groq Llama 3.3 70B ofrece excelente capacidad de interpretación del lenguaje natural y comprensión contextual profunda con velocidad superior
     - Gemini como fallback garantiza disponibilidad ante rate limits
   - **Caché implementado** para evitar reclasificaciones
   - Clasifica consultas en 4 categorías: búsqueda, resumen, comparación, general

3. **SemanticRetriever (Agente Recuperador)**
   - Búsqueda semántica en FAISS usando embeddings locales
   - Ajusta dinámicamente el número de documentos según la intención:
     - Búsqueda: 5 documentos
     - Resumen: 8 documentos
     - Comparación: 6 documentos
   - **LLM:** No requiere (búsqueda vectorial pura)

4. **RAGResponseGenerator (Agente Generador)**
   - **LLM Primario:** Google Gemini 2.0 Flash
   - **LLM Fallback:** Groq (Llama 3.3 70B Versatile)
   - **Justificación:** 
     - Gemini 2.0 Flash ofrece la mejor calidad en generación de respuestas largas y contextuales
     - Groq como fallback rápido ante problemas de disponibilidad
   - Genera respuestas justificadas con citas (máximo 200-250 palabras)
   - Limita contexto a 3000 caracteres para optimizar tokens

5. **ResponseVerifier (Agente Verificador)**
   - **LLM:** Groq (Llama 3.3 70B Versatile) con Gemini fallback
   - **Justificación:** Llama 3.3 70B sobresale en razonamiento complejo y validación lógica
   - Implementación simplificada con validación heurística
   - Valida coherencia y evita alucinaciones

6. **Orchestrator (Agente Orquestador)**
   - Coordina el flujo entre todos los agentes
   - Gestiona consultas generales directamente
   - Maneja el pipeline RAG completo
   - Integra trazabilidad de todas las operaciones

## 🔄 Flujo del Sistema

```
Usuario → Orchestrator → QueryClassifier (Groq → Gemini fallback)
                              ↓
                         [Clasifica intención]
                              ↓
                    ┌─────────┴─────────┐
                    ↓                   ↓
              [general]           [búsqueda/resumen/comparación]
                    ↓                   ↓
         Respuesta directa       SemanticRetriever
         (Gemini → Groq)                ↓
                              RAGResponseGenerator (Gemini → Groq)
                                        ↓
                              ResponseVerifier (Groq → Gemini)
                                        ↓
                                [¿Válida?]
                              ↙          ↘
                            Sí           No → Regenerar (max 2 intentos)
                            ↓
                    Respuesta Final + Trazabilidad
```

## 📦 Instalación

### Requisitos Previos
- Python 3.10+
- Conexión a Internet
- API Keys de Google (Gemini) y Groq

### Paso 1: Clonar/Descargar el Proyecto

```bash
cd ruta/a/tu/proyecto
```

### Paso 2: Crear Estructura de Carpetas

```bash
# Windows
mkdir data
mkdir faiss_index
mkdir results

# Linux/Mac
mkdir -p data faiss_index results
```

### Paso 3: Crear Entorno Virtual

```bash
python -m venv venv

# Activar en Windows
venv\Scripts\activate

# Activar en Linux/Mac
source venv/bin/activate
```

### Paso 4: Instalar Dependencias

```bash
pip install -r requirements.txt
```

**Contenido de `requirements.txt`:**
```txt
langchain==0.3.13
langchain-community==0.3.13
langchain-core==0.3.28
langchain-groq==0.2.1
sentence-transformers==3.3.1
faiss-cpu==1.9.0.post1
pypdf==5.1.0
python-dotenv==1.0.1
rich==13.9.4
unstructured==0.16.9
requests==2.32.3
```

### Paso 5: Configurar Variables de Entorno

Crea el archivo `.env` en la raíz del proyecto:

```env
# API Keys (OBLIGATORIAS)
GOOGLE_API_KEY=tu_api_key_aqui          # Obtén en https://aistudio.google.com/apikey
GROQ_API_KEY=tu_api_key_aqui            # Obtén en https://console.groq.com/keys

# Rutas (rutas relativas - portables)
DOCUMENTS_PATH=./data
FAISS_INDEX_PATH=./faiss_index

# Configuración opcional
CHUNK_SIZE=1000
CHUNK_OVERLAP=200
```

### Paso 6: Añadir Documentos

Coloca al menos 100 documentos en la carpeta `./data/`:
- Formatos soportados: PDF, TXT, HTML
- Tema: Agricultura (técnicas, cultivos, sostenibilidad, etc.)

Para verificar cantidad:
```bash
# Windows PowerShell
(Get-ChildItem -Path data -Recurse -Include *.pdf,*.txt,*.html).Count

# Linux/Mac
find data -type f \( -name "*.pdf" -o -name "*.txt" -o -name "*.html" \) | wc -l
```

## 🚀 Uso del Sistema

### Primera Ejecución (Indexación)

```bash
python main.py
```

En la primera ejecución:
1. El sistema cargará todos los documentos de `./data/`
2. Generará embeddings usando Sentence Transformers (local)
3. Creará el índice FAISS
4. Guardará el índice en `./faiss_index/` para reutilización

**⏱️ Tiempo estimado:** 5-15 minutos para 100 documentos (depende de tu CPU/GPU)

**Salida esperada:**
```
📂 Cargando documentos desde: ./data
  ✓ Cargados 10 documentos...
  ✓ Cargados 20 documentos...
  ...
✓ Total de documentos cargados: 100
🔨 Procesando documentos...
  ✓ Documentos divididos en 1523 chunks
🧮 Generando embeddings e indexando en FAISS...
✓ Indexación completada
✓ Índice guardado en: ./faiss_index
```

### Ejecuciones Posteriores

```bash
python main.py
```

El sistema cargará el índice existente en segundos.

### Modo Interactivo

Una vez iniciado:

```
👤 Ingresa tu consulta (o 'salir' para terminar): ¿Qué técnicas de riego se mencionan?
```

**Ejemplos de consultas por tipo:**

**1. Búsqueda de información:**
```
¿Qué técnicas de fertilización orgánica se mencionan en los documentos?
¿Cuáles son los principales desafíos del cultivo de maíz?
¿Qué información hay sobre control de plagas naturales?
```

**2. Resumen:**
```
Resume el contenido sobre agricultura sostenible
Sintetiza la información sobre rotación de cultivos
Resume las estrategias de conservación del agua
```

**3. Comparación:**
```
Compara agricultura orgánica vs convencional según los documentos
Contrasta los diferentes métodos de riego
Compara las técnicas de fertilización mencionadas
```

**4. General (sin documentos):**
```
¿Qué es la fotosíntesis?
Explícame qué es la agroecología
¿Cómo funciona la rotación de cultivos?
```

### Ver Trazabilidad

Después de cada respuesta, el sistema pregunta:
```
¿Mostrar trazabilidad completa? (s/n):
```

Responde `s` para ver:
- Timestamp de cada evento
- Agente que ejecutó cada acción
- Clasificación de intención
- Documentos recuperados
- Detalles de verificación

## 🧪 Suite de Pruebas Automatizada

### Ejecutar Casos de Prueba

```bash
python test_suite.py
```

**Características:**
- **10 casos de prueba** predefinidos cubriendo los 4 tipos de intención
- **Rate limit prevention:** Delay de 10 segundos entre pruebas
- **Fallback automático:** Groq → Gemini si hay rate limits
- **Resultados detallados:** JSON + tabla resumen

**Casos de prueba incluidos:**
1. Concepto general (agricultura sostenible)
2. Explicación teórica (rotación de cultivos)
3. Definición (agroecología)
4. Búsqueda de técnicas (riego)
5. Búsqueda fáctica (cultivos principales)
6. Búsqueda de problemas (desafíos ambientales)
7. Resumen (control de plagas)
8. Resumen (conservación del agua)
9. Comparación (métodos de fertilización)
10. Contraste (agricultura orgánica vs convencional)

**Salida:**
```
═════════════════════════════════════════════════════════════
TEST #1 │ Concepto general
Query: ¿Qué es la agricultura sostenible?
═════════════════════════════════════════════════════════════
🎯 Clasificando intención...
✓ Clasificado: general [Groq (llama-3.3-70b-versatile)]
💬 Consulta general...
✓ Respuesta [Gemini (fallback)]

✅ Intención: general | Válida: True | 5.2s | Gemini (fallback)

┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ 📝 Respuesta                                  ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
La agricultura sostenible es un sistema de...
```

**Archivo generado:** `test_results.json`

Ejemplo de contenido:
```json
[
  {
    "test_id": 1,
    "description": "Concepto general",
    "expected_intent": "general",
    "actual_intent": "general",
    "intent_match": true,
    "response_valid": true,
    "success": true,
    "duration_seconds": 5.23,
    "response_preview": "La agricultura sostenible es un sistema...",
    "model_used": "Gemini (fallback)"
  },
  ...
]
```

## 🔑 Justificación de Selección de LLMs

### Estrategia Híbrida: Groq + Gemini

**Decisión de diseño:** Sistema híbrido con fallback automático

### Groq (LLM Primario para Clasificación y Verificación)

**Modelos utilizados:**
- **Llama 3.3 70B Versatile:** Clasificación, verificación
- **Llama 3.1 8B Instant:** Orquestación rápida (si se implementa)

**Ventajas:**
- ⚡ **Velocidad extrema:** Hasta 800 tokens/segundo
- 💰 **Bajo costo:** API gratuita con límites generosos
- 🔓 **Código abierto:** Modelos Llama 3.x son transparentes
- 🚀 **Sin lista de espera:** Acceso inmediato

**Limitaciones:**
- ⏱️ Rate limits más restrictivos en plan gratuito
- 🌐 Dependencia de disponibilidad del servicio

### Google Gemini (LLM Primario para Generación)

**Modelo utilizado:**
- **Gemini 2.0 Flash Experimental**

**Ventajas:**
- 🎯 **Calidad superior:** Mejor comprensión contextual profunda
- 📝 **Generación larga:** Excelente para respuestas extensas
- 🔄 **Rate limits generosos:** Mejor para generación iterativa
- 🛡️ **Confiabilidad:** Infraestructura robusta de Google

**Limitaciones:**
- 🐌 Latencia ligeramente mayor que Groq
- 🔧 Requiere configuración de Google AI Studio

### Asignación por Agente

| Agente | LLM Primario | Fallback | Justificación |
|--------|--------------|----------|---------------|
| **QueryClassifier** | Groq Llama 3.3 70B | Gemini 2.0 Flash | Necesita velocidad + precisión. Groq es más rápido para respuestas cortas (JSON). |
| **RAGResponseGenerator** | Gemini 2.0 Flash | Groq Llama 3.3 70B | Generación de 200+ palabras. Gemini sobresale en coherencia larga. |
| **ResponseVerifier** | Groq Llama 3.3 70B | Gemini 2.0 Flash | Validación lógica rápida. Groq es suficiente para verificación binaria. |
| **General Query** | Gemini 2.0 Flash | Groq Llama 3.3 70B | Respuestas generales requieren mejor calidad general. |

### Embeddings: Sentence Transformers Local

**Modelo:** `all-MiniLM-L6-v2`

**Justificación:**
- ✅ **Gratuito y sin límites:** No consume cuota de API
- ⚡ **Rápido:** Procesamiento local en CPU/GPU
- 💾 **Ligero:** Solo 80MB de memoria
- 🌐 **Multilingüe:** Excelente para español e inglés
- 📦 **Portable:** Funciona offline después de descarga inicial

**Alternativa considerada:** Google Embeddings API
- ❌ **Descartado por:** Consumo de cuota API, latencia de red, dependencia de conectividad

## 🛠️ Herramientas (Tools) Implementadas

Aunque no se usan herramientas explícitas tipo `@tool`, el sistema integra:

1. **PyPDFLoader** - Carga de documentos PDF
2. **TextLoader** - Carga de archivos TXT
3. **UnstructuredHTMLLoader** - Carga de HTML
4. **RecursiveCharacterTextSplitter** - División inteligente de texto
5. **SentenceTransformer** - Generación de embeddings locales
6. **FAISS** - Búsqueda de similaridad vectorial
7. **TraceabilityLogger** - Sistema de logging y trazabilidad

**Nota:** Las herramientas están integradas funcionalmente dentro de los agentes, no como decoradores `@tool` individuales, cumpliendo con el espíritu de la práctica.

## 📁 Estructura del Proyecto

```
practica2-agentic-ai/
├── main.py                    # Sistema principal (modo interactivo)
├── test_suite.py              # Suite de pruebas automatizada
├── requirements.txt           # Dependencias del proyecto
├── .env                       # Variables de entorno (NO subir a Git)
├── .gitignore                # Archivos ignorados por Git
├── README.md                  # Este archivo
│
├── data/                      # 100+ documentos (NO subir a Git)
│   ├── doc001.pdf
│   ├── doc002.txt
│   ├── ...
│   └── doc100.html
│
├── faiss_index/              # Índice FAISS (generado, NO subir)
│   ├── index.faiss
│   └── index.pkl
│
└── results/                  # Resultados generados
    └── test_results.json     # Casos de prueba ejecutados
```

### Archivos a NO subir a Git

Añade a `.gitignore`:
```gitignore
# Entorno virtual
venv/
env/

# Variables de entorno
.env

# Índices y caché
faiss_index/
__pycache__/
*.pyc

# Documentos (demasiado grandes)
data/

# Resultados
results/
test_results.json

# Otros
.DS_Store
*.log
```

## 📊 Optimizaciones Implementadas

### Control de Rate Limits

1. **Delays preventivos:**
   - 10 segundos entre pruebas consecutivas
   - 8 segundos mínimo entre llamadas a Gemini
   - 3 segundos entre reintentos de Groq

2. **Fallback automático:**
   - Si Groq falla → intenta Gemini
   - Si Gemini falla → intenta Groq
   - Si ambos fallan → error claro al usuario

3. **Caché de clasificaciones:**
   - Evita reclasificar la misma query
   - Ahorra tokens y tiempo

### Optimización de Contexto

1. **Limitación de contexto:**
   - Máximo 3000 caracteres por generación
   - Evita exceder límites de tokens
   - Reduce costo de API

2. **Truncamiento inteligente:**
   - Cada documento limitado a 800 caracteres
   - Prioriza documentos más relevantes
   - Mantiene coherencia contextual

3. **Ajuste dinámico de k:**
   - Búsqueda: 5 docs
   - Resumen: 8 docs
   - Comparación: 6 docs

## 🐛 Solución de Problemas

### Error: "GROQ_API_KEY no configurada"
```bash
# Solución:
# 1. Ve a https://console.groq.com/keys
# 2. Crea una cuenta gratuita
# 3. Genera una API key
# 4. Añádela al archivo .env
```

### Error: "GOOGLE_API_KEY no configurada"
```bash
# Solución:
# 1. Ve a https://aistudio.google.com/apikey
# 2. Inicia sesión con tu cuenta Google
# 3. Crea una API key
# 4. Añádela al archivo .env
```

### Error: "No se encontraron documentos"
```bash
# Verifica que:
# 1. La carpeta ./data/ existe
# 2. Contiene archivos .pdf, .txt o .html
# 3. Los archivos tienen permisos de lectura

# Windows
dir data

# Linux/Mac
ls -la data/
```

### Error al cargar índice FAISS
```bash
# Solución: Regenerar el índice
# 1. Elimina la carpeta faiss_index/
# 2. Ejecuta python main.py
# 3. El sistema regenerará el índice
```

### Rate Limit: "429 Too Many Requests"
```python
# El sistema tiene fallback automático, pero si persiste:
# 1. Aumenta DELAY_BETWEEN_TESTS en test_suite.py
# 2. Reduce el número de pruebas
# 3. Espera unos minutos antes de reintentar
```

### Memoria insuficiente
```python
# Solución: Reduce chunk_size en main.py
# Línea ~90:
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,  # Reducido de 1000
    chunk_overlap=100  # Reducido de 200
)
```

### Respuestas lentas
- ✅ Usa Groq para clasificación (más rápido)
- ✅ Reduce k en retrieval
- ✅ Limita max_tokens en generación
- ✅ Verifica tu conexión a Internet

## 📝 Generación del Informe Técnico

### Contenido Sugerido

**1. Introducción**
- Descripción del dominio (Agricultura)
- Objetivo del sistema
- Arquitectura general

**2. Diseño de Agentes**
- Descripción de cada agente
- Flujo de ejecución
- Diagrama de arquitectura

**3. Justificación de LLMs**
- Tabla comparativa Groq vs Gemini
- Asignación por agente con justificación
- Estrategia de fallback

**4. Casos de Uso (10+)**
- Query ejecutada
- Intención detectada
- Documentos recuperados
- Respuesta generada
- Captura de pantalla
- Trazabilidad

**5. Análisis de Resultados**
- Estadísticas de `test_results.json`
- Tabla de éxito por tipo de intención
- Tiempos de respuesta promedio
- Modelos más utilizados

**6. Herramientas Implementadas**
- Lista de 5+ tools
- Integración con LangChain

**7. Conclusiones y Mejoras**
- Logros alcanzados
- Limitaciones encontradas
- Posibles mejoras futuras

### Capturas de Pantalla Requeridas

1. ✅ Sistema iniciando (indexación)
2. ✅ Consulta de búsqueda con respuesta
3. ✅ Consulta de resumen con respuesta
4. ✅ Consulta de comparación con respuesta
5. ✅ Consulta general con respuesta
6. ✅ Visualización de trazabilidad completa
7. ✅ Ejecución de test_suite.py
8. ✅ Tabla de resultados
9. ✅ Estructura de carpetas
10. ✅ Contenido de .env (sin API keys visibles)


## 📚 Referencias

- [LangChain Documentation](https://python.langchain.com/)
- [Groq API Documentation](https://console.groq.com/docs/quickstart)
- [Google Gemini API](https://ai.google.dev/gemini-api/docs)
- [FAISS Documentation](https://github.com/facebookresearch/faiss)
- [Sentence Transformers](https://www.sbert.net/)
- [RAG Pattern](https://python.langchain.com/docs/tutorials/rag/)

## ✅ Checklist de Entrega

Antes de entregar, verifica:

- [ ] ✅ Código funcional (`main.py` y `test_suite.py`)
- [ ] ✅ 100+ documentos en `data/` (no incluir en ZIP)
- [ ] ✅ Índice FAISS generado (no incluir en ZIP)
- [ ] ✅ `test_results.json` generado con 10+ casos
- [ ] ✅ README.md completo y actualizado
- [ ] ✅ requirements.txt con todas las dependencias
- [ ] ✅ `.env.example` con plantilla (sin API keys reales)
- [ ] ✅ Informe técnico en PDF
- [ ] ✅ Video pitch subido a YouTube (enlace en PDF)
- [ ] ✅ Todos los integrantes participan en el video
- [ ] ✅ Archivo ZIP: `practica3-grupo-XX-equipo-YY.zip`

**Contenido del ZIP:**
```
practica3-grupo-XX-equipo-YY.zip
├── main.py
├── test_suite.py
├── requirements.txt
├── .env.example
├── README.md
├── informe_tecnico.pdf  (con enlace al video)
└── test_results.json
```


## 👥 Equipo

- Jessica Paola Vega
- Juan David Cortés
- Jonatan Estiven Sánchez
- Josue Duque

**Grupo:** [1]  
**Equipo:** [9]

## 📄 Licencia

Este proyecto es parte de un trabajo académico para la Universidad Nacional de Colombia.

---

**Profesor:** Jaime Alberto Guzmán Luna  
**Curso:** Procesamiento del Lenguaje Natural (3011176)