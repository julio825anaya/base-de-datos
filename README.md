# base-de-datos
CREATE DATABASE plataforma_musical;
  
USE plataforma_musical;

CREATE TABLE grupo (
  nombre     VARCHAR(100) NOT NULL,
  fecha_cre  DATE         NOT NULL,
  PRIMARY KEY (nombre)
);

CREATE TABLE grupo_componente (
  nombre_grupo VARCHAR(100) NOT NULL,
  componente   VARCHAR(100) NOT NULL,
  PRIMARY KEY (nombre_grupo, componente),
  CONSTRAINT fk_componente_grupo
    FOREIGN KEY (nombre_grupo) REFERENCES grupo (nombre)
    ON UPDATE CASCADE ON DELETE CASCADE
);

CREATE TABLE album (
  id_album       INT          NOT NULL AUTO_INCREMENT,
  titulo         VARCHAR(150) NOT NULL,
  fecha_pub      DATE,
  num_canciones  INT          NOT NULL DEFAULT 0,
  nombre_grupo   VARCHAR(100) NOT NULL,
  PRIMARY KEY (id_album),
  CONSTRAINT chk_num_canciones CHECK (num_canciones >= 0),
  CONSTRAINT fk_album_grupo
    FOREIGN KEY (nombre_grupo) REFERENCES grupo (nombre)
    ON UPDATE CASCADE ON DELETE RESTRICT
);

CREATE TABLE cancion (
  id_cancion  INT          NOT NULL AUTO_INCREMENT,
  titulo      VARCHAR(150) NOT NULL,
  publica     DATE,
  PRIMARY KEY (id_cancion)
);

CREATE TABLE album_cancion (
  id_album    INT NOT NULL,
  id_cancion  INT NOT NULL,
  PRIMARY KEY (id_album, id_cancion),
  CONSTRAINT fk_ac_album
    FOREIGN KEY (id_album) REFERENCES album (id_album)
    ON UPDATE CASCADE ON DELETE CASCADE,
  CONSTRAINT fk_ac_cancion
    FOREIGN KEY (id_cancion) REFERENCES cancion (id_cancion)
    ON UPDATE CASCADE ON DELETE CASCADE
);

CREATE TABLE usuario (
  id_usuario  INT          NOT NULL AUTO_INCREMENT,
  correo      VARCHAR(150) NOT NULL,
  fecha_alta  DATE         NOT NULL,
  url         VARCHAR(255),
  PRIMARY KEY (id_usuario),
  CONSTRAINT uq_usuario_correo UNIQUE (correo)
);

CREATE TABLE registrado (
  id_usuario  INT           NOT NULL,
  cuota       DECIMAL(10,2) NOT NULL,
  fecha_fin   DATE,
  PRIMARY KEY (id_usuario),
  CONSTRAINT fk_registrado_usuario
    FOREIGN KEY (id_usuario) REFERENCES usuario (id_usuario)
    ON UPDATE CASCADE ON DELETE CASCADE
);

CREATE TABLE stream (
  id_cancion  INT      NOT NULL,
  id_usuario  INT      NOT NULL,
  fecha       DATETIME NOT NULL,
  PRIMARY KEY (id_cancion, id_usuario, fecha),
  CONSTRAINT fk_stream_cancion
    FOREIGN KEY (id_cancion) REFERENCES cancion (id_cancion)
    ON UPDATE CASCADE ON DELETE CASCADE,
  CONSTRAINT fk_stream_usuario
    FOREIGN KEY (id_usuario) REFERENCES usuario (id_usuario)
    ON UPDATE CASCADE ON DELETE CASCADE
);

CREATE TABLE descarga (
  id_cancion  INT  NOT NULL,
  id_usuario  INT  NOT NULL,
  fecha       DATE NOT NULL,
  n_veces     INT  NOT NULL DEFAULT 1,
  PRIMARY KEY (id_cancion, id_usuario),
  CONSTRAINT chk_n_veces CHECK (n_veces > 0),
  CONSTRAINT fk_descarga_cancion
    FOREIGN KEY (id_cancion) REFERENCES cancion (id_cancion)
    ON UPDATE CASCADE ON DELETE CASCADE,
  CONSTRAINT fk_descarga_registrado
    FOREIGN KEY (id_usuario) REFERENCES registrado (id_usuario)
    ON UPDATE CASCADE ON DELETE CASCADE
);

CREATE INDEX idx_stream_usuario ON stream (id_usuario);
CREATE INDEX idx_album_titulo ON album (titulo);

DELIMITER $$

CREATE TRIGGER trg_ac_insert
AFTER INSERT ON album_cancion
FOR EACH ROW
BEGIN
  UPDATE album
  SET num_canciones = num_canciones + 1
  WHERE id_album = NEW.id_album;
END$$

CREATE TRIGGER trg_ac_delete
AFTER DELETE ON album_cancion
FOR EACH ROW
BEGIN
  UPDATE album
  SET num_canciones = num_canciones - 1
  WHERE id_album = OLD.id_album;
