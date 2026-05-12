# WhatsApp Chatbot (RAG System)

Este repositorio contiene la configuración de un chatbot avanzado para WhatsApp integrado con un sistema de Generación Aumentada por Recuperación (RAG). El bot gestiona consultas de clientes sobre servicios tecnológicos y soporte automatizado.

> [!IMPORTANT]
> Este flujo de trabajo requiere una instancia de n8n con soporte para nodos de LangChain y acceso a las APIs de Meta, Supabase, Groq y Cohere.

## Estructura del Sistema

El flujo de trabajo se divide en cuatro capas principales:
1.  **Ingesta de Mensajes:** Captura vía Webhook y validación de tipos de contenido (texto o documentos).
2.  **Gestión de Estado:** Control de modo (Bot/Humano) y registro de leads en la base de datos.
3.  **Procesamiento RAG:** Agrupación de mensajes, recuperación de contexto vectorial y generación de respuesta mediante LLM.
4.  **Automatizaciones Secundarias:** Reactivación automática de chats inactivos y sistema de alertas de error por correo electrónico.

> [!NOTE]
> El sistema incluye un "Buffer de Mensajes" que espera 3 segundos para agrupar múltiples envíos del usuario en una sola consulta al LLM, optimizando el consumo de tokens y mejorando la coherencia de la respuesta.

## Estructura del Flujo (Mermaid)

```mermaid
graph TD
    A[WhatsApp Webhook] --> B{Validar Tipo}
    B -- Texto/Doc --> C[Registrar Lead en DB]
    B -- No Soportado --> D[Mensaje de Error]
    
    C --> E{Estado: Humano?}
    E -- Sí --> F[Sin Acción / Modo Silencio]
    E -- No --> G[Buffer de Mensajes]
    
    G --> H[Espera 3s] --> I{¿Es el último ID?}
    I -- Sí --> J[Recuperar Contexto RAG]
    J --> K[Agente LLM Groq]
    
    K --> L{Switch de Salida}
    L -- Transferir --> M[Activar Modo Humano]
    L -- Sin Información --> N[Notificar a Soporte]
    L -- Respuesta --> O[Dividir Mensajes]
    
    O --> P[Enviar por WhatsApp]

