<h1 align="center">🌸 API REST de Tareas Personales 🌸</h1>

<p align="center">
  Proyecto CRUD con FastAPI + SQLite + Interfaz Web 
</p>

𝙳𝚎𝚜𝚌𝚛𝚒𝚙𝚌𝚒ó𝚗

Este proyecto consiste en el desarrollo de una **API REST** para la administración de tareas personales, implementando operaciones CRUD (Crear, Leer, Actualizar y Eliminar).

Incluye una **interfaz web con HTML, CSS (tema rosita)** y JavaScript para interactuar con la API.

---

𝙾𝚋𝚓𝚎𝚝𝚒𝚟𝚘

Desarrollar una API REST funcional que permita:

* Crear tareas
* Consultar tareas
* Actualizar tareas
* Eliminar tareas

---

𝚃𝚎𝚌𝚗𝚘𝚕𝚘𝚐í𝚊𝚜 𝚞𝚝𝚒𝚕𝚒𝚣𝚊𝚍𝚊𝚜

* Python
* FastAPI
* Uvicorn
* SQLite
* SQLAlchemy
* HTML
* CSS
* JavaScript
* Jinja2

---

𝙴𝚜𝚝𝚛𝚞𝚌𝚝𝚞𝚛𝚊 𝚍𝚎𝚕 𝚙𝚛𝚘𝚢𝚎𝚌𝚝𝚘

```
api_tareas/
│
├── main.py
├── database.py
├── models.py
├── schemas.py
├── tareas.db
│
├── templates/
│   └── index.html
│
└── static/
    └── styles.css
```

---

𝙸𝚗𝚜𝚝𝚊𝚕𝚊𝚌𝚒ó𝚗 𝚢 𝚎𝚓𝚎𝚌𝚞𝚌𝚒ó𝚗

### 1. Crear proyecto

```bash
mkdir api_tareas
cd api_tareas
```

### 2. Crear entorno virtual

```bash
python -m venv venv
```

Activar en Windows:

```bash
venv\Scripts\activate
```

### 3. Instalar dependencias

```bash
pip install fastapi uvicorn sqlalchemy jinja2
```

### 4. Ejecutar servidor

```bash
uvicorn main:app --reload
```

Código del proyecto

---

## 📄 main.py

```python
from fastapi import FastAPI, Depends, Request
from sqlalchemy.orm import Session
import models, schemas
from database import SessionLocal, engine

from fastapi.templating import Jinja2Templates
from fastapi.responses import HTMLResponse
from fastapi.staticfiles import StaticFiles

app = FastAPI()

models.Base.metadata.create_all(bind=engine)

app.mount("/static", StaticFiles(directory="static"), name="static")

templates = Jinja2Templates(directory="templates")

@app.get("/", response_class=HTMLResponse)
def pagina_inicio(request: Request):
    return templates.TemplateResponse("index.html", {"request": request})

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.post("/tareas")
def crear_tarea(tarea: schemas.TareaCreate, db: Session = Depends(get_db)):
    nueva = models.Tarea(**tarea.dict())
    db.add(nueva)
    db.commit()
    db.refresh(nueva)
    return nueva

@app.get("/tareas")
def obtener_tareas(db: Session = Depends(get_db)):
    return db.query(models.Tarea).all()

@app.put("/tareas/{id}")
def actualizar_tarea(id: int, tarea: schemas.TareaCreate, db: Session = Depends(get_db)):
    tarea_db = db.query(models.Tarea).filter(models.Tarea.id == id).first()
    if tarea_db:
        tarea_db.titulo = tarea.titulo
        tarea_db.descripcion = tarea.descripcion
        tarea_db.estado = tarea.estado
        db.commit()
        return tarea_db
    return {"error": "No encontrada"}

@app.delete("/tareas/{id}")
def eliminar_tarea(id: int, db: Session = Depends(get_db)):
    tarea_db = db.query(models.Tarea).filter(models.Tarea.id == id).first()
    if tarea_db:
        db.delete(tarea_db)
        db.commit()
        return {"mensaje": "Eliminada"}
    return {"error": "No encontrada"}
```

---

## 📄 database.py

