## 1. Descripción

En este laboratorio construirá un pipeline básico de procesamiento de texto para sistemas de NLP y LLMs.

El objetivo es comprender cómo un texto crudo puede ser normalizado, tokenizado, analizado estadísticamente y revisado mediante reglas simples de seguridad antes de ser enviado a un modelo de lenguaje.

El laboratorio se enfoca en dos componentes principales:

- Tokenización y estadísticas básicas de texto.
- Guardrails basados en expresiones regulares para detectar información sensible.

---

## 2. Objetivos De Aprendizaje

Al finalizar este laboratorio, el estudiante será capaz de:

- Implementar distintas estrategias de tokenización en Python.
- Comparar tokenización por espacios, tokenización con Regex básica y tokenización mixta.
- Calcular estadísticas básicas sobre un texto tokenizado.
- Detectar datos sensibles usando expresiones regulares.
- Diseñar acciones simples de guardrail como `ALLOW`, `WARN`, `REDACT` y `BLOCK`.
- Reflexionar sobre las limitaciones de Regex como mecanismo de seguridad.

---

## 3. Contexto

Antes de enviar texto a un LLM, es importante revisar qué información contiene.

Un usuario puede introducir accidentalmente datos como:

- Correos electrónicos.
- Números telefónicos.
- URLs.
- Identificadores personales.
- Contraseñas, tokens o API keys.

En sistemas reales, enviar este tipo de información a un modelo externo puede representar riesgos de privacidad, cumplimiento normativo y seguridad.

En este laboratorio implementará una primera capa de protección usando Regex.

---

## 4. Archivos

Se le proporciona un archivo guía:

```text
laboratorio_1_tokenizacion_guardrails_guia.py
```

Debe trabajar sobre una copia de este archivo y completar las secciones indicadas como ejercicios.

Nombre sugerido para la entrega:

```text
laboratorio_1_apellido_nombre.py
```

---

## 5. Requisitos Funcionales

Su programa debe implementar un pipeline que realice las siguientes etapas:

```text
Texto de entrada
  -> normalización básica
  -> detección de datos sensibles
  -> acción de guardrail
  -> tokenización
  -> estadísticas
  -> salida final
```

---

## 6. Parte A: Textos De Prueba

Debe incluir al menos **5 textos de prueba**.

De esos textos:

- Al menos 1 debe contener un correo electrónico simulado.
- Al menos 1 debe contener un teléfono simulado.
- Al menos 1 debe contener una URL.
- Al menos 1 debe contener una palabra sensible como `password`, `token`, `secret`, `clave` o `api_key`.
- Al menos 1 debe contener un DPI simulado o número largo tipo identificador.

Ejemplos válidos:

```text
Mi correo es ana.lopez@uvg.edu.gt
```

```text
Mi número es +502 5555-1234
```

```text
La documentación está en https://docs.example.com/nlp
```

```text
Mi password temporal es Cambiar123
```

```text
Mi DPI simulado es 1234 56789 0101
```

---

## 7. Parte B: Normalización

Debe implementar una función que normalice espacios.

Ejemplo esperado:

```python
def normalizar_espacios(texto):
    return re.sub(r"\s+", " ", texto).strip()
```

La normalización debe eliminar espacios repetidos y saltos de línea innecesarios.

No es obligatorio convertir todo a minúsculas, ya que en algunos casos puede perderse información relevante.

---

## 8. Parte C: Tokenización

Debe incluir al menos tres formas de tokenización:

1. Tokenización por espacios.
2. Tokenización con Regex básica.
3. Tokenización mixta que conserve correos y URLs cuando sea posible.

Ejemplo de tokenización por espacios:

```python
def tokenizar_por_espacios(texto):
    return texto.split()
```

Ejemplo de tokenización Regex básica:

```python
def tokenizar_con_regex_basico(texto):
    return re.findall(r"\b\w+\b", texto.lower())
```

Ejemplo de tokenización mixta:

```python
def tokenizar_con_regex_mixto(texto):
    patron = r"https?://\S+|www\.\S+|[\w\.-]+@[\w\.-]+\.\w+|\b\w+\b|[^\w\s]"
    return re.findall(patron, texto, flags=re.UNICODE)
```

---

## 9. Parte D: Estadísticas De Texto

Debe calcular estadísticas básicas para cada texto procesado:

- Número de caracteres.
- Número total de tokens.
- Número de tokens únicos.
- Top 10 tokens más frecuentes.

Ejemplo de salida esperada:

```text
Caracteres: 69
Tokens: 17
Tokens únicos: 14
Top 10 tokens:
  - mi: 2
  - correo: 1
  - EMAIL_REDACTED: 1
```

---

## 10. Parte E: Detección De Datos Sensibles

Debe definir un diccionario de patrones Regex para detectar información sensible.

Patrones mínimos requeridos:

```python
PATRONES_SENSIBLES = {
    "EMAIL": r"\b[\w\.-]+@[\w\.-]+\.\w+\b",
    "URL": r"https?://\S+|www\.\S+",
    "PHONE": r"\+?\d[\d\s\-]{7,}\d",
    "SECRET_WORD": r"(?i)\b(password|contrasena|contraseña|clave|secret|token|api_key|apikey)\b",
    "DPI": r"\b\d{4}[\s\-]?\d{5}[\s\-]?\d{4}\b",
    "LONG_NUMBER": r"\b\d{8,}\b",
}
```

Debe implementar una función que retorne los hallazgos detectados.

Cada hallazgo debe incluir:

- Tipo de dato detectado.
- Valor detectado.

Ejemplo:

```text
Datos sensibles detectados:
- EMAIL: ana.lopez@uvg.edu.gt
- PHONE: +502 5555-1234
```

