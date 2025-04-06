# GRUPO_2
--TEORIA BASE DE DATOS
--GRUPO2
--1. Creación de Tablas (Creating Tables)
-- Tabla principal de Empleados
CREATE TABLE empleados (
    empleado_id NUMBER PRIMARY KEY,
    nombre VARCHAR2(50) NOT NULL,
    apellido VARCHAR2(50) NOT NULL,
    fecha_contratacion DATE NOT NULL,
    salario NUMBER(10,2) NOT NULL,
    departamento_id NUMBER,
    cargo VARCHAR2(50),
    CONSTRAINT chk_salario CHECK (salario > 0)
);

-- Tabla de Departamentos
CREATE TABLE departamentos (
    departamento_id NUMBER PRIMARY KEY,
    nombre_departamento VARCHAR2(50) NOT NULL,
    ubicacion VARCHAR2(100),
    director_id NUMBER
);

-- Añadir la restricción de clave foránea
ALTER TABLE empleados 
ADD CONSTRAINT fk_departamento 
FOREIGN KEY (departamento_id) 
REFERENCES departamentos(departamento_id);

-- Tabla para el historial de cambios de salario
CREATE TABLE historial_salarios (
    historial_id NUMBER PRIMARY KEY,
    empleado_id NUMBER,
    salario_anterior NUMBER(10,2),
    salario_nuevo NUMBER(10,2),
    fecha_cambio DATE,
    usuario VARCHAR2(50),
    CONSTRAINT fk_empleado_historial FOREIGN KEY (empleado_id) REFERENCES empleados(empleado_id)
);

-- Secuencia para generar IDs automáticos para empleados
CREATE SEQUENCE seq_empleado_id
START WITH 1
INCREMENT BY 1
NOCACHE
NOCYCLE;

-- Secuencia para generar IDs automáticos para departamentos
CREATE SEQUENCE seq_departamento_id
START WITH 1
INCREMENT BY 1
NOCACHE
NOCYCLE;

-- Secuencia para generar IDs automáticos para historial de salarios
CREATE SEQUENCE seq_historial_id
START WITH 1
INCREMENT BY 1
NOCACHE
NOCYCLE;

////////////////////////////
///////////////////////////

---2. Creación de Triggers (Creating Triggers)
CREATE OR REPLACE TRIGGER trg_actualiza_salario
AFTER UPDATE OF salario ON empleados
FOR EACH ROW
BEGIN
    INSERT INTO historial_salarios (
        historial_id,
        empleado_id,
        salario_anterior,
        salario_nuevo,
        fecha_cambio,
        usuario
    ) VALUES (
        seq_historial_id.NEXTVAL,
        :OLD.empleado_id,
        :OLD.salario,
        :NEW.salario,
        SYSDATE,
        USER
    );
END;
/

-- Trigger para validar datos antes de insertar empleados
CREATE OR REPLACE TRIGGER trg_validar_empleado
BEFORE INSERT OR UPDATE ON empleados
FOR EACH ROW
BEGIN
    -- Validar que el salario sea positivo (ya existe como constraint pero es un ejemplo)
    IF :NEW.salario <= 0 THEN
        RAISE_APPLICATION_ERROR(-20001, 'El salario debe ser mayor que cero');
    END IF;
    
    -- Validar que la fecha de contratación no sea futura
    IF :NEW.fecha_contratacion > SYSDATE THEN
        RAISE_APPLICATION_ERROR(-20002, 'La fecha de contratación no puede ser futura');
    END IF;
END;
////////////////////
///////////////////
---3. Inserción de Datos (Inserting Data)
-- Insertar departamentos
INSERT INTO departamentos (departamento_id, nombre_departamento, ubicacion)
VALUES (seq_departamento_id.NEXTVAL, 'Recursos Humanos', 'Piso 1');

INSERT INTO departamentos (departamento_id, nombre_departamento, ubicacion)
VALUES (seq_departamento_id.NEXTVAL, 'Tecnología', 'Piso 2');

INSERT INTO departamentos (departamento_id, nombre_departamento, ubicacion)
VALUES (seq_departamento_id.NEXTVAL, 'Finanzas', 'Piso 3');