```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

DATABASE_URL = "sqlite:///./tareas.db"

engine = create_engine(DATABASE_URL, connect_args={"check_same_thread": False})

SessionLocal = sessionmaker(bind=engine)

Base = declarative_base()
```

---

## 📄 models.py

```python
from sqlalchemy import Column, Integer, String
from database import Base

class Tarea(Base):
    __tablename__ = "tareas"

    id = Column(Integer, primary_key=True, index=True)
    titulo = Column(String)
    descripcion = Column(String)
    estado = Column(String)
```

---

## 📄 schemas.py

```python
from pydantic import BaseModel

class TareaBase(BaseModel):
    titulo: str
    descripcion: str
    estado: str

class TareaCreate(TareaBase):
    pass

class Tarea(TareaBase):
    id: int

    class Config:
        from_attributes = True
```

---

## 📄 templates/index.html

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Mis Tareas 💖</title>
    <link rel="stylesheet" href="/static/styles.css">
</head>
<body>

<h1>🌸 Lista de Tareas 🌸</h1>

<form id="formulario">
    <input type="text" id="titulo" placeholder="Título" required>
    <input type="text" id="descripcion" placeholder="Descripción" required>
    <input type="text" id="estado" placeholder="Estado" required>
    <button type="submit">Agregar</button>
</form>

<ul id="lista"></ul>

<script>
const API = "http://127.0.0.1:8000/tareas";

async function cargarTareas() {
    const res = await fetch(API);
    const data = await res.json();

    const lista = document.getElementById("lista");
    lista.innerHTML = "";

    data.forEach(t => {
        lista.innerHTML += `
            <li>
                <b>${t.titulo}</b> - ${t.descripcion} (${t.estado})
                <button onclick="eliminar(${t.id})">❌</button>
            </li>
        `;
    });
}

document.getElementById("formulario").addEventListener("submit", async e => {
    e.preventDefault();

    const tarea = {
        titulo: document.getElementById("titulo").value,
        descripcion: document.getElementById("descripcion").value,
        estado: document.getElementById("estado").value
    };

    await fetch(API, {
        method: "POST",
        headers: {"Content-Type": "application/json"},
        body: JSON.stringify(tarea)
    });

    cargarTareas();
});

async function eliminar(id) {
    await fetch(`${API}/${id}`, { method: "DELETE" });
    cargarTareas();
}

cargarTareas();
</script>

</body>
</html>
```

---

## 📄 static/styles.css

```css
body {
    font-family: Arial;
    background-color: #ffe4ec;
    text-align: center;
}

h1 {
    color: #d63384;
}

form {
    margin: 20px;
}

input {
    padding: 8px;
    margin: 5px;
    border: 1px solid #ffb6c1;
    border-radius: 8px;
}

button {
    background-color: #ff69b4;
    color: white;
    border: none;
    padding: 8px 12px;
    border-radius: 8px;
    cursor: pointer;
}

button:hover {
    background-color: #ff1493;
}

li {
    list-style: none;
    background: white;
    margin: 10px auto;
    padding: 10px;
    width: 300px;
    border-radius: 10px;
    box-shadow: 0px 0px 5px pink;
}
```

---

𝙴𝚟𝚒𝚍𝚎𝚗𝚌𝚒𝚊𝚜
<h3 align="center">Interfaz del sistema y GET</h3>

<p align="center">
  <img src="https://github.com/Vivian-14/TASKS/blob/main/PRUEBAS/ninininie.png" width="700">
</p>

<h3 align="center">POST</h3>

<p align="center">
  <img src="https://github.com/Vivian-14/TASKS/blob/main/PRUEBAS/wfjsjf.png" width="700">
</p>

<h3 align="center">DELETE</h3>

<p align="center">
  <img src="https://github.com/Vivian-14/TASKS/blob/main/PRUEBAS/4444.png" width="700">
</p>

<h3 align="center">UPDATE</h3>

<p align="center">
  <img src="https://github.com/Vivian-14/TASKS/blob/main/PRUEBAS/333.png" width="700">
</p>
---

Autora

**Alondra Vianney Hernández Torres**
Grupo: **GTID152**

---
