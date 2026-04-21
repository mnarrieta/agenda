---
name: fastapi-spec-builder
description: >
  Genera código FastAPI (modelos, schemas Pydantic, servicios, routers y tests)
  a partir de specs YAML definidas bajo Spec Driven Development. Usa esta skill
  siempre que el usuario tenga un fichero spec/*.yaml o agents.md con operaciones
  CRUD definidas y quiera convertirlas en código Python real. También úsala cuando
  el usuario diga "genera el agente", "implementa la spec", "construye el endpoint"
  o "pasa la spec a código".
---

# FastAPI Spec Builder

Convierte specs YAML de operaciones CRUD en código FastAPI listo para producción,
siguiendo estrictamente la estructura definida en el fichero `agents.md`.

## Flujo de trabajo

1. **Leer la spec** — Localiza y parsea el fichero YAML de la operación solicitada.
2. **Ejecutar ASSERT** — Verifica dependencias antes de generar código.
3. **Generar BUILD** — Produce los ficheros en el orden correcto.
4. **Confirmar con el usuario** — Muestra un resumen de lo generado.

---

## Fase ASSERT (ejecutar siempre antes del BUILD)

Antes de generar cualquier fichero, verifica:

```python
# assert_checks.py — el agente ejecuta estas comprobaciones
checks = [
    "requirements.txt contiene: fastapi, sqlalchemy, pydantic[email], passlib[bcrypt], python-jose",
    "alembic está inicializado (existe alembic.ini)",
    "app/db/database.py existe con SessionLocal y engine definidos",
    "no hay conflictos de nombre en app/routers/users.py",
]
```

Si algún check falla → detener y reportar al usuario antes de continuar.

---

## Plantillas de generación

### Modelo SQLAlchemy (desde `spec/models/user.yaml`)

```python
# app/models/user.py
from sqlalchemy import Column, Integer, String, Boolean, DateTime, Enum
from sqlalchemy.sql import func
from app.db.database import Base

class User(Base):
    __tablename__ = "users"

    id           = Column(Integer, primary_key=True, index=True)
    name         = Column(String(100), nullable=False)
    email        = Column(String, unique=True, index=True, nullable=False)
    hashed_password = Column(String, nullable=False)
    role         = Column(Enum("admin", "editor", "viewer"), default="viewer")
    is_active    = Column(Boolean, default=True)
    created_at   = Column(DateTime(timezone=True), server_default=func.now())
    updated_at   = Column(DateTime(timezone=True), onupdate=func.now())
```

### Schema Pydantic (desde `request` y `responses` de la spec)

```python
# app/schemas/user_create.py
from pydantic import BaseModel, EmailStr, field_validator
import re

class UserCreate(BaseModel):
    name: str
    email: EmailStr
    password: str
    role: str = "viewer"

    @field_validator("password")
    def password_strength(cls, v):
        if len(v) < 8 or not re.search(r"[A-Z]", v) or not re.search(r"[0-9]", v):
            raise ValueError("Mínimo 8 caracteres, una mayúscula y un número")
        return v

    @field_validator("email")
    def normalize_email(cls, v):
        return v.lower()
```

### Servicio (lógica de negocio de la spec)

```python
# app/services/create_service.py
from sqlalchemy.orm import Session
from passlib.context import CryptContext
from app.models.user import User
from app.schemas.user_create import UserCreate
from fastapi import HTTPException

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def create_user(db: Session, user_in: UserCreate) -> User:
    existing = db.query(User).filter(User.email == user_in.email).first()
    if existing:
        raise HTTPException(status_code=409, detail="Email already registered")

    db_user = User(
        name=user_in.name,
        email=user_in.email,
        hashed_password=pwd_context.hash(user_in.password),
        role=user_in.role,
    )
    db.add(db_user)
    db.commit()
    db.refresh(db_user)
    return db_user
```

### Router (endpoint de la spec)

```python
# app/routers/users.py
from fastapi import APIRouter, Depends
from sqlalchemy.orm import Session
from app.db.database import get_db
from app.schemas.user_create import UserCreate
from app.schemas.user_response import UserResponse
from app.services.create_service import create_user

router = APIRouter(prefix="/users", tags=["users"])

@router.post("/", response_model=UserResponse, status_code=201)
def register_user(user_in: UserCreate, db: Session = Depends(get_db)):
    return create_user(db, user_in)
```

---

## Orden de generación por agente

| Agente | Ficheros generados | Depende de |
|---|---|---|
| `create_agent` | model, schema_create, schema_response, create_service, router, migración | — |
| `read_agent` | schema_response (extend), read_service, auth dependency | create_agent |
| `update_agent` | schema_update, update_service | create_agent + read_agent |
| `delete_agent` | delete_service, migración is_active | todos los anteriores |

---

## Reglas de la skill

- **Nunca generar código sin pasar la fase ASSERT.**
- **Nunca incluir `hashed_password` en ningún schema de respuesta.**
- Los nombres de ficheros deben coincidir exactamente con los definidos en `agents.md`.
- Si la spec tiene `version`, incluirla como comentario en la cabecera del fichero generado.
- Si hay ambigüedad en la spec, preguntar al usuario antes de asumir.

---

## Ejemplo de uso

**Usuario**: "Genera el `create_agent` de la spec"
**Skill**: Lee `spec/operations/create_user.yaml` → ejecuta ASSERT → genera los 6 ficheros del BUILD → muestra resumen.

**Usuario**: "Implementa el endpoint de borrado con soft delete"
