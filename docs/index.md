# Manual de Instalación y Registro de Procedimiento

**Sistema:** Santiago Seguros Analytics (Python + MySQL)  
**Herramienta de Empaquetado:** Inno Setup Compiler  
**Versión del Documento:** 1.0  

---

## 1. Requerimientos de Hardware y Software para la Creación del Instalador (Estación de Trabajo del Desarrollador)
Antes de iniciar el proceso de creación del instalador, la máquina utilizada para el desarrollo debe cumplir con los siguientes prerrequisitos:
- **Sistema Operativo:** Windows 10 o superior (64 bits).
- **Software Base:**
    - Python 3.8 o superior instalado y configurado en el PATH del sistema.
    - MySQL Server (puede ser la versión Community) o XAMPP/WAMP para pruebas locales.
    - Biblioteca `mysql-connector-python` instalada via pip.
    - Inno Setup Compiler (versión 5.4 o superior). Es indispensable marcar la opción de instalar el "Inno Setup Preprocessor" durante su instalación, ya que permite usar scripts más avanzados.
- **Estructura de Archivos (Fuente):** Se debe tener una carpeta raíz del proyecto (ej. `C:\SantiagoSeguros\Dist`) que contenga:
    - El script principal de Python (ej. `main.py`).
    - Los módulos y paquetes del sistema.
    - El script SQL de inicialización de la base de datos (`init_database.sql`).
    - El archivo `.ico` para el ícono de la aplicación.
    - (Opcional) El ejecutable de MySQL si se opta por la instalación embebida.

## 2. Pasos Necesarios para la Configuración, Compilación e Instalación del Instalador (Procedimiento en Inno Setup)

### Paso 1: Preparación de la Aplicación Python (Creación del Ejecutable Base)
Para asegurar la portabilidad y evitar dependencias externas en el equipo del cliente, se recomienda compilar la aplicación Python en un archivo `.exe` antes de crear el instalador final.
1. Abrir la terminal (CMD) en la carpeta del proyecto.
2. Ejecutar el comando: `pip install pyinstaller`.
3. Ejecutar el comando de empaquetado:
    ```cmd
    pyinstaller --onefile --windowed --icon=logo.ico --name "SantiagoSeguros" main.py
    ```
    *Este comando generará una carpeta `dist` que contiene el archivo `SantiagoSeguros.exe`*.

### Paso 2: Configuración Inicial del Script de Inno Setup
1. Abrir el programa **Inno Setup Compiler**.
2. En el menú, seleccionar `File` -> `New`. Esto abrirá el "Application Wizard" (Asistente).
3. Hacer clic en "Next". Rellenar la información básica:
    - **Application name:** Santiago Seguros Analytics.
    - **Application version:** 1.0.
    - **Application publisher:** Santiago Seguros.
    - **Application website:** (Opcional, ej. `www.santiagoseguros.cl`).
4. Hacer clic en "Next". En la pantalla "Application Destination", aceptar la carpeta por defecto (`Program Files\Santiago Seguros Analytics`).

### Paso 3: Configuración de Archivos a Empaquetar
1. En el paso "Application Files", hacer clic en "Add file(s)..." y seleccionar el `SantiagoSeguros.exe` generado en el Paso 1.
2. Volver a hacer clic en "Add file(s)..." y agregar el script `init_database.sql`.
3. Si se incluye MySQL embebido, hacer clic en "Add folder..." y seleccionar la carpeta que contiene los binarios de MySQL.
4. En la ventana de propiedades del archivo `SantiagoSeguros.exe`, marcar la casilla "This is the main executable file".

### Paso 4: Configuración de Accesos Directos y Registro (Íconos)
1. En el paso "Application Icons", hacer clic en "Add entry...".
2. Para la ubicación del acceso directo, seleccionar "Program Menu folder" y "Desktop folder".
3. En "Name", escribir "Santiago Seguros Analytics".
4. En "Filename", hacer clic en el botón "Browse..." y seleccionar `SantiagoSeguros.exe`.
5. En "Icon filename", seleccionar el archivo `.ico` del proyecto.

### Paso 5: Configuración del Lenguaje y Modo de Instalación
1. En el paso "Setup Languages", seleccionar "English" y "Spanish". Inno Setup soporta instalaciones multilingüe, lo cual es útil para entornos corporativos.
2. En el paso "Compiler Settings", definir el nombre del archivo de salida (ej. `SantiagoSeguros_Setup.exe`) y la carpeta donde se guardará.

### Paso 6: Inclusión de las Pruebas y Procedimiento de Marcha Atrás (Código Pascal Script)
Este es el paso más crítico para cumplir con el instructivo formal. Se debe hacer clic en "Next" hasta llegar a "Finish" y luego marcar la opción "Yes" para editar el script `.iss` generado. Se debe agregar el siguiente código en la sección `[Code]` para automatizar la verificación de requisitos e instalación de MySQL.

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

### Paso 7: Compilación del Instalador Final
1. Guardar el script con la extensión `.iss` (ej. `installer_santiago.iss`).
2. En el menú de Inno Setup, hacer clic en `Compile` -> `Compile` (o presionar `F9`).
3. Verificar que no aparezcan errores en la consola de salida. Si la compilación es exitosa, se generará el archivo `SantiagoSeguros_Setup.exe` en la carpeta de salida definida.

## 3. Pruebas para Asegurar la Instalación Correcta
Una vez generado el instalador (`SantiagoSeguros_Setup.exe`), se deben ejecutar las siguientes pruebas en un equipo limpio (sin Python ni MySQL):
- **Prueba de Integridad:** Ejecutar el `setup.exe` y verificar que se descompriman todos los archivos en la carpeta `Program Files`.
- **Prueba de Funcionalidad:** Abrir el acceso directo del escritorio. Verificar que la aplicación se conecte a la base de datos MySQL (ya sea local o remota) y que permita ingresar un nuevo cliente con su tipo de seguro e ingresos anuales.
- **Prueba de Validación:** Confirmar que si el equipo no cumple con los requisitos (sin MySQL), el instalador muestre el mensaje de error configurado en el `InitializeSetup` y cancele la instalación.

## 4. Procedimiento de Marcha Atrás (Desinstalación)
En caso de que la instalación no sea exitosa o se requiera revertir el sistema:
1. Ejecutar el desinstalador que se creó automáticamente en la carpeta `C:\Program Files\Santiago Seguros Analytics\unins000.exe`.
2. El desinstalador eliminará los archivos de la aplicación y los accesos directos.
3. Si se seleccionó la opción durante la desinstalación, el script ejecutará la eliminación de la base de datos, dejando el equipo en el mismo estado en que se encontraba antes de la instalación.
