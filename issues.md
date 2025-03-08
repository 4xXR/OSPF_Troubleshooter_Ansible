**Resumen de Problemas y Soluciones en la Configuración de Ansible para Routers Cisco en GNS3**

### **1. Problema: Conexión SSH Inicial Bloqueada**
**Síntoma:**
- Intentamos conectarnos a los routers con SSH y recibimos el error:
  ```
  Unable to negotiate with A.B.C.D port 22: no matching key exchange method found.
  Their offer: diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1
  ```

**Causa:**
- Los routers solo soportaban algoritmos de cifrado SSH antiguos, los cuales OpenSSH moderno bloquea por seguridad.

**Solución:**
- Forzamos a OpenSSH a aceptar estos algoritmos obsoletos ejecutando:
  ```
  ssh -oKexAlgorithms=+diffie-hellman-group14-sha1 -oCiphers=aes128-cbc -oHostKeyAlgorithms=+ssh-rsa admin1@192.168.1.101
  ```
- Esto nos permitió conectarnos manualmente y realizar la configuración inicial de SSH.

---
### **2. Problema: Ansible No Podía Conectarse por SSH**
**Síntoma:**
- Al ejecutar un playbook en Ansible, recibimos:
  ```
  UNREACHABLE! => {"msg": "Failed to connect to the host via ssh"}
  ```

**Causa:**
- Ansible también bloquea los algoritmos SSH antiguos por defecto.

**Solución:**
- Configuramos Ansible para aceptar estos algoritmos en `ansible.cfg`:
  ```ini
  [ssh_connection]
  ssh_args = -oKexAlgorithms=+diffie-hellman-group14-sha1 -oCiphers=aes128-cbc -oHostKeyAlgorithms=+ssh-rsa
  ```
- Después de esto, Ansible pudo conectarse correctamente.

---
### **3. Problema: "Connection type ssh is not valid for this module"**
**Síntoma:**
- Al intentar ejecutar un playbook de configuración, recibimos:
  ```
  "Connection type ssh is not valid for this module"
  ```

**Causa:**
- Estábamos usando `ansible_connection: ssh`, pero los módulos de red de Cisco requieren `network_cli`.

**Solución:**
- Cambiamos en `inventory.yml`:
  ```yaml
  ansible_connection: network_cli
  ansible_network_os: cisco.ios.ios
  ```
- Esto permitió a Ansible manejar adecuadamente la conexión con los routers.

---
### **4. Problema: "paramiko is not installed"**
**Síntoma:**
- Ansible intentaba usar `paramiko`, pero no estaba instalado:
  ```
  paramiko is not installed: No module named 'paramiko'
  ```

**Causa:**
- El entorno de Ansible no tenía la librería `paramiko` instalada.

**Solución:**
- Instalamos `paramiko` con:
  ```bash
  pip install paramiko
  ```
- Esto permitió a Ansible manejar `network_cli` correctamente.

---
### **5. Problema: "Invalid input detected" en `ios_command`**
**Síntoma:**
- Al ejecutar un comando con `ios_command`, recibimos:
  ```
  % Invalid input detected at '^' marker.
  ```

**Causa:**
- Ansible estaba enviando el comando de manera incorrecta dentro de una lista:
  ```
  commands=['show ip interface brief']
  ```

**Solución:**
- En lugar de usar una lista, enviamos el comando como una cadena de texto:
  ```
  ansible R1 -i inventory.yml -m cisco.ios.ios_command -a "commands='show ip interface brief'" -vvvv
  ```
- Esto resolvió el problema y permitió ejecutar comandos correctamente.

---
### **Conclusiones y Mejores Prácticas**
1. **SSH en IOS antiguo:** OpenSSH moderno bloquea algoritmos obsoletos, hay que habilitarlos manualmente si es necesario.
2. **Ansible y `network_cli`:** Es obligatorio para dispositivos Cisco, `ssh` no funciona.
3. **Manejo de comandos en `ios_command`:** Deben enviarse como cadenas de texto, no como listas.
4. **Paramiko y Ansible:** Si `paramiko` no está instalado, `network_cli` puede fallar.
5. **Usar `-vvvv` para debuggear:** Muestra detalles útiles para resolver problemas de conexión.

Este registro servirá como referencia para solucionar problemas similares en el futuro. 🚀

