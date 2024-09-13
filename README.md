# Configuraci�n de API en Docker con Conexi�n a SQL Server

## Descripci�n

Este documento detalla los pasos necesarios para configurar una API en .NET para conectarse a una instancia de SQL Server, as� como para ejecutarla dentro de un contenedor Docker en un entorno de desarrollo.

## Requisitos Previos

- **Docker**: Instalado en tu m�quina.
- **SQL Server**: Instalado y configurado.
- **.NET SDK**: Instalado en tu m�quina.

## Pasos para Configurar la Conexi�n a SQL Server

### 1. **Configurar SQL Server**

1. **Abrir SQL Server Configuration Manager**:
   - Accede a **SQL Server Configuration Manager**.

2. **Configurar TCP/IP**:
   - Ve a **SQL Server Network Configuration** y selecciona **Protocols for [tu instancia]**.
   - Haz doble clic en **TCP/IP** y ve a la pesta�a **IP Addresses**.
   - En la secci�n **IPAll**, aseg�rate de que el puerto TCP sea **1433** y que la opci�n **Enable** est� habilitada.
   - En **IPAll**, verifica la opci�n **Force Encryption** y, si est� habilitada, desact�vala temporalmente para pruebas.

3. **Reiniciar el Servicio SQL Server**:
   - En **SQL Server Configuration Manager**, selecciona **SQL Server Services**.
   - Haz clic derecho en **SQL Server (MSSQLSERVER)** (o el nombre de tu instancia) y selecciona **Restart**.

4. **Verificar Acceso Remoto**:
   - Aseg�rate de que el firewall permita el tr�fico en el puerto **1433**.

### 2. **Configurar la Cadena de Conexi�n**
#### Obtener la IP local : 
1. Para obtener la IP local ejecutar en un bash 

```
ipconfig
```
2. Seleccionar la ip 
```
Adaptador de LAN inal�mbrica Wi-Fi:

   Sufijo DNS espec�fico para la conexi�n. . :
   Direcci�n IPv6 . . . . . . . . . . : --------------
   Direcci�n IPv6 temporal. . . . . . : --------------
   V�nculo: direcci�n IPv6 local. . . : --------------
   Direcci�n IPv4. . . . . . . . . . . . . . : 192.168.xxx.xx <- ESTA 
```

1. **Actualizar `appsettings.json`**:
   - En tu archivo `appsettings.json`, configura la cadena de conexi�n:
     ```json
     {
       "ConnectionStrings": {
         "DefaultConnection": "Server=tuIpLocal,1433;Database=CustomerTestDocket;User Id=xxxxxxx;Password=xxxxx;TrustServerCertificate=True;"
       }
     }
     ```

2. **Configurar Validaci�n de Certificado** (si se usa SSL/TLS):
   - En tu aplicaci�n .NET, puedes agregar un callback de validaci�n de certificado (para pruebas):
     ```csharp
     using System.Net.Security;
     using System.Security.Cryptography.X509Certificates;

     ServicePointManager.ServerCertificateValidationCallback = 
         (sender, certificate, chain, sslPolicyErrors) => true;
     ```

### 3. **Probar la Conexi�n**

1. **Desde tu M�quina Local**:
   - Usa el comando `Test-NetConnection` para verificar la conectividad:
     ```powershell
     Test-NetConnection -ComputerName tuIpLocal -Port 1433
     ```

2. **Desde el Contenedor Docker**:
   - Abre una terminal en el contenedor y usa `nc` para verificar la conexi�n:
     ```bash
     nc -zv tuIpLocal 1433
     ```

### 4. **Configurar Docker**

1. **Crear el Archivo `Dockerfile`**:
   - Aseg�rate de que tu archivo `Dockerfile` est� configurado para tu aplicaci�n .NET. Aqu� hay un ejemplo b�sico:
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

- **Error de Conexi�n con el Servidor SQL**: Verifica la configuraci�n del firewall y aseg�rate de que el puerto 1433 est� abierto.
- **Errores de SSL/TLS**: Aseg�rate de que la opci�n `TrustServerCertificate=True` est� en la cadena de conexi�n. Para pruebas, puedes desactivar la validaci�n del certificado.

## Conclusi�n

Siguiendo estos pasos, deber�as poder conectar tu API en .NET a una instancia de SQL Server y ejecutarla dentro de un contenedor Docker. Si encuentras problemas adicionales, verifica los registros de Docker y SQL Server para m�s detalles.

