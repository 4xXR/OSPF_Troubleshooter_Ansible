**# Registro de Problemas y Soluciones en la Configuración de Interfaces con Ansible**

Este documento resume los principales problemas encontrados y sus soluciones durante la implementación de un playbook de Ansible para configurar interfaces en routers Cisco 7200 con IOS 12.4.

## **Problemas y Soluciones**

### **1. Error: "network os cisco.ios is not supported"**

- **Causa:** `ansible_network_os` no estaba bien definido en `group_vars/all.yml`.
- **Solución:** Cambiar `ansible_network_os: cisco.ios` por `ansible_network_os: cisco.ios.ios`.
- **Verificación:**
  ```bash
  ansible-inventory -i inventory.yml --list
  ```

### **2. Error: "configure terminal" repetido en modo de configuración**

- **Causa:** Ansible intentaba entrar en `configure terminal` dos veces.
- **Solución:** Usar `parents:` en lugar de escribir `interface X/X` en `lines:` en `ios_config`.
- **Corrección en el playbook:**
  ```yaml
  - name: Configure interfaces
    cisco.ios.ios_config:
      parents: "interface {{ item.interface }}"
      lines:
        - "ip address {{ item.ip }} 255.255.255.0"
        - "no shutdown"
  ```

### **3. Error: "MODULE FAILURE" con **``

- **Causa:** El router no estaba en modo privilegiado antes de configurar.
- **Solución:** Habilitar `ansible_become` y `ansible_become_method: enable` en `group_vars/all.yml`:
  ```yaml
  ansible_become: yes
  ansible_become_method: enable
  ```
- **Verificación:**
  ```bash
  ansible R1 -m cisco.ios.ios_command -a "commands='show privilege'" -i inventory.yml
  ```
  **Salida esperada:** `Current privilege level is 15`

### **4. Error: "interfaces" no está definido en los routers**

- **Causa:** Ansible no estaba detectando `interfaces` en `host_vars/`.
- **Solución:** Asegurar que `host_vars/R1.yml` tiene la estructura correcta:
  ```yaml
  interfaces:
    - { interface: "FastEthernet0/0", ip: "A.B.C.D" }
    - { interface: "FastEthernet1/0", ip: "E.F.G.H" }
  ```
- **Verificación:**
  ```bash
  ansible R1 -m debug -a "var=interfaces" -i inventory.yml
  ```

### **5. Error: "No matching key exchange method found" en SSH**

- **Causa:** El router solo soporta `diffie-hellman-group14-sha1`.
- **Solución:** Conectarse manualmente y verificar:
  ```bash
  ssh -oKexAlgorithms=+diffie-hellman-group14-sha1 -oHostKeyAlgorithms=+ssh-rsa admin@192.168.0.1
  ```

---

## **Resultados Finales**

Después de implementar estas soluciones:

- Se logró la conexión exitosa con los routers.
- Se ejecutó el playbook sin errores.
- Las interfaces se configuraron correctamente.
- Se generó un sistema modular y escalable para futuras automatizaciones.

### **Prueba Final**

```bash
ansible routers -m cisco.ios.ios_command -a "commands='show ip interface brief'" -i inventory.yml
```

**Salida esperada:** Interfaces configuradas con IPs y en estado `up`.

---

