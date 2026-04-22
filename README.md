# Sistema de gestión de eventos y calendarios colaborativos desarrollado para la comuna de Humberto Primo.
Contexto y Objetivo

Este sistema de calendario fue desarrollado para la comuna de Humberto Primo con el objetivo de centralizar y organizar la gestión de eventos a nivel regional. Surge ante la necesidad de contar con una herramienta que permita registrar, coordinar y dar seguimiento a actividades de manera estructurada, reemplazando procesos manuales y desorganizados.

La solución implementa un modelo de calendarios individuales y compartidos, facilitando la coordinación entre distintas áreas involucradas en cada evento. A través del sistema, es posible gestionar tareas, pedidos de recursos y responsabilidades específicas, mejorando la comunicación interna y reduciendo errores operativos.

Como resultado, el sistema permitió:

Unificar la planificación de eventos en una única plataforma accesible
Mejorar la visibilidad y trazabilidad de tareas asociadas a cada evento
Optimizar la coordinación entre equipos y responsables
Mantener un registro histórico preciso de actividades y decisiones

El código fuente no se encuentra disponible públicamente debido a su carácter privado, pero la solución se encuentra operativa y utilizada en un entorno real.

# README Técnico - Sistema HID (Gestión de Calendario y Eventos)

Este documento detalla la arquitectura, el stack tecnológico y el funcionamiento técnico del módulo de Calendario dentro del ecosistema HID (Humberto Primo Interconectado).

## 1. Descripción Técnica
El sistema es una **Web App Fullstack** diseñada para la gestión de eventos, coordinación de equipos y asignación de recursos. Funciona como un sistema centralizado de planificación colaborativa donde los usuarios pueden gestionar calendarios personales y grupales, realizar un seguimiento de tareas vinculadas a eventos y automatizar recurrencias.

## 2. Arquitectura del Sistema
El sistema sigue una arquitectura de **Cliente-Servidor (Monolito Modular)**:

*   **Frontend (SPA):** Desarrollado en React, se comunica con el servidor mediante una API RESTful. Implementa una gestión de estado local eficiente y renderizado dinámico de vistas temporales (Mes, Semana, Día).
*   **Backend (API REST):** Un servidor Node.js/Express que maneja la lógica de negocio, autenticación y comunicación con la base de datos.
*   **Capa de Datos:** Utiliza una base de datos relacional (SQLite) para garantizar la integridad de las relaciones entre usuarios, calendarios, eventos y tareas.
*   **Infraestructura:** El sistema está desplegado detrás de un proxy inverso (**Nginx**) que maneja el SSL y la terminación de tráfico, con **PM2** asegurando la alta disponibilidad de los procesos Node.js.

## 3. Stack Tecnológico

### Frontend
*   **Framework:** React (Vite como build tool)
*   **Estilos:** Tailwind CSS & Lucide React (Iconografía)
*   **Comunicación:** Axios para peticiones HTTP
*   **Tipografía:** Inter (Cargada dinámicamente para performance)

### Backend
*   **Runtime:** Node.js
*   **Framework:** Express.js
*   **Base de Datos:** SQLite (vía `sqlite3` driver)
*   **Seguridad:** JWT (JSON Web Tokens), Helmet, CORS, Rate-Limiter
*   **Procesamiento:** Multer (Uploads), Node-cron (Tareas programadas)

## 4. Componentes Principales

### Frontend
*   **Calendario.jsx:** Componente principal que implementa algoritmos de layout para eventos solapados en vistas horarias y gestión de estados de arrastrar y soltar (Drag & Drop) para edición rápida.
*   **Navigation.jsx:** Sistema de navegación lateral (Sidebar) que integra la gestión de múltiples calendarios y filtros activos.
*   **EventoDetalle.jsx:** Módulo de gestión granular que permite administrar el historial de cambios, asignación de tareas específicas y pedidos de recursos mediante un chat de persistencia local.

### Backend
*   **Rutas de Calendario (`/api/calendario`):** Implementa la lógica de permisos complejos (RBAC), permitiendo la distinción entre propietarios, coordinadores e invitados.
*   **Motor de Recurrencia:** Lógica integrada para la generación masiva de eventos basada en frecuencias (semanal, anual) con propagación de cambios a toda la serie.
*   **Sistema de Notificaciones:** Motor interno que genera alertas en tiempo real y notificaciones por correo electrónico (SMTP) ante invitaciones o asignación de tareas.

## 5. Flujo de Funcionamiento (Operación Típica)

1.  **Request:** El cliente (Frontend) envía una petición firmada con un JWT en el header `Authorization`.
2.  **Middleware:** El servidor valida el token, verifica los permisos del usuario para el calendario específico y sanitiza el payload para prevenir inyecciones.
3.  **Lógica de Negocio:** Se procesa la solicitud (ej: creación de evento). Si el evento es repetible, se inicia una transacción SQL para insertar múltiples registros vinculados por un `parent_id`.
4.  **Persistencia:** Se ejecutan las queries SQL en la base de datos SQLite.
5.  **Side Effects:** Se disparan promesas asíncronas para el envío de correos electrónicos y la creación de registros en la tabla de notificaciones.
6.  **Response:** El servidor retorna un objeto JSON con el estado de la operación y los datos actualizados.

## 6. Decisiones Técnicas Relevantes

*   **SQLite como motor de DB:** Se eligió por su portabilidad y baja latencia en entornos de un solo servidor, eliminando la sobrecarga de mantenimiento de un servicio de base de datos externo.
*   **Transacciones Manuales:** Se implementó `BEGIN TRANSACTION` en operaciones críticas (como el borrado en cascada manual) para asegurar la consistencia de datos, dado que SQLite requiere configuraciones específicas para `FOREIGN KEY` cascades.
*   **Polling de Notificaciones:** El sistema utiliza un intervalo de refresco de 30s en el frontend para sincronización de estados, priorizando simplicidad sobre la complejidad de WebSockets en la fase actual.
*   **Layout Engine en Frontend:** El cálculo de columnas para eventos solapados se realiza en el cliente para reducir la carga de procesamiento del servidor y mejorar la interactividad.

## 7. Escalabilidad y Mejoras Posibles

*   **Base de Datos:** Migración a PostgreSQL para soportar mayores niveles de concurrencia y bloqueos de fila más granulares.
*   **Tiempo Real:** Implementación de Socket.io para actualizaciones instantáneas de eventos y chat sin necesidad de polling.
*   **Caché:** Introducción de Redis para cachear consultas frecuentes de eventos y sesiones de usuario.
*   **Arquitectura:** Extracción de la lógica de negocio de los archivos de rutas hacia una capa de Servicios (Service Pattern) para mejorar la testeabilidad.

## 8. Seguridad

*   **RBAC (Role-Based Access Control):** Validación de permisos en cada endpoint basada en la relación usuario-calendario.
*   **Sanitización:** Middleware dedicado a la detección de patrones maliciosos y limpieza de `req.body`.
*   **Security Headers:** Implementación exhaustiva de Helmet para mitigar ataques XSS y Clickjacking.
*   **Rate Limiting:** Protección contra ataques de fuerza bruta y denegación de servicio a nivel de aplicación.
