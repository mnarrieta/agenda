# SPEC.md — Agenda de Contactos

## 1. Visión General

Aplicación web de agenda personal que permite gestionar contactos con nombre y número de teléfono. La interfaz es simple, rápida y accesible desde cualquier dispositivo.

---

## 2. Actores

| Actor | Descripción |
|-------|-------------|
| **Usuario** | Persona que usa la agenda para gestionar sus contactos |

---

## 3. Casos de Uso

### UC-01 · Añadir contacto
- **Actor:** Usuario
- **Precondición:** —
- **Flujo principal:**
  1. El usuario introduce un nombre y un número de teléfono.
  2. El sistema valida los datos.
  3. El sistema guarda el contacto y lo muestra en la lista.
- **Flujos alternativos:**
  - Si el nombre está vacío → error: "El nombre es obligatorio".
  - Si el número tiene formato inválido → error: "Número no válido".
  - Si ya existe un contacto con el mismo nombre → advertencia de duplicado.

### UC-02 · Listar contactos
- **Actor:** Usuario
- **Flujo principal:**
  1. Al abrir la app, se muestran todos los contactos ordenados alfabéticamente.
  2. Cada contacto muestra nombre y número.

### UC-03 · Buscar contacto
- **Actor:** Usuario
- **Flujo principal:**
  1. El usuario escribe en el campo de búsqueda.
  2. La lista se filtra en tiempo real por nombre o número.

### UC-04 · Editar contacto
- **Actor:** Usuario
- **Flujo principal:**
  1. El usuario selecciona un contacto.
  2. Modifica nombre y/o número.
  3. El sistema valida y guarda los cambios.

### UC-05 · Eliminar contacto
- **Actor:** Usuario
- **Flujo principal:**
  1. El usuario pulsa "Eliminar" en un contacto.
  2. El sistema pide confirmación.
  3. El sistema elimina el contacto de la lista.

---

## 4. Modelo de Datos

```
Contact {
  id:        string   // UUID generado al crear
  name:      string   // Nombre completo del contacto (1–100 chars)
  phone:     string   // Número de teléfono (formato E.164 o local)
  createdAt: ISO8601  // Fecha de creación
  updatedAt: ISO8601  // Fecha de última modificación
}
```

---

## 5. Reglas de Negocio

| ID | Regla |
|----|-------|
| RN-01 | El nombre es obligatorio y debe tener entre 1 y 100 caracteres. |
| RN-02 | El número es obligatorio. Se aceptan formatos: `+34 612 345 678`, `612345678`, `(612) 345-678`. |
| RN-03 | No se permiten dos contactos con exactamente el mismo nombre (insensible a mayúsculas). Se avisa al usuario, pero puede continuar. |
| RN-04 | Los contactos se ordenan alfabéticamente por nombre (A→Z) por defecto. |
| RN-05 | La búsqueda filtra por nombre o número, insensible a mayúsculas y acentos. |
| RN-06 | La eliminación requiere confirmación explícita del usuario. |

---

## 6. Requisitos No Funcionales

| ID | Requisito |
|----|-----------|
| RNF-01 | La app debe funcionar sin backend: persistencia en `localStorage`. |
| RNF-02 | Tiempo de respuesta < 100 ms para todas las operaciones. |
| RNF-03 | Diseño responsive: usable en móvil (≥ 320 px) y escritorio. |
| RNF-04 | Accesibilidad mínima: etiquetas ARIA, contraste WCAG AA. |
| RNF-05 | Sin dependencias externas de red en runtime (funciona offline). |

---

## 7. Pantallas / Estados UI

```
┌─────────────────────────────┐
│  🔍 Buscar contacto         │
├─────────────────────────────┤
│  [+ Nuevo contacto]         │
├─────────────────────────────┤
│  A                          │
│    Ana García  · 612 000 001│
│  B                          │
│    Blas López  · 622 000 002│
│  ...                        │
└─────────────────────────────┘

Modal "Nuevo / Editar contacto":
┌─────────────────────────────┐
│  Nombre:  [______________]  │
│  Teléfono:[______________]  │
│        [Cancelar] [Guardar] │
└─────────────────────────────┘
```

---

## 8. Criterios de Aceptación

| ID | Criterio |
|----|----------|
| CA-01 | Un contacto creado aparece inmediatamente en la lista ordenada. |
| CA-02 | Buscar "ana" devuelve todos los contactos cuyo nombre o teléfono contiene "ana". |
| CA-03 | Intentar guardar un contacto sin nombre muestra un mensaje de error visible. |
| CA-04 | Tras eliminar un contacto con confirmación, desaparece de la lista. |
| CA-05 | Al recargar la página, los contactos persisten (localStorage). |
| CA-06 | La app es usable con teclado (Tab, Enter, Escape en modales). |