INSERT INTO departamentos (departamento_id, nombre_departamento, ubicacion)
VALUES (seq_departamento_id.NEXTVAL, 'Marketing', 'Piso 2');

-- Insertar empleados
INSERT INTO empleados (
    empleado_id,
    nombre,
    apellido,
    fecha_contratacion,
    salario,
    departamento_id,
    cargo
) VALUES (
    seq_empleado_id.NEXTVAL,
    'Juan',
    'Pérez',
    TO_DATE('15-01-2020', 'DD-MM-YYYY'),
    3500,
    1,
    'Analista de RRHH'
);


INSERT INTO empleados (
    empleado_id,
    nombre,
    apellido,
    fecha_contratacion,
    salario,
    departamento_id,
    cargo
) VALUES (
    seq_empleado_id.NEXTVAL,
    'María',
    'González',
    TO_DATE('22-03-2019', 'DD-MM-YYYY'),
    4200,
    2,
    'Desarrollador Senior'
);

INSERT INTO empleados (
    empleado_id,
    nombre,
    apellido,
    fecha_contratacion,
    salario,
    departamento_id,
    cargo
) VALUES (
    seq_empleado_id.NEXTVAL,
    'Carlos',
    'Rodríguez',
    TO_DATE('10-06-2021', 'DD-MM-YYYY'),
    3800,
    3,
    'Contador'
);

INSERT INTO empleados (
    empleado_id,
    nombre,
    apellido,
    fecha_contratacion,
    salario,
    departamento_id,
    cargo
) VALUES (
    seq_empleado_id.NEXTVAL,
    'Laura',
    'Martínez',
    TO_DATE('05-09-2020', 'DD-MM-YYYY'),
    3600,
    4,
    'Especialista en Marketing Digital'
);

-- Actualizar los directores de departamento
UPDATE departamentos SET director_id = 1 WHERE departamento_id = 1;
UPDATE departamentos SET director_id = 2 WHERE departamento_id = 2;
UPDATE departamentos SET director_id = 3 WHERE departamento_id = 3;
UPDATE departamentos SET director_id = 4 WHERE departamento_id = 4;

-- Añadir clave foránea para director
ALTER TABLE departamentos
ADD CONSTRAINT fk_director
FOREIGN KEY (director_id)
REFERENCES empleados(empleado_id);
/////////////////
///////////////
---4. Indexación de Columnas (Indexing Columns)
-- Índice para búsqueda por apellido
CREATE INDEX idx_empleados_apellido ON empleados(apellido);

-- Índice para búsqueda por departamento
CREATE INDEX idx_empleados_departamento ON empleados(departamento_id);

-- Índice compuesto para búsquedas combinadas
CREATE INDEX idx_empleados_nombre_apellido ON empleados(nombre, apellido);

-- Índice para búsquedas de salario
CREATE INDEX idx_empleados_salario ON empleados(salario);
////////////////////
///////////////////
---5. Consultas de Datos (Querying Data)
-- Consultar todos los empleados
SELECT * FROM empleados;

-- Consultar empleados con salario mayor a 3700
SELECT empleado_id, nombre, apellido, salario 
FROM empleados 
WHERE salario > 3700;

-- Consultar empleados por departamento
SELECT e.empleado_id, e.nombre, e.apellido, d.nombre_departamento
FROM empleados e
JOIN departamentos d ON e.departamento_id = d.departamento_id
ORDER BY d.nombre_departamento, e.apellido;

-- Consultar información del empleado y su director
SELECT e.nombre AS nombre_empleado, 
       e.apellido AS apellido_empleado,
       d.nombre_departamento,
       dir.nombre AS nombre_director,
       dir.apellido AS apellido_director
FROM empleados e
JOIN departamentos d ON e.departamento_id = d.departamento_id
JOIN empleados dir ON d.director_id = dir.empleado_id;
////////////////////
///////////////////
------6. Adición de Columnas (Adding Columns)
-- Añadir columna para correo electrónico
ALTER TABLE empleados
ADD correo_electronico VARCHAR2(100);

