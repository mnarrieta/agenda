# architecture.md — Arquitectura Técnica: Agenda de Contactos

> Generado por: `architect`  
> Basado en: `SPEC.md` v1.0, `constitution.md` v1.0, `decisions.md` DEC-01..03  
> Fecha: 2026-04-21

---

## 1. Stack Tecnológico

| Capa | Tecnología | Justificación |
|------|------------|---------------|
| UI | React 18 | DEC-01 aprobada |
| Build | Vite 5 | DEC-01 aprobada |
| Lenguaje | JavaScript (JSDoc types) | Sin overhead de compilación TS para app pequeña |
| Estilos | CSS puro con custom properties | DEC-03 aprobada |
| Persistencia | localStorage | DEC-02 aprobada |
| Testing | Vitest + Testing Library | Ligero, compatible con Vite |

---

## 2. Estructura de Carpetas

```
agenda/
├── index.html
├── vite.config.js
├── package.json
│
└── src/
    ├── main.jsx                  # Entry point, monta <App />
    ├── App.jsx                   # Raíz: estado global + layout
    │
    ├── components/
    │   ├── SearchBar.jsx         # UC-03: campo de búsqueda
    │   ├── ContactList.jsx       # UC-02: lista agrupada A-Z
    │   ├── ContactItem.jsx       # UC-02: fila individual de contacto
    │   ├── ContactModal.jsx      # UC-01, UC-04: modal crear/editar
    │   └── ConfirmDialog.jsx     # UC-05: diálogo de confirmación
    │
    ├── hooks/
    │   ├── useContacts.js        # Estado + CRUD + persistencia
    │   └── useSearch.js          # Lógica de filtrado (UC-03)
    │
    ├── storage/
    │   └── contactStorage.js     # Encapsula localStorage (DEC-02)
    │
    ├── utils/
    │   ├── validators.js         # RN-01, RN-02: validaciones
    │   ├── sorter.js             # RN-04: ordenación A-Z
    │   └── normalizer.js         # RN-05: normalización para búsqueda
    │
    └── styles/
        ├── global.css            # Reset + custom properties
        ├── components.css        # Estilos de componentes
        └── modal.css             # Estilos de modales
```

---

## 3. Árbol de Componentes

```
<App>                          ← estado: contacts[], searchQuery, modal
  │
  ├── <SearchBar />            ← props: query, onChange
  │
  ├── <button> Nuevo contacto  ← onClick → abre ContactModal (modo CREATE)
  │
  ├── <ContactList />          ← props: contacts (filtrados + ordenados)
  │     └── <ContactItem />    ← props: contact, onEdit, onDelete
  │           ├── nombre + teléfono
  │           ├── [✏ Editar]  → abre ContactModal (modo EDIT)
  │           └── [🗑 Eliminar] → abre ConfirmDialog
  │
  ├── <ContactModal />         ← props: mode, contact|null, onSave, onClose
  │     ├── input Nombre
  │     ├── input Teléfono
  │     ├── mensajes de error (RN-01, RN-02)
  │     ├── [Cancelar]
  │     └── [Guardar]
  │
  └── <ConfirmDialog />        ← props: contact, onConfirm, onCancel
        ├── "¿Eliminar a [nombre]?"
        ├── [Cancelar]
        └── [Eliminar]
```

---

## 4. Modelo de Datos (Tipos)

```javascript
/**
 * @typedef {Object} Contact
 * @property {string} id          - UUID v4
 * @property {string} name        - Nombre completo (1–100 chars)
 * @property {string} phone       - Teléfono (formato libre validado)
 * @property {string} createdAt   - ISO 8601
 * @property {string} updatedAt   - ISO 8601
 */

/**
 * @typedef {'CREATE' | 'EDIT'} ModalMode
 */

/**
 * @typedef {Object} ValidationResult
 * @property {boolean} valid
 * @property {{ name?: string, phone?: string }} errors
 */
```

---

## 5. Interfaz del Storage (`contactStorage.js`)

