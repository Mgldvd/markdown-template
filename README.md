# Markdown Template
# 📄 Technical Specification: Dynamic Markdown Template System (Extended Mustache Model)

## 1. Objective

Define the standard format for creating **dynamic Markdown templates** using an extended **Mustache** syntax.
This format allows variables with type, maximum length, visual width, required flag, and **regular expression (`regex`) validation**, while also supporting variables not declared in advance.

The system must:

- Separate variable definitions from the document body.
- Support **implicit (undeclared)** variables.
- Validate field properties such as maximum length, visual width, required status, and regex pattern.
- Maintain full compatibility with Markdown syntax.

---

## 2. General File Structure

Each Markdown template consists of **two sections**, separated by a single line containing only:

```
:---
```

### Structure:

```
[SECTION 1: Variable Definitions]
:---
[SECTION 2: Template Content]
```

### General Example:

```md
:---

{{ variable | type | length | width | regex }}
{{ anotherVariable | type }}

:---

Text using {{ variable }} and {{ anotherVariable }}.
```

---

## 3. Section 1 — Variable Definitions

This section contains the list of variables used or anticipated in the template.
Each line defines one variable using the following syntax:

```
{{ variableName | type | length | width | regex }}
```

### Parameters

|Parameter|Description|Required|
|---|---|---|
|`variableName`|Unique variable identifier. Cannot contain spaces or underscores (`_`). Use *camelCase* for multi-word names (e.g., `fullName`).|✅|
|`type`|Field type (`text`, `textarea`, `date`, etc.).|✅|
|`length`|Maximum number of allowed characters.|Optional|
|`width`|Minimum visual width of the field (supports CSS units).|Optional|
|`regex`|Regular expression defining the valid input format.|Optional|

---

### 3.1. Required Fields

A field is considered **required** if its definition ends with a space followed by `?}}`.

#### Valid examples:

```
{{ company | text | 200 | 60 ?}}
{{ address | textarea | 400 | 80% ?}}
{{ email | text | 255 | 40em | ^[\w.%+-]+@[\w.-]+\.[A-Za-z]{2,}$ ?}}
{{ phone | text | 15 | 200px | ^\+\d{1,3}\s?\d{4,14}$ ?}}
```

#### Rules:

- There must be **a space between the last parameter and the `?`**.
  Example: `| 60 ?}}` or `| 80% ?}}`
- The field must be marked as `required` when rendered in the form.
- The interface may visually indicate required fields (e.g., “(required)” label or color).
- If a required field is empty or fails the `regex` pattern, the system must block document generation.

---

### 3.2. Parameter `width`

- Defines the **minimum visual width** of the input in the form.
- Accepts numeric values or **valid CSS units** (`px`, `%`, `em`, `rem`, `vw`, etc.).
- Affects only the visual display, not the text length limit.

#### Examples:

```
{{ company | text | 200 | 60 }}
{{ phone | text | 15 | 250px }}
{{ address | textarea | 400 | 90% }}
{{ comment | textarea | 500 | 40em ?}}
```

---

### 3.3. Parameter `length`

- Specifies the **maximum number of characters** allowed for a field.
- Used as a `maxlength` validation rule.
- If not defined, the default value is 255 for `text`, or unlimited for `textarea`.

#### Examples:

```
{{ company | text | 200 }}
{{ description | textarea | 500 | 80% }}
```

---

### 3.4. Parameter `regex`

- Defines a **regular expression** that the input value must match to be valid.
- Useful for pattern validation (e.g., emails, phone numbers, IPs).
- Does **not** require delimiters (`/ /`), only the raw regex expression.

#### Examples:

```
{{ email | text | 255 | 60 | ^[\w.%+-]+@[\w.-]+\.[A-Za-z]{2,}$ }}
{{ phone | text | 15 | 200px | ^\+\d{1,3}\s?\d{4,14}$ ?}}
{{ ipAddress | text | 15 | 200px | ^(?:\d{1,3}\.){3}\d{1,3}$ }}
{{ postalCode | text | 5 | 60 | ^\d{5}$ ?}}
```

#### Rules:

- The input value must **fully match** the regex pattern.
- If `regex` is defined and validation fails, the field is considered invalid.
- `regex` can be combined with other parameters (`length`, `width`, `?`).

---

### 3.5. Complete Variable Header Example

```
{{ date | date }}
{{ company | text | 200 | 60 ?}}
{{ address | address | 300 | 80% }}
{{ email | text | 255 | 60 | ^[\w.%+-]+@[\w.-]+\.[A-Za-z]{2,}$ ?}}
{{ phone | text | 15 | 250px | ^\+\d{1,3}\s?\d{4,14}$ ?}}
{{ ipAddress | text | 15 | 200px | ^(?:\d{1,3}\.){3}\d{1,3}$ }}
{{ fullName | text | 100 | 50 ?}}
```

