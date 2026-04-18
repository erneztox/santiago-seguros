# Manual de Instalación y Registro de Procedimiento

**Sistema:** Santiago Seguros Analytics (Python + MySQL)  
**Herramienta de Empaquetado:** Inno Setup Compiler  
**Versión del Documento:** 1.0  

---

## 1. Requerimientos de Hardware y Software

Antes de iniciar el proceso de creación del instalador, la máquina utilizada para el desarrollo debe cumplir con los siguientes prerrequisitos:

### 💻 Sistema Operativo
- **Windows 10** o superior (64 bits).

### ⚙️ Software Base
- **Python 3.8 o superior** instalado y configurado en el PATH del sistema.
- **MySQL Server** (versión Community) o XAMPP/WAMP para pruebas locales.
- **Biblioteca Python:** `mysql-connector-python` instalada vía pip.
- **Inno Setup Compiler** (versión 5.4 o superior).

!!! warning "Importante: Inno Setup Preprocessor"
    Es indispensable marcar la opción de instalar el **"Inno Setup Preprocessor"** durante la instalación de Inno Setup. Esto permite utilizar scripts avanzados necesarios para automatizar tareas complejas.

### 📁 Estructura de Archivos (Fuente)
Se debe contar con una carpeta raíz del proyecto (ej. `C:\SantiagoSeguros\Dist`) estructurada de la siguiente manera:

- `main.py` (Script principal de Python).
- Módulos y paquetes dependientes del sistema.
- `init_database.sql` (Script SQL para inicializar la base de datos).
- `logo.ico` (Archivo de ícono de la aplicación).
- *(Opcional)* El ejecutable instalador de MySQL si se opta por una instalación embebida.

---

## 2. Configuración y Compilación (Procedimiento en Inno Setup)

### Paso 1: Preparación del Ejecutable Base (Python)
Para asegurar la portabilidad y evitar dependencias en el equipo del cliente, compilaremos la aplicación Python en un archivo `.exe`.

1. Abre la terminal (CMD) en la carpeta del proyecto.
2. Instala la herramienta de empaquetado ejecutando:
   ```cmd
   pip install pyinstaller
   ```
3. Ejecuta el comando de empaquetado:
   ```cmd
   pyinstaller --onefile --windowed --icon=logo.ico --name "SantiagoSeguros" main.py
   ```
   > **Nota:** Este comando generará una carpeta `dist` que contendrá tu nuevo archivo `SantiagoSeguros.exe`.

### Paso 2: Configuración del Script de Inno Setup
1. Abre el programa **Inno Setup Compiler**.
2. En el menú, selecciona `File` -> `New` para abrir el asistente (Application Wizard).
3. Haz clic en **Next** y rellena la información básica:
    - **Application name:** Santiago Seguros Analytics.
    - **Application version:** 1.0.
    - **Application publisher:** Santiago Seguros.
4. En **Application Destination**, acepta la ruta por defecto (`Program Files\Santiago Seguros Analytics`).

### Paso 3: Selección de Archivos
1. En **Application Files**, haz clic en `Add file(s)...` y selecciona el `SantiagoSeguros.exe` generado en el paso anterior.
2. Marca la casilla **"This is the main executable file"** en sus propiedades.
3. Vuelve a hacer clic en `Add file(s)...` y agrega `init_database.sql`.
4. *(Si aplica)* Usa `Add folder...` para incluir los binarios de MySQL.

### Paso 4: Accesos Directos
1. En **Application Icons**, haz clic en `Add entry...`.
2. Ubicación: Selecciona `Program Menu folder` y `Desktop folder`.
3. Nombre: `Santiago Seguros Analytics`.
4. Archivo: Selecciona `SantiagoSeguros.exe`.
5. Ícono: Selecciona el archivo `.ico` de tu proyecto.

### Paso 5: Opciones Finales
1. En **Setup Languages**, selecciona **Spanish** (y English si se desea).
2. En **Compiler Settings**, define el nombre de salida del instalador (ej. `SantiagoSeguros_Setup.exe`).

### Paso 6: Script de Automatización (Pascal Script)
Este es el paso más crítico. Haz clic en "Finish", selecciona "Yes" para editar el script generado y agrega el siguiente código al final del archivo en la sección `[Code]`.

```pascal
[Code]
function InitializeSetup(): Boolean;
var
  ResultCode: Integer;
  ErrorMsg: String;
begin
  // Prueba 1: Verificar si MySQL está instalado consultando el registro de Windows
  if not RegKeyExists(HKLM, 'SOFTWARE\MySQL AB') then
  begin
    ErrorMsg := 'MySQL Server no está instalado en este sistema. El instalador intentará instalar los componentes necesarios.';
    MsgBox(ErrorMsg, mbInformation, MB_OK);

    // Ejecutar instalador embebido de MySQL (Ejemplo de acción automática)
    if not Exec(ExpandConstant('{tmp}\mysql_installer.exe'), '/quiet', '', SW_HIDE, ewWaitUntilTerminated, ResultCode) then
    begin
      MsgBox('Error al instalar MySQL. Código: ' + IntToStr(ResultCode), mbError, MB_OK);
      Result := False;
      Exit;
    end;
  end;

  // Prueba 2: Verificar si Python está instalado
  if not RegKeyExists(HKLM, 'SOFTWARE\Python') then
  begin
    ErrorMsg := 'Python 3 no está instalado. La aplicación no funcionará correctamente. Contacte al administrador.';
    MsgBox(ErrorMsg, mbError, MB_OK);
    Result := False;
    Exit;
  end;

  Result := True;
end;

// Procedimiento de Marcha Atrás (Rollback)
procedure CurUninstallStepChanged(CurUninstallStep: TUninstallStep);
begin
  if CurUninstallStep = usUninstall then
  begin
    if MsgBox('¿Desea eliminar también la base de datos de clientes?', mbConfirmation, MB_YESNO) = IDYES then
    begin
      // Lógica para ejecutar script de eliminación de base de datos
      Exec('mysql', '-u root -p -e "DROP DATABASE santiago_seguros_db;"', '', SW_HIDE, ewWaitUntilTerminated, ResultCode);
    end;
  end;
end;
```

### Paso 7: Compilación Final
1. Guarda tu trabajo con la extensión `.iss` (ej. `installer.iss`).
2. Haz clic en `Compile` -> `Compile` (o presiona **F9**).
3. Si todo está correcto, se generará el archivo `SantiagoSeguros_Setup.exe` listo para usar.

---

## 3. Pruebas de Validación

Una vez generado el instalador, valídalo en un equipo limpio:

- **Prueba de Integridad:** Ejecuta el `setup.exe` y verifica que los archivos se descompriman en `Program Files`.
- **Prueba de Funcionalidad:** Abre el sistema desde el acceso directo y verifica la conexión a la base de datos MySQL.
- **Prueba de Excepciones:** Simula un equipo sin MySQL instalado; el instalador debe mostrar la advertencia configurada en el script de validación.

## 4. Procedimiento de Marcha Atrás (Desinstalación)

En caso de requerir revertir la instalación:

1. Ejecuta el archivo `unins000.exe` ubicado en el directorio de la aplicación.
2. Sigue el asistente para eliminar archivos y accesos directos.
3. Cuando el sistema lo pregunte, aprueba la eliminación de la base de datos para dejar el equipo en su estado original.
