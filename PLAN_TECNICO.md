# PROYECTO: CHATBOT RAG PARA FILOSOFIA.MX (GCP + GEMINI)

## STACK MLOps COMPLETO + AGENTIC LOOP

---

## 📋 Tabla de Contenidos

### 🎯 Información General
- [Objetivo](#-objetivo)
- [Estimado de Costo Mensual](#-estimado-de-costo-mensual)

### 🏗️ Arquitectura e Infraestructura
- [Arquitectura (Stack MLOps Completo)](#️-arquitectura-stack-mlops-completo)
  - [Procesamiento de Updates](#procesamiento-de-updates-wordpress--pubsub--cloud-run)
  - [Notas Clave](#notas-clave)

### 🤖 Agent Loop (Query Rewriting)
- [Agentic Layer](#-agentic-layer)
- [Agent Loop: Query Rewriting](#-agent-loop-query-rewriting)
  - [Objetivo](#objetivo)
  - [Condiciones de Activación](#condiciones-de-activación-heurísticas-pre-retrieval)
  - [Costo Real por Request](#costo-real-por-request-con-query-rewriting)
  - [Configuración](#configuración-relacionada-cfg)
- [Estrategias de Rewriting](#-estrategias-de-rewriting-por-tipo-de-pregunta)
- [Flujo del Agent Loop](#-flujo-del-agent-loop-plan--act--observe--decide)
- [Evaluación del Agent Loop](#-evaluación-del-agent-loop)
- [Integración en Pipeline RAG](#-integración-en-el-pipeline-rag-principal)

### 💻 Componentes Técnicos
- [Control de Consumo (Guardrails)](#-control-de-consumo-guardrails)
- [Modelos de IA](#-modelos-de-ia)
  - [Generación (LLM)](#generación-llm)
  - [Embeddings](#embeddings)
  - [Rate Limiting Vertex AI](#rate-limiting-vertex-ai)
- [Chunking Strategy](#-chunking-strategy)
- [Base Vectorial (Qdrant Cloud)](#️-base-vectorial-qdrant-cloud)
  - [Colección Principal](#colección-principal-filosofia_chunks)
  - [Qdrant Point IDs (UUID v5)](#qdrant-point-ids-uuid-v5)
  - [Estrategia de Actualización](#estrategia-de-actualización)
- [Cache + Contadores (Upstash Redis)](#-cache--contadores-upstash-redis)
  - [Schemas de Cache](#schemas-de-cache)
  - [Config Management](#config-management)
  - [Locks Distribuidos](#locks-distribuidos)
- [Cloud Storage](#️-cloud-storage)
- [BigQuery (Analytics y Feedback)](#-bigquery-analytics-y-feedback)
  - [Configuración del Sink](#configuración-del-sink)
  - [Consultas Útiles](#consultas-útiles-para-agent-loop)

### 📊 Observabilidad y Monitoreo
- [Observabilidad (OpenTelemetry + Cloud Trace)](#-observabilidad-opentelemetry--cloud-trace)
  - [OpenTelemetry Trace Format](#opentelemetry-trace-format-en-cloud-logging)
  - [Spans Instrumentados](#spans-instrumentados-con-agent-loop)
- [SLOs y Thresholds](#-slos-y-thresholds)
  - [Latencia](#latencia)
  - [Error Rate](#error-rate)
  - [Cache Performance](#cache-performance)
  - [Quality Metrics](#quality-metrics)
  - [Agent Loop](#agent-loop)
- [Métricas de Calidad del RAG (MLOps)](#-métricas-de-calidad-del-rag-mlops)
  - [Retrieval](#retrieval)
  - [Generación](#generación)
  - [Golden Dataset Schema](#golden-dataset-schema)
  - [Método de Evaluación: LLM-as-judge](#método-de-evaluación-llm-as-judge)

### 🔧 Configuración y Gestión
- [Versionado de Prompts](#-versionado-de-prompts)
- [Logging Estructurado](#-logging-estructurado)
- [Sistema de Feedback Loop](#-sistema-de-feedback-loop)
- [Data Drift Detection](#-data-drift-detection)

### 🚀 Deployment y Operaciones
- [CI/CD Pipeline](#-cicd-pipeline)
  - [GitHub Actions](#github-actions)
  - [Versionado de Imágenes](#versionado-de-imágenes)
- [Procesamiento de Updates (WordPress → Pub/Sub)](#-procesamiento-de-updates-wordpress--pubsub)
  - [WordPress Hook](#wordpress-hook-save_post)
  - [Pub/Sub Configuration](#pubsub-configuration)
  - [Cloud Run Consumer](#cloud-run-consumer)

### 🔐 Seguridad
- [Seguridad y Autenticación](#-seguridad-y-autenticación)
  - [API Gateway v2](#api-gateway-v2)
  - [Cloud Run](#cloud-run)
  - [Secret Manager](#secret-manager)
  - [Rate Limiting Server-Side](#rate-limiting-server-side)

### 🎨 Implementación RAG
- [Estrategia RAG (Optimizada)](#-estrategia-rag-optimizada)
- [Flujo Completo del Sistema](#-flujo-completo-del-sistema)
  - [Chat](#chat)
  - [Updates de Artículos](#updates-de-artículos)
  - [Jobs](#jobs)

### 📅 Planificación
- [Plan de Ejecución](#-plan-de-ejecución)
  - [Fase 1: Setup y Fundación (Día 1-2)](#fase-1-setup-y-fundación-día-1-2)
  - [Fase 2: Data Pipeline (Día 3)](#fase-2-data-pipeline-día-3)
  - [Fase 3: Motor RAG + Agent Loop + MLOps (Día 4-5)](#fase-3-motor-rag--agent-loop--mlops-día-4-5)
  - [Fase 4: Frontend + Integración (Día 6)](#fase-4-frontend--integración-día-6)
  - [Fase 5: CI/CD + Operación (Día 7)](#fase-5-cicd--operación-día-7)
- [Cuotas y Límites](#-cuotas-y-límites)
- [Próximos Pasos Post-Lanzamiento](#-próximos-pasos-post-lanzamiento)
  - [Semana 1-2](#semana-1-2)
  - [Mes 1](#mes-1)
  - [Ongoing](#ongoing)

### 📚 Referencias
- [Recursos Adicionales](#-recursos-adicionales)

---

## 📋 OBJETIVO

Crear un chatbot RAG (Retrieval Augmented Generation) para responder preguntas de filosofía usando ~4,000 artículos de WordPress (www.filosofia.mx) como base de conocimiento.

### Prioridades

- ✅ Costo mínimo posible
- ✅ Arquitectura MLOps profesional
- ✅ Observabilidad completa
- ✅ Mejora continua basada en métricas
- ✅ Buena experiencia de usuario
- ✅ Comportamiento Agentic explícito y medible

**SITIO:** filosofia.mx

---

## 💰 Estimado de Costo Mensual

### Costo Recurrente Mensual

**$25 - $65 USD/mes**

- **Escenario optimista** (free tiers): ~$25/mes
- **Escenario realista**: ~$45/mes  
- **Escenario conservador**: ~$65/mes

### Costo Inicial (One-Time)

**~$40 USD** (embeddings iniciales del corpus completo)

### Desglose por Componente

| Componente | Costo Mensual | Notas |
|------------|---------------|-------|
| **Vertex AI (LLM)** | $15-35 | Depende del tráfico y uso de agent loop |
| **Vertex AI (Embeddings)** | $5-10 | Batch processing + agent loop queries |
| **Qdrant Cloud** | $0-10 | Free tier hasta 1GB |
| **Upstash Redis** | $0 | Free tier 10K commands/day |
| **Cloud Run** | $0-5 | Free tier + mínimo uso |
| **BigQuery** | $0-5 | Free tier 1TB queries/mes |
| **Cloud Storage** | $0-2 | Mínimo almacenamiento |
| **Cloud Logging** | $0-3 | Free tier 50GB/mes |
| **Pub/Sub** | $0 | Free tier suficiente |
| **API Gateway** | $0 | Free tier 2M calls/mes |

### Consideraciones de Costo

- **Agent Loop Impact:** Cuando activo (~20-40% requests), incrementa costos ~3x por request afectado
- **Cache Strategy:** Hit ratio del 30-50% reduce significativamente costos de LLM
- **Tráfico estimado:** 5,000-10,000 requests/mes para estos estimados
- **Escalamiento:** Costos aumentan linealmente con tráfico después de free tiers

**Nota:** Estimaciones conservadoras. El uso de free tiers y caché efectivo puede mantener costos en el rango bajo.

---

## 🏗️ ARQUITECTURA (STACK MLOps COMPLETO)

```
Usuario (navegador)
  ↓
Plugin WordPress (UI: chatbot + feedback)
  ↓ HTTPS
API Gateway v2 (GCP)
  ↓
Cloud Run (Motor RAG + Agent Controller)
  ↓
┌──────────────────┬──────────────────┬──────────────────┬──────────────────┐
│ Qdrant Cloud     │ Upstash Redis    │ Vertex AI        │ BigQuery         │
│ Vectores +       │ Cache +          │ Gemini Pro/Flash │ Analytics via    │
│ Metadata         │ Contadores       │ + Embeddings API │ Cloud Logging    │
└──────────────────┴──────────────────┴──────────────────┴──────────────────┘
  ↓
┌──────────────────┬──────────────────┬──────────────────┐
│ Cloud Monitoring │ Cloud Trace      │ Cloud Storage    │
│ Métricas custom  │ Distributed      │ Prompts +        │
│ Dashboards       │ tracing          │ Configs +        │
│ Alertas + SLOs   │ (OTLP/exporter)  │ Exports          │
└──────────────────┴──────────────────┴──────────────────┘
```

### Procesamiento de Updates (WordPress → Pub/Sub → Cloud Run)

```
WordPress
  ↓ publish message
Pub/Sub topic: filosofia-article-updated
  ↓ subscription (con retry policy + DLT)
Cloud Run consumer (procesa embeddings y actualiza Qdrant)
```

### Notas Clave

- **API Gateway:** API Gateway v2 para mejor integración con Cloud Run
- **BigQuery:** Logs exportados desde Cloud Logging a tabla específica `filosofia_rag_logs`
- **Webhooks:** Asíncronos vía Pub/Sub (devuelven 202)
- **Locks:** Lock con token + heartbeat para evitar races
- **Deduplicación:** event_id único para idempotencia en Pub/Sub
- **Config keys:** Estándar único `cfg:*` y validación con Pydantic
- **Secretos:** IAM nativo de Cloud Run (no service-account keys)
- **Qdrant Point IDs:** UUID v5 determinístico basado en `post_id:chunk_index`
- **OpenTelemetry:** Formato GCP para correlación `logging.googleapis.com/trace`
- **CI/CD:** GitHub Actions con versionado semántico multi-tag
- **AGENT LOOP:** Query Rewriting con heurísticas pre-retrieval, paralelización y merge

---

## 🤖 AGENTIC LAYER

El sistema incluye un **Agent Loop mínimo y determinístico**, sin frameworks, diseñado para mejorar retrieval y demostrar razonamiento operacional.

### Agent Loop Implementado

**Loop de "Query Rewriting"** - Mejora retrieval con overhead controlado

Este loop se ejecuta SOLO cuando heurísticas pre-retrieval lo indican.

---

## 🔄 AGENT LOOP: QUERY REWRITING

### Objetivo

Mejorar recall y precisión del retrieval cuando la query original es ambigua, larga, comparativa o produce resultados potencialmente débiles.

### Condiciones de Activación (Heurísticas Pre-Retrieval)

Se decide **ANTES** del primer retrieval usando señales heurísticas:

```python
def should_rewrite_query_heuristic(question: str, classification: str) -> bool:
    """
    Decide ANTES del primer retrieval si aplicar query rewriting.
    
    Ventajas:
    - No requiere retrieval inicial
    - Más eficiente (evita retrieval doble innecesario)
    - Costo predecible
    """
    
    # Señal 1: Pregunta muy larga (probablemente compleja)
    if len(question) > 200:
        return True
    
    # Señal 2: Pregunta comparativa (requiere múltiples conceptos)
    comparative_keywords = [
        "diferencia", "comparar", "comparación", 
        " vs ", "versus", "distinto", "similar",
        "contraste", "relación entre"
    ]
    if any(kw in question.lower() for kw in comparative_keywords):
        return True
    
    # Señal 3: Sintaxis compleja (múltiples cláusulas)
    if question.count(",") > 3 or question.count(" y ") > 3:
        return True
    
    # Señal 4: Clasificación profunda CON múltiples entidades filosóficas
    if classification == "profunda":
        entities = extract_philosophical_entities(question)
        if len(entities) > 1:
            return True
    
    return False
```

**Activación:**
- ❌ NO requiere retrieval inicial
- ✅ Decisión basada solo en características de la pregunta
- ✅ Evita overhead cuando no es necesario

### Costo Real por Request con Query Rewriting

```yaml
CUANDO SE ACTIVA (heurística positiva):
  
  LLM Calls:
    - 1x Gemini Flash (query rewriting): ~150-300 tokens
      Costo: ~$0.000075 - $0.00015
    - 1x Gemini Pro/Flash (generación final): ~2000-3000 tokens
      Costo: ~$0.0015 - $0.005
  
  Embeddings API:
    - 1x embedding query original: ~50-200 tokens
    - 2x embeddings queries reescritas: ~100-400 tokens total
      Total: 3x llamadas a text-embedding-004
      Costo: ~$0.000003 - $0.000012
  
  Qdrant Vector Searches:
    - 3x searches (original + 2 rewrites)
      Overhead: despreciable (Qdrant es rápido)
  
  LATENCIA AGREGADA:
    - Rewriting (Flash): ~500-800ms
    - Embeddings paralelos: ~300-500ms (batch)
    - Searches paralelos: ~200-400ms (concurrent)
    Total overhead: ~1000-1700ms adicionales
  
  COSTO TOTAL POR REQUEST:
    Con rewriting: ~3x costo de request normal
    Sin rewriting: baseline
    
  TRADE-OFF:
    - Costo: +200%
    - Latencia: +1-1.7s
    - Beneficio: +15-30% mejora en retrieval quality (estimado)
```

### Configuración Relacionada (cfg:*)

```yaml
cfg:agent_query_rewrite_enabled: true
cfg:query_rewrite_max_variants: 2
cfg:query_rewrite_similarity_gain_min: 0.05  # Ganancia mínima para aplicar
cfg:query_rewrite_model: "gemini-1.5-flash"
cfg:query_rewrite_temperature: 0.0  # Determinístico
cfg:query_rewrite_max_tokens: 300
cfg:query_rewrite_include_original: true  # Incluir query original en evaluación
```

---

## 🎯 ESTRATEGIAS DE REWRITING POR TIPO DE PREGUNTA

### 1. Estrategia Comparativa

Para preguntas "X vs Y" → genera query independiente por concepto

**Ejemplo:**
```
Input: "¿Cuál es la diferencia entre el ser y el ente en Heidegger?"
Output: [
    "el ser Heidegger ontología fundamental",
    "el ente Heidegger diferencia ontológica"
]
```

### 2. Estrategia Simplificación

Para preguntas largas/complejas → extrae términos clave

**Ejemplo:**
```
Input: "Me gustaría entender cómo el concepto de libertad en Sartre 
        se relaciona con la angustia existencial y la responsabilidad"
Output: [
    "libertad Sartre existencialismo concepto",
    "angustia existencial responsabilidad Sartre"
]
```

### 3. Estrategia Expansión

Para preguntas muy cortas/vagas → expande con contexto

**Ejemplo:**
```
Input: "¿Qué es la dialéctica?"
Output: [
    "dialéctica concepto filosofía explicación",
    "dialéctica Hegel Marx método filosófico"
]
```

---

## 📊 FLUJO DEL AGENT LOOP (PLAN → ACT → OBSERVE → DECIDE)

### Implementación Completa

```python
async def agent_query_rewrite_loop(
    question: str,
    classification: str,
    config: dict
) -> dict:
    """
    Agent loop completo con query rewriting.
    
    Flujo:
    1. PLAN: Decidir si reescribir (heurística pre-retrieval)
    2. ACT: Generar rewrites según estrategia
    3. OBSERVE: Evaluar todas las queries (paralelo)
    4. DECIDE: Seleccionar mejor o mergear
    5. FALLBACK: Manejar caso de fallo total
    """
    
    # STEP 1: PLAN
    should_rewrite = should_rewrite_query_heuristic(question, classification)
    
    if not should_rewrite:
        # Path rápido sin rewriting
        embedding = await get_embedding(question)
        chunks = await qdrant_search(embedding, top_k=10)
        return {
            "final_query": question,
            "chunks": chunks,
            "agent_decisions": {"rewrite_applied": False}
        }
    
    # STEP 2: ACT
    strategy = select_strategy(question, classification)
    rewritten_queries = await strategy(question)
    
    # STEP 3: OBSERVE (paralelo)
    all_queries = [question] + rewritten_queries
    embeddings = await get_embeddings_batch(all_queries)
    search_results = await asyncio.gather(*[
        qdrant_search(emb, top_k=10) for emb in embeddings
    ])
    
    # STEP 4: DECIDE
    if is_comparative:
        final_chunks = await merge_comparative_chunks(evaluated_queries)
        selected_query = "merged"
    else:
        best = max(evaluated_queries, key=lambda x: x["mean_similarity"])
        final_chunks = best["chunks"]
        selected_query = best["query"]
    
    # STEP 5: FALLBACK
    if best_similarity < threshold:
        return await handle_low_similarity_fallback(...)
    
    return {
        "final_query": selected_query,
        "chunks": final_chunks,
        "agent_decisions": {...}
    }
```

---

## 📈 EVALUACIÓN DEL AGENT LOOP

### Métricas Específicas

```yaml
agent_loop_metrics:
  # Activación
  - agent.query_rewrite.invocations_total
  - agent.query_rewrite.activation_rate
  - agent.query_rewrite.activation_by_heuristic
  
  # Performance
  - agent.query_rewrite.avg_similarity_gain
  - agent.query_rewrite.success_rate  # % que mejoran >= min_gain
  - agent.query_rewrite.avg_latency_ms
  
  # Retrieval quality impact
  - agent.query_rewrite.precision_improvement
  - agent.query_rewrite.recall_improvement
  - agent.query_rewrite.mrr_improvement
  
  # Estrategias
  - agent.query_rewrite.strategy_distribution
  - agent.query_rewrite.comparative_merge_rate
  - agent.query_rewrite.fallback_rate
```

### SLOs Específicos del Agent Loop

```yaml
agent_loop_slos:
  activation_rate:
    min: 0.20  # Al menos 20% activation
    max: 0.50  # Máximo 50% activation
    
  success_rate:
    target: 0.70  # 70% de rewrites mejoran similarity
    minimum: 0.60
    
  avg_similarity_gain:
    minimum: 0.05  # Ganancia mínima promedio
    
  avg_latency_overhead:
    maximum: 2000ms  # Overhead máximo aceptable
```

---

## 🔧 INTEGRACIÓN EN EL PIPELINE RAG PRINCIPAL

### Pipeline con Agent Loop

```
1. Usuario pregunta
2. Clasificación (cacheada)
3. Cache lookup
4. ──► AGENT LOOP: Query Rewriting
   ├─ PLAN: ¿Necesita rewriting?
   ├─ ACT: Generar rewrites (estrategia seleccionada)
   ├─ OBSERVE: Evaluar queries (paralelo)
   ├─ DECIDE: Seleccionar mejor / merge
   └─ FALLBACK: Manejar fallo si aplica
5. Chunks seleccionados (del agent loop)
6. Generación (Pro/Flash según tipo)
7. Evaluación (LLM-as-judge opcional)
8. Cache + logging + feedback
```

---

## 💰 CONTROL DE CONSUMO (GUARDRAILS)

### Métricas Trackeadas

- Número de requests
- Tokens estimados (input + output)
- Proyección mensual estimada
- Agent loop invocations (con overhead asociado)

### Umbrales Configurables

```yaml
cfg:alert_threshold: [valor]
cfg:critical_threshold: [valor]
cfg:models_enabled: true
cfg:agent_query_rewrite_enabled: true
```

### Acción al Alcanzar Umbral Crítico

1. Desactivar modelos (`cfg:models_enabled=false`)
2. Desactivar agent loop (`cfg:agent_query_rewrite_enabled=false`)
3. Modo limitado:
   - Solo respuestas cacheadas
   - Modo "búsqueda sin LLM": devuelve chunks + citaciones
   - Mensaje visible: "Modo limitado por presupuesto"

---

## 🤖 MODELOS DE IA

### Generación (LLM)

- **Modelo principal:** Gemini 1.5 Pro
- **Modelo económico:** Gemini 1.5 Flash
- **Agent loop:** Gemini 1.5 Flash (query rewriting)

### Embeddings

- **API:** Text Embeddings API (text-embedding-004)
- **Dimensión:** 768

### Contexto

- **Default:** 3 chunks (~1,500 palabras)
- **Máximo:** 5 chunks cuando amerite

### Rate Limiting Vertex AI

```python
@retry(
    retry=retry_if_exception_type((ResourceExhausted, TooManyRequests)),
    wait=wait_exponential(multiplier=1, min=4, max=60),
    stop=stop_after_attempt(5)
)
async def get_embeddings_batch(
    texts: list[str],
    batch_size: int = 250
) -> list[list[float]]:
    """Procesa embeddings en batches respetando rate limits."""
    # Implementación...
```

**Configuración recomendada:**
- `batch_size`: 250
- `max_retries`: 5
- `backoff`: exponencial (min=4s, max=60s)

---

## 📝 CHUNKING STRATEGY

### Configuración

```python
CHUNK_CONFIG = {
    "target_tokens": 512,        # ~2000 caracteres
    "overlap_tokens": 50,         # Mantiene contexto
    "separators": [
        "\n\n\n",  # Secciones principales
        "\n\n",    # Párrafos
        "\n",      # Líneas
        ". ",      # Oraciones
        " "        # Palabras
    ],
    "min_chunk_tokens": 100,      # Descartar muy pequeños
    "max_chunk_tokens": 1024,     # Split forzado
    "preserve_metadata": True
}
```

### Estrategia de Overlap

- Mantiene contexto entre chunks contiguos
- 50 tokens ≈ 200 caracteres de overlap
- Mejora recall en búsquedas que caen en límites de chunks

---

## 🗄️ BASE VECTORIAL (QDRANT CLOUD)

### Colección Principal: `filosofia_chunks`

**Configuración:**
- Vector: embedding (768 dims)
- Distancia: Cosine
- HNSW: `m=16`, `ef_construct=100`

### Payload/Metadata por Chunk

```yaml
chunk_id: string (UUID v5 determinístico)
post_id: int
titulo: string
url: string
fecha: datetime
categoria: string
tags: list[string]
filosofo_principal: string | null
filosofos_mencionados: list[string]
escuela: string
texto_chunk: string
chunk_index: int
token_count: int
version: string
updated_at: datetime
```

### Qdrant Point IDs (UUID v5)

```python
import uuid

FILOSOFIA_NAMESPACE = uuid.UUID('12345678-1234-5678-1234-567812345678')

def generate_point_id(post_id: int, chunk_index: int) -> str:
    """
    Genera UUID v5 determinístico basado en post_id y chunk_index.
    
    Ventajas:
    - Determinístico: mismo input → mismo UUID
    - Compatible: formato válido para Qdrant
    - Sin colisiones: namespace + name único
    """
    name = f"{post_id}:{chunk_index}"
    point_id = uuid.uuid5(FILOSOFIA_NAMESPACE, name)
    return str(point_id)
```

### Estrategia de Actualización

**IDs determinísticos + upsert directo**

- `point_id` = UUID v5 de `{post_id}:{chunk_index}`
- Upsert reemplaza el punto sin delete masivo
- `version`/`updated_at` se actualiza

---

## 💾 CACHE + CONTADORES (UPSTASH REDIS)

### Uso

- Cache de respuestas (TTL)
- Cache de clasificación (TTL)
- Contadores (requests/tokens)
- Config dinámica (`cfg:*`)
- Locks/idempotencia
- Deduplicación de eventos Pub/Sub (`event_id`)

### Schemas de Cache

```yaml
# Respuestas
cache:resp:<hash>:
  value: {response, citations, timestamp}
  TTL: 7 días

# Clasificación
cache:cls:<hash>:
  value: "saludo" | "generica" | "profunda"
  TTL: 24 horas

# Deduplicación
processed:<event_id>:
  value: "1"
  TTL: 7 días
```

### Config Management

```python
class ConfigManager:
    PREFIX = "cfg:"
    
    async def get(self, key: str) -> Any:
        """Lee cfg:model_primary sin duplicar prefijo"""
        full_key = f"{self.PREFIX}{key}"
        return await self.redis.get(full_key)
    
    async def set(self, key: str, value: Any):
        """Escribe cfg:temperature"""
        full_key = f"{self.PREFIX}{key}"
        await self.redis.set(full_key, value)
```

### Locks Distribuidos

```python
class DistributedLock:
    """Lock con token + heartbeat para evitar races."""
    
    async def acquire(self) -> bool:
        """Intenta adquirir lock con SET NX + TTL"""
        acquired = await self.redis.set(
            self.key, self.token, nx=True, ex=self.ttl
        )
        if acquired:
            self._heartbeat_task = asyncio.create_task(self._heartbeat())
        return acquired
    
    async def release(self):
        """Libera lock verificando ownership (Lua script)"""
        # Solo borra si el token coincide
```

---

## ☁️ CLOUD STORAGE

### Buckets

```
filosofia-mx-terraform-state/
  ├── backend Terraform
  └── versionado habilitado

filosofia-mx-data/
  ├── wordpress_exports/
  ├── prompts/ (versionado)
  ├── configs/
  ├── backups/ (Qdrant)
  └── evaluation/ (golden datasets)
```

**Cloud Storage NO se usa para:**
- Embeddings (Qdrant)
- Cache (Redis)
- Logs (Cloud Logging)

---

## 📊 BIGQUERY (ANALYTICS Y FEEDBACK)

### Estrategia de Ingesta

```
Cloud Run → Cloud Logging (structured logs en jsonPayload)
           → BigQuery Sink (export de LogEntry)
```

### Configuración del Sink

```bash
# Crear dataset
bq mk --dataset --location=us-central1 filosofia_mx:logs

# Crear sink
gcloud logging sinks create filosofia-rag-logs \
  bigquery.googleapis.com/projects/PROJECT_ID/datasets/logs \
  --log-filter='
    resource.type="cloud_run_revision"
    AND labels.service_name="filosofia-rag"
    AND jsonPayload.app="filosofia-rag"
  '
```

**Tabla resultante:** `PROJECT_ID.logs.filosofia_rag_logs`

### Campos Importantes

- `timestamp`: DateTime del log
- `severity`: INFO, WARNING, ERROR, CRITICAL
- `jsonPayload.*`: Todos los campos custom (incluye agent loop metrics)
- `resource.labels.*`: Metadata del recurso
- `trace`: Linked trace ID

### Ejemplo de Consulta

```sql
SELECT
  TIMESTAMP(timestamp) as ts,
  severity,
  jsonPayload.trace_id,
  jsonPayload.question_type,
  jsonPayload.latency_ms,
  jsonPayload.agent_rewrite_applied,
  jsonPayload.agent_similarity_gain
FROM `PROJECT_ID.logs.filosofia_rag_logs`
WHERE 
  DATE(timestamp) = CURRENT_DATE()
  AND jsonPayload.app = "filosofia-rag"
ORDER BY timestamp DESC
LIMIT 100;
```

### Consultas Útiles para Agent Loop

```sql
-- Agent loop performance
SELECT
  DATE(timestamp) as date,
  COUNTIF(jsonPayload.agent_rewrite_applied = true) as rewrites_applied,
  COUNT(*) as total_requests,
  SAFE_DIVIDE(
    COUNTIF(jsonPayload.agent_rewrite_applied = true),
    COUNT(*)
  ) as rewrite_rate,
  AVG(CAST(jsonPayload.agent_similarity_gain AS FLOAT64)) as avg_similarity_gain
FROM `PROJECT_ID.logs.filosofia_rag_logs`
WHERE 
  jsonPayload.app = "filosofia-rag"
  AND jsonPayload.event = "agent_query_rewrite_completed"
GROUP BY date
ORDER BY date DESC;
```

---

## 🔍 OBSERVABILIDAD (OPENTELEMETRY + CLOUD TRACE)

### Requisitos Explícitos

1. **Propagación de contexto:** W3C traceparent + tracestate
2. **Exporter:** OTLP configurado → Cloud Trace
3. **Sampling:** 10% normal + 100% errores
4. **Correlación logs ↔ trace:** Formato GCP específico

### OpenTelemetry Trace Format en Cloud Logging

```python
def add_trace_context(logger, method_name, event_dict):
    """Agrega trace_id en formato GCP a los logs."""
    span = trace.get_current_span()
    if span and span.is_recording():
        span_context = span.get_span_context()
        
        # Formato requerido por GCP
        trace_id = f"projects/{PROJECT_ID}/traces/{format(span_context.trace_id, '032x')}"
        span_id = format(span_context.span_id, '016x')
        
        # Campos especiales de GCP
        event_dict["logging.googleapis.com/trace"] = trace_id
        event_dict["logging.googleapis.com/spanId"] = span_id
        
        # También en jsonPayload para queries
        event_dict["trace_id"] = format(span_context.trace_id, '032x')
        event_dict["span_id"] = span_id
    
    return event_dict
```

### Spans Instrumentados (con Agent Loop)

```
root_span: POST /chat
├── classify_question (300ms)
├── cache_lookup_response (40ms)
├── agent_query_rewrite_loop (1500ms)
│   ├── should_rewrite_heuristic (10ms)
│   ├── generate_rewrites (600ms)
│   │   └── vertex_ai_flash_api (580ms)
│   ├── evaluate_queries_parallel (800ms)
│   │   ├── get_embeddings_batch (300ms)
│   │   └── qdrant_search_parallel (400ms)
│   └── select_best_or_merge (90ms)
├── llm_inference (3000ms)
├── format_response (100ms)
└── cache_store (60ms)

Total: ~5000ms (vs ~3500ms sin agent loop)
```

---

## 🎯 SLOs Y THRESHOLDS

### Latencia

```yaml
latency:
  p50: 800ms   # Con agent overhead
  p95: 3000ms
  p99: 6000ms
  
  # Alertas
  alert_p95_threshold: 4000ms
  alert_p99_threshold: 10000ms
```

### Error Rate

```yaml
error_rate:
  target: 0.01  # < 1%
  threshold: 0.02  # Alertar si > 2%
  window: 5m
```

### Cache Performance

```yaml
cache:
  hit_ratio:
    minimum: 0.30
    target: 0.50
    alert_threshold: 0.20
  window: 1h
```

### Quality Metrics

```yaml
quality:
  mean_topk_similarity:
    minimum: 0.70
    alert_threshold: 0.65
  
  satisfaction_rate:
    target: 0.80
    minimum: 0.70
    alert_threshold: 0.65
  
  window: 24h
```

### Agent Loop

```yaml
agent_loop:
  activation_rate:
    min: 0.20  # Al menos 20%
    max: 0.50  # Máximo 50%
    alert_threshold_low: 0.10
    alert_threshold_high: 0.60
  
  success_rate:
    target: 0.70  # 70% mejoran similarity
    minimum: 0.60
    alert_threshold: 0.50
  
  avg_similarity_gain:
    minimum: 0.05
    alert_threshold: 0.03
  
  avg_latency_overhead:
    maximum: 2000ms
    alert_threshold: 3000ms
  
  window: 24h
```

---

## 📏 MÉTRICAS DE CALIDAD DEL RAG (MLOPS)

### Retrieval

- Precision@k
- Recall@k
- MRR (Mean Reciprocal Rank)
- NDCG (Normalized Discounted Cumulative Gain)

### Generación

- Coherencia
- Fidelidad / groundedness
- Relevancia

### Agent Loop

- Activation rate
- Success rate (mejora en similarity)
- Avg similarity gain
- Latency overhead
- Impact on retrieval metrics

### Golden Dataset Schema

```json
{
  "version": "1.0",
  "total_questions": 75,
  "questions": [
    {
      "id": "q001",
      "question": "¿Qué es el imperativo categórico de Kant?",
      "expected_chunks": [...],
      "expected_philosophers": ["Immanuel Kant"],
      "difficulty": "media",
      "category": "ética",
      "expects_agent_rewrite": false
    },
    {
      "id": "q002",
      "question": "¿Cuál es la diferencia entre la alegoría de la caverna y el velo de ignorancia?",
      "expected_chunks": [...],
      "difficulty": "alta",
      "category": "comparativa",
      "expects_agent_rewrite": true
    }
  ]
}
```

### Método de Evaluación: LLM-as-judge

```python
class LLMEvaluator:
    """Evalúa calidad usando Gemini Flash."""
    
    async def evaluate_response(
        self,
        question: str,
        response: str,
        chunks: List[str]
    ) -> Dict[str, float]:
        """
        Returns:
            {
                "fidelity": 0.0-1.0,
                "relevance": 0.0-1.0,
                "coherence": 0.0-1.0,
                "completeness": 0.0-1.0
            }
        """
```

---

## 📦 VERSIONADO DE PROMPTS

### Estructura en Cloud Storage

```
prompts/
  ├── v1.0_base.txt
  ├── v1.1_improved_citations.txt
  ├── v1.2_mejor_coherencia.txt
  ├── v1.3_optimizado_tokens.txt
  └── metadata.json
```

### Prompt Manager

```python
class PromptManager:
    """Gestiona carga y caché de prompts desde GCS."""
    
    async def get_prompt(
        self,
        version: Optional[str] = None,
        force_reload: bool = False
    ) -> str:
        """
        Carga prompt con caché en memoria (TTL 1h).
        Fallback a versión default si falla.
        """
```

**Config activa:** `cfg:prompt_version => "v1.3"`

---

## 📝 LOGGING ESTRUCTURADO

### Formato jsonPayload (con Agent Fields)

```json
{
  "app": "filosofia-rag",
  "severity": "INFO",
  "trace_id": "abc123def456",
  "span_id": "span789",
  
  "user_id": "hash_usuario_xyz",
  "question_type": "profunda",
  "question_hash": "def456",
  
  "model_used": "gemini-1.5-pro",
  "cache_hit": false,
  
  "latency_ms": 5523,
  "latency_breakdown": {
    "classify": 300,
    "cache_lookup": 40,
    "agent_loop": 1500,
    "llm_inference": 3000,
    "format": 100
  },
  
  "tokens_input": 2340,
  "tokens_output": 456,
  
  "embedding_model": "text-embedding-004",
  "corpus_version": "20250115_093000",
  
  "top1_similarity": 0.92,
  "mean_topk_similarity": 0.836,
  
  "prompt_version": "v1.3",
  
  "agent_rewrite_applied": true,
  "agent_strategy": "comparative",
  "agent_queries_evaluated": 3,
  "agent_similarity_gain": 0.12
}
```

### Severity Levels

```python
class LogSeverity(str, Enum):
    DEBUG = "DEBUG"
    INFO = "INFO"
    WARNING = "WARNING"
    ERROR = "ERROR"
    CRITICAL = "CRITICAL"
```

---

## 💬 SISTEMA DE FEEDBACK LOOP

### UI Plugin

```
[👍 Útil] [👎 No útil] + categorías + texto opcional
```

### Backend: POST /feedback

- Log a Cloud Logging (jsonPayload) → BigQuery
- Incrementa contadores en Redis

### Feedback Anti-Spam

```python
class FeedbackRateLimiter:
    """Rate limiting para prevenir spam."""
    
    max_feedback_per_hour: int = 10
    max_feedback_per_day: int = 50
    
    async def check_rate_limit(self, user_hash: str) -> tuple[bool, str]:
        """Verifica límites por usuario."""
```

---

## 📉 DATA DRIFT DETECTION

### Inputs Necesarios (en logs)

- `embedding_model`
- `embedding_version`
- `corpus_version`
- `top1_similarity` / `mean_topk_similarity`

### Drift Baseline

```python
@dataclass
class DriftBaseline:
    mean_topk_similarity: float
    std_topk_similarity: float
    embedding_model: str
    corpus_version: str
    start_date: str
    end_date: str
    sample_count: int
    window_days: int = 7
    update_threshold: float = 0.05
```

### Drift Config

```yaml
drift_config:
  baseline:
    window_days: 7
    min_samples: 1000
    update_threshold: 0.05
  
  detection:
    similarity_drop_threshold: 0.10  # 10% drop
    alert_threshold: 0.15  # Alertar si > 15%
    consecutive_days: 2
  
  monitoring:
    check_frequency: "daily"
    lookback_days: 7
```

---

## 🚀 CI/CD PIPELINE

### GitHub Actions

```yaml
name: Build and Deploy

on:
  push:
    branches: [main, develop]
    tags: ['v*.*.*']

jobs:
  lint-and-test:
    - ruff check
    - mypy src/
    - pytest --cov
  
  build-and-push:
    - Build Docker image
    - Push to Artifact Registry
    - Multi-tag: SHA + semver + branch + latest
  
  deploy-production:
    - Deploy con 10% tráfico
    - Monitor métricas (5min)
    - Gradual: 50% → 100%
    - Rollback automático si error_rate > threshold
```

### Versionado de Imágenes

```bash
# Push a main (commit a1b2c3d4):
# - filosofia-rag:a1b2c3d4
# - filosofia-rag:main

# Tag v1.2.3:
# - filosofia-rag:v1.2.3
# - filosofia-rag:a1b2c3d4
# - filosofia-rag:latest
```

---

## 🔄 PROCESAMIENTO DE UPDATES (WORDPRESS → PUB/SUB)

### WordPress Hook (save_post)

```php
public function on_post_updated($post_id, $post, $update) {
    // Filtros
    if ($post->post_type !== 'post') return;
    if (wp_is_post_autosave($post_id)) return;
    if (wp_is_post_revision($post_id)) return;
    if ($post->post_status !== 'publish') return;
    
    // Prevenir loops
    if (get_transient("filosofia_processing_$post_id")) return;
    set_transient("filosofia_processing_$post_id", true, 60);
    
    // Generar event_id único
    $event_id = $this->generate_event_id($post_id);
    
    // Publicar a Pub/Sub (non-blocking)
    wp_remote_post($this->pubsub_endpoint, [
        'body' => json_encode(['event_id' => $event_id, ...]),
        'blocking' => false
    ]);
}
```

### Pub/Sub Configuration

```yaml
topic: filosofia-article-updated
  message_retention: 7 días

subscription: filosofia-article-processor
  ack_deadline: 600s  # 10 minutos
  
  retry_policy:
    minimum_backoff: 10s
    maximum_backoff: 600s
  
  dead_letter_policy:
    max_delivery_attempts: 5
    dead_letter_topic: filosofia-article-failed
```

### Cloud Run Consumer

```python
@app.post("/process-article")
async def process_article(request: Request):
    """Procesa mensajes de Pub/Sub."""
    
    # Parse mensaje
    article_data = json.loads(base64.b64decode(message_data))
    event_id = article_data["event_id"]
    
    # Deduplicación
    if await redis.exists(f"processed:{event_id}"):
        return {"status": "success"}
    
    # Lock distribuido
    lock = DistributedLock(redis, f"embed:{post_id}")
    if not await lock.acquire():
        return {"status": "retry"}
    
    try:
        # Chunking + Embeddings + Qdrant upsert
        # ...
        
        # Marcar como procesado
        await redis.setex(f"processed:{event_id}", 604800, "1")
        await lock.release()
        
        return {"status": "success"}
    except Exception as e:
        await lock.release()
        raise  # Pub/Sub reintentará
```

---

## 🔐 SEGURIDAD Y AUTENTICACIÓN

### API Gateway v2

- API Key para identificación + rate limiting
- OpenAPI spec para validación

### Cloud Run

- **Ingress:** `internal-and-cloud-load-balancing`
- **Requiere:** IAM Invoker
- **Invoker:** API Gateway

### Secret Manager

```yaml
secrets:
  - qdrant-api-key
  - qdrant-url
  - upstash-redis-url
  - upstash-redis-token
  - wordpress-webhook-api-key
```

**NO usar:** service-account keys (usar IAM nativo)

### Service Account Permissions

```yaml
permissions:
  - Vertex AI (invocar modelos/embeddings)
  - Secret Manager Accessor
  - Pub/Sub Publisher/Subscriber
  - Logging/Monitoring/Trace
  - BigQuery Data Editor
  - Cloud Storage Object Viewer
```

### Rate Limiting Server-Side

```python
async def check_rate_limit(request: Request) -> bool:
    """Rate limit basado en user_hash + IP + User-Agent."""
    
    user_identifier = (
        request.client.host +
        request.headers.get("user-agent", "") +
        "filosofia-salt-2025"
    )
    user_hash = hashlib.sha256(user_identifier.encode()).hexdigest()[:16]
    
    # 100 requests/hora
    key = f"ratelimit:{today}:{hour}:{user_hash}"
    count = await redis.incr(key)
    
    return count <= 100
```

---

## 🎨 ESTRATEGIA RAG (OPTIMIZADA)

### Clasificador Rule-Based

```
saludo → Cache / Flash
generica → Embedding → Qdrant (1-2 chunks) → Flash
profunda → AGENT LOOP → Qdrant (3-5 chunks) → Pro
```

---

## 🔄 FLUJO COMPLETO DEL SISTEMA

### CHAT

1. Usuario pregunta
2. Plugin → API Gateway v2 → `/chat`
3. API Gateway: validación + rate limit
4. Cloud Run procesa:
   - Trace OTel (formato GCP)
   - Logs jsonPayload + severity
   - Clasificación (cache)
   - Cache lookup
   - **AGENT LOOP: Query Rewriting** (si heurística positiva)
   - LLM generación (prompt versionado)
   - Citaciones + respuesta
   - Cache store + contadores
5. UI: respuesta + feedback
6. Feedback → anti-spam → logs → BigQuery

### UPDATES DE ARTÍCULOS

1. WordPress: `save_post` (filtrado)
2. Endpoint `/enqueue-article-update` (202)
3. Pub/Sub: mensaje + `event_id`
4. Consumer: deduplicación
5. Lock distribuido
6. Embeddings + Qdrant (UUID v5)
7. Marcar procesado + release lock
8. Ack mensaje

### JOBS

- **Evaluación semanal** (con y sin agent loop)
- **Drift semanal** (baseline comparisons)
- **Reconciliación** (si aplica)

---

## 📅 PLAN DE EJECUCIÓN

### FASE 1: SETUP Y FUNDACIÓN (DÍA 1-2)

#### DÍA 1 – Setup Local + GCP + Servicios Externos

**Local:**
- gcloud, Terraform, Docker, Python 3.10+
- structlog, opentelemetry, pytest, ruff, mypy, tenacity

**GCP:**
- Habilitar APIs
- Service Accounts + IAM
- Artifact Registry

**Externos:**
- Qdrant Cloud (colección `filosofia_chunks`)
- Upstash Redis
- Secret Manager

**ENTREGABLE:**
- ✅ Entorno listo
- ✅ Secrets correctos
- ✅ Permisos configurados

#### DÍA 2 – Infraestructura Base (Terraform)

**Terraform:**
- `cloud_run.tf`
- `api_gateway.tf` (v2)
- `pubsub.tf` (retry + DLT)
- `storage.tf`
- `iam.tf`
- `logging.tf` (sink a BigQuery)
- `monitoring.tf` (SLOs)

**ENTREGABLE:**
- ✅ Infraestructura desplegada
- ✅ Cloud Run cerrado
- ✅ Pub/Sub funcionando
- ✅ Logs → BigQuery

### FASE 2: DATA PIPELINE (DÍA 3)

- Export WordPress (WP-CLI)
- Limpieza HTML
- Chunking (512 tokens, 50 overlap)
- Embeddings batch
- Upload Qdrant (UUID v5)
- Corpus version + baseline drift

**ENTREGABLE:**
- ✅ Chunks en Qdrant
- ✅ Búsqueda validada
- ✅ Baseline establecido

### FASE 3: MOTOR RAG + AGENT LOOP + MLOPS (DÍA 4-5)

#### DÍA 4

- FastAPI
- Config (Pydantic + ConfigManager)
- OTel (W3C + OTLP + formato GCP)
- Structured logs + severity
- PromptManager
- Cache + counters
- RAG pipeline básico
- **AGENT LOOP: Query Rewriting**
  - Heurísticas activación
  - Estrategias por tipo
  - Paralelización
  - Merge comparativas
  - Fallback
- Tests

#### DÍA 5

- Golden dataset (schema JSON)
- LLM-as-judge evaluator
- Evaluación semanal (con/sin agent)
- Drift detector
- Dashboards (incluye agent metrics)
- SLO monitoring

**ENTREGABLE:**
- ✅ Motor RAG + Agent Loop
- ✅ Observabilidad completa
- ✅ Evaluación continua
- ✅ Métricas agent trackeadas

### FASE 4: FRONTEND + INTEGRACIÓN (DÍA 6)

**Plugin WordPress:**
- UI chatbot + feedback
- Anti-spam
- Rate limit (server-side)
- Hooks `save_post` (filtrados)
- Deduplicación (`event_id`)

**ENTREGABLE:**
- ✅ Plugin funcionando
- ✅ Updates vía Pub/Sub
- ✅ Feedback operativo

### FASE 5: CI/CD + OPERACIÓN (DÍA 7)

**CI/CD (GitHub Actions):**
- Lint/test
- Build + push Artifact Registry
- Versionado multi-tag
- Deploy Cloud Run
- Gradual rollout (10% → 50% → 100%)
- Rollback automático

**Runbooks + Alerts:**
- Error rate, latency, cache hit
- Quality drop, drift
- Agent loop performance
- SLO violations

**ENTREGABLE:**
- ✅ Deploys seguros
- ✅ Rollback automático
- ✅ Alertas configuradas

---

## 📊 CUOTAS Y LÍMITES

- Vertex AI quotas
- Qdrant free tier
- Upstash: commands/day
- BigQuery: queries/volumen
- Pub/Sub: mensajes/segundo
- Cloud Run: concurrent requests

---

## 🔮 PRÓXIMOS PASOS POST-LANZAMIENTO

### Semana 1-2

- Monitorear latencia, error rate, cache hit
- Validar agent loop activation rate (20-40%)
- Ajustar heurísticas si necesario
- Validar Pub/Sub (sin duplicados)
- Iterar prompts
- Verificar SLOs

### Mes 1

- Baseline drift (2-4 semanas datos)
- A/B de prompts
- Evaluar impacto real agent loop
- Mejorar chunking si necesario
- Optimizar thresholds
- Análisis feedback

### Ongoing

- Evaluación semanal (con/sin agent)
- Drift semanal
- Mejoras basadas en feedback
- Iteración prompts
- Refinamiento SLOs
- Optimización estrategias rewriting

---

## 📚 RECURSOS ADICIONALES

### Documentación

- [Vertex AI API](https://cloud.google.com/vertex-ai/docs)
- [Qdrant Documentation](https://qdrant.tech/documentation/)
- [OpenTelemetry Python](https://opentelemetry.io/docs/instrumentation/python/)
- [Upstash Redis](https://docs.upstash.com/redis)

---

**Última actualización:** 2025-12-27  
**Versión:** 1.0
