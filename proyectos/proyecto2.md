# Estructura del Dominio

El dominio educativo implementado sigue una estructura organizada y optimizada para la administración eficiente de usuarios, equipos y recursos. 

## Dominio Principal

**sanandres.local**

### Unidades Organizativas (UO)

#### Alumnos
- **ASIR**
  - **Primero** (30 alumnos)
    - **Aula1** (15 equipos)
  - **Segundo** (15 alumnos)
    - **Aula2** (15 equipos)
- **SMR**
  - **Primero** (30 alumnos)
    - **Aula3** (15 equipos)
  - **Segundo** (15 alumnos)
    - **Aula4** (15 equipos)
- **DAM**
  - **Primero** (30 alumnos)
    - **Aula5** (15 equipos)
  - **Segundo** (15 alumnos)
    - **Aula6** (15 equipos)
- **DAW**
  - **Primero** (30 alumnos)
    - **Aula7** (15 equipos)
  - **Segundo** (15 alumnos)
    - **Aula8** (15 equipos)

#### Profesores
- **Grupo: Profesores**

## Justificación de la Estructura

### Separación de Alumnos y Profesores
La división de los usuarios en unidades organizativas separadas permite establecer políticas de seguridad personalizadas para cada grupo. Esto facilita la administración de permisos, accesos y restricciones, garantizando un entorno seguro y funcional.

### Organización por Ciclos Formativos
Cada ciclo formativo se organiza en su propia unidad organizativa con subdivisiones para los cursos de primero y segundo. Esto permite una gestión más eficaz de usuarios y recursos, asegurando que cada grupo tenga acceso a las herramientas necesarias.

### Aulas como Unidades Organizativas
Cada aula se ha definido como una UO dentro del curso correspondiente. Esto permite aplicar políticas específicas a los equipos dentro de cada aula y segmentar la gestión de recursos de manera ordenada.

### Centralización de Profesores
Los profesores se agrupan en una única UO, facilitando la administración de permisos y accesos a recursos compartidos. Además, se pueden establecer excepciones a ciertas políticas aplicadas a los alumnos para garantizar su capacidad administrativa.

## Automatización del Alta de Usuarios y Creación de Carpetas
Se ha implementado un script en PowerShell que automatiza la creación de usuarios en Active Directory y la asignación de carpetas personales y compartidas. Este proceso garantiza que todos los alumnos sean dados de alta correctamente en el dominio, reduciendo errores y optimizando el tiempo de administración.

### Justificación del Script de Automatización
El uso de PowerShell permite gestionar Active Directory de forma programática y eficiente. La automatización evita errores manuales y asegura la consistencia en la creación de usuarios y asignación de permisos. Además, al integrarlo con `Import-CSV`, se pueden importar listas de alumnos fácilmente, facilitando la gestión de nuevas matriculaciones.

## Estructura de Carpetas

Cada alumno tiene una carpeta personal en un recurso compartido del servidor, con permisos exclusivos. Además, se han creado carpetas compartidas para cada grupo de alumnos, permitiendo la colaboración en trabajos y proyectos.

- **Carpetas Personales:** `\\Servidor\Usuarios\NombreUsuario`
- **Carpetas de Grupo:** `\\Servidor\Grupos\CicloCurso`

Esta segmentación garantiza que cada usuario tenga un espacio privado, mientras que los grupos disponen de un entorno colaborativo controlado.

## Políticas de Seguridad
Se han implementado diversas políticas de grupo (GPO) para mejorar la seguridad y administración del entorno:

### Directivas de Contraseña
- Longitud mínima: **10 caracteres**
- Complejidad obligatoria: **Sí**
- Expiración: **180 días**

### Bloqueo de Cuentas
- Número de intentos fallidos: **6**
- Duración del bloqueo: **10 minutos**

### Restricciones de Acceso
- Prohibido acceso al **CMD** para alumnos (excepto profesores).
- Filtrado de contenido en Internet para restringir acceso a páginas no educativas.
- Deshabilitado el cambio de fondo de pantalla en equipos de alumnos.
- Prohibición de acceso al **Panel de Control** y configuración para alumnos.

## Script
```bash
   # Generación del UPN
    $upn = "$nombreUsuario@$dom"
    $contador = 1
    while (Get-ADUser -Filter {UserPrincipalName -eq $upn}) {
        $nombreUsuario = "$nombreUsuario$contador"
        $upn = "$nombreUsuario@$dom"
        $contador++
    }
    
    # Definición de la contraseña por defecto
    $password = ConvertTo-SecureString -String "Passw0rd!" -AsPlainText -Force
    
    # Creación del usuario en Active Directory
    New-ADUser -GivenName $alumno.Nombre `
                -Surname "$($alumno.'Primer Apellido') $($alumno.'Segundo Apellido')" `
                -Name "$($alumno.Nombre) $($alumno.'Primer Apellido')" `
                -SamAccountName $nombreUsuario `
                -UserPrincipalName $upn `
                -Path "OU=$($alumno.Curso),OU=$($alumno.Ciclo),OU=Alumnos,DC=micentro,DC=local" `
                -HomeDirectory "\\MICENTRO\instituto\$nombreUsuario" `
                -HomeDrive 'H:' `
                -AccountPassword $password `
                -Enabled $true
    
    # Creación de la carpeta personal del usuario
    $carpetaUsuario = "C:\Shares\instituto\$nombreUsuario"
    New-Item -Path $carpetaUsuario -ItemType Directory -ErrorAction SilentlyContinue
    
    # Configuración de permisos de la carpeta personal
    $acl = Get-Acl $carpetaUsuario
    $acl.SetAccessRuleProtection($true, $false)
    $permiso = New-Object System.Security.AccessControl.FileSystemAccessRule("$nombreUsuario", "FullControl", "ContainerInherit,ObjectInherit", "None", "Allow")
    $acl.SetAccessRule($permiso)
    Set-Acl -Path $carpetaUsuario -AclObject $acl
    
    # Agregando usuario al grupo del curso
    Add-ADGroupMember -Identity "$($alumno.Curso)" -Members $nombreUsuario
    
} catch {
    Write-Host "Error al crear usuario $($alumno.Nombre): $_"
}
```

## Conclusión
La implementación de esta infraestructura de dominio asegura un entorno controlado, seguro y eficiente para el centro educativo. La automatización reduce errores y tiempo de administración, mientras que la aplicación de políticas garantiza la seguridad y estabilidad del sistema.