---

## 4. Section 2 — Template Content

After the `:---` separator, the Markdown body of the document is defined.
Variables declared in the first section, as well as new inline variables, may be used here.

### Valid forms:

1. **Simple use:**
    ```
    {{ variable }}
    ```
2. **Extended inline use:**
    ```
    {{ variable | type | length | width | regex }}
    ```
3. **Required inline field:**
    ```
    {{ variable | type | length | width | regex ?}}
    ```

---

### 4.1. Variables Not Defined in the Header

Variables used in the body that **are not listed** in the header are considered **implicit** and automatically generated with default values.

|Property|Default value|
|---|---|
|`type`|`text`|
|`length`|unlimited|
|`width`|default size (e.g., 40)|
|`regex`|none|
|`required`|`false`|

#### Valid examples:

```
{{ signature }}
{{ signature | text }}
{{ signature | text | 200 | 80 }}
```

> Although inline variable definitions are allowed, declaring them in the header is recommended for clarity.

---

## 5. Full Markdown Template Example

```md
{{ date | date }}
{{ company | text | 200 | 60 ?}}
{{ address | address | 300 | 80% }}
{{ phone | text | 15 | 250px | ^\+\d{1,3}\s?\d{4,14}$ ?}}
{{ fullName | text | 100 | 50 ?}}

:---

{{ date }}

Human Resources Department
{{ company }}
{{ address }}

Dear Sir or Madam,

My name is {{ fullName }} and I am a {{ job }}.
I am writing to express my interest in joining the {{ company }} team.

Sincerely,
{{ fullName }}

{{ signature }}
```

> In this example, `signature` is not defined in the header, so it is treated as an **implicit variable** of type `text`.

---

## 6. Example of Parsed Structure (Expected Output)

```json
[
  {"variable": "date", "type": "date", "length": null, "width": null, "regex": null, "required": false},
  {"variable": "company", "type": "text", "length": 200, "width": "60", "regex": null, "required": true},
  {"variable": "address", "type": "address", "length": 300, "width": "80%", "regex": null, "required": false},
  {"variable": "phone", "type": "text", "length": 15, "width": "250px", "regex": "^\+\d{1,3}\s?\d{4,14}$", "required": true},
  {"variable": "fullName", "type": "text", "length": 100, "width": "50", "regex": null, "required": true},
  {"variable": "signature", "type": "text", "length": null, "width": null, "regex": null, "required": false}
]
```

---

## 7. Supported Field Types

|Type|Description|Example|
|---|---|---|
|`text`|Single-line text input.|`"Telelejos SA"`|
|`textarea`|Multi-line text input.|`"I am a motivated and hardworking person..."`|
|`date`|Date selector.|`"2025-10-17"`|
|`number`|Numeric input.|`"25"`|
|`email`|Email address.|`"laura@example.com"`|
|`address`|Address or long text.|`"Condor Avenue 8"`|
|`boolean`|Checkbox (yes/no).|`true` / `false`|

**Note:** The system should support additional future field types such as `phone`, `url`, `currency`, `signature`, etc.

---

## 8. Validation Rules

### Structure
- The document must contain a single `:---` separator between the variable header and the body.
- If the header section is omitted, inline variable definitions remain valid.
- The content must be valid Markdown.

### Variables
- Variable names must use **camelCase** — underscores (`_`) are not allowed.
- Parameters are separated by `|` (whitespace ignored).
- Required fields end with a space and `?}}`.
- `width` accepts CSS-compatible units (`px`, `%`, `em`, `rem`, `vw`, etc.).
- `regex` defines formatting patterns (no delimiters).
- Undefined variables are created automatically with default settings.

### Replacement
- All occurrences of the same variable share the same value.
- If a required variable (`?`) is empty or fails regex validation, document generation must stop.
- Markdown formatting, spacing, and line breaks must be preserved.

---

## 9. Expected System Behavior

1. **Parsing**
   - Detects all variables across both sections.
   - Builds a JSON schema with properties (`type`, `length`, `width`, `regex`, `required`).
   - Assigns default values to implicit variables.

2. **Dynamic Form Rendering**
   - Generates form inputs based on variable types.
   - Applies validations (`maxlength`, `required`, `width`, `regex`).
   - Properly interprets CSS width units.

3. **Final Rendering**
   - Replaces placeholders with user-provided values.
   - Maintains full Markdown formatting and style.
   - Supports exporting as `.md`, `.html`, or `.pdf`.
