---
Title: Aplicación para la automatización de guardias del Instituto
Author: Miguel Núñez
Date: 22/04/2026
Category: Desarrollo en Python y PHP
Tags: PHP, Python
---
# Aplicación para la automatización de guardias del Instituto
Aplicación para la automatización de guardias del Instituto desarrollada en Python, PHP y base de datos MariaDB

## Índice

## Estructura de la Base de Datos
La tabla principal de nuestra aplicación tiene el siguiente formato:

| Campo | Tipo | Descripción |
| :--- | :--- | :--- |
| `Tabla profesorado`  |
| :--- | :--- | :--- |
| `id` | Integer | Clave primaria autoincremental |
| `profesor` | String | Nombre de profesor (único) |
| `horario` | String | String con horas |
| `dia_semana` | Integer | Día de la semana de guardia |
