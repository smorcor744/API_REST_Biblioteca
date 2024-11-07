
# **Actividad: Implementación de un CRUD para "Libros" y "Autores"**

## **Objetivo:**
Desarrollar una API REST en Kotlin utilizando Spring Boot que permita realizar operaciones CRUD sobre las entidades **Libros** y **Autores**, trabajando directamente con las entidades y estableciendo relaciones entre ellas.

---

## **1. Requerimientos**

1. **Endpoints para Autores**:
    - Crear un autor.
    - Consultar todos los autores.
    - Consultar un autor por su ID.
    - Modificar un autor por su ID.
    - Eliminar un autor por su ID.
    - Consultar los libros de un autor.

2. **Endpoints para Libros**:
    - Crear un libro.
    - Consultar todos los libros.
    - Consultar un libro por su ID.
    - Modificar un libro por su ID.
    - Eliminar un libro por su ID.
    - Consultar libros por género.

3. **Persistencia**:
    - Usar una base de datos **MySQL**.
    - Configurar JPA/Hibernate para manejar las operaciones con la base de datos.

---

## **2. Guía de Desarrollo**

### **2.1. Configuración del Proyecto**
1. Crear un proyecto en **Spring Boot** con Kotlin usando las siguientes dependencias:
    - Spring Web.
    - Spring Data JPA.
    - MySQL Driver.

2. Configurar el archivo `application.properties`:
    ```properties
    # Configuracion para el acceso a la Base de Datos
   spring.jpa.hibernate.ddl-auto=create
   spring.jpa.properties.hibernate.globally_quoted_identifiers=true
   spring.jpa.show-sql=true
   
   # Puerto donde escucha el servidor una vez se inicie
   server.port=8080
   
   # Datos de conexion con la base de datos MySQL
   spring.jpa.database-platform=org.hibernate.dialect.MySQLDialect
   spring.datasource.url=jdbc:mysql://localhost:3306/libreria_bd
   spring.datasource.username=root
   spring.datasource.password=

# Para popular la BBDD
spring.jpa.properties.javax.persistence.sql-load-script-source=sql/datos-prueba.sql
    ```

---

### **2.2. Modelado de Entidades**

#### **Entidad `Autor`**
```kotlin
import jakarta.persistence.*
import org.hibernate.annotations.Cascade
import org.hibernate.annotations.CascadeType

@Entity
data class Autor(
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long = 0,

    var nombre: String,
    var nacionalidad: String,
    var anioNacimiento: Int,

    @Column(length = 1000)
    var biografia: String? = null,

    @OneToMany(mappedBy = "autor", cascade = [jakarta.persistence.CascadeType.ALL], orphanRemoval = true)
    val libros: MutableList<Libro> = mutableListOf()
)
```

#### **Entidad `Libro`**
```kotlin
import jakarta.persistence.*
import java.time.LocalDate

@Entity
data class Libro(
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long = 0,

    var titulo: String,
    var genero: String,
    var anioPublicacion: Int,
    var precio: Double,

    @ManyToOne
    @JoinColumn(name = "autor_id", nullable = false)
    var autor: Autor
)
```

---

### **2.3. Crear Repositorios**

```kotlin
import org.springframework.data.jpa.repository.JpaRepository

interface AutorRepository : JpaRepository<Autor, Long>
interface LibroRepository : JpaRepository<Libro, Long>
```

---

### **2.4. Crear Servicios**

#### **Servicio para `Autor`**
```kotlin
import org.springframework.stereotype.Service

@Service
class AutorService(private val autorRepository: AutorRepository) {

    fun crearAutor(autor: Autor): Autor {
        return autorRepository.save(autor)
    }

    fun obtenerTodosLosAutores(): List<Autor> {
        return autorRepository.findAll()
    }

    fun obtenerAutorPorId(id: Long): Autor? {
        return autorRepository.findById(id).orElse(null)
    }

    fun eliminarAutor(id: Long) {
        autorRepository.deleteById(id)
    }
}
```

