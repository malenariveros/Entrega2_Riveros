# Entrega2_Riveros
Base de datos Restaurante.
Script: 


DROP DATABASE if exists restaurante; -- en caso de que la base de datos exista, la elimina
CREATE DATABASE restaurante; -- crea la base de datos 

USE restaurante; -- selecciona la base de datos con la que vamos a trabajar

-- creamos las tablas de la base de datos restaurante

CREATE TABLE clientes ( 
	id_clientes INT NOT NULL PRIMARY KEY,
    nombre VARCHAR (50) NOT NULL, 
    apellido VARCHAR(50) NOT NULL,
    telefono VARCHAR(15),
    email VARCHAR(45),
    fecha_nacimiento datetime
);
	
    
CREATE TABLE mesas (
	id_mesas INT auto_increment NOT NULL PRIMARY KEY,
    capacidad_pax INT,
    estado_mesas VARCHAR (50), -- indica si la mesa está libre, ocupada o sucia.
    ubicacion VARCHAR (50)
);

CREATE TABLE reservas (
	id_reservas INT NOT NULL auto_increment PRIMARY KEY,
	hora TIME NOT NULL,
    fecha DATE NOT NULL,
    cantidad_pax VARCHAR(50),
    id_mesas INT, 
    id_clientes INT,
    foreign key (id_mesas) references mesas (id_mesas),
    foreign key (id_clientes) references clientes (id_clientes)
);


CREATE TABLE comandas (
	id_comandas INT NOT NULL PRIMARY KEY,
    hora TIME NOT NULL,
    fecha DATE NOT NULL,
    detalle TEXT NOT NULL,
    observaciones TEXT, -- indica pedidos especiales de clientes, como el punto de una carne, alergias, menú vegano/sin tacc, etc.
    id_clientes INT,
    id_mesas INT,
    FOREIGN KEY (id_clientes) REFERENCES clientes (id_clientes),
    FOREIGN KEY (id_mesas) REFERENCES mesas (id_mesas)
);

-- eliminamos la columna "observaciones" y "detalle" de la tabla "comandas", que pasarán a estar en la tabla "detalle_comandas" 
-- que crearemos después de la tabla menu y todas la que referencian a la misma.
ALTER TABLE comandas
DROP observaciones;

ALTER TABLE comandas
DROP detalle;


CREATE TABLE menu (
	id_menu INT auto_increment PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL,
    descripcion VARCHAR(100),
    precio INT,
    tipo VARCHAR (50), -- indica si es un menú de pasos (pudiendo ser de 3, 5 o 7 pasos)
    tiempo_elaboracion TIME,
    id_comandas INT,
    FOREIGN KEY (id_comandas) REFERENCES comandas (id_comandas)
);

-- eliminamos la columna "tiempo_elaboracion", porque utilizaremos una funcion para calcularlo en base a cada plato y postre,
-- de esta manera evitamos tener que actualizar manualmente los tiempos de elaboración de cada menú en caso de quitar o modificar platos.

ALTER TABLE menu
DROP tiempo_elaboracion;

CREATE TABLE platos(
	id_platos INT auto_increment NOT NULL PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL,
    descripcion VARCHAR (100),
    categoria VARCHAR (50),
    tiempo_preparacion TIME,
    disponible TINYINT, -- indica disponibilidad en el menú
    id_menu INT,
    FOREIGN KEY (id_menu) REFERENCES menu (id_menu)
);

-- agregamos la columna "precio" a la tabla "platos"
ALTER TABLE platos
ADD precio DECIMAL(10, 2) NOT NULL;

CREATE TABLE bebidas(
	id_bebidas INT auto_increment NOT NULL PRIMARY KEY,
    nombre VARCHAR(50),
    descripcion VARCHAR(100),
    categoria VARCHAR(50), 
    disponible TINYINT,
    id_menu INT,
    FOREIGN KEY (id_menu) REFERENCES menu (id_menu)
);

-- agregamos la columna "precio" a la tabla "bebidas" 
ALTER TABLE bebidas
ADD precio DECIMAL(10, 2) NOT NULL;
 

