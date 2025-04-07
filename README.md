# htb-intentions-privilege-escalation

Este archivo describe una vulnerabilidad de escalación de privilegios que permite extraer la flag de root (`/root/root.txt`) carácter por carácter. A continuación, se explica cómo funciona el código y por qué es efectivo.

---

## **Descripción de la vulnerabilidad**

El programa explota una vulnerabilidad en una herramienta llamada `scanner`, que parece ser un binario con permisos elevados (SUID). Este binario permite a usuarios no privilegiados calcular hashes MD5 de partes específicas de un archivo protegido, como `/root/root.txt`. Sin embargo, no valida adecuadamente quién puede acceder a esta funcionalidad, lo que permite a un atacante reconstruir el contenido del archivo protegido.

---

## **Funcionamiento del código**

El script utiliza un enfoque iterativo para extraer la flag carácter por carácter. A continuación, se detalla el proceso paso a paso:

### 1. **Inicio del proceso**
El script comienza llamando a la función `extract_root_flag()`, que inicia la extracción de la flag desde `/root/root.txt`.

```python
print("[*] Iniciando extracción de /root/root.txt...")
``` 

2. Iteración por posición

El script utiliza un bucle while para extraer cada carácter de la flag. En cada iteración:

- Se calcula el hash MD5 del contenido del archivo hasta una posición específica.
- Se intenta deducir el carácter correspondiente que produce ese hash.

```python
cmd = f"/opt/scanner/scanner -l {position} -s root_flag -c {target_file} -p"
result = subprocess.check_output(cmd, shell=True)
```

El comando ejecuta el binario scanner con los siguientes parámetros:

-l {position}: Longitud del contenido a calcular.
-c {target_file}: Archivo objetivo (/root/root.txt).  
-p: [Debug] Imprime el hash calculado del archivo. Solo es compatible con -c.


3. Deducción del caracter
La función checkhash() toma el hash objetivo y prueba todos los caracteres posibles (string.printable) para encontrar cuál genera el mismo hash cuando se concatena al contenido actual.

```python
for char in charset:
    test_content = current_content + char
    if hashlib.md5(test_content.encode()).hexdigest() == target_hash:
        return char
``` 

Cuando encuentra el carácter correcto, lo devuelve y lo añade al contenido reconstruido.


- Falta de validación de permisos en scanner
    - El binario scanner permite a cualquier usuario calcular hashes MD5 de archivos protegidos, como /root/root.txt. Esto expone información sensible.

- Reconstrucción basada en hashes
    - Al conocer el hash MD5 de cada carácter, el script puede deducir el contenido del archivo probando todas las combinaciones posibles.

- Iteración carácter por carácter
    - La extracción iterativa permite reconstruir el archivo completo sin necesidad de acceso directo al mismo.

