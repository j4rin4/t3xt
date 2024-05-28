### Asegurando los Secretos en Proyectos con SOPS y AGE

Recientemente, he estado trabajando en cómo asegurar los secretos en nuestros proyectos utilizando las tecnologías SOPS y AGE. La razón principal para utilizar estas herramientas es evitar que contraseñas y otros datos sensibles queden expuestos en los repositorios Git, un error de seguridad común.

### ¿Qué es SOPS?

Mozilla SOPS (Secrets OPerationS) es una herramienta que permite cifrar y descifrar archivos en formatos como YAML, JSON, ENV, INI y binarios. La ventaja de SOPS es su capacidad de integrarse con servicios de gestión de claves en la nube, como AWS KMS, GCP KMS y Azure Key Vault, lo que facilita un manejo seguro y centralizado de nuestras claves de cifrado. Además, soporta el cifrado con PGP, ofreciendo flexibilidad para distintos entornos y necesidades.

### ¿Qué es AGE?

AGE es otra herramienta y biblioteca Go diseñada para el cifrado simple y seguro. Su enfoque minimalista con claves explícitas y sin configuraciones complejas hace que sea fácil de usar y muy eficiente para nuestros propósitos. Un archivo AGE se compone de un encabezado de texto con la clave y una carga útil binaria cifrada, lo que asegura que los datos sean inalterables sin la clave adecuada.

### Instalación y Configuración

Primero, instalé SOPS descargando el binario desde GitHub y configurándolo en mi sistema. Luego, hice lo mismo con AGE, asegurándome de que ambos estuvieran correctamente instalados y listos para usar. Generé un par de claves con AGE y las almacené de forma segura en un directorio específico, configurando también una variable de entorno para facilitar su uso.

#### Instalación de Herramientas

##### Instalación de SOPS

1. **Descargar el binario de SOPS desde GitHub:**

```
sudo curl --silent --location --remote-name https://github.com/getsops/sops/releases/download/v3.8.1/sops-v3.8.1.linux.amd64
```

2. **Instalar SOPS en el sistema:**

```
sudo install --owner root --group root --mode 555 sops-v3.8.1.linux.amd64 /usr/local/bin/sops
```

3. **Verificar la instalación:**

```
sops --version
```

##### Instalación de AGE

1. **Descargar el archivo tar de AGE desde GitHub:**

```
sudo curl --silent --location --remote-name https://github.com/FiloSottile/age/releases/download/v1.1.1/age-v1.1.1-linux-amd64.tar.gz
```

2. **Extraer y mover los binarios:**

```
sudo tar --extract --gzip --file age-v1.1.1-linux-amd64.tar.gz --strip-components 1 --directory /usr/local/bin age/age age/age-keygen`
 ```

3. **Verificar la instalación:**

```
age --version
 ```

#### Configuración de Claves

##### Generación de Claves AGE

1. **Crear un par de claves pública y privada:**

```
age-keygen -o key.txt
```

2. **Mover la clave a un directorio seguro:**

```
mkdir ~/.sops mv key.txt ~/.sops
```

3. **Exportar la variable de entorno para la clave:**

```
echo "export SOPS_AGE_KEY_FILE=$HOME/.sops/key.txt" >> ~/.bashrc
```

### Ejemplos Prácticos

Para ilustrar cómo funcionan estas herramientas, realicé algunos ejemplos prácticos:

#### Cifrado y Descifrado de un Archivo de Texto

1. **Crear un archivo de texto simple y cifrarlo:**
```
echo "Este es un ejemplo de cifrado con SOPS & AGE" >> ejemplo.txt sops --encrypt --age $(cat $SOPS_AGE_KEY_FILE | grep -oP "public key: \K(.*)") --in-place ./ejemplo.txt
```

2. **Verificar el contenido cifrado:**
```
cat ejemplo.txt
```

3. **Descifrar el archivo:**
```
sops --decrypt --age $(cat $SOPS_AGE_KEY_FILE | grep -oP "public key: \K(.*)") --in-place ./ejemplo.txt
```

4. **Verificar el contenido descifrado:**
```
cat ejemplo.txt
```

#### Cifrado y Descifrado de un Archivo YAML

1. **Crear un archivo YAML con datos sensibles:**
```
apiVersion: v1 kind: Secret metadata:   name: mysql-secret   namespace: default stringData:   MYSQL_USER: myuserroot   MYSQL_PASSWORD: my_secret_password
```

2. **Cifrar las secciones especificadas del archivo YAML:**
```
sops --encrypt --age $(cat $SOPS_AGE_KEY_FILE | grep -oP "public key: \K(.*)") --encrypted-regex '^(data|stringData)$' --in-place ./midocker.yml
```

3. **Verificar el contenido cifrado:**

```
cat midocker.yml
```

4. **Descifrar el archivo YAML:**
```
sops --decrypt --age $(cat $SOPS_AGE_KEY_FILE | grep -oP "public key: \K(.*)") --encrypted-regex '^(data|stringData)$' --in-place ./midocker.yml
```

5. **Verificar el contenido descifrado:**
```
cat midocker.yml
```

#### Cifrado Parcial en un Archivo docker-compose

1. **Cifrar solo la parte del archivo que contiene la contraseña de MySQL:**

```
sops --encrypt --age $(cat $SOPS_AGE_KEY_FILE | grep -oP "public key: \K(.*)") --encrypted-regex '^(services.mysql.environment.MYSQL_PASSWORD)$' --in-place ./docker-compose.yml
```

2. **Verificar el contenido cifrado:**

    ```
   `cat docker-compose.yml`
    ```


### Buenas Prácticas

Es fundamental no subir archivos descifrados a los repositorios Git. Para ello, agregué los archivos descifrados a `.gitignore`, asegurando que no se subieran accidentalmente al control de versiones.

```
`echo "ejemplo.txt" >> .gitignore echo "midocker.yml" >> .gitignore echo "docker-compose.yml" >> .gitignore`
```

Este enfoque asegura que nuestros datos sensibles permanezcan seguros mientras colaboramos en proyectos, evitando la exposición accidental de información crítica.