Encapsula **todo** acceso a `localStorage`. Nadie más lo toca directamente.

```javascript
const STORAGE_KEY = 'agenda_contacts';

export const contactStorage = {
  /** @returns {Contact[]} */
  getAll()        // Lee y parsea localStorage

  /** @param {Contact[]} contacts */
  saveAll(contacts) // Serializa y guarda

  /** @param {Contact} contact */
  add(contact)    // getAll → push → saveAll

  /** @param {Contact} contact */
  update(contact) // getAll → map → saveAll

  /** @param {string} id */
  remove(id)      // getAll → filter → saveAll
}
```

---

## 6. Hook `useContacts.js`

Gestiona el estado en memoria y delega la persistencia en `contactStorage`.

```javascript
export function useContacts() {
  // state: Contact[]
  // Al montar: carga desde contactStorage.getAll()

  return {
    contacts,          // Contact[] (sin filtrar)
    addContact,        // (name, phone) → Contact  [UC-01]
    updateContact,     // (id, name, phone) → void [UC-04]
    deleteContact,     // (id) → void              [UC-05]
  }
}
```

---

## 7. Hook `useSearch.js`

```javascript
export function useSearch(contacts) {
  // state: query string

  return {
    query,             // string actual
    setQuery,          // actualiza el filtro
    filteredContacts,  // Contact[] filtrados por nombre o teléfono [UC-03, RN-05]
  }
}
```

---

## 8. Utilidades

### `validators.js` — Reglas RN-01 y RN-02

```javascript
export function validateContact(name, phone) → ValidationResult

// RN-01: name no vacío, máx 100 chars
// RN-02: phone regex: /^[+\d\s\(\)\-]{6,20}$/
// RN-03: duplicado → { valid: true, warning: 'duplicate' }
```

### `sorter.js` — Regla RN-04

```javascript
export function sortContacts(contacts) → Contact[]
// Ordena A-Z por name, insensible a mayúsculas
// Agrupa por primera letra para ContactList
export function groupByLetter(contacts) → Map<string, Contact[]>
```

### `normalizer.js` — Regla RN-05

```javascript
export function normalize(str) → string
// Minúsculas + quita acentos (NFD + replace /\p{Mn}/gu)
// Usado en useSearch para comparar query vs name/phone
```

---

## 9. Flujo de Datos

```
localStorage
    │ (al montar)
    ▼
useContacts → contacts[] → useSearch → filteredContacts[]
                                              │
                                              ▼
                                        ContactList → ContactItem[]
                                                          │
                                              ┌───────────┴───────────┐
                                              ▼                       ▼
                                        ContactModal           ConfirmDialog
                                              │                       │
                                        onSave(name,phone)      onConfirm(id)
                                              │                       │
                                              ▼                       ▼
                                       useContacts.add/update   useContacts.delete
                                              │
                                              ▼
                                       contactStorage.saveAll()
                                              │
                                              ▼
                                         localStorage
```

---

## 10. Criterios de Aceptación → Componentes responsables

| CA | Descripción | Responsable |
|----|-------------|-------------|
| CA-01 | Contacto creado aparece en lista ordenada | `useContacts` + `sorter.js` |
| CA-02 | Búsqueda filtra por nombre y teléfono | `useSearch` + `normalizer.js` |
| CA-03 | Sin nombre → error visible | `validators.js` + `ContactModal` |
| CA-04 | Eliminar con confirmación | `ConfirmDialog` + `useContacts` |
| CA-05 | Recarga persiste datos | `contactStorage` + `useContacts` (mount) |
| CA-06 | Navegable con teclado | `ContactModal` (Escape/Enter) + ARIA labels |

---

## 11. Decisiones Pendientes / Abiertas

| ID | Pregunta | Bloqueante |
|----|----------|------------|
| — | ¿Exportar/importar contactos (JSON)? | No, fuera de scope v1 |
| — | ¿Foto de contacto? | No, fuera de scope v1 |
| — | ¿Múltiples teléfonos por contacto? | No, SPEC define uno |
