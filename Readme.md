## Prueba Parcial 2 ##

**Instrucciones del examen**

Responda a la siguiente pregunta con la URL del repositorio de GitHub. Recuerde que en caso de usar IA generativa, su repositorio debe tener 2 branch. El primero con su código y el segundo con el código generado por IA y el prompt que usó para generarlo.

Agregar en el archivo README, capturas de ejecución del programa.


**VERSIÓN 2**

- Descargue el archivo CSV de ataques informáticosLinks to an external site.. 
- Genere la tabla en MYSQL para esta información.

- Elabore un programa que inyecte los datos del archivo CSV a la base de datos. 

# Resolucion.

Primero hice uso de las librerias vistas y usadas en las clases de Programacion Funcianol y Reactiva.
<img width="1086" height="783" alt="image" src="https://github.com/user-attachments/assets/3485019d-db69-44fe-abaa-102128f30c46" />


Seguido use los imports, y realice la coneccion a la base de datos que use para la resolucion del ejercicio. En este caso MySql.

```Scala
import doobie._
import doobie.implicits._
import cats.effect._
import cats.implicits._
import scala.io.Source

object Prueba2 extends IOApp.Simple {

  case class Ataque(
                     id: Int,
                     tipo: String,
                     ipOrigen: String,
                     ipDestino: String,
                     severidad: String,
                     paisOrigen: String,
                     paisDestino: String,
                     fecha: String,
                     duracion: Int,
                     sensible: Boolean
                   )

  // CONFIGURACIÓN DB
  val xa = Transactor.fromDriverManager[IO](
    driver = "com.mysql.cj.jdbc.Driver",
    url = "jdbc:mysql://localhost:3306/Prueba2",
    user = "root",
    password = "Francis27S",
    logHandler = None
  )
```

Despues cree la tabla Sql en mi codigo de Scala, haciendo uso de la libreria doobie. Para que no genere un error.

```Scala
val createTable: ConnectionIO[Int] =
    sql"""
      CREATE TABLE IF NOT EXISTS ataques_registrados (
        id_csv INT,
        tipo_ataque VARCHAR(100),
        ip_origen VARCHAR(50),
        ip_destino VARCHAR(50),
        severidad VARCHAR(20),
        pais_origen VARCHAR(100),
        pais_destino VARCHAR(100),
        fecha_ataque DATETIME,
        duracion_minutos INT,
        sensible BOOLEAN
      )
    """.update.run
```

Luego realizo un mapeo de la lista Ataques y hago la insercion de los datos 

```Scala
def insertMany(ataques: List[Ataque]): ConnectionIO[Int] = {
    val sql = """
      INSERT INTO ataques_registrados
      (id_csv, tipo_ataque, ip_origen, ip_destino, severidad, pais_origen, pais_destino, fecha_ataque, duracion_minutos, sensible)
      VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
    """
    Update[Ataque](sql).updateMany(ataques)
  }
```

Luego mapeo las columnas del csv, para convertir cada columna de texto en su tipo correspondiente.

```Scala
def loadCsvData(filePath: String): IO[List[Ataque]] = IO {
    val source = Source.fromFile(filePath)
    val data = source.getLines()
      .drop(1) //
      .map { line =>
        val cols = line.split(",")

        Ataque(
          id          = cols(0).trim.toInt,
          tipo        = cols(1).trim,
          ipOrigen    = cols(2).trim,
          ipDestino   = cols(3).trim,
          severidad   = cols(4).trim,
          paisOrigen  = cols(5).trim,
          paisDestino = cols(6).trim,
          fecha       = cols(7).trim,
          duracion    = cols(8).trim.toInt,
          sensible    = cols(9).trim.toBoolean
        )
      }.toList
    source.close()
    data
  }
```

Por ultimo realizo la ejecucion de todo el programa.

```Scala
override def run: IO[Unit] = {

    val csvPath = "C:\\Scala\\b2-proyecto-integrador-c-echo\\Prueba Parcial2\\src\\main\\resources\\ataques.csv"

    for {
      _       <- IO.println("Inicio de Procedimiento")

      _       <- createTable.transact(xa)
      _       <- IO.println("Tabla lista.")

      attacks <- loadCsvData(csvPath)
      _       <- IO.println(s"Datos leídos del CSV: ${attacks.size}.")

      count   <- insertMany(attacks).transact(xa)
      _       <- IO.println(s"Inserción completa. Filas afectadas: $count")

    } yield ()
  }
}
```

## Reusltados del Codigo.

**Scala**

<img width="646" height="467" alt="image" src="https://github.com/user-attachments/assets/0ef4f157-b9a7-4ed8-a859-bcee03bc0ae6" />


**MySql**

<img width="835" height="671" alt="image" src="https://github.com/user-attachments/assets/32c5f06e-fda1-4df6-8649-5e48ba8febfc" />