END$$

DELIMITER ;

INSERT INTO grupo (nombre, fecha_cre) VALUES
  ('Aurora Boreal', '2015-03-10'),
  ('Ritmo Andino',  '2018-07-22');

INSERT INTO grupo_componente (nombre_grupo, componente) VALUES
  ('Aurora Boreal', 'Laura Pérez'),
  ('Aurora Boreal', 'Camilo Rojas'),
  ('Aurora Boreal', 'Andrea Gómez'),
  ('Ritmo Andino',  'Julián Torres'),
  ('Ritmo Andino',  'Marta Díaz');

INSERT INTO album (id_album, titulo, fecha_pub, nombre_grupo) VALUES
  (1, 'Luces del Norte',  '2019-05-17', 'Aurora Boreal'),
  (2, 'Horizonte',        '2022-09-30', 'Aurora Boreal'),
  (3, 'Montañas y Ríos',  '2021-11-12', 'Ritmo Andino');

INSERT INTO cancion (id_cancion, titulo, publica) VALUES
  (1, 'Amanecer',      '2019-05-17'),
  (2, 'Noche Polar',   '2019-05-17'),
  (3, 'Cielo Abierto', '2022-09-30'),
  (4, 'Valle Verde',   '2021-11-12'),
  (5, 'Cóndor',        '2021-11-12');

INSERT INTO album_cancion (id_album, id_cancion) VALUES
  (1,1), (1,2), (2,3), (3,4), (3,5);

INSERT INTO usuario (id_usuario, correo, fecha_alta, url) VALUES
  (1, 'ana@mail.com',   '2020-01-15', 'https://musica.example/u/ana'),
  (2, 'bruno@mail.com', '2021-03-02', 'https://musica.example/u/bruno'),
  (3, 'carla@mail.com', '2022-06-20', 'https://musica.example/u/carla'),
  (4, 'diego@mail.com', '2023-02-11', 'https://musica.example/u/diego');

INSERT INTO registrado (id_usuario, cuota, fecha_fin) VALUES
  (2, 24900.00, '2026-12-31'),
  (4, 24900.00, '2027-03-31');

INSERT INTO stream (id_cancion, id_usuario, fecha) VALUES
  (1, 1, '2026-10-01 08:15:00'),
  (1, 2, '2026-10-02 19:40:00'),
  (2, 1, '2026-10-03 21:05:00'),
  (3, 3, '2026-10-05 10:30:00'),
  (4, 4, '2026-10-06 18:20:00'),
  (1, 4, '2026-10-07 07:55:00');

INSERT INTO descarga (id_cancion, id_usuario, fecha, n_veces) VALUES
  (1, 2, '2026-10-02', 2),
  (4, 2, '2026-10-04', 1),
  (4, 4, '2026-10-08', 3),
  (3, 4, '2026-10-09', 1);
  
  SELECT id_album, titulo, fecha_pub
FROM album
WHERE fecha_pub >= '2021-01-01'
ORDER BY fecha_pub;

SELECT id_album, titulo, fecha_pub
FROM album
WHERE fecha_pub >= '2021-01-01'
ORDER BY fecha_pub;

SELECT g.nombre AS grupo, COUNT(a.id_album) AS total_albumes
FROM grupo g
JOIN album a ON a.nombre_grupo = g.nombre
GROUP BY g.nombre
HAVING COUNT(a.id_album) >= 1;

SELECT c.titulo, COUNT(s.id_usuario) AS reproducciones
FROM cancion c
LEFT JOIN stream s ON s.id_cancion = c.id_cancion
GROUP BY c.id_cancion, c.titulo
ORDER BY reproducciones DESC;

UPDATE registrado
SET cuota = 29900.00
WHERE id_usuario = 4;

DELETE FROM stream
WHERE id_cancion = 1
  AND id_usuario = 4
  AND fecha = '2026-10-07 07:55:00';
  
SELECT id_cancion, titulo
FROM cancion
WHERE id_cancion NOT IN (SELECT id_cancion FROM descarga);

SELECT u.id_usuario, u.correo
FROM usuario u
WHERE EXISTS (
  SELECT 1
  FROM stream s
  JOIN album_cancion ac ON ac.id_cancion = s.id_cancion
  WHERE s.id_usuario = u.id_usuario
    AND ac.id_album = 1
    );
    
 CREATE VIEW v_descargas_por_cancion AS
SELECT c.id_cancion,
       c.titulo AS cancion,
       a.nombre_grupo AS grupo,
       SUM(d.n_veces) AS total_descargas
FROM descarga d
JOIN cancion c        ON c.id_cancion = d.id_cancion
JOIN album_cancion ac ON ac.id_cancion = c.id_cancion
JOIN album a          ON a.id_album = ac.id_album
GROUP BY c.id_cancion, c.titulo, a.nombre_grupo;

SELECT * FROM v_descargas_por_cancion ORDER BY total_descargas DESC;
   
