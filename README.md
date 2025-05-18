# FastAPI---Tutorial
[![Ver video en YouTube](https://img.youtube.com/vi/dAQENEPAqsc/hqdefault.jpg)](https://www.youtube.com/watch?v=dAQENEPAqsc "Haz clic para reproducir")
# Tutorial Básico de FastAPI 
## ¿Qué es una API REST?

Antes de entrar en FastAPI, vamos a reapasar qué es una API REST:

Una **API** (Interfaz de Programación de Aplicaciones) es un conjunto de reglas que permite que diferentes aplicaciones se comuniquen entre sí. 

Una **API REST** es un tipo específico de API que utiliza el protocolo HTTP y sigue ciertos principios:

- Utiliza URLs (endpoints) para acceder a recursos
- Emplea métodos HTTP como GET, POST, PUT y DELETE para realizar operaciones
- Devuelve datos en formatos como JSON o XML
- No mantiene estado entre solicitudes (stateless)


## ¿Por qué FastAPI?

FastAPI es un framework hecho Python que nos facilita crear APIs REST. Es especialmente adecuado en esta intancia porque:

- Es fácil de aprender y usar
- Tiene excelente documentación
- Proporciona validación automática de datos
- Genera documentación interactiva sin esfuerzo adicional
- Es muy rápido (de ahí su nombre "Fast")

## Preparando nuestro entorno

### Paso 1: Asegúrate de tener Python instalado

Necesitamos Python 3.7 o superior. Para verificar tu versión:

```bash
python --version
```

### Paso 2: Crea un entorno virtual (recomendado)

Un entorno virtual nos permite tener un espacio aislado para nuestro proyecto:

```bash
# Crear el entorno virtual
python -m venv mi_entorno

# Activar el entorno virtual
# En Windows:
mi_entorno\Scripts\activate
# En macOS/Linux:
source mi_entorno/bin/activate
```

### Paso 3: Instala FastAPI y Uvicorn

```bash
pip install fastapi uvicorn
```

- **FastAPI** es nuestro framework para crear la API
- **Uvicorn** es el servidor que ejecutará nuestra aplicación

## Nuestro primer proyecto: Una API de tareas pendientes

Vamos a crear una API simple paso a paso para gestionar tareas pendientes (todo list).

### Paso 1: Crear la estructura del proyecto

Crea una carpeta para tu proyecto y dentro de ella un archivo llamado `main.py`.

### Paso 2: "Hola Mundo" en FastAPI

Abre `main.py` y escribe este código:

```python
# Importamos FastAPI
from fastapi import FastAPI

# Creamos una instancia (objeto) de FastAPI
app = FastAPI()

# Definimos nuestra primera "ruta" o "endpoint"
@app.get("/")
async def raiz():
    # Esta función se ejecutará cuando alguien visite la URL raíz
    return {"mensaje": "¡Hola, bienvenido a mi primera API!"}
```

### Paso 3: Ejecuta tu servidor

En la terminal, dentro de la carpeta de tu proyecto, ejecuta:

```bash
uvicorn main:app --reload
```

Explicación:
- `main`: el nombre del archivo (main.py)
- `app`: el nombre de la variable que contiene la instancia de FastAPI
- `--reload`: hace que el servidor se reinicie automáticamente cuando cambias el código

### Paso 4: ¡Prueba tu API!

1. Abre tu navegador y visita: http://127.0.0.1:8000/
2. Deberías ver: `{"mensaje": "¡Hola, bienvenido a mi primera API!"}`

¡Felicidades! Has creado tu primera API con FastAPI.

## Explorando la documentación automática

Una de las características más útiles de FastAPI es que genera documentación automáticamente.

Visita:
- http://127.0.0.1:8000/docs - Interfaz Swagger UI
- http://127.0.0.1:8000/redoc - Interfaz ReDoc

Estas páginas muestran todos los endpoints de tu API y te permiten probarlos directamente desde el navegador.

## Entendiendo los métodos HTTP

En una API REST, utilizamos diferentes métodos HTTP para diferentes operaciones:

- **GET**: Para obtener información (como leer un artículo)
- **POST**: Para crear nuevos recursos (como publicar un comentario)
- **PUT/PATCH**: Para actualizar recursos existentes (como editar un perfil)
- **DELETE**: Para eliminar recursos (como borrar una foto)

## Ampliando nuestra API: Gestión de tareas

Vamos a expandir nuestra API para gestionar tareas. Primero, definiremos un modelo de datos usando Pydantic:
https://docs.pydantic.dev/latest/

```python
from fastapi import FastAPI
from pydantic import BaseModel
from typing import Optional, List

app = FastAPI(title="API de Tareas", description="Una API sencilla para gestionar tareas")

# Modelo de datos - define la estructura de una tarea, que campos son obligatorios y que tipo de datos tiene, cuando refactoricemos lo llevaremos a schemas.
class Tarea(BaseModel):
    id: Optional[int] =
    titulo: str # estas anotaiciones nos indican por ejemplo que titulo es obligatorio y de tipo string, no sifnifica que para python sera asi, pero es lo validaremos con pydantic
    descripcion: Optional[str] = None  # descripcion es opcional y su valor default es None
    completada: bool = False

# Simulamos una base de datos usando una lista, debermos reeplczar esta lista por una base de datos real con persistencia
base_de_datos_tareas = []
contador_id = 1  # Para asignar IDs únicos a cada tarea
```

### Crear una tarea (POST) - VER ANEXO A

```python
@app.post("/tareas/", response_model=Tarea, summary="Crear una nueva tarea")
async def crear_tarea(tarea: Tarea):
    """
    Crea una nueva tarea en la lista de tareas.
    
    - **titulo**: El título de la tarea (obligatorio)
    - **descripcion**: Una descripción opcional de la tarea
    - **completada**: Si la tarea está completada (por defecto: False)
    """
    global contador_id
    
    # Asignamos un ID a la tarea
    tarea.id = contador_id
    contador_id += 1
    
    # Añadimos la tarea a nuestra "base de datos"
    base_de_datos_tareas.append(tarea)
    
    return tarea
```

### Obtener todas las tareas (GET)

```python
@app.get("/tareas/", response_model=List[Tarea], summary="Obtener todas las tareas")
async def obtener_tareas():
    """
    Devuelve la lista completa de tareas.
    """
    return base_de_datos_tareas
```

### Obtener una tarea específica (GET con parámetro de ruta)

```python
@app.get("/tareas/{tarea_id}", response_model=Tarea, summary="Obtener una tarea por ID")
async def obtener_tarea(tarea_id: int):
    """
    Busca y devuelve una tarea específica según su ID.
    
    - **tarea_id**: El ID de la tarea que se desea obtener
    """
    # Buscar la tarea con el ID proporcionado
    for tarea in base_de_datos_tareas:
        if tarea.id == tarea_id:
            return tarea
    
    # Si no encontramos la tarea, devolvemos un error 404
    raise HTTPException(status_code=404, detail=f"Tarea con ID {tarea_id} no encontrada")
```

¡Ups! Nos falta importar HTTPException. Actualiza la primera línea de importaciones:

```python
from fastapi import FastAPI, HTTPException
```

### Actualizar una tarea (PUT)

```python
@app.put("/tareas/{tarea_id}", response_model=Tarea, summary="Actualizar una tarea")
async def actualizar_tarea(tarea_id: int, tarea_actualizada: Tarea):
    """
    Actualiza una tarea existente.
    
    - **tarea_id**: El ID de la tarea que se desea actualizar
    - **tarea_actualizada**: Los nuevos datos de la tarea
    """
    # Buscar la tarea y actualizarla
    for i, tarea in enumerate(base_de_datos_tareas):
        if tarea.id == tarea_id:
            # Conservamos el ID original
            tarea_actualizada.id = tarea_id
            # Reemplazamos la tarea antigua con la actualizada
            base_de_datos_tareas[i] = tarea_actualizada
            return tarea_actualizada
    
    # Si no encontramos la tarea, devolvemos un error 404
    raise HTTPException(status_code=404, detail=f"Tarea con ID {tarea_id} no encontrada")
```

### Eliminar una tarea (DELETE)

```python
@app.delete("/tareas/{tarea_id}", response_model=Tarea, summary="Eliminar una tarea")
async def eliminar_tarea(tarea_id: int):
    """
    Elimina una tarea existente.
    
    - **tarea_id**: El ID de la tarea que se desea eliminar
    """
    # Buscar la tarea y eliminarla
    for i, tarea in enumerate(base_de_datos_tareas):
        if tarea.id == tarea_id:
            # Eliminamos la tarea y la devolvemos
            return base_de_datos_tareas.pop(i)
    
    # Si no encontramos la tarea, devolvemos un error 404
    raise HTTPException(status_code=404, detail=f"Tarea con ID {tarea_id} no encontrada")
```

## El código completo

Aquí está el código completo de nuestra API de tareas:

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import Optional, List

app = FastAPI(title="API de Tareas", description="Una API sencilla para gestionar tareas")

# Modelo de datos - define la estructura de una tarea
class Tarea(BaseModel):
    id: Optional[int] = None
    titulo: str
    descripcion: Optional[str] = None
    completada: bool = False

# Simulamos una base de datos usando una lista
base_de_datos_tareas = []
contador_id = 1  # Para asignar IDs únicos a cada tarea

# Ruta raíz
@app.get("/")
async def raiz():
    return {"mensaje": "¡Bienvenido a la API de Tareas!"}

# Crear una tarea
@app.post("/tareas/", response_model=Tarea, summary="Crear una nueva tarea")
async def crear_tarea(tarea: Tarea):
    """
    Crea una nueva tarea en la lista de tareas.
    
    - **titulo**: El título de la tarea (obligatorio)
    - **descripcion**: Una descripción opcional de la tarea
    - **completada**: Si la tarea está completada (por defecto: False)
    """
    global contador_id
    
    # Asignamos un ID a la tarea
    tarea.id = contador_id
    contador_id += 1
    
    # Añadimos la tarea a nuestra "base de datos"
    base_de_datos_tareas.append(tarea)
    
    return tarea

# Obtener todas las tareas
@app.get("/tareas/", response_model=List[Tarea], summary="Obtener todas las tareas")
async def obtener_tareas():
    """
    Devuelve la lista completa de tareas.
    """
    return base_de_datos_tareas

# Obtener una tarea específica
@app.get("/tareas/{tarea_id}", response_model=Tarea, summary="Obtener una tarea por ID")
async def obtener_tarea(tarea_id: int):
    """
    Busca y devuelve una tarea específica según su ID.
    
    - **tarea_id**: El ID de la tarea que se desea obtener
    """
    # Buscar la tarea con el ID proporcionado
    for tarea in base_de_datos_tareas:
        if tarea.id == tarea_id:
            return tarea
    
    # Si no encontramos la tarea, devolvemos un error 404
    raise HTTPException(status_code=404, detail=f"Tarea con ID {tarea_id} no encontrada")

# Actualizar una tarea
@app.put("/tareas/{tarea_id}", response_model=Tarea, summary="Actualizar una tarea")
async def actualizar_tarea(tarea_id: int, tarea_actualizada: Tarea):
    """
    Actualiza una tarea existente.
    
    - **tarea_id**: El ID de la tarea que se desea actualizar
    - **tarea_actualizada**: Los nuevos datos de la tarea
    """
    # Buscar la tarea y actualizarla
    for i, tarea in enumerate(base_de_datos_tareas):
        if tarea.id == tarea_id:
            # Conservamos el ID original
            tarea_actualizada.id = tarea_id
            # Reemplazamos la tarea antigua con la actualizada
            base_de_datos_tareas[i] = tarea_actualizada
            return tarea_actualizada
    
    # Si no encontramos la tarea, devolvemos un error 404
    raise HTTPException(status_code=404, detail=f"Tarea con ID {tarea_id} no encontrada")

# Eliminar una tarea
@app.delete("/tareas/{tarea_id}", response_model=Tarea, summary="Eliminar una tarea")
async def eliminar_tarea(tarea_id: int):
    """
    Elimina una tarea existente.
    
    - **tarea_id**: El ID de la tarea que se desea eliminar
    """
    # Buscar la tarea y eliminarla
    for i, tarea in enumerate(base_de_datos_tareas):
        if tarea.id == tarea_id:
            # Eliminamos la tarea y la devolvemos
            return base_de_datos_tareas.pop(i)
    
    # Si no encontramos la tarea, devolvemos un error 404
    raise HTTPException(status_code=404, detail=f"Tarea con ID {tarea_id} no encontrada")
```

## Probando nuestra API

1. Guarda el código anterior en `main.py`
2. Ejecuta el servidor: `uvicorn main:app --reload`
3. Abre el navegador y visita: http://127.0.0.1:8000/docs

En esta interfaz interactiva podrás:
- Crear nuevas tareas
- Ver la lista de tareas
- Obtener una tarea específica
- Actualizar tareas
- Eliminar tareas

## Pasos siguientes

implementa las siguientes mejoras:
1. **Bases de datos reales**: Conectar a PostgreSQL, MySQL o SQLite
2. **Parámetros de consulta**: Filtrar, ordenar y paginar resultados
5. **implementa un nuevo Endpoint con paginacion**

*Investigar SQLAlchemy https://www.sqlalchemy.org/ para estas tareas.


## Recursos adicionales

- [Documentación oficial de FastAPI](https://fastapi.tiangolo.com/)
- [Tutorial completo de FastAPI](https://fastapi.tiangolo.com/tutorial/)
- [Aprende más sobre APIs REST](https://restfulapi.net/)


### Anexo A · Anatomía de un endpoint en FastAPI

Este anexo complementa el tutorial principal y explica en detalle cómo funcionan los decoradores de ruta y sus parámetros en **FastAPI**, usando como ejemplo el endpoint que crea una tarea.

```python
@app.post(
    "/tareas/",
    response_model=Tarea,       # modelo de salida
    summary="Crear una nueva tarea"
)
async def crear_tarea(tarea: Tarea):
    ...
````

| Elemento                          | Propósito                                                                       | Puntos clave                                                                                                                  |
| --------------------------------- | ------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `@app.post`                       | Registra la operación como **HTTP POST**.                                       | Cada método HTTP tiene su decorador: `@app.get`, `@app.put`, `@app.delete`, etc.                                              |
| `"/tareas/"`                      | Ruta relativa sobre la que escucha el endpoint.                                 | Puedes incluir parámetros de ruta: `"/tareas/{tarea_id}"` asigna el valor entre llaves al parámetro `tarea_id` de la función. |
| `response_model=Tarea`            | Indica que la respuesta se serializa y valida con el modelo Pydantic **Tarea**. | • Filtra campos extra.<br>• Genera el esquema JSON en la documentación automática.                                            |
| `summary="Crear una nueva tarea"` | Título breve que aparece en **Swagger UI** / **ReDoc**.                         | Se pueden añadir `description`, `tags`, `status_code`, etc.                                                                   |

---

#### Flujo de una petición **POST /tareas/**

1. El cliente envía un JSON que coincide con el modelo **Tarea**.
2. FastAPI **convierte y valida** el cuerpo → entrega una instancia `Tarea` al parámetro `tarea`.
3. Tu lógica (guardar en BD, lista en memoria, etc.) se ejecuta.
4. Devuelves un objeto (o `dict`) → FastAPI lo **re-valida y serializa** con `response_model`.
5. Respuesta HTTP: cuerpo JSON con la tarea creada y código (por defecto **200 OK**, o **201** si lo estableces).

---

#### Parámetros adicionales útiles

| Parámetro                                           | Descripción rápida                                    | Ejemplo                                                            |
| --------------------------------------------------- | ----------------------------------------------------- | ------------------------------------------------------------------ |
| `status_code=201`                                   | Cambia el código devuelto (ideal en POST).            | `@app.post("/tareas/", status_code=201)`                           |
| `tags=["tareas"]`                                   | Agrupa endpoints bajo la misma pestaña en Swagger UI. | `tags=["tareas"]`                                                  |
| `description="..."`                                 | Texto largo para la documentación.                    | `description="Crea una tarea nueva y la añade a la base de datos"` |
| `deprecated=True`                                   | Marca el endpoint como obsoleto.                      | `deprecated=True`                                                  |
| `responses={404: {"description": "No encontrada"}}` | Esquemas/documentación de respuestas adicionales.     | `responses={404: {"model": ErrorModel}}`                           |

---

#### ¿Por qué `async def`?

* **Libera el hilo** mientras espera operaciones de E/S (consultas SQL, peticiones HTTP externas, etc.).
* Permite **manejar más clientes concurrentes** con los mismos recursos.
* No cambia la forma de llamar a la función: simplemente precede la definición con `async`.
