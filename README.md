#  AetherDFS: Sistema de Archivos Distribuido por Bloques

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue?logo=python&logoColor=white)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.110-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)

**AetherDFS** es un Sistema de Archivos Distribuido (DFS) minimalista, seguro y tolerante a fallos inspirado en arquitecturas de nivel industrial como **HDFS (Hadoop)** y **Google File System (GFS)**. Diseñado desde cero para garantizar la resiliencia de los datos, fragmentación inteligente y recuperación automática de desastres en clústeres distribuidos.

---

##  Características Principales

*  **Fragmentación Dinámica (Chunking):** Divide de forma automática los archivos masivos en bloques de tamaño configurable (Por defecto: 64MB) para ser distribuidos en múltiples nodos.
*  **Tolerancia a Fallos y Replicación:** Cada bloque es replicado en al menos dos DataNodes distintos de forma nativa. 
*  **Self-Healing (Auto-Curación):** El NameNode monitorea constantemente el clúster. Si un DataNode colapsa y se pierde una réplica, el sistema ordena de inmediato a un nodo sano que duplique el bloque huérfano para restaurar la seguridad.
*  **Autenticación Segura (JWT):** Acceso multi-usuario encriptado. Cada cliente opera en un ecosistema virtual privado.
*  **Block Reporting:** Los DataNodes escanean de forma autónoma sus discos y envían reportes de inventario regulares al NameNode, evitando registros fantasmas o corrupciones por desconexiones.
*  **Integridad Criptográfica (Checksums MD5):** Prevención de corrupción física. Al descargar, el cliente audita criptográficamente cada bloque transferido contra la firma digital del NameNode; si encuentra daños, rechaza el bloque e invoca una copia sana en microsegundos.
*  **Docker-Native:** Completamente dockerizado. Listo para orquestarse en la nube o AWS mediante un simple `docker-compose`.

---

##  Arquitectura del Sistema (Master-Workers)

AetherDFS funciona bajo un modelo `Master-Worker` compuesto por tres actores principales comunicados mediante **API REST** de alta velocidad:

1. **NameNode (Master):** El cerebro central. Almacena todos los metadatos (jerarquías de carpetas, tamaños, propietarios, Hashes MD5 y mapa de bloques). Utiliza SQLite en modo WAL para transacciones ácidas (ACID) y delega la carga física a los trabajadores.
2. **DataNodes (Workers):** Servidores de almacenamiento bruto. Reciben, guardan y envían fragmentos de archivos a los clientes, además de ejecutar un *Garbage Collector* para eliminar fragmentos de archivos borrados.
3. **Cliente (CLI):** Aplicación de consola interactiva en Python que simula una terminal nativa de Unix (`ls`, `cd`, `mkdir`, `put`, `get`, `rm`) para orquestar subidas y bajadas complejas hacia los nodos de forma transparente para el usuario final.

---

##  Despliegue Rápido (Quickstart)

El proyecto está diseñado para levantarse en segundos mediante Docker Compose.

### 1. Clonar e Iniciar el Clúster
```bash
git clone https://github.com/tu-usuario/aether-dfs.git
cd aether-dfs

# Levantar el NameNode central y 3 DataNodes perimetrales
docker compose up -d --build
```
> El NameNode se expondrá en el puerto `8000`, y los DataNodes mapearán sus puertos internamente de forma dinámica en `8001`, `8002` y `8003`.

### 2. Iniciar el Cliente Interactivo (CLI)
```bash
cd client
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Ejecutar la Terminal del DFS
python3 dfs_cli.py
```

### 3. Operaciones Soportadas
Dentro de la consola `AetherDFS >`, prueba los siguientes comandos:
- `register <user> <pass>` : Crea un usuario nuevo.
- `login <user> <pass>` : Inicia sesión en el clúster.
- `mkdir <carpeta>` : Crea un directorio virtual.
- `ls` : Lista archivos y directorios (Muestra pesos exactos).
- `cd <carpeta>` : Navega por la jerarquía.
- `put <ruta_local> <nombre_remoto>` : Fragmenta y sube un archivo masivo al clúster de forma distribuida.
- `get <nombre_remoto> <ruta_local>` : Reconstruye y audita un archivo desde los múltiples nodos.
- `rm <nombre_remoto>` : Elimina el archivo y dispara el recolector de basura en los nodos.
- `rmdir <carpeta>` : Elimina un directorio vacío.

---

##  Demostración de Tolerancia a Desastres

AetherDFS brilla cuando las cosas salen mal. Puedes someterlo a pruebas extremas:
1. **Prueba de Fuego (Nodos Caídos):** Mientras descargas un archivo masivo, detén un DataNode (`docker stop sistemas_distribuidos-datanode1-1`). Verás cómo el cliente intercepta la falla de red y extrae los bloques faltantes de los nodos de réplica sobrevivientes sin interrumpir la descarga.
2. **Prueba de Auto-Curación:** Tras apagar un nodo, espera 30 segundos y observa los logs del NameNode (`docker logs -f sistemas_distribuidos-namenode-1`). Verás al sistema darse cuenta de que perdió copias e invocar órdenes de replicación entre los nodos sanos.
3. **Prueba de Corrupción Física:** Entra a la fuerza bruta al disco de un DataNode y altera el texto de un bloque binario. Al intentar hacer un `get`, el cliente gritará ` Bloque corrupto detectado (MD5 mismatch)` y automáticamente buscará el bloque intacto en el nodo de respaldo.

---
*Desarrollado para el proyecto final de Arquitecturas de Nube y Sistemas Distribuidos (2026).*
