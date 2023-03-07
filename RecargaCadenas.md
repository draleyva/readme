# RecargaCadenas

## Modificaciones

La aplicación ha sido modificada para dar soporte a múltiples cadenas. Los cambios se han producido en los siguientes ámbitos

1. Base de datos
	Creación de dos nuevas tablas que incluyen el posfijo MT para indicar la naturaleza multithread de la aplicación y cambios en la estructura
	
![Diagrama](https://iili.io/HW4OuUb.png)

- tb_RecargaCadenasConfiguracionMT

	**identidad** es la llave numérica que identificará cada cadena
	Se añadió un campo **Estado** para habilitar cadena según conveniencia. El símbolo 0 indica deshabilitado
	**TopUpIdEntidad** que anteriormente estuvo en el archivo de configuración de la aplicación ahora es parte de cada cadena.
	Los demás parámetros tienen el mismo significado que se ha usado hasta el momento 
- tb_RecargaCadenasConfiguracion
	
	**identidad** llave foránea parte de  la llave principal para encadenar los registros
	**NombreArchivo** Dado que toda operación debería se identificada de forma inequívoca por la fuente de la que proviene
	**Terminal** añadido para identificar la operación  
	

2.  Código fuente
Soporte multithread para manejar las cadenas de forma independiente. La aplicación espera por la finalización de cada subproceso asociado a una cadena

```C#
     List<Task> tasklist = new List<Task>();
     string formatid;

     foreach (var chain in chains)
     {
       if(!chain.Estado)
       {
         formatid = String.Format("{0:000000}", chain.Id);
         log.Info($"Cadena [{formatid}] <{chain.Description}> está deshablitada");
         continue;
       }
       var logic = new Logic(chain);
       tasklist.Add(logic.start());
     }

     Task.WaitAll(tasklist.ToArray());
```

En el archivo log se podrá identificar las operaciones de cada una de las cadenas por el nombre del thread que ha sido renombrado en código para que use el **identidad** formateado en cuatro posiciones y completado con '0' 

3. Configuración Log (log4net.config)

	Se modificado el patrón de fecha para usar yyyy-MM-dd

```xml
	<datePattern value="yyyy-MM-dd'.log'" />
```

## Operativa

La operativa de la aplicación es similar a la que estaba implementada pero permite el procesamiento simultáneo de múltiples cadenas que serán configuradas con los parámetros adecuados en la tabla tb_RecargaCadenasConfiguracion por cada cadena.

Los nombre de los archivos para cada cadena de preferencia deben ser colocados en ubicaciones diferentes. Por cada ejecución la aplicación soporta el procesamiento de múltiples archivos por cadena que serán procesados de forma secuencial.

El patrón de nombre de archivo es **"RECARGAS_{RUC}_*.csv** donde el nombre de RUC varía por cada cadena.

Cada archivo luego de ser descargado desde el sftp será movido a una carpeta con el nombre **descargado** para que no sea encontrado en el siguiente procesamiento. Por esta razón es necesario contar con permisos de escritura en la carpeta sftp.

Para establecer la ruta del archivo log se recomienda establecer una ruta absoluta en la propiedad correspondiente

```xml
 <appender name="RollingLogFileAppender" type="log4net.Appender.RollingFileAppender">
      ...
      <file value="C:\GKNdevelopment\RecargaCadenas\RecargaCadenas\bin\Release\netcoreapp3.1\log\" />

```