-- Añadir columna para número de teléfono
ALTER TABLE empleados
ADD telefono VARCHAR2(20);

-- Añadir columna para dirección
ALTER TABLE empleados
ADD direccion VARCHAR2(200);

-- Actualizar datos para las nuevas columnas
UPDATE empleados
SET correo_electronico = LOWER(nombre || '.' || apellido || '@empresa.com')
WHERE empleado_id > 0;
//////////
/////////
---7. Consulta del Diccionario de Datos de Oracle (Querying the Oracle Data Dictionary)
-- Consultar todas las tablas del esquema actual
SELECT table_name
FROM user_tables
ORDER BY table_name;

-- Consultar columnas de una tabla específica
SELECT column_name, data_type, data_length, nullable
FROM user_tab_columns
WHERE table_name = 'EMPLEADOS'
ORDER BY column_id;

-- Consultar restricciones de la tabla
SELECT constraint_name, constraint_type, search_condition
FROM user_constraints
WHERE table_name = 'EMPLEADOS';

-- Consultar índices de la tabla
SELECT index_name, index_type, uniqueness
FROM user_indexes
WHERE table_name = 'EMPLEADOS';

-- Consultar triggers asociados a la tabla
SELECT trigger_name, trigger_type, triggering_event
FROM user_triggers
WHERE table_name = 'EMPLEADOS';
//////////////////
/////////////////
---8. Actualización de Datos (Updating Data)
-- Actualizar el salario de un empleado (activará el trigger)
UPDATE empleados
SET salario = 3800
WHERE empleado_id = 1;

-- Actualizar el departamento de un empleado
UPDATE empleados
SET departamento_id = 2
WHERE empleado_id = 3;

-- Actualizar múltiples columnas
UPDATE empleados
SET telefono = '555-123-4567',
    direccion = 'Calle Principal 123'
WHERE empleado_id = 2;

-- Actualizar con base en una condición
UPDATE empleados
SET salario = salario * 1.05
WHERE departamento_id = 2;
/////////////
////////////
---9. Consultas Agregadas (Aggregate Queries)
-- Calcular el salario promedio de todos los empleados
SELECT AVG(salario) AS salario_promedio
FROM empleados;

-- Calcular el salario promedio por departamento
SELECT d.nombre_departamento, AVG(e.salario) AS salario_promedio
FROM empleados e
JOIN departamentos d ON e.departamento_id = d.departamento_id
GROUP BY d.nombre_departamento;

-- Contar empleados por departamento
SELECT d.nombre_departamento, COUNT(e.empleado_id) AS total_empleados
FROM empleados e
JOIN departamentos d ON e.departamento_id = d.departamento_id
GROUP BY d.nombre_departamento;

-- Encontrar el salario máximo y mínimo
SELECT 
    MIN(salario) AS salario_minimo,
    MAX(salario) AS salario_maximo,
    MAX(salario) - MIN(salario) AS diferencia
FROM empleados;

-- Agrupar y filtrar con HAVING
SELECT d.nombre_departamento, AVG(e.salario) AS salario_promedio
FROM empleados e
JOIN departamentos d ON e.departamento_id = d.departamento_id
GROUP BY d.nombre_departamento
HAVING AVG(e.salario) > 3700;
/////////////
////////////
---10. Compresión de Datos (Compressing Data)
-- Crear una tabla de historial con compresión
CREATE TABLE historial_extendido (
    id NUMBER PRIMARY KEY,
    empleado_id NUMBER,
    evento VARCHAR2(100),
    descripcion VARCHAR2(4000),
    fecha TIMESTAMP,
    usuario VARCHAR2(50)
) COMPRESS FOR OLTP;

-- Insertar algunos datos de ejemplo en la tabla comprimida
BEGIN
    FOR i IN 1..10 LOOP
        INSERT INTO historial_extendido VALUES (
            i,
            FLOOR(DBMS_RANDOM.VALUE(1, 5)),
            'CAMBIO_DATOS',
            'Descripción del evento ' || i,
            SYSTIMESTAMP - INTERVAL '1' DAY * i,
            USER
        );
    END LOOP;
    COMMIT;
