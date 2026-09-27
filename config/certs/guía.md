## 1. Configuración de seguridad TLS (generación de certificados)

Realizar **una sola vez** en un nodo de administración. Luego distribuir los certificados a cada nodo.

### 1.1 Crear Autoridad Certificadora (CA)
```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-certutil ca --out /etc/elasticsearch/certs/soc-ca.p12 
```


#### 5.1.2 Guardar contraseña en keystore

Si tu `soc-ca.p12` tiene contraseña:

```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-keystore add  
xpack.security.transport.ssl.truststore.secure_password
```

y

```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-keystore add 
xpack.security.http.ssl.truststore.secure_password
```

### 5.2 Crear archivo de instancias (`instances.yml`)
```bash
sudo mkdir -p /etc/elasticsearch/certs
sudo nano /etc/elasticsearch/certs/instances.yml
```

Contenido (adaptar IPs y nombres):
```yaml
instances:
  - name: "master-1"
    dns: ["master-1.soc.com"]
  - name: "master-2"
    dns: ["master-2.soc.com"]
  - name: "master-3"
    dns: ["master-3.soc.com"]
  - name: "data-1"
    dns: ["data-1.soc.com"]
  - name: "data-2"
    dns: ["data-2.soc.com"]
  - name: "data-3"
    dns: ["data-3.soc.com"]
  - name: "ingest-1"
    dns: ["ingest-1.soc.com"]
  - name: "ingest-2"
    dns: ["ingest-2.soc.com"]
  - name: "coord-1"
    dns: ["coord-1.soc.com"]    
  - name: "coord-2"
    dns: ["coord-2.soc.com"]    
  - name: "transform-1"
    dns: ["transform-1.soc.com"]
  - name: "ml-1"
    dns: ["ml-1.soc.com"]
  - name: "kibana-1"
    dns: ["kibana-1.soc.com"]
  - name: "fleet-server"
    dns: ["fleet-1.soc.com"]

```
### 1.3 Generar certificados para cada nodo
```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-certutil cert \
  --ca /etc/elasticsearch/certs/soc-ca.p12 \
  --in /etc/elasticsearch/certs/instances.yml \
  --out /etc/elasticsearch/certs/elastic-certs.zip \
  --pass ""
```
Extraer:
```bash
sudo unzip /etc/elasticsearch/certs/elastic-certs.zip -d /etc/elasticsearch/certs/
```


#### 1.3.1 Guardar contraseña en keystore

Si `master-1.p12` tiene contraseña, ejecuta:
```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-keystore add \  
xpack.security.transport.ssl.keystore.secure_password
```

y escribe la contraseña.

Luego también para HTTP:

```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-keystore add \  
xpack.security.http.ssl.keystore.secure_password
```

>**Nota**: Realizar este paso para cada nodo 

### 1.4 Distribuir certificados a cada nodo
En cada nodo remoto, crear la misma estructura y copiar su carpeta correspondiente.

```bash
scp -r /etc/elasticsearch/certs/master-2 ubuntu@master-2.soc.com:/tmp/

sudo mkdir -p /etc/elasticsearch/certs
sudo mv /tmp/master-2 /etc/elasticsearch/certs/
sudo chown -R elasticsearch:elasticsearch /etc/elasticsearch/certs
```

>**Nota**: Para Kibana y Fleet-Server varia el nombre.

### 1.5 Ajustar permisos
```bash
sudo chown -R elasticsearch:elasticsearch /etc/elasticsearch/certs;
sudo chmod 750 /etc/elasticsearch/certs
```
