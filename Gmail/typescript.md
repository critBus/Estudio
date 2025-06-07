# Leer tus correos

necesitas tener la <mark style="background-color: #FF5733;">contraseña de aplicación</mark> 

```bash
pnpm install tsx imap-simple mailparser dotenv
```



```
GMAIL_USER="xxxx@gmail.com"
GMAIL_APP_PASSWORD="xxxx xxxx xxxx xxxx"
```



```tsx
// tests/helpers/gmailHelper.ts

import imaps from "imap-simple";
import { simpleParser, ParsedMail } from "mailparser";
import "dotenv/config"; // Asegúrate de que las variables de entorno se carguen

// Configuración para la conexión IMAP de Gmail
const gmailConfig = {
  imap: {
    user: process.env.GMAIL_USER!,
    password: process.env.GMAIL_APP_PASSWORD!,
    host: "imap.gmail.com",
    port: 993,
    tls: true,
    authTimeout: 3000,
    tlsOptions: {
      rejectUnauthorized: false,
    },
  },
};

interface SearchOptions {
  subject?: string;
  from?: string;
  to?: string;
  since?: string; // Fecha en formato 'YYYY-MM-DD'
}

/**
 * Busca el correo más reciente que coincida con los criterios.
 * @param options Criterios de búsqueda como subject, from, to, o since.
 * @returns El correo parseado (ParsedMail) o null si no se encuentra.
 */
export async function findLatestEmail(
  options: SearchOptions
): Promise<ParsedMail | null> {
  let connection: imaps.ImapSimple | null = null;

  try {
    connection = await imaps.connect(gmailConfig);
    await connection.openBox("INBOX");

    // Construye el criterio de búsqueda.
    const searchCriteria: any[] = ["ALL"];

    // Filtro por fecha (correos de hoy o después)
    // Esto es clave para tu requisito de no leer correos antiguos.
    const today = new Date().toISOString().split("T")[0];
    searchCriteria.push(["SINCE", options.since || today]);

    if (options.subject) {
      searchCriteria.push(["SUBJECT", options.subject]);
    }
    if (options.from) {
      searchCriteria.push(["FROM", options.from]);
    }
    if (options.to) {
      searchCriteria.push(["TO", options.to]);
    }

    const messages = await connection.search(searchCriteria, {
      bodies: [""], // Pide el cuerpo completo del mensaje
      markSeen: false, // No marcar el correo como leído en tu bandeja de entrada
    });

    if (messages.length === 0) {
      console.log("No se encontraron correos con los criterios especificados.");
      return null;
    }

    // Los mensajes suelen venir del más antiguo al más nuevo. Tomamos el último.
    const latestMessage = messages[messages.length - 1];
    const rawBody = latestMessage.parts.find((part) => part.which === "")?.body;

    if (!rawBody) {
      console.log("El correo más reciente no tiene cuerpo.");
      return null;
    }

    // Parsea el contenido crudo del correo a un objeto utilizable
    const parsedEmail = await simpleParser(rawBody);
    return parsedEmail;
  } catch (error) {
    console.error("Error al leer el correo:", error);
    return null;
  } finally {
    if (connection) {
      connection.end();
    }
  }
}

```



test

```typescript
// prueba.ts
import { findLatestEmail } from "./helpers/gmailHelper";
import type { ParsedMail } from "mailparser";

/**
 * Función que sondea repetidamente hasta encontrar un correo o que se agote el tiempo.
 * @param searchOptions Las opciones para buscar el correo.
 * @param timeoutMs Tiempo total de espera en milisegundos.
 * @param intervalMs Tiempo de espera entre cada intento.
 * @returns Una promesa que se resuelve con el correo encontrado.
 */
function pollForEmail(
  searchOptions: { subject: string; to: string },
  timeoutMs = 30000, // 30 segundos por defecto
  intervalMs = 2000 // 2 segundos entre intentos
): Promise<ParsedMail> {
  const startTime = Date.now();

  return new Promise((resolve, reject) => {
    const check = async () => {
      console.log("Buscando el correo...");
      const email = await findLatestEmail(searchOptions);

      if (email) {
        // ¡Éxito! Correo encontrado.
        resolve(email);
      } else if (Date.now() - startTime > timeoutMs) {
        // Se acabó el tiempo.
        reject(
          new Error(`El correo no se encontró en ${timeoutMs / 1000} segundos.`)
        );
      } else {
        // Aún no se encuentra, volver a intentar después del intervalo.
        setTimeout(check, intervalMs);
      }
    };

    check(); // Iniciar el primer intento
  });
}

// --- CÓMO USARLA ---
console.log("Iniciando la prueba de búsqueda de correo...");
pollForEmail({
  subject: "¡Bienvenido a nuestra aplicación!",
  to: process.env.GMAIL_USER!, // Asegúrate de que tus variables de entorno están cargadas
})
  .then((email) => {
    console.log("✅ ¡Correo encontrado!");
    console.log("Asunto:", email.subject);
    console.log("Cuerpo (texto):", email.text?.substring(0, 100) + "..."); // Muestra los primeros 100 caracteres
  })
  .catch((error) => {
    console.error("❌ Error:", error.message);
  })
  .finally(() => {
    console.log("Prueba finalizada.");
  });

```
