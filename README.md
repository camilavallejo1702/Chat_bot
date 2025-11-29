{
  "name": "My workflow DEF",
  "nodes": [
    {
      "parameters": {
        "mode": "insert",
        "qdrantCollection": {
          "__rl": true,
          "value": "mis_vectores",
          "mode": "id"
        },
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.vectorStoreQdrant",
      "typeVersion": 1.3,
      "position": [
        -224,
        -576
      ],
      "id": "38298f06-d421-408d-ad91-85a86a31d74f",
      "name": "Qdrant Vector Store",
      "credentials": {
        "qdrantApi": {
          "id": "V00g7HiNc1AJiwqv",
          "name": "QdrantApi account"
        }
      }
    },
    {
      "parameters": {
        "model": "nomic-embed-text:v1.5"
      },
      "type": "@n8n/n8n-nodes-langchain.embeddingsOllama",
      "typeVersion": 1,
      "position": [
        -272,
        -368
      ],
      "id": "5bfc4de4-efe7-452f-bb44-447669bea234",
      "name": "Embeddings Ollama",
      "credentials": {
        "ollamaApi": {
          "id": "LV87SgpX1s6SbI4s",
          "name": "Ollama account"
        }
      }
    },
    {
      "parameters": {
        "model": "qwen2.5:0.5b",
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.lmChatOllama",
      "typeVersion": 1,
      "position": [
        -272,
        80
      ],
      "id": "a5e287ff-1af7-41aa-98ca-b9ef9eb416e4",
      "name": "Ollama Chat Model1",
      "credentials": {
        "ollamaApi": {
          "id": "LV87SgpX1s6SbI4s",
          "name": "Ollama account"
        }
      },
      "notes": "Eres un modelo de lenguaje utilizado dentro de un agente de n8n.\nTu comportamiento debe ser consistente, preciso y completamente orientado a herramientas (tool-aware).\n\nDebes cumplir las siguientes reglas:\n\n1. Cuando el agente o el usuario pidan información basada en documentos, usa siempre la herramienta \"Answer questions with a vector store”.\nNunca respondas directamente si la información depende de documentos almacenados en Qdrant.\n\n2. Si tu controlador (el AI Agent) te envía una instrucción de ejecutar una herramienta, hazlo sin cuestionar.\nCuando uses una herramienta, devuelve exclusivamente el JSON requerido por n8n.\n\n3. No inventes información.\nSi no existe contexto recuperado desde el vector store, indica que no hay datos suficientes.\n\n4. Mantén un estilo claro, conciso y adecuado para mensajes enviados por Telegram.\n\n5. Si la entrada no requiere consultar documentos (saludos, interacción ligera), responde normalmente.\n\n6. Tienes acceso a memoria (Buffer Window en n8n). Úsala solo para el diálogo, no para inventar contenido faltante.\n\n7. No hagas suposiciones ni “alucines”.\nPrioriza siempre:\nHerramienta → Memoria → Razonamiento propio."
    },
    {
      "parameters": {
        "sessionIdType": "customKey",
        "sessionKey": "messages",
        "contextWindowLength": 4
      },
      "type": "@n8n/n8n-nodes-langchain.memoryBufferWindow",
      "typeVersion": 1.3,
      "position": [
        -80,
        80
      ],
      "id": "0c7bc3bf-ee64-400b-8d0b-0d3bc0f57e66",
      "name": "Simple Memory1"
    },
    {
      "parameters": {
        "model": "nomic-embed-text:v1.5"
      },
      "type": "@n8n/n8n-nodes-langchain.embeddingsOllama",
      "typeVersion": 1,
      "position": [
        112,
        352
      ],
      "id": "0bddbeff-5e85-4f33-9f21-5c414d7bfd3e",
      "name": "Embeddings Ollama2",
      "credentials": {
        "ollamaApi": {
          "id": "LV87SgpX1s6SbI4s",
          "name": "Ollama account"
        }
      }
    },
    {
      "parameters": {
        "promptType": "define",
        "text": "=Eres un agente de n8n que DEBE usar herramientas.  \nTienes una herramienta llamada \"Answer questions with a vector store\".  \nDebes seguir estas reglas de manera OBLIGATORIA:\n\n1. Si el usuario hace una pregunta, SIEMPRE debes llamar a la herramienta \"Answer questions with a vector store\" antes de responder.\n2. Después de usar la herramienta, responde SOLO basándote en los resultados entregados por ella.\n3. Si no existe información relevante devuelta por la herramienta, responde:\n   \"No encontré información sobre eso en tus documentos.\"\n4. Nunca inventes datos o explicaciones que no provengan de la herramienta.\n5. No uses razonamiento interno ni cadenas de pensamiento.  \n6. Usa solo texto plano, simple y directo.\n7. Si el usuario hace un saludo o un mensaje que NO requiera consultar documentos, puedes responder sin usar la herramienta.\n\n----  \n⚠️ Mensaje del usuario:  \n{{ $('Telegram Trigger').item.json.message.text }}\n",
        "options": {
          "systemMessage": "ou are a PDF document assistant. You have ONE tool: 'buscar_documentos'.\n\nRULES:\n1. User asks question → Use buscar_documentos tool → Read result → Answer\n2. Tool gives you TEXT from PDF. Use ONLY that text to answer\n3. Answer in Spanish. Keep it short (2-3 sentences)\n4. If tool finds nothing, say: 'No encontré esa información en el documento'\n5. NEVER make up information\n\nEXAMPLE:\nUser: What is Hightower?\nTool returns: 'Hightower is a port city...'\nYou say: 'Según el documento, Hightower es una ciudad portuaria...'\n\nALWAYS use the tool when user asks about documents. Trust what the tool returns."
        }
      },
      "type": "@n8n/n8n-nodes-langchain.agent",
      "typeVersion": 3,
      "position": [
        -112,
        -176
      ],
      "id": "e33de818-fd19-4a2d-824d-ec336ce20deb",
      "name": "AI Agent1",
      "notes": "Eres un asistente de preguntas y respuestas basado en recuperación (RAG).\nTu tarea ESPECÍFICA es usar SIEMPRE la herramienta \"Answer questions with a vector store\" para responder preguntas del usuario basadas en los documentos almacenados en el vector store mis_vectores.\n\nInstrucciones estrictas:\n\n1. Cuando el usuario envíe una pregunta, usa la herramienta de vector store para hacer una búsqueda semántica y obtener contexto relevante antes de responder.\n\n2. Si la herramienta devuelve información, responde usando SOLO ese contenido.\n\n3. Si la búsqueda NO encuentra nada relevante, responde claramente:\n\n“No encontré información sobre eso en tus documentos.”\n\n4. Sé claro, directo y evita especular.\n\n5. Mantén el formato de respuesta amigable para Telegram (texto plano).\n\n6. Si la pregunta no requiere datos del vector store (ej: saludo), responde normalmente.\n\n7. Nunca inventes información que no esté en los documentos recuperados."
    },
    {
      "parameters": {
        "updates": [
          "message"
        ],
        "additionalFields": {
          "download": true
        }
      },
      "type": "n8n-nodes-base.telegramTrigger",
      "typeVersion": 1.2,
      "position": [
        -816,
        -480
      ],
      "id": "a14dffdd-8530-4263-900f-9845e65e21a2",
      "name": "Telegram Trigger",
      "webhookId": "37a841b6-dd3e-4a6e-b9ab-02a01eee4f15",
      "credentials": {
        "telegramApi": {
          "id": "Gd1V2A5DUlGYsAcp",
          "name": "Telegram account"
        }
      }
    },
    {
      "parameters": {
        "chatId": "={{ $('Telegram Trigger').item.json.message.chat.id }}",
        "text": "={{ $json.output }}",
        "additionalFields": {}
      },
      "type": "n8n-nodes-base.telegram",
      "typeVersion": 1.2,
      "position": [
        240,
        -176
      ],
      "id": "09c787f8-ccfb-4834-a842-4258d0be99d6",
      "name": "Send a text message",
      "webhookId": "9f2ff89e-a8a8-4ddb-b7fb-661ac92d5876",
      "credentials": {
        "telegramApi": {
          "id": "Gd1V2A5DUlGYsAcp",
          "name": "Telegram account"
        }
      }
    },
    {
      "parameters": {
        "conditions": {
          "options": {
            "caseSensitive": true,
            "leftValue": "",
            "typeValidation": "loose",
            "version": 2
          },
          "conditions": [
            {
              "id": "e95afe4b-fce8-4d64-b4fe-d03a706bd366",
              "leftValue": "={{ $json.message.document }}",
              "rightValue": "",
              "operator": {
                "type": "string",
                "operation": "notEmpty",
                "singleValue": true
              }
            }
          ],
          "combinator": "and"
        },
        "looseTypeValidation": true,
        "options": {}
      },
      "type": "n8n-nodes-base.if",
      "typeVersion": 2.2,
      "position": [
        -608,
        -464
      ],
      "id": "3e8a8651-66c7-4015-9b34-2e308ca1255b",
      "name": "If"
    },
    {
      "parameters": {
        "chatId": "={{ $('Telegram Trigger').item.json.message.chat.id }}",
        "text": "✅ Documento guardado correctamente. Ahora puedes hacerme preguntas sobre su contenido.",
        "additionalFields": {}
      },
      "type": "n8n-nodes-base.telegram",
      "typeVersion": 1.2,
      "position": [
        272,
        -576
      ],
      "id": "85d1670a-99d4-452a-b910-0a604d872946",
      "name": "Send a text message1",
      "webhookId": "d65aa115-4c97-4716-817c-c3c1ad52add7",
      "credentials": {
        "telegramApi": {
          "id": "Gd1V2A5DUlGYsAcp",
          "name": "Telegram account"
        }
      }
    },
    {
      "parameters": {
        "aggregate": "aggregateAllItemData",
        "options": {}
      },
      "type": "n8n-nodes-base.aggregate",
      "typeVersion": 1,
      "position": [
        96,
        -576
      ],
      "id": "09683a7b-f9ca-4fab-b5ab-d99bf24f918e",
      "name": "Aggregate"
    },
    {
      "parameters": {
        "dataType": "binary",
        "binaryMode": "specificField",
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.documentDefaultDataLoader",
      "typeVersion": 1.1,
      "position": [
        -48,
        -400
      ],
      "id": "bdb4c098-c5f8-4135-a2e0-01d02760598b",
      "name": "Default Data Loader"
    },
    {
      "parameters": {
        "qdrantCollection": {
          "__rl": true,
          "value": "mis_vectores",
          "mode": "list",
          "cachedResultName": "mis_vectores"
        },
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.vectorStoreQdrant",
      "typeVersion": 1.3,
      "position": [
        80,
        208
      ],
      "id": "a7836f21-36b2-4e8e-8338-c6130cf3c165",
      "name": "Qdrant Vector Store1",
      "credentials": {
        "qdrantApi": {
          "id": "V00g7HiNc1AJiwqv",
          "name": "QdrantApi account"
        }
      }
    },
    {
      "parameters": {
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.memoryManager",
      "typeVersion": 1.1,
      "position": [
        -448,
        -176
      ],
      "id": "1692a364-73be-489b-8313-18cfaf47ae42",
      "name": "Chat Memory Manager"
    },
    {
      "parameters": {
        "topK": "={{ /*n8n-auto-generated-fromAI-override*/ $fromAI('Limit', ``, 'number') }}"
      },
      "type": "@n8n/n8n-nodes-langchain.toolVectorStore",
      "typeVersion": 1.1,
      "position": [
        112,
        32
      ],
      "id": "c21102cc-33d9-4478-bba3-324bd1a78eea",
      "name": "Answer questions with a vector store"
    }
  ],
  "pinData": {},
  "connections": {
    "Embeddings Ollama": {
      "ai_embedding": [
        [
          {
            "node": "Qdrant Vector Store",
            "type": "ai_embedding",
            "index": 0
          }
        ]
      ]
    },
    "Ollama Chat Model1": {
      "ai_languageModel": [
        [
          {
            "node": "AI Agent1",
            "type": "ai_languageModel",
            "index": 0
          },
          {
            "node": "Answer questions with a vector store",
            "type": "ai_languageModel",
            "index": 0
          }
        ]
      ]
    },
    "Simple Memory1": {
      "ai_memory": [
        [
          {
            "node": "AI Agent1",
            "type": "ai_memory",
            "index": 0
          },
          {
            "node": "Chat Memory Manager",
            "type": "ai_memory",
            "index": 0
          }
        ]
      ]
    },
    "Embeddings Ollama2": {
      "ai_embedding": [
        [
          {
            "node": "Qdrant Vector Store1",
            "type": "ai_embedding",
            "index": 0
          }
        ]
      ]
    },
    "Telegram Trigger": {
      "main": [
        [
          {
            "node": "If",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "AI Agent1": {
      "main": [
        [
          {
            "node": "Send a text message",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "If": {
      "main": [
        [
          {
            "node": "Qdrant Vector Store",
            "type": "main",
            "index": 0
          }
        ],
        [
          {
            "node": "Chat Memory Manager",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Qdrant Vector Store": {
      "main": [
        [
          {
            "node": "Aggregate",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Aggregate": {
      "main": [
        [
          {
            "node": "Send a text message1",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Default Data Loader": {
      "ai_document": [
        [
          {
            "node": "Qdrant Vector Store",
            "type": "ai_document",
            "index": 0
          }
        ]
      ]
    },
    "Qdrant Vector Store1": {
      "ai_tool": [
        []
      ],
      "ai_vectorStore": [
        [
          {
            "node": "Answer questions with a vector store",
            "type": "ai_vectorStore",
            "index": 0
          }
        ]
      ]
    },
    "Chat Memory Manager": {
      "main": [
        [
          {
            "node": "AI Agent1",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Answer questions with a vector store": {
      "ai_tool": [
        [
          {
            "node": "AI Agent1",
            "type": "ai_tool",
            "index": 0
          }
        ]
      ]
    }
  },
  "active": false,
  "settings": {
    "executionOrder": "v1"
  },
  "versionId": "638ed40b-25d9-4a76-859a-a7568dd0a25b",
  "meta": {
    "instanceId": "558d88703fb65b2d0e44613bc35916258b0f0bf983c5d4730c00c424b77ca36a"
  },
  "id": "gf9er3pPFrjkOpBG",
  "tags": []
}