CREATE TABLE postres (
	id_postre INT NOT NULL auto_increment PRIMARY KEY,
    nombre VARCHAR(100),
    descripcion VARCHAR(100),
    disponible TINYINT,
    id_menu INT, 
    FOREIGN KEY (id_menu) REFERENCES menu (id_menu)
);

-- agregamos la columna "tiempo_preparacion" a la tabla "postres", necesario para la funcion "tiempo_menu_pasos"
ALTER TABLE postres
ADD tiempo_preparacion TIME;

-- agregamos la columna "categoria" a la tabla "postres".

ALTER TABLE postres
ADD categoria VARCHAR(50);


ALTER TABLE postres
ADD precio INT;

-- creamos la tabla "detalles_comandas" que normaliza los datos de las tablas "comandas" y "menu" para 
-- facilitar el manejo de la tabla "facturas_clientes".

CREATE TABLE detalle_comandas (
    id_detalle_comandas INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
	id_comandas INT NOT NULL,
    id_platos INT,
    id_bebidas INT,
    id_menu INT,
    cantidad INT NOT NULL,
    observaciones TEXT, -- para especificaciones de los clientes/ pedidos especiales.
    FOREIGN KEY (id_comandas) REFERENCES comandas(id_comandas),
    FOREIGN KEY (id_platos) REFERENCES platos(id_platos),
    FOREIGN KEY (id_bebidas) REFERENCES bebidas(id_bebidas),
    FOREIGN KEY (id_menu) REFERENCES menu (id_menu)
);


CREATE TABLE facturas_clientes (
	id_facturas INT NOT NULL auto_increment PRIMARY KEY,
    monto_total INT NOT NULL,
    detalle TEXT NOT NULL,
    id_clientes INT,
    id_comandas INT, 
    FOREIGN KEY (id_clientes) REFERENCES clientes (id_clientes),
    FOREIGN KEY (id_comandas) REFERENCES comandas (id_comandas)
);

-- eliminamos la columna detalles de la tabla "facturas_clientes" para utilizarla
-- en una nueva tabla "detalle_facturas_clientes"

ALTER TABLE facturas_clientes
DROP detalle;

-- agregamos la columna "fecha" a la tabla "facturas_clientes"
ALTER TABLE facturas_clientes
ADD fecha TIME;


CREATE TABLE detalle_facturas_clientes (
    id_clientes INT NULL,
    id_facturas INT NOT NULL,
    id_detalle_comandas INT NOT NULL,
    FOREIGN KEY (id_clientes) REFERENCES clientes (id_clientes),
    FOREIGN KEY (id_facturas) REFERENCES facturas_clientes (id_facturas),
    FOREIGN KEY (id_detalle_comandas) REFERENCES detalle_comandas (id_detalle_comandas)
);



CREATE TABLE encuestas_satisfaccion (
	id_encuestas INT auto_increment primary key,
    comentarios TEXT,
    fecha DATE,
    hora TIME, 
    id_clientes INT,
    FOREIGN KEY (id_clientes) REFERENCES clientes (id_clientes)
);


CREATE TABLE proveedores (
	id_proveedores INT not null auto_increment PRIMARY KEY,
    nombre VARCHAR (50) NOT NULL,
    telefono VARCHAR (15) NOT NULL,
    direccion VARCHAR (50)
);

CREATE TABLE pedidos_proveedores(
	id_pedidos_proveedores INT not null auto_increment PRIMARY KEY,
    detalle TEXT NOT NULL,
    fecha DATE NOT NULL,
    id_proveedores INT,
    FOREIGN KEY (id_proveedores) REFERENCES proveedores (id_proveedores)
);

-- creamos la tabla "detalle_pedidos_proveedores" para almacenar el detalle de los pedidos 

