# ansible-playbooks

## Propuesta de naming de los playbooks
Hasta una solución mejor se recomienda:
- Comenzar con "win-" o "linux-" para indicar el objetivo concreto.
- Una opción adecuada puede ser añadir el grupo "smr-" o "des1-" para indicar el aula, pero ésto no es necesario si se marca el objetivo `hosts: des1` dentro de la play.
- Un nombre descriptivo o las iniciales del creador en fase de pruebas.

## Encabezado
Hasta solución mejor se recomienda un encabezado descriptivo almenos con:
```txt
# Estado: Playbook probada y funcionando
# Autor: Luis Ferreira
# Comentario: 
#  - hosts debe coincidir con el aula
#
```

Posibles **estados**:
- En redacción
- En pruebas
- Probada y funcionando
- Legacy
- Deprecated (o como sea en español)
- Obsoleto

**Autor** para posibles consultas.

