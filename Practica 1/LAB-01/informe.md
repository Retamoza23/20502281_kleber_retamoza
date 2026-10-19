# Informe Técnico: Estrategias de Privacidad, Anonimización y Retención de Datos

---

## 19. Gestión Avanzada de Privacidad y Ciclo de Vida de Datos

### 19.1 Clasificación
* **¿Qué datos requieren protección y por qué?**  
  Requieren protección todos aquellos datos clasificados como Información Personal Identificiciable (PII por sus siglas en inglés), datos financieros, credenciales de acceso y secretos de infraestructura. Su protección es fundamental para cumplir con normativas de privacidad, mitigar riesgos legales, evitar fraudes financieros y salvaguardar la reputación y confianza de los usuarios y de la organización.

### 19.2 Enmascaramiento
* **Qué se ocultó:** Fragmentos críticos de información sensible (como los dígitos centrales de tarjetas de crédito, contraseñas, o correos electrónicos).
* **Para quién:** Personal de soporte operativo o usuarios con privilegios estándar que no requieren ver el dato completo para realizar sus funciones.
* **Por qué:** Para minimizar la exposición de datos sensibles (*principio de minimización*) y evitar fugas accidentales por visualización en pantalla.
* **Qué mecanismo se utilizó:** Vistas dinámicas en SQL con funciones de manipulación de cadenas (ej. `SUBSTRING`, concatenación de asteriscos).

### 19.3 Seudonimización
* **Cómo se generaron los tokens:** Utilizando identificadores únicos irrepetibles (UUIDs) o funciones criptográficas con sal (*hashing* aleatorio) para reemplazar los identificadores directos.
* **Dónde se encuentra el mapa de correspondencia:** En una tabla de mapeo relacional altamente restringida y cifrada, separada físicamente de las tablas analíticas o operativas.
* **Quién debería poder acceder a él:** Exclusivamente administradores de seguridad o personal de cumplimiento normativo (*Compliance*) bajo estrictas auditorías.
* **Por qué se separaron los permisos:** Para aplicar el principio de segregación de funciones, impidiendo que analistas o usuarios comunes puedan correlacionar los datos seudonimizados con la identidad real de las personas.

### 19.4 Anonimización
* **Qué identificadores fueron eliminados:** Nombres completos, correos electrónicos, números telefónicos y RFCs directos.
* **Qué atributos fueron generalizados:** Rangos de edad en lugar de fechas de nacimiento exactas, y códigos postales truncados a nivel de región o estado.
* **Qué grupos fueron suprimidos:** Registros individuales correspondientes a valores atípicos (*outliers*) que pudieran facilitar la reidentificación por unicidad.
* **Qué riesgo de reidentificación permanece:** El riesgo de ataques por combinación de atributos mediante el cruce con bases de datos externas públicas (*linkage attacks*).

### 19.5 Retención
* **Qué criterio se utilizó:** Requerimientos legales, fiscales y normativos vigentes, además de la necesidad operativa de negocio.
* **Qué información debería conservarse:** Registros de auditoría financiera y transaccionales durante los plazos obligatorios establecidos por la ley (ej. 5 a 10 años).
* **Qué información podría eliminarse:** Datos de sesiones temporales, registros de logs obsoletos y perfiles de usuario inactivos tras un periodo determinado sin actividad.
* **Qué controles deberían existir antes de eliminarla:** Políticas de aprobación múltiple, verificación de retenciones legales activas (*legal holds*) y respaldos seguros previos a la destrucción.

### 19.6 Conclusiones Aplicadas
* **¿Qué técnica utilizarías para un área de soporte?**  
  **Enmascaramiento dinámico**, ya que permite al personal visualizar parcialmente la información necesaria para atender al cliente sin exponer datos críticos.
* **¿Qué técnica utilizarías para un área de analítica?**  
  **Seudonimización**, permitiendo a los analistas estudiar patrones de comportamiento a lo largo del tiempo sin conocer la identidad real de los usuarios.
* **¿Qué técnica utilizarías para publicar estadísticas?**  
  **Anonimización y agregación total**, garantizando que los datos sean completamente desvinculados de cualquier individuo.
* **¿Por qué ninguna de estas técnicas sustituye al control de acceso?**  
  Porque las técnicas de privacidad modifican o enmascaran el contenido y la estructura de los datos, pero no determinan quién tiene el derecho legítimo de conectarse, consultar o ejecutar comandos sobre los recursos de la base de datos (se requiere *Defense in Depth*).

---

## 20. Preguntas de Reflexión

1. **¿Cuál es la diferencia entre enmascaramiento y seudonimización?**  
   El enmascaramiento oculta visualmente una parte del dato original (ej. `****-1234`), mientras que la seudonimización sustituye el identificador por un token único, manteniendo la trazabilidad mediante un mapa de correspondencia seguro y reversible.

2. **¿Por qué la seudonimización no significa anonimización?**  
   Porque la seudonimización es un proceso reversible; los datos siguen vinculados a una persona física a través de una clave o tabla de mapeo, por lo que legalmente siguen considerándose datos personales.

3. **¿Por qué eliminar nombre y RFC no garantiza que un conjunto de datos sea anónimo?**  
   Debido a los cuasi-identificadores (como fecha de nacimiento, género y código postal), los cuales al cruzarse con fuentes de datos externas permiten reidentificar unívocamente a los individuos.

4. **¿Qué función tienen los cuasi-identificadores?**  
   Son atributos que no identifican directamente por sí solos, pero que en combinación con otros datos aumentan drásticamente el riesgo de reidentificación de una persona.

5. **¿Por qué es importante separar el mapa de seudonimización?**  
   Para garantizar que la posesión de los datos analíticos no otorgue acceso a las identidades reales, previniendo abusos o brechas de correlación directa.

6. **¿Qué principio de seguridad se aplica al impedir que el analista acceda al mapa?**  
   El **Principio de Menor Privilegio** y la **Separación de Funciones** (*Segregation of Duties*).

7. **¿Por qué una vista puede ser preferible a entregar acceso directo a la tabla?**  
   Porque permite aplicar lógica de enmascaramiento, filtrado y transformación de forma transparente y controlada, protegiendo la estructura interna de la tabla base.

8. **¿Qué diferencia existe entre proteger un dato para visualización y protegerlo contra acceso no autorizado?**  
   La protección de visualización altera cómo se presenta el dato (ej. enmascarando caracteres), mientras que el control de acceso regula quién tiene autorización para autenticarse y consultar el almacenamiento subyacente.

9. **¿Por qué la retención forma parte de la protección de datos?**  
   Porque conservar información indefinidamente incrementa de forma innecesaria la superficie de ataque y el impacto potencial ante una filtración de seguridad.

10. **¿Qué riesgo existe si una organización conserva información indefinidamente?**  
    Mayor exposición ante brechas de datos, acumulación de costos operativos y posibles sanciones legales por incumplimiento de normativas de privacidad que exigen la eliminación de datos una vez cumplida su finalidad.