CREATE TABLE detalle_pedidos_proveedores(
	id_detalle INT NOT NULL auto_increment PRIMARY KEY,
	id_proveedores INT NOT NULL,
    id_pedidos_proveedores INT NOT NULL,
    productos TEXT NOT NULL,
    cantidad INT NOT NULL,
    FOREIGN KEY (id_proveedores) REFERENCES proveedores (id_proveedores),
    FOREIGN KEY (id_pedidos_proveedores) REFERENCES pedidos_proveedores (id_pedidos_proveedores)
);


CREATE TABLE facturas_proveedores(
	id_facturas_proveedores INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    detalle TEXT,
    fecha DATE,
    monto INT,
    id_proveedores INT,
	id_pedidos_proveedores INT,
    FOREIGN KEY (id_proveedores) REFERENCES proveedores(id_proveedores),
    FOREIGN KEY (id_pedidos_proveedores) REFERENCES pedidos_proveedores (id_pedidos_proveedores)
); 

-- creamos la tabla "detalle_facturas_proveedores" para almacenar el detalle de las mismas.

CREATE TABLE detalle_facturas_proveedores(
    id_detalle INT auto_increment PRIMARY KEY,
    id_facturas_proveedores INT NOT NULL,
    productos TEXT,
    cantidad INT,
    precio INT,
    FOREIGN KEY (id_facturas_proveedores) REFERENCES facturas_proveedores (id_facturas_proveedores)
);

CREATE TABLE empleados (
	id_empleados INT auto_increment NOT NULL PRIMARY KEY,
    nombre VARCHAR (50),
    apellido VARCHAR(50),
    fecha_nacimiento DATE NOT NULL,
    tipo_documento VARCHAR(15) NOT NULL,
    numero_documento INT NOT NULL,
    direccion VARCHAR (100),
    email VARCHAR (50),
    telefono VARCHAR (15),
    fecha_contratacion DATE NOT NULL,
    puesto VARCHAR (50) NOT NULL,
    alergias TEXT
);

CREATE TABLE turnos_empleados (
	id_turnos INT auto_increment NOT NULL PRIMARY KEY,
    tipo VARCHAR (50), -- indica si es turno de mañana o tarde
    fecha DATE NOT NULL,
    hora_inicio TIME NOT NULL,
    hora_fin TIME NOT NULL,
    puesto VARCHAR (50) NOT NULL,
    observaciones VARCHAR (50), 
    id_empleados INT,
    FOREIGN KEY (id_empleados) REFERENCES empleados (id_empleados)
);


CREATE TABLE empleados_despedidos (
	id_empleados_despedidos INT NOT NULL auto_increment PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL,
    apellido VARCHAR(50) NOT NULL,
    fecha_contratacion DATE NOT NULL,
    fecha_despido DATE NOT NULL,
    motivo_despido TEXT NOT NULL,
    puesto VARCHAR(45),
    id_empleados INT,
    FOREIGN KEY (id_empleados) REFERENCES empleados (id_empleados)
);

-- creamos una vista que nos permita ver las resvar del día, hechas por los clientes. 

CREATE VIEW vista_reservas_diarias AS
SELECT r.id_reservas, r.hora, r.fecha, r.cantidad_pax, c.nombre AS nombre_cliente, c.apellido AS apellido_cliente
FROM reservas r
JOIN clientes c ON r.id_clientes = c.id_clientes
WHERE r.fecha = CURDATE();

-- creamos una vista que no de el estado de las mesas. 

SELECT
    m.id_mesas,
    m.estado_mesas,
    CASE
        WHEN r.id_reservas IS NOT NULL THEN 'Ocupada'
        ELSE 'Libre'
    END AS estado_actual
FROM mesas m
LEFT JOIN reservas r ON m.id_mesas = r.id_mesas
AND r.fecha = CURDATE();


-- inserción de datos para las tablas platos y postres.

INSERT INTO platos (nombre, descripcion, categoria, tiempo_preparacion, disponible, precio)
VALUES ('Gazpacho', 'Sopa fría de tomates de estación con pesto albahaca.', 'entrada', '00:05:00', '1', 9.99);

