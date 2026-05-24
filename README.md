# DFS Minimalista: Sistema de Archivos Distribuido por Bloques

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue?logo=python&logoColor=white)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.110-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)

**DFS Minimalista** es un Sistema de Archivos Distribuido (DFS) minimalista, seguro y tolerante a fallos inspirado en arquitecturas de nivel industrial como **HDFS (Hadoop)** y **Google File System (GFS)**. Diseñado desde cero para garantizar la resiliencia de los datos, fragmentación inteligente y recuperación automática de desastres en clústeres distribuidos.

---

## Características Principales

* **Fragmentación dinámica (chunking):** Divide de forma automática los archivos masivos en bloques de tamaño configurable (por defecto: 64 MB) para ser distribuidos en múltiples nodos.
* **Tolerancia a fallos y replicación:** Cada bloque es replicado en al menos dos DataNodes distintos de forma nativa.
* **Self-healing (auto-curación):** El NameNode monitorea constantemente el clúster. Si un DataNode colapsa y se pierde una réplica, el sistema ordena de inmediato a un nodo sano que duplique el bloque huérfano para restaurar la seguridad.
* **Autenticación segura (JWT):** Acceso multi-usuario encriptado. Cada cliente opera en un ecosistema virtual privado.
* **Block reporting:** Los DataNodes escanean de forma autónoma sus discos y envían reportes de inventario regulares al NameNode, evitando registros fantasmas o corrupciones por desconexiones.
* **Integridad criptográfica (checksums MD5):** Prevención de corrupción física. Al descargar, el cliente audita criptográficamente cada bloque transferido contra la firma del NameNode; si encuentra daños, rechaza el bloque e invoca una copia sana en el nodo de respaldo.
* **Docker-native:** Completamente dockerizado. Listo para orquestarse en la nube mediante un simple `docker compose`.

---

## Arquitectura del Sistema (Master-Workers)

DFS Minimalista funciona bajo un modelo `Master-Worker` compuesto por tres actores principales comunicados mediante **API REST**:

1. **NameNode (Master):** El cerebro central. Almacena todos los metadatos (jerarquías de carpetas, tamaños, propietarios, hashes MD5 y mapa de bloques). Utiliza SQLite en modo WAL para transacciones ACID y delega la carga física a los trabajadores.
2. **DataNodes (Workers):** Servidores de almacenamiento bruto. Reciben, guardan y envían fragmentos de archivos a los clientes, además de ejecutar un *garbage collector* para eliminar fragmentos de archivos borrados.
3. **Cliente (CLI):** Aplicación de consola interactiva en Python que simula una terminal (`ls`, `cd`, `mkdir`, `put`, `get`, `rm`) para orquestar subidas y bajadas hacia los nodos de forma transparente.

---

## Despliegue Rápido (Quickstart)

El proyecto está diseñado para levantarse en segundos mediante Docker Compose.

### 1. Clonar e iniciar el clúster

```bash
git clone https://github.com/jagfxx/DFS-minimalista.git
cd DFS-minimalista

# Levantar el NameNode central y 3 DataNodes
docker compose up -d --build
```

> El NameNode se expondrá en el puerto `8000`, y los DataNodes en `8001`, `8002` y `8003`.

### 2. Iniciar el cliente interactivo (CLI)

```bash
cd client
python3 -m venv venv
source venv/bin/activate   # En Windows: venv\Scripts\activate
pip install -r requirements.txt

# Ejecutar la terminal del DFS
python3 dfs_cli.py
```

### 3. Operaciones soportadas

Dentro de la consola `DFS >`, prueba los siguientes comandos:

- `register <user> <pass>` : Crea un usuario nuevo.
- `login <user> <pass>` : Inicia sesión en el clúster.
- `mkdir <carpeta>` : Crea un directorio virtual.
- `ls` : Lista archivos y directorios (muestra pesos exactos).
- `cd <carpeta>` : Navega por la jerarquía.
- `put <ruta_local> <nombre_remoto>` : Fragmenta y sube un archivo masivo al clúster de forma distribuida.
- `get <nombre_remoto> <ruta_local>` : Reconstruye y audita un archivo desde los múltiples nodos.
- `rm <nombre_remoto>` : Elimina el archivo y dispara el recolector de basura en los nodos.
- `rmdir <carpeta>` : Elimina un directorio vacío.

---

## Demostración de Tolerancia a Desastres

Puedes someter el sistema a pruebas extremas:

1. **Prueba de nodos caídos:** Mientras descargas un archivo masivo, detén un DataNode (`docker compose stop datanode1`). El cliente intercepta la falla de red y extrae los bloques faltantes de los nodos de réplica sobrevivientes sin interrumpir la descarga.
2. **Prueba de auto-curación:** Tras apagar un nodo, espera 30 segundos y observa los logs del NameNode (`docker compose logs -f namenode`). Verás al sistema detectar copias perdidas e invocar órdenes de replicación entre los nodos sanos.
3. **Prueba de corrupción física:** Entra al volumen de un DataNode y altera un bloque binario. Al intentar un `get`, el cliente reportará `Bloque corrupto detectado (MD5 mismatch)` y buscará automáticamente el bloque intacto en el nodo de respaldo.

---

*Desarrollado para el proyecto final de Arquitecturas de Nube y Sistemas Distribuidos (2026).*
