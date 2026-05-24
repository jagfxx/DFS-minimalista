# Sistema de Archivos Distribuido por Bloques (DFS Minimalista)

Este repositorio contiene el diseño e implementación de un sistema de archivos distribuido por bloques, desarrollado para el proyecto de Arquitecturas de Nube y Sistemas Distribuidos (2026).

## Objetivo General
Diseñar e implementar un DFS minimalista que permita almacenar y acceder a archivos masivos de forma distribuida, aplicando particionamiento en bloques y replicación (simulando sistemas como HDFS).

## Componentes del Sistema
- **NameNode (Master):** Servidor central de metadatos (gestiona la ubicación de los bloques).
- **DataNodes (Workers):** Servidores que almacenan físicamente los bloques de los archivos.
- **Cliente (CLI):** Interfaz para interactuar con el sistema (`put`, `get`, `ls`, `mkdir`, `rm`).

## Tecnologías a Utilizar
- Python (FastAPI)
- Docker (Contenedores)
- SQLite (Base de datos para metadatos)
- API REST para comunicación entre nodos

## Integrantes
- [Tu Nombre]
- [Nombre de tu Compañero]
