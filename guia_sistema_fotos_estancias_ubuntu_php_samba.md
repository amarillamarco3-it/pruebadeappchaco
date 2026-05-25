# Sistema de Fotos para Estancias

## Objetivo

Sistema interno para:

- Subir fotos desde celulares.
- Visualizar fotos desde navegador.
- Compartir carpeta desde Windows.
- Centralizar imágenes de estancias.

---

# Tecnologías utilizadas

- Ubuntu Server
- Nginx
- PHP
- MariaDB
- Samba

---

# Instalación inicial

Actualizar paquetes:

```bash
sudo apt update
```

Instalar servidor web + PHP + MariaDB:

```bash
sudo apt install nginx php-fpm mariadb-server php-mysql
```

Instalar extensiones PHP útiles:

```bash
sudo apt install php-gd php-mbstring php-xml php-curl
```

---

# Verificar servicios

Verificar Nginx:

```bash
sudo systemctl status nginx
```

Verificar MariaDB:

```bash
sudo systemctl status mariadb
```

Verificar PHP:

```bash
php -v
```

---

# Crear proyecto

Crear carpeta principal:

```bash
sudo mkdir -p /var/www/estancias
```

Dar permisos al usuario actual:

```bash
sudo chown -R $USER:$USER /var/www/estancias
```

---

# Configuración Nginx

Crear archivo del sitio:

```bash
sudo nano /etc/nginx/sites-available/estancias
```

Contenido:

```nginx
server {
    listen 80;
    server_name _;

    root /var/www/estancias;
    index index.php index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
    }
}
```

IMPORTANTE:

Verificar versión PHP instalada:

```bash
ls /run/php/
```

Si aparece por ejemplo:

```text
php8.2-fpm.sock
```

Entonces cambiar esta línea:

```nginx
fastcgi_pass unix:/run/php/php8.3-fpm.sock;
```

por:

```nginx
fastcgi_pass unix:/run/php/php8.2-fpm.sock;
```

---

# Activar sitio

```bash
sudo ln -s /etc/nginx/sites-available/estancias /etc/nginx/sites-enabled/
```

Desactivar sitio default:

```bash
sudo rm /etc/nginx/sites-enabled/default
```

Verificar configuración:

```bash
sudo nginx -t
```

Reiniciar Nginx:

```bash
sudo systemctl restart nginx
```

---

# Crear index.php

Editar archivo:

```bash
nano /var/www/estancias/index.php
```

Contenido:

```php
<?php

// Busca todas las imagenes dentro de uploads
$imagenes = glob("uploads/*");

?>

<h1>Galeria de Fotos</h1>

<?php foreach($imagenes as $img): ?>

<div>
    <img src="<?php echo $img; ?>" width="300">
</div>

<?php endforeach; ?>
```

---

# Crear carpeta uploads

```bash
mkdir /var/www/estancias/uploads
```

Permisos:

```bash
sudo chown -R www-data:www-data /var/www/estancias/uploads
```

```bash
sudo chmod 755 /var/www/estancias/uploads
```

---

# Crear upload.php

Editar archivo:

```bash
nano /var/www/estancias/upload.php
```

Contenido:

```php
<?php

// Verifica si el formulario fue enviado
if ($_SERVER['REQUEST_METHOD'] == 'POST') {

    // Obtiene el archivo enviado
    $archivo = $_FILES['foto'];

    // Genera ruta destino
    $ruta = "uploads/" . basename($archivo['name']);

    // Mueve el archivo al servidor
    if (move_uploaded_file($archivo['tmp_name'], $ruta)) {
        echo "Imagen subida correctamente";
    } else {
        echo "Error al subir";
    }
}

?>

<form method="POST" enctype="multipart/form-data">

    <!-- Selector de archivo -->
    <input type="file" name="foto">

    <!-- Boton subir -->
    <button type="submit">Subir</button>

</form>
```

---

# Acceso desde navegador

Galería:

```text
http://IP-DEL-SERVIDOR/
```

Upload:

```text
http://IP-DEL-SERVIDOR/upload.php
```

Ejemplo:

```text
http://192.168.60.158
```

---

# Instalar Samba

Instalar:

```bash
sudo apt install samba
```

Verificar:

```bash
sudo systemctl status smbd
```

---

# Configurar Samba

Editar:

```bash
sudo nano /etc/samba/smb.conf
```

Agregar al FINAL:

```ini
[estancias]
   path = /var/www/estancias/uploads
   browseable = yes
   writable = yes
   read only = no
   guest ok = yes
   public = yes
   guest only = yes
   force user = www-data
```

Reiniciar Samba:

```bash
sudo systemctl restart smbd
```

---

# Crear usuario Samba (opcional recomendado)

Crear usuario Linux:

```bash
sudo adduser secretaria
```

Agregar usuario Samba:

```bash
sudo smbpasswd -a secretaria
```

Reiniciar Samba:

```bash
sudo systemctl restart smbd
```

---

# Acceso desde Windows

Abrir explorador de archivos y entrar:

```text
\\192.168.60.158\estancias
```

O usando nombre del servidor:

```text
\\NOMBRE-SERVIDOR\estancias
```

---

# Arquitectura del sistema

## Encargados de estancia

- Suben imágenes desde la web.
- No acceden al servidor.
- No borran archivos.

## Secretaria / dueño

- Visualizan desde Windows.
- Pueden administrar imágenes.
- Acceden mediante Samba.

## TI

- Administra servidor.
- Gestiona permisos.
- Gestiona backups.
- Mantiene sistema.

---

# Próximas mejoras recomendadas

## ETAPA 1

- Renombrar imágenes automáticamente.
- Evitar archivos duplicados.

Ejemplo:

```php
$nombre = uniqid() . ".jpg";
```

---

## ETAPA 2

Conectar MariaDB:

Guardar:

- estancia
- fecha
- usuario
- comentario
- categoría
- nombre original
- nombre interno

---

## ETAPA 3

Usuarios y permisos:

- encargado
- secretaria
- dueño
- admin TI

---

## ETAPA 4

Miniaturas automáticas:

- Mejor rendimiento.
- Galerías más rápidas.
- Menor consumo de red.

---

# Estructura futura recomendada

```text
/var/www/estancias/
 ├── uploads/
 ├── thumbnails/
 ├── backups/
 ├── app/
 └── logs/
```

---

# Notas importantes

- No guardar imágenes dentro de MariaDB.
- Guardar solamente rutas.
- Mantener backups frecuentes.
- Evitar permisos 777.
- Validar tipos de archivos en producción.

---

# Estado actual del proyecto

Actualmente funcionando:

- Ubuntu Server
- Nginx
- PHP
- MariaDB
- Upload de imágenes
- Galería web
- Samba compartido a Windows

