Para asegurarte de que **Veyon Service** esté siempre activo y en ejecución en los equipos de los alumnos mediante Ansible y WinRM, el playbook debe realizar tres acciones clave:

1. **Asegurar el servicio:** Cambiar su inicio a automático y forzar que esté en estado *running*.
2. **Crear una Tarea Programada de Windows (Schedule Task):** Configurar una tarea en cada equipo que verifique y reinicie el servicio Veyon cada 5 o 10 minutos por si los alumnos intentan detenerlo.
3. **Bloquear la modificación del servicio:** Ajustar los permisos (opcional) o la tarea recurrente para que se reejecute automáticamente.

### Requisitos previos

En los equipos de los alumnos, la interfaz WinRM debe estar habilitada (por ejemplo, mediante PowerShell con `Enable-PSRemoting -Force` o a través de Directivas de Grupo GPO).

---

### Playbook de Ansible (`ensure_veyon.yml`)

```yaml
---
- name: Asegurar y mantener activo Veyon Service en equipos de alumnos
  hosts: alumnos
  gather_facts: no
  vars:
    ansible_user: "Administrador"
    ansible_password: "TuPasswordAdmin"
    ansible_connection: winrm
    ansible_winrm_server_cert_validation: ignore
    ansible_winrm_transport: ntlm

  tasks:
    - name: Configurar e iniciar el servicio VeyonService
      ansible.windows.win_service:
        name: VeyonService
        start_mode: auto
        state: started

    - name: Crear Tarea Programada recurrente para reactivar Veyon si es detenido
      community.windows.win_scheduled_task:
        name: "MantenerVeyonActivo"
        description: "Inicia el servicio Veyon si un alumno lo ha desactivado"
        actions:
          - path: 'powershell.exe'
            arguments: '-NoProfile -ExecutionPolicy Bypass -Command "if ((Get-Service -Name VeyonService).Status -ne ''Running'') { Start-Service -Name VeyonService }"'
        triggers:
          - type: boot
            repetition_interval: PT5M  # Se ejecuta cada 5 minutos
        username: "SYSTEM"
        state: present
        enabled: yes

```

---

### Verificación del funcionamiento

Para comprobar que el playbook se ha aplicado correctamente en los clientes de los alumnos:

1. Ejecuta el playbook desde tu equipo maestro de Ansible:
```bash
ansible-playbook -i inventario.ini ensure_veyon.yml

```


2. Accede al cliente de un alumno y abre PowerShell como administrador.
3. Intenta detener el servicio manualmente:
```powershell
Stop-Service -Name VeyonService

```


4. Espera un máximo de 5 minutos (o fuerza el inicio de la tarea en el **Programador de tareas** de Windows) y verifica que el estado vuelve a ser `Running`:
```powershell
Get-Service -Name VeyonService

```



---

### Medidas adicionales recomendadas

Si los alumnos disponen de permisos de administrador en sus cuentas localmente, podrán seguir deteniendo la tarea o el servicio. Para evitarlo por completo:

* **Quitar permisos de Administrador:** Asegúrate de que las cuentas de los alumnos sean cuentas de usuario estándar.
* **Ocultar la bandeja del sistema:** En la configuración de Veyon Configurator, desactiva la opción *"Mostrar icono en la bandeja del sistema"* para que no sepan que el servicio está corriendo ni puedan interactuar con él directamente.
