# 📄 Especificación Técnica: Sistema Dinámico de Plantillas Markdown (Modelo Mustache Ampliado)

## 1. Objetivo

Definir el formato estándar para la creación de **plantillas Markdown dinámicas** utilizando una sintaxis **Mustache ampliada**.
Este formato permite variables con tipo, longitud máxima, ancho visual, indicador de obligatoriedad y **validación mediante expresiones regulares (`regex`)**, y también admite variables no declaradas previamente.

El sistema debe:

* Separar las definiciones de variables del cuerpo del documento.
* Soportar variables **implícitas (no declaradas)**.
* Validar propiedades como longitud máxima, ancho visual, obligatoriedad y patrón `regex`.
* Mantener compatibilidad total con la sintaxis Markdown.

---

## 2. Estructura General del Archivo

Cada plantilla Markdown consta de **dos secciones**, separadas por una línea que contenga únicamente:

```
:---
```

### Estructura:

```
[SECCIÓN 1: Definición de Variables]
:---
[SECCIÓN 2: Contenido de la Plantilla]
```

### Ejemplo General:

```md
:---

{{ variable | tipo | longitud | ancho | regex }}
{{ otraVariable | tipo }}

:---

Texto con {{ variable }} y {{ otraVariable }}.
```

---

## 3. Sección 1 — Definición de Variables

Esta sección contiene la lista de variables utilizadas o previstas en la plantilla.
Cada línea define una variable utilizando la siguiente sintaxis:

```
{{ nombreVariable | tipo | longitud | ancho | regex }}
```

### Parámetros

| Parámetro        | Descripción                                                                                                                              | Obligatorio |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------- | ----------- |
| `nombreVariable` | Identificador único. No debe contener espacios ni guiones bajos (`_`). Usar *camelCase* para nombres compuestos (ej.: `nombreCompleto`). | ✅           |
| `tipo`           | Tipo de campo (`text`, `textarea`, `date`, etc.).                                                                                        | ✅           |
| `longitud`       | Número máximo de caracteres permitidos.                                                                                                  | Opcional    |
| `ancho`          | Ancho visual mínimo del campo (acepta unidades CSS).                                                                                     | Opcional    |
| `regex`          | Expresión regular que define el formato válido.                                                                                          | Opcional    |

---

### 3.1. Campos Obligatorios

Un campo se considera **obligatorio** si su definición termina con un espacio seguido de `?}}`.

#### Ejemplos válidos:

```
{{ empresa | text | 200 | 60 ?}}
{{ direccion | textarea | 400 | 80% ?}}
{{ correo | text | 255 | 40em | ^[\w.%+-]+@[\w.-]+\.[A-Za-z]{2,}$ ?}}
{{ telefono | text | 15 | 200px | ^\+\d{1,3}\s?\d{4,14}$ ?}}
```

#### Reglas:

* Debe haber **un espacio entre el último parámetro y el `?`**.
  Ejemplo: `| 60 ?}}` o `| 80% ?}}`
* El campo se marcará como obligatorio al renderizarse en el formulario.
* La interfaz puede indicarlo visualmente (etiqueta “(obligatorio)” o color).
* Si un campo obligatorio está vacío o falla la validación `regex`, se debe bloquear la generación del documento.

---

### 3.2. Parámetro `ancho`

* Define el **ancho visual mínimo** del campo en el formulario.
* Acepta valores numéricos o unidades CSS válidas (`px`, `%`, `em`, `rem`, `vw`, etc.).
* Solo afecta a la presentación visual, no a la longitud del texto.

#### Ejemplos:

```
{{ empresa | text | 200 | 60 }}
{{ telefono | text | 15 | 250px }}
{{ direccion | textarea | 400 | 90% }}
{{ comentario | textarea | 500 | 40em ?}}
```

---

### 3.3. Parámetro `longitud`

* Especifica el **número máximo de caracteres** permitidos.
* Se utiliza como regla de validación `maxlength`.
* Si no se define, el valor por defecto es 255 para `text`, o ilimitado para `textarea`.

#### Ejemplos:

```
{{ empresa | text | 200 }}
{{ descripcion | textarea | 500 | 80% }}
```

---

### 3.4. Parámetro `regex`

* Define una **expresión regular** que debe cumplir el valor ingresado.
* Útil para validar formatos (correos, teléfonos, IPs).
* No requiere delimitadores (`/ /`), solo la expresión en bruto.

#### Ejemplos:

```
{{ correo | text | 255 | 60 | ^[\w.%+-]+@[\w.-]+\.[A-Za-z]{2,}$ }}
{{ telefono | text | 15 | 200px | ^\+\d{1,3}\s?\d{4,14}$ ?}}
{{ ip | text | 15 | 200px | ^(?:\d{1,3}\.){3}\d{1,3}$ }}
{{ codigoPostal | text | 5 | 60 | ^\d{5}$ ?}}
```

#### Reglas:

* El valor debe **coincidir completamente** con el patrón.
* Si se define `regex` y la validación falla, el campo se considera inválido.
* Puede combinarse con los demás parámetros (`longitud`, `ancho`, `?`).

---

### 3.5. Ejemplo Completo de Cabecera de Variables

```
{{ fecha | date }}
{{ empresa | text | 200 | 60 ?}}
{{ direccion | address | 300 | 80% }}
{{ correo | text | 255 | 60 | ^[\w.%+-]+@[\w.-]+\.[A-Za-z]{2,}$ ?}}
{{ telefono | text | 15 | 250px | ^\+\d{1,3}\s?\d{4,14}$ ?}}
{{ ip | text | 15 | 200px | ^(?:\d{1,3}\.){3}\d{1,3}$ }}
{{ nombreCompleto | text | 100 | 50 ?}}
```

