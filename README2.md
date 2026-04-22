import os

# Contenido del archivo Markdown solicitado
markdown_content = """---
title: "Ejercicio Práctico: Desarrollo de una Agenda en Python"
author: "Departamento de Informática"
date: "2024-05-20"
tags: [Python, Programación, Estructuras de Datos, Markdown]
version: 1.0
---

# Guía de Ejercicio: Creación de una Agenda Telefónica

En este ejercicio aprenderás a estructurar una aplicación básica de consola utilizando Python para gestionar contactos (nombre y número).

---

## ÍNDICE

1. [Introducción](#introducción)
2. [Requisitos del Sistema](#requisitos)
3. [Estructura del Proyecto](#estructura)
4. [Diagrama de Flujo](#diagrama)
5. [Implementación del Código](#código)
6. [Criterios de Evaluación](#evaluación)

---

## 1. Introducción <a name="introducción"></a>

El objetivo de esta práctica es desarrollar una **Agenda de Contactos**. Los alumnos deberán aplicar conceptos de *bucles*, *diccionarios* y *manejo de excepciones*.

> **Nota importante:** La persistencia de datos (guardar en archivo) se tratará en la siguiente unidad didáctica[^1].

---

## 2. Requisitos del Sistema <a name="requisitos"></a>

Para realizar este ejercicio, asegúrate de cumplir con lo siguiente:

* **Lenguaje:** Python 3.8 o superior.
* **Editor:** Visual Studio Code o PyCharm.
* **Conocimientos previos:**
    - Manipulación de listas y diccionarios.
    - Definición de funciones (`def`).

Puedes consultar la documentación oficial en este [enlace a Python.org](https://docs.python.org/3/).

---

## 3. Estructura del Proyecto <a name="estructura"></a>

El proyecto debe contener al menos las siguientes funciones:

| Función | Descripción |
| :--- | :--- |
| `añadir_contacto()` | Solicita nombre y teléfono y lo guarda en el diccionario. |
| `buscar_contacto()` | Permite localizar un número mediante el nombre. |
| `mostrar_todos()` | Imprime la lista completa de contactos en pantalla. |
| `menu()` | Gestiona la interfaz de usuario por consola. |

---

## 4. Diagrama de Flujo <a name="diagrama"></a>

A continuación se muestra la lógica de navegación del programa:

```mermaid
graph TD
    A[Inicio] --> B{¿Seleccionar Opción?}
    B -->|1| C[Añadir Contacto]
    B -->|2| D[Buscar Contacto]
    B -->|3| E[Ver Todos]
    B -->|4| F[Salir]
    C --> B
    D --> B
    E --> B
    F --> G[Fin]
