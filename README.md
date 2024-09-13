# Configuración de API en Docker con Conexión a SQL Server

## Descripción

Este documento detalla los pasos necesarios para configurar una API en .NET para conectarse a una instancia de SQL Server, así como para ejecutarla dentro de un contenedor Docker en un entorno de desarrollo.

## Requisitos Previos

- **Docker**: Instalado en tu máquina.
- **SQL Server**: Instalado y configurado.
- **.NET SDK**: Instalado en tu máquina.

## Pasos para Configurar la Conexión a SQL Server

### 1. **Configurar SQL Server**

1. **Abrir SQL Server Configuration Manager**:
   - Accede a **SQL Server Configuration Manager**.

2. **Configurar TCP/IP**:
   - Ve a **SQL Server Network Configuration** y selecciona **Protocols for [tu instancia]**.
   - Haz doble clic en **TCP/IP** y ve a la pestaña **IP Addresses**.
   - En la sección **IPAll**, asegúrate de que el puerto TCP sea **1433** y que la opción **Enable** esté habilitada.
   - En **IPAll**, verifica la opción **Force Encryption** y, si está habilitada, desactívala temporalmente para pruebas.

3. **Reiniciar el Servicio SQL Server**:
   - En **SQL Server Configuration Manager**, selecciona **SQL Server Services**.
   - Haz clic derecho en **SQL Server (MSSQLSERVER)** (o el nombre de tu instancia) y selecciona **Restart**.

4. **Verificar Acceso Remoto**:
   - Asegúrate de que el firewall permita el tráfico en el puerto **1433**.

### 2. **Configurar la Cadena de Conexión**
#### Obtener la IP local : 
1. Para obtener la IP local ejecutar en un bash 

```
ipconfig
```
2. Seleccionar la ip 
```
Adaptador de LAN inalámbrica Wi-Fi:

   Sufijo DNS específico para la conexión. . :
   Dirección IPv6 . . . . . . . . . . : --------------
   Dirección IPv6 temporal. . . . . . : --------------
   Vínculo: dirección IPv6 local. . . : --------------
   Dirección IPv4. . . . . . . . . . . . . . : 192.168.xxx.xx <- ESTA 
```

1. **Actualizar `appsettings.json`**:
   - En tu archivo `appsettings.json`, configura la cadena de conexión:
     ```json
     {
       "ConnectionStrings": {
         "DefaultConnection": "Server=tuIpLocal,1433;Database=CustomerTestDocket;User Id=xxxxxxx;Password=xxxxx;TrustServerCertificate=True;"
       }
     }
     ```

2. **Configurar Validación de Certificado** (si se usa SSL/TLS):
   - En tu aplicación .NET, puedes agregar un callback de validación de certificado (para pruebas):
     ```csharp
     using System.Net.Security;
     using System.Security.Cryptography.X509Certificates;

     ServicePointManager.ServerCertificateValidationCallback = 
         (sender, certificate, chain, sslPolicyErrors) => true;
     ```

### 3. **Probar la Conexión**

1. **Desde tu Máquina Local**:
   - Usa el comando `Test-NetConnection` para verificar la conectividad:
     ```powershell
     Test-NetConnection -ComputerName tuIpLocal -Port 1433
     ```

2. **Desde el Contenedor Docker**:
   - Abre una terminal en el contenedor y usa `nc` para verificar la conexión:
     ```bash
     nc -zv tuIpLocal 1433
     ```

### 4. **Configurar Docker**

1. **Crear el Archivo `Dockerfile`**:
   - Asegúrate de que tu archivo `Dockerfile` esté configurado para tu aplicación .NET. Aquí hay un ejemplo básico:
     ```dockerfile
     # Use the official .NET SDK image
     FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
     WORKDIR /app
     EXPOSE 80

     # Use the official .NET SDK image to build the app
     FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
     WORKDIR /src
     COPY ["YourProject.csproj", "./"]
     RUN dotnet restore "YourProject.csproj"
     COPY . .
     WORKDIR "/src/."
     RUN dotnet build "YourProject.csproj" -c Release -o /app/build

     FROM build AS publish
     RUN dotnet publish "YourProject.csproj" -c Release -o /app/publish

     FROM base AS final
     WORKDIR /app
     COPY --from=publish /app/publish .
     ENTRYPOINT ["dotnet", "YourProject.dll"]
     ```

2. **Construir y Ejecutar el Contenedor**:
   - Construye tu imagen de Docker:
     ```bash
     docker build -t yourapp .
     ```
   - Ejecuta el contenedor:
     ```bash
     docker run -d -p 8080:80 yourapp
     ```

## Problemas Comunes y Soluciones

- **Error de Conexión con el Servidor SQL**: Verifica la configuración del firewall y asegúrate de que el puerto 1433 esté abierto.
- **Errores de SSL/TLS**: Asegúrate de que la opción `TrustServerCertificate=True` esté en la cadena de conexión. Para pruebas, puedes desactivar la validación del certificado.

## Conclusión

Siguiendo estos pasos, deberías poder conectar tu API en .NET a una instancia de SQL Server y ejecutarla dentro de un contenedor Docker. Si encuentras problemas adicionales, verifica los registros de Docker y SQL Server para más detalles.