---

## 11. Parte F: Acciones De Guardrail

Debe implementar una política de decisión con las siguientes acciones:

| Acción | Significado |
|---|---|
| `ALLOW` | El texto puede continuar sin modificación. |
| `WARN` | El texto puede continuar, pero requiere advertencia o revisión. |
| `REDACT` | El texto contiene datos sensibles que deben reemplazarse. |
| `BLOCK` | El texto no debe enviarse al modelo. |

La política mínima requerida es:

| Condición | Acción |
|---|---|
| Contiene `SECRET_WORD` | `BLOCK` |
| Contiene `EMAIL`, `PHONE`, `DPI` o `LONG_NUMBER` | `REDACT` |
| Contiene solo `URL` | `WARN` |
| No contiene hallazgos | `ALLOW` |

Ejemplo:

```python
def decidir_accion(hallazgos):
    tipos = {hallazgo["tipo"] for hallazgo in hallazgos}

    if "SECRET_WORD" in tipos:
        return "BLOCK"

    if tipos.intersection({"EMAIL", "PHONE", "DPI", "LONG_NUMBER"}):
        return "REDACT"

    if "URL" in tipos:
        return "WARN"

    return "ALLOW"
```

---

## 12. Parte G: Redacción De Texto

Cuando la acción sea `REDACT`, debe reemplazar los datos sensibles por etiquetas.

Ejemplos:

| Dato | Reemplazo |
|---|---|
| Correo electrónico | `[EMAIL_REDACTED]` |
| Teléfono | `[PHONE_REDACTED]` |
| DPI | `[DPI_REDACTED]` |
| Número largo | `[NUMBER_REDACTED]` |

Ejemplo:

```text
Entrada:
Mi correo es ana.lopez@uvg.edu.gt

Salida:
Mi correo es [EMAIL_REDACTED]
```

Si la acción es `BLOCK`, el programa no debe tokenizar el texto ni calcular estadísticas sobre el contenido original.

---

## 13. Parte H: Salida Esperada

Para cada texto, el programa debe imprimir una salida similar a esta:

```text
================================================================================
TEXTO 1
================================================================================
Texto original:
Hola!!! necesito ayuda con mi cuenta :( mi correo es ana.lopez@uvg.edu.gt

Texto normalizado:
Hola!!! necesito ayuda con mi cuenta :( mi correo es ana.lopez@uvg.edu.gt

Datos sensibles detectados:
  - EMAIL: ana.lopez@uvg.edu.gt

Acción recomendada: REDACT

Texto seguro:
Hola!!! necesito ayuda con mi cuenta :( mi correo es [EMAIL_REDACTED]

Tokens:
['Hola', '!', '!', '!', 'necesito', 'ayuda', 'con', 'mi', 'cuenta', ':', '(', 'mi', 'correo', 'es', '[', 'EMAIL_REDACTED', ']']

Estadísticas:
Caracteres: 69
Tokens: 17
Tokens únicos: 14
```

---

## 14. Preguntas De Reflexión

Incluya al final de su entrega una reflexión breve de **150 a 250 palabras** respondiendo:

- ¿Qué falsos positivos encontró o podrían ocurrir?
- ¿Qué falsos negativos encontró o podrían ocurrir?
- ¿Qué información se pierde al redactar datos sensibles?
- ¿Regex sería suficiente para proteger información de una empresa real?
- ¿Qué otra capa de seguridad agregaría?

---

## 15. Entregables

Debe entregar:

- Archivo `.py` con la implementación completa.
- Evidencia de ejecución, ya sea salida copiada en un `.txt`, captura o notebook ejecutado.
- Reflexión breve de 150 a 250 palabras.

Nombre sugerido:

```text
laboratorio_1_apellido_nombre.py
```

---

## 16. Rúbrica

| Criterio | Descripción | Peso |
|---|---|---:|
| Tokenización funcional | Implementa y compara tokenización por espacios, Regex básica y Regex mixta. | 25% |
| Estadísticas de texto | Calcula caracteres, tokens, tokens únicos y frecuencias. | 20% |
| Detección con Regex | Detecta email, URL, teléfono, secreto, DPI o número largo. | 30% |
| Acciones de guardrail | Implementa correctamente `ALLOW`, `WARN`, `REDACT` y `BLOCK`. | 15% |
| Reflexión crítica | Discute limitaciones, falsos positivos y falsos negativos. | 10% |

Total: **100%**

---

## 17. Criterios De Calidad

Se valorará que el código:

- Sea claro y legible.
- Use nombres descriptivos.
- Esté dividido en funciones.
- Incluya comentarios útiles cuando sea necesario.
- No exponga datos reales de ninguna persona.
- Use únicamente datos simulados.

---

## 18. Restricción Importante

No utilice datos personales reales.

Todos los correos, teléfonos, identificadores, contraseñas o tokens incluidos en este laboratorio deben ser simulados.

Ejemplos aceptables:

```text
ana.lopez@example.com
+502 5555-1234
1234 56789 0101
API_KEY=abc123-simulada
```

---

## 19. Comando De Ejecución

Para ejecutar el laboratorio:

```bash
python laboratorio_1_apellido_nombre.py
```

Si trabaja sobre el archivo guía:

```bash
python laboratorio_1_tokenizacion_guardrails_guia.py
```

---

## 20. Conexión Con El Siguiente Tema

Este laboratorio prepara el camino para representación semántica.

Después de tokenizar y proteger el texto, el siguiente paso será convertirlo en vectores:

```text
Texto
  -> tokens
  -> embeddings
  -> similitud semántica
  -> búsqueda semántica
```