#### **Servicio para `Libro`**
```kotlin
import org.springframework.stereotype.Service

@Service
class LibroService(private val libroRepository: LibroRepository) {

    fun crearLibro(libro: Libro): Libro {
        return libroRepository.save(libro)
    }

    fun obtenerTodosLosLibros(): List<Libro> {
        return libroRepository.findAll()
    }

    fun obtenerLibroPorId(id: Long): Libro? {
        return libroRepository.findById(id).orElse(null)
    }

    fun eliminarLibro(id: Long) {
        libroRepository.deleteById(id)
    }

    fun obtenerLibrosPorGenero(genero: String): List<Libro> {
        return libroRepository.findAll().filter { it.genero.equals(genero, ignoreCase = true) }
    }
}
```

---

### **2.5. Crear Controladores**

#### **Controlador para `Autor`**
```kotlin
import org.springframework.http.ResponseEntity
import org.springframework.web.bind.annotation.*

@RestController
@RequestMapping("/autores")
class AutorController(private val autorService: AutorService) {

    @PostMapping
    fun crearAutor(@RequestBody autor: Autor): ResponseEntity<Autor> {
        return ResponseEntity.ok(autorService.crearAutor(autor))
    }

    @GetMapping
    fun obtenerTodosLosAutores(): ResponseEntity<List<Autor>> {
        return ResponseEntity.ok(autorService.obtenerTodosLosAutores())
    }

    @GetMapping("/{id}")
    fun obtenerAutorPorId(@PathVariable id: Long): ResponseEntity<Autor> {
        val autor = autorService.obtenerAutorPorId(id)
        return if (autor != null) ResponseEntity.ok(autor) else ResponseEntity.notFound().build()
    }

    @DeleteMapping("/{id}")
    fun eliminarAutor(@PathVariable id: Long): ResponseEntity<Unit> {
        autorService.eliminarAutor(id)
        return ResponseEntity.noContent().build()
    }
}
```

#### **Controlador para `Libro`**
```kotlin
import org.springframework.http.ResponseEntity
import org.springframework.web.bind.annotation.*

@RestController
@RequestMapping("/libros")
class LibroController(private val libroService: LibroService) {

    @PostMapping
    fun crearLibro(@RequestBody libro: Libro): ResponseEntity<Libro> {
        return ResponseEntity.ok(libroService.crearLibro(libro))
    }

    @GetMapping
    fun obtenerTodosLosLibros(): ResponseEntity<List<Libro>> {
        return ResponseEntity.ok(libroService.obtenerTodosLosLibros())
    }

    @GetMapping("/{id}")
    fun obtenerLibroPorId(@PathVariable id: Long): ResponseEntity<Libro> {
        val libro = libroService.obtenerLibroPorId(id)
        return if (libro != null) ResponseEntity.ok(libro) else ResponseEntity.notFound().build()
    }

    @GetMapping("/genero/{genero}")
    fun obtenerLibrosPorGenero(@PathVariable genero: String): ResponseEntity<List<Libro>> {
        return ResponseEntity.ok(libroService.obtenerLibrosPorGenero(genero))
    }

    @DeleteMapping("/{id}")
    fun eliminarLibro(@PathVariable id: Long): ResponseEntity<Unit> {
        libroService.eliminarLibro(id)
        return ResponseEntity.noContent().build()
    }
}
```

---

## **3. Actividades para los Estudiantes**

1. **Configurar el Proyecto:**
    - Crear el proyecto con Spring Boot para Kotlin y configurar las dependencias necesarias.

2. **Modelar las Entidades:**
    - Crear las clases `Autor` y `Libro` según los ejemplos proporcionados.

3. **Implementar CRUD:**
    - Crear servicios y controladores para gestionar las operaciones CRUD.

4. **Pruebas:**
    - Probar los endpoints con Postman o Swagger.

5. **Extras (Opcional):**
    - Agregar validaciones con anotaciones como `@NotNull`, `@Size`, etc.
    - Documentar la API usando Swagger.

---

## **4. Criterios de Evaluación**

1. Correcta implementación de los endpoints.
2. Uso adecuado de relaciones entre entidades.
3. Código limpio y organizado.
4. Pruebas exitosas de los endpoints.
5. Documentación y comentarios en el código.
