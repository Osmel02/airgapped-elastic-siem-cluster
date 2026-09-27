## 1. Configuración en Kibana (Fleet UI)

1.1 **Ajustar Settings:** Ir a `Management > Fleet > Settings`.
    
    - **Fleet Server Hosts:** Añadir `https://fleet-1.soc.com`.
        
    - **Elasticsearch Output:** Asegurarse que apunte al balanceador: `https://balanceador.soc.com/ingest`.
        
    - **CA de Confianza:** En el campo `Advanced YAML` del output, pegar el contenido de  `soc-ca.crt`.
        
1.2 **Generar Token:**
    
    - Ir `Fleet > Agents > Add Fleet Server`.
        
    - Crear una nueva política (ej: `Fleet-Server-1`).

---

## 2. Instalación de Fleet Server

Descargar el agente en el servidor dedicado y ejecuta la instalación combinada (Enroll + Install):

```Bash
sudo ./elastic-agent install \
  --url=https://fleet-1.soc.com \
  --fleet-server-es=https://balanceador.soc.com/ingest \
  --fleet-server-service-token=TOKEN_GENERADO_KIBANA \
  --fleet-server-policy=SOC-Fleet-Policy \
  --fleet-server-es-ca=/etc/fleet-certs/elastic-stack-ca.crt \
  --certificate-authorities=/etc/fleet-certs/soc-ca.crt \
  --fleet-server-cert=/etc/fleet-certs/fleet-server.crt \
  --fleet-server-cert-key=/etc/fleet-certs/fleet-server.key \
  --fleet-server-port=8220
```

### Parámetros Clave:

- `--fleet-server-es-ca`: Permite que Fleet Server confíe en Elasticsearch.
    
- `--certificate-authorities`: Es la CA que Fleet Server entregará a los futuros agentes para que confíen en él.
    

---

## 3. Verificación Final

### 3.1 En el Servidor (CLI):

```Bash
sudo elastic-agent status
```

Debe mostrar `Status: HEALTHY` tanto para el daemon como para el componente `fleet-server`.


