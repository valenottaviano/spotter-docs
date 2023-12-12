Bienvenido al futuro del *parking* sin estrés con nuestra innovadora startup dedicada a revolucionar tu experiencia de aparcamiento. El núcleo de nuestra innovación es una tecnología de código QR de última generación integrada a la perfección en una aplicación Next.js, que facilita validaciones de usuario rápidas y seguras. Atrás quedaron los días de la tediosa búsqueda de lugares de aparcamiento; nuestros algoritmos de IA elevan la optimización del aparcamiento a niveles sin precedentes, gestionando de forma inteligente las plazas para lograr la máxima eficiencia.

## Key Features:

- **Tecnología de código QR:** Nuestro sistema emplea tecnología avanzada de código QR para validaciones de usuario seguras y sin problemas, agilizando el proceso de entrada.

- **Integración de la aplicación Next.js:** El corazón de nuestra solución se encuentra en una aplicación Next.js, que proporciona una interfaz fácil de usar para una interacción sin esfuerzo con nuestro sistema de aparcamiento.

## System Overview

![[Pasted image 20231112153009.png]]
## 1. Aplicación web fullstack (Next.js)

Nuestro sistema lector de códigos QR se basa en una robusta aplicación web fullstack desarrollada utilizando el framework Next.js. Alojada en Vercel, esta aplicación sirve como interfaz para interacciones fluidas con el usuario. Los componentes clave incluyen:

- **Marco Next.js:** La aplicación está programada utilizando el marco Next.js, proporcionando un entorno de desarrollo potente y eficiente.

- **Alojamiento Vercel:** Todo el sistema está alojado en Vercel, lo que garantiza su fiabilidad y escalabilidad.

- **Numerosos Endpoints:** La aplicación expone múltiples endpoints para facilitar la integración con sistemas externos.

- **Componente de lectura de códigos QR:** Un componente especializado dentro de la aplicación se dedica a la lectura y validación de códigos QR, garantizando un proceso de entrada seguro y eficiente.

## 2. Proxy de validación de códigos QR

Para validar los códigos QR, nuestro sistema emplea un proxy que se comunica con el sitio "UNSTA", verificando la información extraída por el componente lector de códigos QR. Las principales características son:

- **Comunicación con el sitio "UNSTA":** El proxy de validación establece comunicación con el sitio "UNSTA", garantizando la validación en tiempo real de la información del código QR.

- **Transmisión segura de datos:** Todos los datos intercambiados entre el componente lector de códigos QR y el sitio "UNSTA" se transmiten de forma segura a través del proxy de validación.

- **Validación en tiempo real:** El proxy facilita la validación en tiempo real, confirmando la autenticidad de la información del código QR antes de conceder el acceso.

## 3. Base de datos para el historial de transacciones

![[Pasted image 20231112153825.png]]

Nuestro sistema mantiene una completa base de datos que almacena todas las transacciones, proporcionando valiosos datos históricos para la gestión del aparcamiento. Los atributos clave de la base de datos incluyen:

- **Registro de transacciones:** Cada evento de entrada y salida se registra, creando un registro histórico de las transacciones de aparcamiento.

- Análisis de datos históricos:** La base de datos facilita el análisis detallado de los patrones y tendencias de aparcamiento a lo largo del tiempo.

- Integridad de los datos:** Los sólidos mecanismos de almacenamiento de datos garantizan la integridad y fiabilidad de los datos históricos de aparcamiento.
