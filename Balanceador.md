# **Balanceador**
<div align="justify">

**Objetivo dentro del proyecto:** 

El balanceador en tu proyecto tiene como objetivo repartir las solicitudes de los usuarios entre las dos aplicaciones que tienes desplegadas en diferentes servidores, mejorando así la disponibilidad, escalabilidad y rendimiento del sistema. Actúa como un punto de entrada único, de modo que los usuarios no necesitan conocer las direcciones específicas de cada backend, y además ayuda a que si una instancia falla, la otra pueda seguir atendiendo las peticiones, asegurando así la continuidad del servicio.  


## **Configuración de IP en la máquina virtual**

* **Cambio de IP dinámica a estática:**
```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

Dentro del archivo /etc/...
```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: false
      addresses: [192.168.105.10/24]
      routes:
      - to: default
        via: 192.168.105.10
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

## **configuracion e instalacion de Nginx:**

* **Instalación de Nginx:**
```bash
sudo apt update && sudo apt install nginx
```

* **Verificación del servicio Nginx:**
```bash
sudo systemctl status nginx
```

* **Instalación de módulos PHP:**
```bash
sudo apt install php8.3-fpm php8.3-mbstring php8.3-curl php8.3-xml php8.3-mysql php8.3-zip
```

* **Restablecimiento del servicio:**
```bash
sudo systemctl restart nginx
```

## **Configuración del balanceador**

* **Configuración del sitio habilitado:**
```bash
sudo nano /etc/nginx/sites-available/balanceo_SIS313
```

Dentro del archivo:
```nginx
upstream mis_apps {
    server 192.168.105.11:3000;
    server 192.168.105.12:3000;
}

server {
    listen 80;
    server_name sis313.final;

    location / {
        proxy_pass http://mis_apps;
        proxy_http_version 1.1;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_cache_bypass $http_upgrade;

        # Evita que Nginx cambie las redirecciones de la app
        proxy_redirect off;
    }
}
```

* **Enlace simbólico:**
```bash
sudo ln -s /etc/nginx/sites-available/balanceo_SIS313 /etc/nginx/sites-enabled/
```

* **Eliminar el enlace `default`:**
```bash
sudo rm /etc/nginx/sites-enabled/default
```

* **Revisión de sintaxis del archivo Nginx:**
```bash
sudo nginx -t
```


* **Verificación del enlace creado:**
```bash
ls /etc/nginx/sites-enabled/
```

* **Reinicio del servicio con los cambios:**
```bash
sudo systemctl restart nginx
```

# **Añadir a hosts IP y DNS del balanceador**

se añade esto dentro de c:/windows/system32/drivers/etc/host, para que se pueda d¿verificar la app desde el navegador.

```bash
192.168.105.10 sis313.final
```

</div>