END;
/

-- Aplicar compresión a tabla existente
ALTER TABLE historial_salarios COMPRESS FOR OLTP;

-- Mover datos a una partición comprimida 
CREATE TABLE historial_salarios_comp 
COMPRESS FOR OLTP 
AS SELECT * FROM historial_salarios;
////////////////////
/////////////////////
/////////////// --ocurre cuando intentas eliminar un registro que está siendo referenciado por otra tabla a través de una clave foránea.
--11. Eliminación de Datos (Deleting Data)
-- Eliminar un empleado específico
DELETE FROM empleados 
WHERE empleado_id = 4;
-- ocurre cuando intentas eliminar un registro que está siendo referenciado por otra tabla a través de una clave foránea.
-- Opción 1: Actualizar el departamento para que tenga otro director
UPDATE departamentos
SET director_id = NULL
WHERE director_id = 4;

-- Ahora sí puedes eliminar el empleado
DELETE FROM empleados 
WHERE empleado_id = 4;

-- Eliminar empleados de un departamento específico
-- Primero elimina las referencias
UPDATE departamentos
SET director_id = NULL
WHERE director_id = 4;

-- Ahora elimina el empleado
DELETE FROM empleados 
WHERE empleado_id = 4;

-- Comprobar si la papelera está habilitada
SELECT value FROM v$parameter WHERE name = 'recyclebin';

-- Para el resto de los ejemplos
-- Antes de eliminar empleados de un departamento, actualiza las referencias
UPDATE departamentos
SET director_id = NULL
WHERE director_id IN (SELECT empleado_id FROM empleados WHERE departamento_id = 4);

-- Ahora puedes eliminar los empleados del departamento

DELETE FROM empleados 
WHERE departamento_id = 4;

-- Eliminar registros con condición combinada

DELETE FROM historial_salarios
WHERE fecha_cambio < SYSDATE - 365;

-- Truncar tabla (eliminar todos los registros pero mantener estructura)
TRUNCATE TABLE historial_extendido;
////////////
////////////
-----12. Eliminación de Tablas (Dropping Tables)
-- Eliminar tabla de historial extendido
DROP TABLE historial_extendido;

-- Eliminar todo el esquema (todas las tablas en cascada)

BEGIN
    FOR cur_rec IN (SELECT table_name FROM user_tables) LOOP
        EXECUTE IMMEDIATE 'DROP TABLE ' || cur_rec.table_name || ' CASCADE CONSTRAINTS';
    END LOOP;
END;
/

----13. Restauración de Tablas Eliminadas (Un-dropping Tables)
-- Consultar la papelera de reciclaje
SELECT * FROM recyclebin;

-- Restaurar una tabla desde la papelera
FLASHBACK TABLE historial_extendido TO BEFORE DROP;

-- Purgar una tabla específica de la papelera
PURGE TABLE historial_extendido;

-- Purgar toda la papelera de reciclaje
PURGE RECYCLEBIN;
///////////////////// ---
-- 14. para la elimminacion sino funciona
-- Crear una tabla de prueba
CREATE TABLE tabla_prueba_recuperacion (
    id NUMBER PRIMARY KEY,
    descripcion VARCHAR2(100)
);

-- Insertar algunos datos
INSERT INTO tabla_prueba_recuperacion VALUES (1, 'Prueba de recuperación');

-- Verificar que la tabla existe
SELECT * FROM tabla_prueba_recuperacion;

-- Eliminar la tabla
DROP TABLE tabla_prueba_recuperacion;

-- Verificar la papelera de reciclaje
SELECT object_name, original_name, type 
FROM recyclebin;
-- Restaurar la tabla si aparece en la papelera
FLASHBACK TABLE tabla_prueba_recuperacion TO BEFORE DROP;
-- Verificar que la tabla se ha recuperado
SELECT * FROM tabla_prueba_recuperacion;

-- Crear tabla de respaldo
CREATE TABLE tabla_prueba_backup AS
SELECT * FROM tabla_prueba_recuperacion;