INSERT INTO platos (nombre, descripcion, categoria, tiempo_preparacion, disponible, precio)
VALUES ('Langostinos Jumbo', 'Langostinos en salsa vasca, cítrica a base de mariscos.', 'entrada', '00:10:00', '1', 14.99);

INSERT INTO platos (nombre, descripcion, categoria, tiempo_preparacion, disponible, precio)
VALUES ('Taco filet con salsa de vino', 'Acompañado de verduras de estación, con salsa de vino y frutos rojos.', 'principal', '00:15:00', '1', 19.99);

INSERT INTO platos (nombre, descripcion, categoria, tiempo_preparacion, disponible, precio)
VALUES ('Rotolo de espinaca', 'Pasta rellena de espinaca, con salsa de queso azul.', 'principal', '00:15:00', '1', 17.99);

INSERT INTO postres (nombre, descripcion, categoria, tiempo_preparacion, disponible, precio)
VALUES ('Panqueques frambeados', 'Rellenos de dulce de leche.', 'postre', '00:05:00', '1', 7.99);

INSERT INTO postres (nombre, descripcion, categoria, tiempo_preparacion, disponible, precio)
VALUES ('Nemesis', 'Lingote de chocolate semi amargo, con salsa de frutillas y frutos secos crocantes.', 'postre', '00:05:00', '1', 8.99);



-- creamos un stored procedure que nos de el detalle de las comandas.

DELIMITER //

CREATE PROCEDURE ver_detalle_comandas(IN comanda_id INT)
BEGIN
    SELECT dc.id_detalle_comandas, dc.id_comandas, dc.id_platos, dc.id_bebidas, dc.id_menu, dc.cantidad, dc.observaciones
    FROM detalle_comandas dc
    WHERE dc.id_comandas = comanda_id;
END //

DELIMITER ;


-- nos muestra el detalle de comanda, en este caso la comanda con id 1.
CALL ver_detalle_comandas(1);



-- creamos un trigger para actualizar el estado de las mesas, con las reservas. 

DELIMITER //

CREATE TRIGGER actualizar_estado_mesas
AFTER INSERT ON reservas
FOR EACH ROW
BEGIN
    UPDATE mesas
    SET estado_mesas = 'ocupada'
    WHERE id_mesas = NEW.id_mesas;
END;
//

DELIMITER ;

-- creamos una funcion que nos permita ver los platos más pedidos. 

CREATE FUNCTION platos_mas_vendidos()
RETURNS TABLE
AS
SELECT
  p.nombre AS nombre_plato,
  COUNT(*) AS veces_vendido
FROM detalle_comandas dc
JOIN platos p ON dc.id_platos = p.id_platos
GROUP BY p.nombre
ORDER BY veces_vendido DESC;
END;



-- creamos una vista para obtener las facturas pendientes a pagar de los clientes.alter

CREATE VIEW vista_facturas_pendientes AS
SELECT f.id_facturas, f.monto_total, f.fecha, c.nombre AS nombre_cliente, c.apellido AS apellido_cliente
FROM facturas_clientes f
JOIN clientes c ON f.id_clientes = c.id_clientes
WHERE f.pagada = 0;


-- creamos una vista para las ventas diarias.

CREATE VIEW vista_ventas_diarias AS
SELECT
    v.fecha AS fecha_venta,
    SUM(d.cantidad * (p.precio + b.precio + m.precio)) AS total_ventas
FROM ventas v
JOIN detalle_ventas d ON v.id_ventas = d.id_ventas
JOIN platos p ON d.id_platos = p.id_platos
LEFT JOIN bebidas b ON d.id_bebidas = b.id_bebidas
LEFT JOIN menu m ON d.id_menu = m.id_menu
WHERE v.fecha = CURDATE();


-- creamos una vista que nos diga los empleados por turno.

CREATE VIEW vista_empleados_por_turno AS
SELECT e.nombre, e.apellido, t.tipo AS turno, t.fecha, t.hora_inicio, t.hora_fin
FROM empleados e
JOIN turnos_empleados t ON e.id_empleados = t.id_empleados;
