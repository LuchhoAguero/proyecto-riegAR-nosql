# Proyecto riegAR: Sistema de Gestión y Cálculos de Riego

## 1. Contexto y Dominio del Problema
**riegAR** es un sistema diseñado para la gestión y monitoreo del riego en establecimientos agrícolas. Originalmente estructurado en un modelo relacional, el sistema administra perfiles de usuarios, fincas y cálculos hidráulicos (dimensiones de canales, caudales, etc.). 

El objetivo de migrar a una arquitectura de base de datos orientada a documentos (NoSQL) es optimizar al máximo las consultas de lectura. Esta nueva base de datos nos permitirá acceder a toda la configuración topográfica e hidráulica de una finca en una única consulta, sin necesidad de realizar múltiples "JOINs", lo cual mejora significativamente el rendimiento de la aplicación al mostrarle los datos al ingeniero o productor.

---

## 2. Esquema de Colecciones (Mockups)

Para esta fase, transformamos las entidades principales en colecciones de documentos JSON.

### Colección: Usuarios
El documento del usuario mantiene la información de autenticación y perfil, e incluye referencias a las fincas que administra.

```json
{
  "_id": "ObjectId('user_5')",
  "nombre": "jose",
  "email": "jose@gmail.com",
  "fincas_asignadas": [
    "ObjectId('finca_3')",
    "ObjectId('finca_4')"
  ],
  "created_at": "2025-11-27T22:21:27Z"
}
```

### Colección: Fincas
El documento de la finca concentra la información principal de ubicación y contiene, de forma anidada, todo su historial de cálculos hidráulicos para los canales de riego.

```json
{
  "_id": "ObjectId('finca_3')",
  "nombre_finca": "Agüero",
  "ubicacion": "irrazabal 450",
  "propietario_id": "ObjectId('user_5')",
  "calculos_hidraulicos": [
    {
      "nombre_calculo": "canal noroeste",
      "tipo_canal": "rectangular",
      "parametros": {
        "b": 2.0000,
        "h": 0.5000,
        "z": 1.0000,
        "n": 0.0120,
        "S": 0.000500
      },
      "resultados": {
        "A": 1.0000,
        "P": 3.0000,
        "Q_m3s": 0.895824
      }
    }
  ],
  "created_at": "2025-11-27T22:22:49Z"
}
```

---

## 3. Fundamentación Arquitectónica de la Lógica No Relacional

Basándonos en los patrones de acceso de la aplicación **riegAR**, hemos tomado las siguientes decisiones de modelado (Anidado vs. Referenciado):

*   **Datos Referenciados (Usuarios -> Fincas):** 
    Decidimos implementar el patrón de referencia (guardando el `ObjectId`) para vincular a los Usuarios con sus Fincas. La justificación técnica radica en evitar documentos con tamaño desproporcionado. Un usuario puede administrar múltiples establecimientos, y si anidáramos toda la configuración de la finca dentro del documento del usuario, impactaría negativamente en los tiempos de carga al consultar simplemente el perfil de la persona. Además, referenciar permite que en el futuro varios usuarios puedan compartir acceso a una misma finca sin duplicar datos.
    
*   **Datos Anidados (Fincas -> Cálculos):** 
    Decidimos utilizar el patrón *Embedded Documents* (anidamiento) para guardar los registros de los cálculos hidráulicos directamente dentro de la colección `Fincas`. Esto se debe a que un cálculo (como las dimensiones y el caudal de un canal) pertenece estrictamente a una finca específica y siempre se consulta en conjunto con esta. Al anidar estos datos, logramos que el frontend reciba toda la información técnica de la finca en una sola lectura a la base de datos, optimizando notablemente los tiempos de respuesta del sistema.<img width="1672" height="941" alt="ChatGPT Image 17 ago 2026, 13_27_15" src="https://github.com/user-attachments/assets/6b28559d-0aa3-4154-b26e-eb7ea5481500" />
