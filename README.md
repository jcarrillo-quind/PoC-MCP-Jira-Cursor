# PoC MCP Jira Cursor

Este documento describe una **Prueba de Concepto (PoC)** para integrar **Jira ↔ MCP ↔ Cursor ↔ Repo (Ionic Angular)**, con el objetivo de generar automáticamente código base a partir de tickets en Jira.

---

## 🚀 Flujo propuesto

1. **Jira → MCP**  
   - Jira emite un **webhook** cuando se crea/actualiza un ticket.  
   - El MCP recibe datos del ticket:  
     - `key` (ej: `APP-123`)  
     - `summary` (ej: "Crear pantalla de login")  
     - `description` (ej: "La pantalla debe tener campos de email y contraseña").  

2. **MCP → Cursor**  
   - MCP construye un *prompt estructurado* con los datos del ticket.  
   - Envía el prompt a la API local de Cursor (`http://localhost:5100`).  
   - Cursor genera el código correspondiente en el repo.  

3. **Cursor → Repo + Branch**  
   - Cursor crea una rama `feature/APP-123`.  
   - Genera archivos solicitados (componentes Angular/Ionic, pruebas unitarias, routing, etc).  
   - (Opcional) MCP crea un **Pull Request** automáticamente usando GitHub/GitLab API.  

---

## 🛠 Ejemplo MCP en Node.js

```js
import express from "express";
import axios from "axios";
import dotenv from "dotenv";

dotenv.config();
const app = express();
app.use(express.json());

// Jira webhook
app.post("/jira-webhook", async (req, res) => {
  try {
    const issue = req.body.issue;
    const key = issue.key;
    const summary = issue.fields.summary;
    const description = issue.fields.description;

    console.log(`📥 Nuevo ticket Jira: ${key} - ${summary}`);

    // Prompt para Cursor
    const prompt = `
      Genera un componente Angular standalone para la tarea ${key}.
      - Nombre: ${summary}
      - Descripción: ${description}
      - Framework: Ionic Angular 7, Angular 19+
      - Estructura: crear en src/app/features/${key.toLowerCase()}
      - Incluir un módulo de routing si aplica.
      - Generar pruebas unitarias básicas.
    `;

    // Enviar prompt a Cursor
    const cursorResponse = await axios.post("http://localhost:5100/command", {
      command: "generate",
      prompt,
    });

    console.log("✅ Respuesta Cursor:", cursorResponse.data);

    res.status(200).send("Ticket procesado y enviado a Cursor.");
  } catch (error) {
    console.error("❌ Error:", error.message);
    res.status(500).send("Error procesando Jira webhook");
  }
});

app.listen(3000, () => {
  console.log("🚀 MCP corriendo en http://localhost:3000");
});
```
## Integraciones clave

### Jira

Configura un webhook apuntando a:
http://localhost:3000/jira-webhook

### MCP

Procesa el ticket y genera el prompt adecuado.

### Cursor

Recibe el prompt, genera código y lo guarda en el repo.

(Opcional)

MCP usa GitHub/GitLab API para crear Pull Requests automáticamente.

📌 Beneficios

🚀 Automatización de scaffolding de código desde Jira.

🔗 Trazabilidad: cada ticket genera su propio branch y código base.

⚡ Reducción de tareas repetitivas: Cursor escribe esqueletos, el equipo ajusta lógica de negocio.

