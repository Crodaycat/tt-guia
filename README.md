# Guia MasterClass 2: App web

## Recursos

[Back-end](https://github.com/Crodaycat/tt-backend)

[Front-end](https://github.com/Crodaycat/tt-frontend)

## Requisitos

[PostgreSQL](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads)

[PgAdmin](https://www.pgadmin.org/download/)

[Intellij Community](https://www.jetbrains.com/idea/download/)

[NodeJs](https://nodejs.org/en/)

Angular CLI: Una vez instalado node ejecutar en una consola `npm install -g @angular/cli`

[VS Code](https://code.visualstudio.com/)

## Base de datos

* Generamos nuestra tabla con el siguiente script SQL:

```sql


CREATE SEQUENCE "flower".flowers_id_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;

CREATE TABLE IF NOT EXISTS flower.flowers
(
    id bigint NOT NULL DEFAULT nextval('flower.flowers_id_seq'::regclass),
    name character varying(48) COLLATE pg_catalog."default" NOT NULL,
    color character varying(24) COLLATE pg_catalog."default" NOT NULL,
    image_url character varying(4096) COLLATE pg_catalog."default",
    price numeric NOT NULL,
    CONSTRAINT flowers_pkey PRIMARY KEY (id)
);
```

## Back-end

* Spring Initilizer

  Ingresar a la siguiente [URL](https://start.spring.io/#!type=maven-project&language=java&platformVersion=3.3.5&packaging=jar&jvmVersion=17&groupId=com.flowers&artifactId=ordering-system&name=ordering-system&description=CRUD%20flowers%20app&packageName=com.flowers.ordering.system&dependencies=web,data-jpa,postgresql)

* Descargar el proyecto usando el botón `GENERATE`

* Abrir projecto en IntelliJ

* Configurar el JDK y Maven

* Vamos a crear los siguientes paquetes:

  ```text
  config
  controller
  entity
  mapper
  model
  repository
  services
  ```

* Creemos la clase `Flower` en el paquete `model`:

```java

package com.flowers.ordering.system.model;

import java.math.BigDecimal;

public class Flower {
    private Integer id;
    private String name;
    private String color;
    private String imageUrl;
    private BigDecimal price;

    public Flower(Integer id, String name, String color, String imageUrl, BigDecimal price) {
        this.id = id;
        this.name = name;
        this.color = color;
        this.imageUrl = imageUrl;
        this.price = price;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }

    public String getImageUrl() {
        return imageUrl;
    }

    public void setImageUrl(String imageUrl) {
        this.imageUrl = imageUrl;
    }

    public BigDecimal getPrice() {
        return price;
    }

    public void setPrice(BigDecimal price) {
        this.price = price;
    }
}
```

* Creemos la entidad `FlowersEntity` en el paquete `entity`:

```java
package com.flowers.ordering.system.entity;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Table;

import java.math.BigDecimal;

@Table(name = "flowers")
@Entity
public class FlowerEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    private String name;
    private String color;
    private String imageUrl;
    private BigDecimal price;

    public FlowerEntity() {}

    public FlowerEntity(Integer id, String name, String color, String imageUrl, BigDecimal price) {
        this.id = id;
        this.name = name;
        this.color = color;
        this.imageUrl = imageUrl;
        this.price = price;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }

    public String getImageUrl() {
        return imageUrl;
    }

    public void setImageUrl(String imageUrl) {
        this.imageUrl = imageUrl;
    }

    public BigDecimal getPrice() {
        return price;
    }

    public void setPrice(BigDecimal price) {
        this.price = price;
    }
}
```

* Creemos el repositorio JPA en el paquete `repository`:

```java
package com.flowers.ordering.system.repository;

import com.flowers.ordering.system.entity.FlowerEntity;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface FlowerJpaRepository extends JpaRepository<FlowerEntity, Integer> {
}

```

* Creemos la clase `FlowerMapper` en paquete `mappers`:

```java
package com.flowers.ordering.system.mapper;

import com.flowers.ordering.system.entity.FlowerEntity;
import com.flowers.ordering.system.model.Flower;
import org.springframework.stereotype.Component;

@Component
public class FlowerMapper {

    public FlowerEntity flowerToFlowerEntity(Flower flower) {
        return new FlowerEntity(
                flower.getId(),
                flower.getName(),
                flower.getColor(),
                flower.getImageUrl(),
                flower.getPrice()
        );
    }

    public Flower flowerEntityToFlower(FlowerEntity flowerEntity) {
        return new Flower(
                flowerEntity.getId(),
                flowerEntity.getName(),
                flowerEntity.getColor(),
                flowerEntity.getImageUrl(),
                flowerEntity.getPrice()
        );
    }

}
```

* Creemos la clase `FlowersRepository` en el paquete `repository`:

```java
package com.flowers.ordering.system.repository;

import com.flowers.ordering.system.entity.FlowerEntity;
import com.flowers.ordering.system.mapper.FlowerMapper;
import com.flowers.ordering.system.model.Flower;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.Optional;

@Component
public class FlowerRepository {

    private final FlowerJpaRepository flowerJpaRepository;
    private final FlowerMapper flowerMapper;

    public FlowerRepository(FlowerJpaRepository flowerJpaRepository, FlowerMapper flowerMapper) {
        this.flowerJpaRepository = flowerJpaRepository;
        this.flowerMapper = flowerMapper;
    }

    public List<Flower> findAll() {
        return flowerJpaRepository.findAll().stream()
                .map(this.flowerMapper::flowerEntityToFlower)
                .toList();
    }

    // Create and update
    public Flower save(Flower flower) {
        final FlowerEntity createdFlower = flowerJpaRepository.save(this.flowerMapper.flowerToFlowerEntity(flower));

        return this.flowerMapper.flowerEntityToFlower(createdFlower);
    }

    public Optional<Flower> findById(Integer id) {
        return this.flowerJpaRepository.findById(id)
                .map(flowerEntity -> this.flowerMapper.flowerEntityToFlower(flowerEntity));
    }

    public void deleteById(Integer id) {
        this.flowerJpaRepository.deleteById(id);
    }
}
```

* Creemos la clase `FlowerService` en el paquete `services`:

```java
package com.flowers.ordering.system.services;

import com.flowers.ordering.system.model.Flower;
import com.flowers.ordering.system.repository.FlowerRepository;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
public class FlowerService {

    private final FlowerRepository flowerRepository;

    public FlowerService(FlowerRepository flowerRepository) {
        this.flowerRepository = flowerRepository;
    }


    public List<Flower> findAll() {
        return flowerRepository.findAll();
    }

    public Optional<Flower> findById(Integer id) {
        return this.flowerRepository.findById(id);
    }

    // Create and update
    public Flower save(Flower flower) {
        return flowerRepository.save(flower);
    }

    public Flower deleteById(Integer id) {
        final var flower = this.flowerRepository.findById(id);

        if (flower.isPresent()) {
            this.flowerRepository.deleteById(id);
        }

        return flower.orElse(null);
    }

}

```

* creemos la clase `FlowersController` en el paquete `controller`:

```java
package com.flowers.ordering.system.controller;

import com.flowers.ordering.system.model.Flower;
import com.flowers.ordering.system.services.FlowerService;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;
import java.util.Optional;

@RestController
@RequestMapping(value = "/flowers")
public class FlowersController {

    private final FlowerService flowerService;

    public FlowersController(FlowerService flowerService) {
        this.flowerService = flowerService;
    }

    @GetMapping
    public List<Flower> findAll() {
        return this.flowerService.findAll();
    }

    @GetMapping("{flowerId}")
    public Optional<Flower> findById(@PathVariable Integer flowerId) {
        return this.flowerService.findById(flowerId);
    }

    @PostMapping
    public Flower create(@RequestBody Flower flower) {
        return this.flowerService.save(flower);
    }

    @PutMapping("{flowerId}")
    public Flower update(@PathVariable Integer flowerId, @RequestBody Flower flower) {
        flower.setId(flowerId);

        return this.flowerService.save(flower);
    }

    @DeleteMapping("{flowerId}")
    public Flower delete(@PathVariable Integer flowerId) {
        return this.flowerService.deleteById(flowerId);
    }
}
```

* Agregamos la clase `WebConfig` en el paquete `config` para las configuraciones de CORS:

```java
package com.flowers.ordering.system.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("http://localhost:4200")
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                .allowedHeaders("*")
                .allowCredentials(true);
    }
}
```

* Agregemos las configuraciones al proyecto, creamos el archivo `application.yml` en la carpeta `resources` del proyecto:

```yml
server:
  port: 18100

spring:
  application:
    name: flowers-ordering-system
  jpa:
    open-in-view: false
    show-sql: true
    database-platform: org.hibernate.dialect.PostgreSQLDialect
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
  datasource:
    url: jdbc:postgresql://localhost:5432/postgres?currentSchema=flower&binaryTransfer=true&reWriteBatchedInserts=true&stringtype=unspecified
    username: postgres
    password: root
    driver-class-name: org.postgresql.Driver
```

## Front-end

* Generamos la aplicación con el comando `ng new flowers-store`.
* Instalamos angular material con el comando `ng add angular-material@17`.
* Agregamos bootstrap como librería de estilos. En el archivo `index.html` en la sección `<head>...</head>` agregamos el link a los estilos.

```html
<head>
  ...
  <link
      href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
      rel="stylesheet"
      integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH"
      crossorigin="anonymous"
    />
    <script
      src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"
      integrity="sha384-YvpcrYf0tY3lHB60NNkmXc5s9fDVZLESaAA55NDzOxhy9GkcIdslK1eN7N6jIeHz"
      crossorigin="anonymous"
    ></script>
  ...
</head>
```

* Creamos un módulo compartido usando el comando `ng generate module shared`

* Añadirmos las siguientes páginas:

```text

ng generate component pages/flowers-create-page
ng generate component pages/flowers-edit-page
ng generate component pages/flowers-list-page

```

* Generamos los siguientes components:

```text

ng generate component shared/components/flower-card
ng generate component shared/components/flowers-form
ng generate component shared/components/nav

```

* Generamos el siguiente servicio:

* Generamos los siguientes components:

```text

ng generate service shared/services/flowers-api

```

* Ejecutamos la aplicación conel comando `npm start`.
