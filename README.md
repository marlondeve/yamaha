# yamaha
1: 


CREATE TABLE Departamentos (
    id_departamento INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
) ENGINE=InnoDB;

CREATE TABLE Ciudades (
    id_ciudad INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    id_departamento INT NOT NULL,
    FOREIGN KEY (id_departamento) REFERENCES Departamentos(id_departamento) ON DELETE RESTRICT ON UPDATE CASCADE
) ENGINE=InnoDB;

CREATE TABLE Tiendas (
    id_tienda INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    id_ciudad INT NOT NULL,
    FOREIGN KEY (id_ciudad) REFERENCES Ciudades(id_ciudad) ON DELETE RESTRICT ON UPDATE CASCADE
) ENGINE=InnoDB;

CREATE TABLE Generos (
    id_genero INT AUTO_INCREMENT PRIMARY KEY,
    descripcion VARCHAR(10) NOT NULL
) ENGINE=InnoDB;

CREATE TABLE Clientes (
    id_cliente INT AUTO_INCREMENT PRIMARY KEY,
    cedula VARCHAR(20) NOT NULL UNIQUE,
    nombres VARCHAR(100) NOT NULL,
    apellidos VARCHAR(100) NOT NULL,
    direccion VARCHAR(255),
    fecha_nacimiento DATE,
    genero_id INT NOT NULL,
    celular VARCHAR(20) NOT NULL,
    email VARCHAR(150) NOT NULL,
    FOREIGN KEY (genero_id) REFERENCES Generos(id_genero) ON DELETE RESTRICT ON UPDATE CASCADE
) ENGINE=InnoDB;

CREATE TABLE Vehiculos (
    id_vehiculo INT AUTO_INCREMENT PRIMARY KEY,
    modelo VARCHAR(50) NOT NULL,
    numero_motor VARCHAR(50) NOT NULL UNIQUE,
    cilindraje DECIMAL(5,2) NOT NULL,  -- Cambio a DECIMAL por si hay valores con decimales
    color VARCHAR(50) NOT NULL,
    fecha_ensamble DATE NOT NULL,
    modelo_anio INT NOT NULL
) ENGINE=InnoDB;

CREATE TABLE Ventas (
    id_venta INT AUTO_INCREMENT PRIMARY KEY,
    fecha DATE NOT NULL,
    numero_factura VARCHAR(50) NOT NULL UNIQUE,
    precio DECIMAL(12,2) NOT NULL,
    id_vehiculo INT NOT NULL,
    id_cliente INT NOT NULL,
    id_tienda INT NOT NULL,
    vendedor VARCHAR(100),
    FOREIGN KEY (id_vehiculo) REFERENCES Vehiculos(id_vehiculo) ON DELETE CASCADE ON UPDATE CASCADE,
    FOREIGN KEY (id_cliente) REFERENCES Clientes(id_cliente) ON DELETE CASCADE ON UPDATE CASCADE,
    FOREIGN KEY (id_tienda) REFERENCES Tiendas(id_tienda) ON DELETE RESTRICT ON UPDATE CASCADE
) ENGINE=InnoDB;

CREATE INDEX idx_clientes_cedula ON Clientes(cedula);
CREATE INDEX idx_vehiculos_numero_motor ON Vehiculos(numero_motor);
CREATE INDEX idx_ventas_numero_factura ON Ventas(numero_factura);
  return go(f, seed, [])
}


4:
CREATE VIEW ClientesFrecuentes AS
SELECT 
    c.id_cliente, 
    c.nombres, 
    c.apellidos, 
    v.modelo, 
    COUNT(v.id_vehiculo) AS total_compras, 
    ROUND(AVG(DATEDIFF(v2.fecha, v1.fecha)), 2) AS periodicidad_promedio,
    DATE_ADD(MAX(v2.fecha), INTERVAL ROUND(AVG(DATEDIFF(v2.fecha, v1.fecha))) DAY) AS proxima_compra_estimada
FROM Clientes c
JOIN Ventas v1 ON c.id_cliente = v1.id_cliente
JOIN Ventas v2 ON c.id_cliente = v2.id_cliente AND v1.id_vehiculo <> v2.id_vehiculo
JOIN Vehiculos v ON v1.id_vehiculo = v.id_vehiculo
WHERE c.id_cliente IN (
    SELECT id_cliente FROM Ventas GROUP BY id_cliente HAVING COUNT(id_vehiculo) > 2
)
GROUP BY c.id_cliente, c.nombres, c.apellidos, v.modelo;