---

## 4. Sección 2 — Contenido de la Plantilla

Tras el separador `:---`, se define el cuerpo Markdown del documento.
Pueden utilizarse tanto variables declaradas como otras nuevas en línea.

### Formas válidas:

1. **Uso simple:**

   ```
   {{ variable }}
   ```
2. **Uso extendido en línea:**

   ```
   {{ variable | tipo | longitud | ancho | regex }}
   ```
3. **Campo obligatorio en línea:**

   ```
   {{ variable | tipo | longitud | ancho | regex ?}}
   ```

---

### 4.1. Variables No Definidas en la Cabecera

Las variables utilizadas en el cuerpo que **no estén en la cabecera** se consideran **implícitas**, y se generan automáticamente con valores por defecto.

| Propiedad     | Valor por defecto            |
| ------------- | ---------------------------- |
| `tipo`        | `text`                       |
| `longitud`    | ilimitada                    |
| `ancho`       | tamaño por defecto (ej.: 40) |
| `regex`       | ninguna                      |
| `obligatorio` | `false`                      |

#### Ejemplos válidos:

```
{{ firma }}
{{ firma | text }}
{{ firma | text | 200 | 80 }}
```

> Aunque se permite definir variables en línea, se recomienda declararlas en la cabecera para mayor claridad.

---

## 5. Ejemplo Completo de Plantilla Markdown

```md
{{ fecha | date }}
{{ empresa | text | 200 | 60 ?}}
{{ direccion | address | 300 | 80% }}
{{ telefono | text | 15 | 250px | ^\+\d{1,3}\s?\d{4,14}$ ?}}
{{ nombreCompleto | text | 100 | 50 ?}}

:---

{{ fecha }}

Departamento de Recursos Humanos
{{ empresa }}
{{ direccion }}

Estimados señores:

Mi nombre es {{ nombreCompleto }} y soy {{ puesto }}.
Les escribo para expresar mi interés en formar parte del equipo de {{ empresa }}.

Atentamente,
{{ nombreCompleto }}

{{ firma }}
```

> En este ejemplo, `firma` no está definida en la cabecera, por lo tanto se trata como variable **implícita** de tipo `text`.

---

## 6. Ejemplo de Estructura Analizada (Salida Esperada)

```json
[
  {"variable": "fecha", "type": "date", "length": null, "width": null, "regex": null, "required": false},
  {"variable": "empresa", "type": "text", "length": 200, "width": "60", "regex": null, "required": true},
  {"variable": "direccion", "type": "address", "length": 300, "width": "80%", "regex": null, "required": false},
  {"variable": "telefono", "type": "text", "length": 15, "width": "250px", "regex": "^\\+\\d{1,3}\\s?\\d{4,14}$", "required": true},
  {"variable": "nombreCompleto", "type": "text", "length": 100, "width": "50", "regex": null, "required": true},
  {"variable": "firma", "type": "text", "length": null, "width": null, "regex": null, "required": false}
]
```

---

## 7. Tipos de Campo Soportados

| Tipo       | Descripción                      | Ejemplo                                       |
| ---------- | -------------------------------- | --------------------------------------------- |
| `text`     | Texto de una sola línea.         | `"Telelejos SA"`                              |
| `textarea` | Texto de varias líneas.          | `"Soy una persona motivada y trabajadora..."` |
| `date`     | Selector de fecha.               | `"2025-10-17"`                                |
| `number`   | Campo numérico.                  | `"25"`                                        |
| `email`    | Dirección de correo electrónico. | `"laura@example.com"`                         |
| `address`  | Dirección o texto largo.         | `"Avenida Cóndor 8"`                          |
| `boolean`  | Casilla de verificación (sí/no). | `true` / `false`                              |

**Nota:** El sistema debe soportar tipos adicionales en el futuro como `phone`, `url`, `currency`, `signature`, etc.

---

## 8. Reglas de Validación

### Estructura

* El documento debe contener un único separador `:---` entre la cabecera y el cuerpo.
* Si se omite la cabecera, siguen siendo válidas las definiciones en línea.
* El contenido debe ser Markdown válido.

### Variables

* Los nombres de variables deben seguir el formato **camelCase** — no se permiten guiones bajos (`_`).
* Los parámetros se separan con `|` (se ignoran los espacios).
* Los campos obligatorios terminan con un espacio seguido de `?}}`.
* El parámetro `ancho` acepta unidades CSS compatibles (`px`, `%`, `em`, `rem`, `vw`, etc.).
* El parámetro `regex` define patrones de formato (sin delimitadores).
* Las variables no definidas se crean automáticamente con valores por defecto.

### Sustitución

* Todas las apariciones de una misma variable comparten el mismo valor.
* Si una variable obligatoria (`?`) está vacía o falla la validación `regex`, se debe interrumpir la generación del documento.
* Se debe conservar el formato Markdown, los espacios y saltos de línea.

---

## 9. Comportamiento Esperado del Sistema

1. **Análisis**

   * Detecta todas las variables en ambas secciones.
   * Construye un esquema JSON con propiedades (`type`, `length`, `width`, `regex`, `required`).
   * Asigna valores por defecto a las variables implícitas.

2. **Renderizado Dinámico del Formulario**

   * Genera campos de entrada según el tipo de variable.
   * Aplica validaciones (`maxlength`, `required`, `width`, `regex`).
   * Interpreta correctamente las unidades CSS de ancho.

3. **Renderizado Final**

   * Sustituye los marcadores por los valores proporcionados.
   * Mantiene el formato Markdown original.
   * Permite exportación como `.md`, `.html` o `.pdf`.
