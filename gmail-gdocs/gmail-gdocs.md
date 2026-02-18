# gmail-gdocs_auth — Workflow: Gmail a Google Docs vía OAuth2

| Necesidad | Ubicación |
| :--- | :--- |
| Ver definición completa del workflow | [`gmail-docs-auth/workflow-gmail-docs.auth.json`](../../gmail-docs-auth/workflow-gmail-docs.auth.json) |
| Configurar credenciales Gmail | [Credenciales](#credenciales) |
| Configurar credenciales Google Docs | [Credenciales](#credenciales) |
| Depurar inserción vacía en el Doc | [Resolución de Errores](#resolución-de-errores) |
| Activar el workflow en producción | [Configuración](#configuración) |

## Componentes

| Nodo | Tipo | Versión | Posición |
| :--- | :--- | :--- | :--- |
| `Gmail Trigger` | `n8n-nodes-base.gmailTrigger` | 1.3 | `[-336, -160]` |
| `Update a document` | `n8n-nodes-base.googleDocs` | 2 | `[-112, -160]` |
| `Get Document Content` | `n8n-nodes-base.googleDocs` | 2 | `[112, -160]` |
| `Preview Output` | `n8n-nodes-base.noOp` | 1 | `[336, -160]` |

## Flujo de Ejecución

`Gmail Trigger` → `Update a document` → `Get Document Content` → `Preview Output`

El trigger hace polling cada minuto. Al detectar un correo, extrae `$json.text` e inserta el valor en el documento. Después lee el estado actualizado del documento y lo pasa al nodo de previsualización.

## Credenciales

| Credencial | Tipo n8n  | Nombre configurado |
| :--- | :--- | :---  |
| Gmail OAuth2 | `gmailOAuth2` |  `Gmail account` |
| Google Docs OAuth2 | `googleDocsOAuth2Api` | `Google Docs account` |

Ambas credenciales deben configurarse en **Settings > Credentials** de la instancia n8n antes de importar el workflow.

## Configuración

| Parámetro | Nodo | Valor actual | Descripción |
| :--- | :--- | :--- | :--- |
| `pollTimes.item[0].mode` | `Gmail Trigger` | `everyMinute` | Frecuencia de polling |
| `simple` | `Gmail Trigger` | `false` | Retorna objeto completo del email |
| `documentURL` | `Update a document` | `https://docs.google.com/document/d/1zaCzOf9-sZBit-J8NAu1b4Q8eZ0hGe6Yb2O5ulKb8f8/edit?tab=t.0` | Doc destino (hardcodeado) |
| `actionsUi.actionFields[0].action` | `Update a document` | `insert` | Operación sobre el Doc |
| `actionsUi.actionFields[0].text` | `Update a document` | `={{ $json.text }}` | Campo mapeado del email |
| `active` | Workflow | `false` | Estado: inactivo |
| `executionOrder` | Settings | `v1` | Orden de ejecución |
| `binaryMode` | Settings | `separate` | Almacenamiento binario |

## Rutas por Rol

| Rol | Tarea | Referencia |
| :--- | :--- | :--- |
| **Desarrollador** | Cambiar el campo mapeado del email | Nodo `Update a document` > `actionsUi.actionFields[0].text` |
| **Desarrollador** | Parametrizar `documentURL` con expresión dinámica | Nodo `Update a document` > `documentURL` |
| **Operaciones** | Activar el workflow en producción | `active: true` en el JSON o toggle en la UI |
| **Operaciones** | Ajustar frecuencia de polling | `pollTimes.item[0].mode` en nodo `Gmail Trigger` |
| **Usuario Final** | Verificar contenido insertado | Google Docs ID `1zaCzOf9-sZBit-J8NAu1b4Q8eZ0hGe6Yb2O5ulKb8f8` |

## Resolución de Errores

| Síntoma | Causa Raíz | Solución Técnica |
| :--- | :--- | :--- |
| El Doc recibe texto vacío | `$json.text` no existe en el output del trigger | Verificar campos disponibles en el panel de ejecución; usar `$json.snippet` o `$json.body.html` según el trigger |
| Error de autenticación Gmail | Credencial `hViz8oFFv0RTwAgU` expirada o inválida | Reconectar OAuth2 en **Settings > Credentials > Gmail account** |
| Error de autenticación Google Docs | Credencial `4JHC5L3aYq44h8J6` sin permisos de escritura | Verificar scope OAuth2: requiere `https://www.googleapis.com/auth/documents` |
| El workflow no se ejecuta | `active: false` en la definición | Activar mediante toggle en la UI o cambiar `active` a `true` antes de importar |
| Inserción duplicada en el Doc | Polling detecta el mismo email múltiples veces | Agregar filtro por `$json.id` con un nodo `If` antes de `Update a document` |

---

| Campo | Valor |
| :--- | :--- |
| **Mantenedor** | amillanaol (https://orcid.org/0009-0003-1768-7048) |
| **Estado** | Borrador |
| **Última Actualización** | 2026-02-17 |
