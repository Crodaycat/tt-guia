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

- Generamos nuestra tabla con el siguiente script SQL:

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

1. **Spring Initializer**

   - Ingresar a la siguiente [URL](https://start.spring.io/#!type=maven-project&language=java&platformVersion=3.3.5&packaging=jar&jvmVersion=17&groupId=com.flowers&artifactId=ordering-system&name=ordering-system&description=CRUD%20flowers%20app&packageName=com.flowers.ordering.system&dependencies=web,data-jpa,postgresql)
   - Descargar el proyecto usando el botón `GENERATE`
   - Abrir el proyecto en IntelliJ
   - Configurar el JDK y Maven

2. **Estructura de Paquetes**

   - Crear los siguientes paquetes:
     ```text
     config
     controller
     entity
     mapper
     model
     repository
     services
     ```

3. **Modelo y Entidad**

   - Crear la clase `Flower` en el paquete `model`:

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

   - Crear la entidad `FlowerEntity` en el paquete `entity`:

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

4. **Repositorio**

   - Crear el repositorio JPA en el paquete `repository`:

   ```java
   package com.flowers.ordering.system.repository;

   import com.flowers.ordering.system.entity.FlowerEntity;
   import org.springframework.data.jpa.repository.JpaRepository;
   import org.springframework.stereotype.Repository;

   @Repository
   public interface FlowerJpaRepository extends JpaRepository<FlowerEntity, Integer> {
   }
   ```

5. **Mapper**

   - Crear la clase `FlowerMapper` en el paquete `mapper`:

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

6. Crear la clase `FlowerRepository` en el paquete `repository`:

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

7. **Servicios**

   - Crear la clase `FlowerService` en el paquete `services`:

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

8. **Controlador**

   - Crear la clase `FlowersController` en el paquete `controller`:

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

9. **Configuración de CORS**

   - Agregar la clase `WebConfig` en el paquete `config`:

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

10. **Configuraciones del Proyecto**

    - Crear el archivo `application.yml` en la carpeta `resources` del proyecto:

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

1. **Generar la Aplicación**

   - Ejecutar el comando `ng new flowers-store`.

2. **Instalar Angular Material**

   - Ejecutar el comando `ng add @angular/material@17`.

   > **Nota:** Se utiliza Angular Material versión 17 ya que la versión 18 aún no tiene todos los estilos completamente integrados.
   > **Nota2:** El comando se debe ejecutar dentro de la carpeta del nuevo proyecto genera que se llama `flowers-store`.

3. **Agregar Bootstrap**

   - En el archivo `index.html` en la sección `<head>...</head>` agregar el link a los estilos:

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

   > **Nota:** La ubicación de este archivo es `./src/index.html`.

4. **Crear Módulo Compartido**

   - Ejecutar el comando `ng generate module shared`.

5. **Crear Páginas**

   - Ejecutar los siguientes comandos:

   ```text
   ng generate component pages/flowers-create-page
   ng generate component pages/flowers-edit-page
   ng generate component pages/flowers-list-page
   ```

6. **Crear Componentes Compartidos**

   - Ejecutar los siguientes comandos:

   ```text
   ng generate component shared/components/flower-card
   ng generate component shared/components/flowers-form
   ng generate component shared/components/nav
   ```

7. **Crear Servicio**

   - Ejecutar el comando:

   ```text
   ng generate service shared/services/flowers-api
   ```

8. **Crear Servicio**

   - Ejecutar el comando:

   ```text
   ng generate module shared
   ```

9. **Ejecutar la Aplicación**

   - Para iniciar la aplicación, ejecuta el comando `npm start` en la terminal.
   - Una vez que la aplicación esté en ejecución, abre tu navegador y navega a `http://localhost:4200` para acceder a la página web desplegada localmente.

10. **Crear Barra de Navegación y Configurar Rutas de la Aplicación**

- Eliminar todo el contenido del archivo `./src/app/app.component.html` y agregar el siguiente contenido:

  ```html
  <app-nav></app-nav>

  <section class="container py-4">
    <router-outlet></router-outlet>
  </section>
  ```

  - Modificar el archivo `./src/app/app.component.ts` con el siguiente contenido:

  ```typescript
  import { Component } from "@angular/core";
  import { RouterOutlet } from "@angular/router";
  import { SharedModule } from "./shared/shared.module";

  @Component({
    standalone: true,
    selector: "app-root",
    imports: [RouterOutlet, SharedModule],
    templateUrl: "./app.component.html",
    styleUrl: "./app.component.css",
  })
  export class AppComponent {
    title = "flowers-store";
  }
  ```

  - Modificar el archivo `./src/app/shared/shared.module.ts` con el siguiente contenido:

  ```typescript
  import { CommonModule } from "@angular/common";
  import { NgModule } from "@angular/core";
  import { MatButtonModule } from "@angular/material/button";
  import { MatToolbarModule } from "@angular/material/toolbar";
  import { RouterModule } from "@angular/router";
  import { NavComponent } from "./components/nav/nav.component";

  @NgModule({
    declarations: [NavComponent],
    imports: [CommonModule, MatToolbarModule, RouterModule, MatButtonModule],
    exports: [NavComponent],
  })
  export class SharedModule {}
  ```

  - Modificar el archivo `./src/app/shared/components/nav/nav.component.html` con el siguiente contenido:

  ```html
  <mat-toolbar color="primary" class="nav-bar">
    <span>La florecita</span>

    <a mat-button routerLink="/flowers" routerLinkActive="active-link"
      >Flores</a
    >
    <a mat-button routerLink="/flowers/create" routerLinkActive="active-link"
      >Registrar flores</a
    >
  </mat-toolbar>
  ```

  - Modificar el archivo `./src/app/shared/components/nav/nav.component.ts` con el siguiente contenido:

  ```typescript
  import { Component } from "@angular/core";

  @Component({
    selector: "app-nav",
    templateUrl: "./nav.component.html",
    styleUrl: "./nav.component.css",
  })
  export class NavComponent {}
  ```

  - Modificar el archivo `./src/app/app.routes.ts` con el siguiente contenido:

  ```typescript
  import { Routes } from "@angular/router";
  import { FlowersCreatePageComponent } from "./pages/flowers-create-page/flowers-create-page.component";
  import { FlowersEditPageComponent } from "./pages/flowers-edit-page/flowers-edit-page.component";
  import { FlowersListPageComponent } from "./pages/flowers-list-page/flowers-list-page.component";

  export const routes: Routes = [
    {
      path: "flowers",
      component: FlowersListPageComponent,
    },
    { path: "flowers/create", component: FlowersCreatePageComponent },
    {
      path: "flowers/:flowerId/edit",
      component: FlowersEditPageComponent,
    },
    { path: "**", redirectTo: "/flowers" },
  ];
  ```

11. **Crear Carpeta de Entornos**

    - En Angular 18, puedes generar la carpeta de entornos (`environments`) utilizando el siguiente comando de Angular CLI:

    ```bash
    ng generate environments
    ```

    Este comando creará automáticamente los archivos `environment.ts` y `environment.prod.ts` dentro de la carpeta `src/environments`.

    - El archivo `environment.development.ts` tendrá el siguiente contenido:

    ```typescript
    export const environment = {
      production: false,
      apiUrl: "http://localhost:18100",
    };
    ```

    - El archivo `environment.ts` tendrá el siguiente contenido:

    ```typescript
    export const environment = {
      production: true,
      apiUrl: "http://localhost:18100",
    };
    ```

    > Nota: A la hora de querer publicar tu aplicacion a internet debes modificar el `environment.prod.ts` para utilizar tu apiUrl con la ruta en internet. Ejemplo: `https://mi-dominio.com`

    > **Importante:** En este punto te recomiendo detener la ejecucion de la aplicacion usando `ctrl + c` en la terminal donde ejecutaste `npm start` y que vuelvas a ejecutar el comando para volver a iniciar la app.

12. **Crear el archivo `flower.ts`**

    1. **Crear el archivo `flower.ts`**

    - Navega a la carpeta `src/app/shared/model` y crea un archivo llamado `flower.ts`.

    2. **Definir la interfaz `flower`**

    - Abre el archivo `flower.ts` y define la interfaz `Flower` con el siguiente contenido:

    ```typescript
    export interface Flower {
      id?: number;
      name: string;
      color: string;
      imageUrl: string;
      price: number;
    }
    ```

    - **Explicación del código:**
      - `id?`: Propiedad opcional que representa el identificador único de la flor.
      - `name`: Propiedad que representa el nombre de la flor.
      - `color`: Propiedad que representa el color de la flor.
      - `imageUrl`: Propiedad que representa la URL de la imagen de la flor.
      - `price`: Propiedad que representa el precio de la flor.

13. **Crear Servicio para la Comunicación con el Backend**

    - Modificar el archivo `src/app/shared/services/flowers-api.service.ts` con el siguiente contenido:

    ```typescript
    import { HttpClient } from "@angular/common/http";
    import { Injectable } from "@angular/core";
    import { Observable } from "rxjs";
    import { environment } from "../../../environments/environment";
    import { Flower } from "../model/flower";

    @Injectable({
      providedIn: "root",
    })
    export class FlowersApiService {
      constructor(private client: HttpClient) {}

      findAll(): Observable<Flower[]> {
        return this.client.get<Flower[]>(`${environment.apiUrl}/flowers`);
      }

      findById(id: number): Observable<Flower> {
        return this.client.get<Flower>(`${environment.apiUrl}/flowers/${id}`);
      }

      create(flower: Flower): Observable<Flower> {
        return this.client.post<Flower>(
          `${environment.apiUrl}/flowers`,
          flower
        );
      }

      update(flower: Flower): Observable<Flower> {
        return this.client.put<Flower>(
          `${environment.apiUrl}/flowers/${flower.id}`,
          flower
        );
      }

      delete(id: number) {
        return this.client.delete<Flower>(
          `${environment.apiUrl}/flowers/${id}`
        );
      }
    }
    ```

    - Modificar el archivo `./src/app/shared/shared.module.ts` con el siguiente contenido:

    ```typescript
    import { CommonModule } from "@angular/common";
    import { NgModule } from "@angular/core";
    import { MatButtonModule } from "@angular/material/button";
    import { MatToolbarModule } from "@angular/material/toolbar";
    import { RouterModule } from "@angular/router";
    import { NavComponent } from "./components/nav/nav.component";

    import {
      provideHttpClient,
      withInterceptorsFromDi,
    } from "@angular/common/http";
    import { FlowersApiService } from "./services/flowers-api.service";

    @NgModule({
      declarations: [NavComponent],
      imports: [CommonModule, MatToolbarModule, RouterModule, MatButtonModule],
      exports: [NavComponent],
      providers: [
        provideHttpClient(withInterceptorsFromDi()),
        FlowersApiService,
      ],
    })
    export class SharedModule {}
    ```

14. **Crear el Formulario de Registro de Flores**

    - Modificar el archivo `./src/app/shared/shared.module.ts` con el siguiente contenido:

    ```typescript
    import { CommonModule } from "@angular/common";
    import { NgModule } from "@angular/core";
    import { ReactiveFormsModule } from "@angular/forms";
    import { MatButtonModule } from "@angular/material/button";
    import { MAT_FORM_FIELD_DEFAULT_OPTIONS } from "@angular/material/form-field";
    import { MatInputModule } from "@angular/material/input";
    import { MatToolbarModule } from "@angular/material/toolbar";
    import { RouterModule } from "@angular/router";
    import { FlowersFormComponent } from "./components/flowers-form/flowers-form.component";
    import { NavComponent } from "./components/nav/nav.component";

    import {
      provideHttpClient,
      withInterceptorsFromDi,
    } from "@angular/common/http";
    import { FlowersApiService } from "./services/flowers-api.service";

    @NgModule({
      declarations: [NavComponent, FlowersFormComponent],
      imports: [
        CommonModule,
        MatToolbarModule,
        RouterModule,
        MatButtonModule,
        ReactiveFormsModule,
        MatInputModule,
      ],
      exports: [NavComponent, FlowersFormComponent],
      providers: [
        {
          provide: MAT_FORM_FIELD_DEFAULT_OPTIONS,
          useValue: { appearance: "outline" },
        },
        provideHttpClient(withInterceptorsFromDi()),
        FlowersApiService,
      ],
    })
    export class SharedModule {}
    ```

    - Modificar el archivo `./src/app/shared/components/flowers-form/flowers-form.component.ts` con el siguiente contenido:

    ```typescript
    import { Component, EventEmitter, Input, Output } from "@angular/core";
    import { FormBuilder, FormGroup, Validators } from "@angular/forms";
    import { Flower } from "../../model/flower";

    @Component({
      selector: "app-flowers-form",
      templateUrl: "./flowers-form.component.html",
      styleUrl: "./flowers-form.component.css",
    })
    export class FlowersFormComponent {
      flowersForm!: FormGroup;

      @Input() set defaultValue(value: Flower) {
        this.intilizeForm(value);
      }

      @Input() isEdit = false;

      @Output()
      onSubmit: EventEmitter<Flower> = new EventEmitter<Flower>();

      constructor(private formBuilder: FormBuilder) {
        this.intilizeForm();
      }

      intilizeForm(defaultValue?: Flower) {
        const { name, color, imageUrl, price } = defaultValue || {
          name: "",
          color: "",
          imageUrl: "",
          price: 0,
        };

        this.flowersForm = this.formBuilder.group({
          name: [name, Validators.required],
          color: [color, Validators.required],
          imageUrl: [imageUrl, Validators.required],
          price: [price, [Validators.required, Validators.min(1)]],
        });
      }

      submit() {
        if (this.flowersForm.valid) {
          const { name, color, imageUrl, price } = this.flowersForm.value;

          this.onSubmit.emit({ name, color, imageUrl, price });
        }
      }
    }
    ```

    - Modificar el archivo `./src/app/shared/components/flowers-form/flowers-form.component.html` con el siguiente contenido:

    ```html
    <form [formGroup]="flowersForm" (ngSubmit)="submit()">
      <div class="row mb-3">
        <mat-form-field class="col-12 col-md-6 col-lg-4">
          <mat-label>Nombre</mat-label>
          <input
            type="text"
            matInput
            formControlName="name"
            placeholder="Ingresa el nombre de la flor"
          />

          @if (flowersForm.get('name')?.hasError('required')) {
          <mat-error>El nombre es <strong>requerido</strong></mat-error>
          }
        </mat-form-field>

        <mat-form-field class="col-12 col-md-6 col-lg-4">
          <mat-label>Color</mat-label>
          <input
            type="text"
            matInput
            formControlName="color"
            placeholder="Ingresa el color de la flor"
          />

          @if (flowersForm.get('color')?.hasError('required')) {
          <mat-error>El color es <strong>requerido</strong></mat-error>
          }
        </mat-form-field>
      </div>

      <div class="row mb-3">
        <mat-form-field class="col-12 col-md-6 col-lg-4">
          <mat-label>Imagen</mat-label>
          <input
            type="text"
            matInput
            formControlName="imageUrl"
            placeholder="Ingresa la URL de la imagen de la flor"
          />

          @if (flowersForm.get('imageUrl')?.hasError('required')) {
          <mat-error>La imagen es <strong>requerida</strong></mat-error>
          }
        </mat-form-field>

        <mat-form-field class="col-12 col-md-6 col-lg-4">
          <mat-label>Precio</mat-label>
          <input
            type="number"
            matInput
            formControlName="price"
            placeholder="Ingresa el precio de la flor"
          />

          @if (flowersForm.get('price')?.hasError('required')) {
          <mat-error>El precio es <strong>requerido</strong></mat-error>
          } @if (flowersForm.get('price')?.hasError('min')) {
          <mat-error
            >El precio debe ser <strong>mayor</strong> a cero
          </mat-error>
          }
        </mat-form-field>
      </div>

      <button
        type="submit"
        mat-flat-button
        color="primary"
        [disabled]="flowersForm.invalid"
      >
        {{ isEdit ? "Actualizar" : "Registrar" }}
      </button>
    </form>
    ```

15. **Página de Creación de Flores**

    - Vamos a crear la página para registrar flores. Modificamos el archivo `./src/app/pages/flowers-create-page/flowers-create-page.component.ts` con el siguiente contenido:

    ```typescript
    import { Component } from "@angular/core";
    import {
      MatSnackBar,
      MatSnackBarModule,
    } from "@angular/material/snack-bar";
    import { finalize } from "rxjs";
    import { Flower } from "../../shared/model/flower";
    import { FlowersApiService } from "../../shared/services/flowers-api.service";
    import { SharedModule } from "../../shared/shared.module";

    @Component({
      selector: "app-flowers-create-page",
      standalone: true,
      imports: [SharedModule, MatSnackBarModule],
      templateUrl: "./flowers-create-page.component.html",
      styleUrl: "./flowers-create-page.component.css",
    })
    export class FlowersCreatePageComponent {
      isLoading = false;

      constructor(
        private flowersApi: FlowersApiService,
        private snackBar: MatSnackBar
      ) {}

      onSubmit(flower: Flower) {
        this.isLoading = true;

        this.flowersApi
          .create(flower)
          .pipe(finalize(() => (this.isLoading = false)))
          .subscribe({
            next: () => {
              this.snackBar.open("Flor creada con éxito", "Cerrar", {
                panelClass: "snack-success",
              });
            },
            error: () => {
              this.snackBar.open(
                "Error, la flor no pudo ser registrada.",
                "Cerrar",
                {
                  panelClass: "snack-warning",
                }
              );
            },
          });
      }
    }
    ```

    - **Explicación del código:**

      - `isLoading`: Una propiedad booleana que indica si la operación de creación está en curso.
      - `constructor`: Inyecta los servicios `FlowersApiService` y `MatSnackBar` para manejar las solicitudes HTTP y mostrar notificaciones, respectivamente.
      - `onSubmit(flower: Flower)`: Este método se llama cuando se envía el formulario. Establece `isLoading` en `true` para indicar que la operación está en curso. Luego, llama al método `create` del servicio `FlowersApiService` para enviar una solicitud HTTP que crea una nueva flor. Utiliza el operador `finalize` para establecer `isLoading` en `false` una vez que la solicitud se completa, independientemente de si tuvo éxito o falló. Si la solicitud es exitosa, muestra una notificación de éxito utilizando `MatSnackBar`. Si la solicitud falla, muestra una notificación de error.

    - Ahora, modificamos el archivo HTML `./src/app/pages/flowers-create-page/flowers-create-page.component.html` con el siguiente contenido:

    ```html
    <h1>Registrar flor</h1>

    <app-flowers-form (onSubmit)="onSubmit($event)"></app-flowers-form>
    ```

    - **Explicación del HTML:**

      - `<h1>Registrar flor</h1>`: Título de la página.
      - `<app-flowers-form (onSubmit)="onSubmit($event)"></app-flowers-form>`: Utiliza el componente `FlowersFormComponent` y maneja el evento `onSubmit` para crear una nueva flor.

16. **Crear Componente Flower Card**

- Modificar el archivo `./src/app/shared/shared.module.ts` para declarar y exportar el componente `FlowerCardComponent`:

  ```typescript
  import { CommonModule } from "@angular/common";
  import { NgModule } from "@angular/core";
  import { ReactiveFormsModule } from "@angular/forms";
  import { MatButtonModule } from "@angular/material/button";
  import { MAT_FORM_FIELD_DEFAULT_OPTIONS } from "@angular/material/form-field";
  import { MatInputModule } from "@angular/material/input";
  import { MatToolbarModule } from "@angular/material/toolbar";
  import { RouterModule } from "@angular/router";
  import { FlowersFormComponent } from "./components/flowers-form/flowers-form.component";
  import { NavComponent } from "./components/nav/nav.component";

  import {
    provideHttpClient,
    withInterceptorsFromDi,
  } from "@angular/common/http";
  import { MatCardModule } from "@angular/material/card";
  import { FlowerCardComponent } from "./components/flower-card/flower-card.component";
  import { FlowersApiService } from "./services/flowers-api.service";

  @NgModule({
    declarations: [NavComponent, FlowersFormComponent, FlowerCardComponent],
    imports: [
      CommonModule,
      MatToolbarModule,
      RouterModule,
      MatButtonModule,
      ReactiveFormsModule,
      MatInputModule,
      MatCardModule,
    ],
    exports: [NavComponent, FlowersFormComponent, FlowerCardComponent],
    providers: [
      {
        provide: MAT_FORM_FIELD_DEFAULT_OPTIONS,
        useValue: { appearance: "outline" },
      },
      provideHttpClient(withInterceptorsFromDi()),
      FlowersApiService,
    ],
  })
  export class SharedModule {}
  ```

- **Explicación del código:**

  - `FlowerCardComponent` se declara y se exporta para que pueda ser utilizado en otros módulos.
  - Se importa `MatCardModule` para utilizar el componente de tarjeta de Angular Material.

- Modificar el archivo `./src/app/shared/components/flower-card/flower-card.component.ts` con el siguiente contenido:

  ```typescript
  import { Component, Input } from "@angular/core";
  import { Flower } from "../../model/flower";

  @Component({
    selector: "app-flower-card",
    templateUrl: "./flower-card.component.html",
    styleUrl: "./flower-card.component.css",
  })
  export class FlowerCardComponent {
    @Input() flower!: Flower;
  }
  ```

- **Explicación del código:**

  - `@Input() flower!: Flower;`: Define una propiedad de entrada `flower` que recibe un objeto `Flower`.

- Modificar el archivo `./src/app/shared/components/flower-card/flower-card.component.html` con el siguiente contenido:

```html
<mat-card class="example-card">
  <mat-card-header>
    <mat-card-title>{{ flower?.name }}</mat-card-title>
  </mat-card-header>

  <img
    class="flower-card__image"
    mat-card-image
    [src]="flower?.imageUrl"
    alt="Foto de la flor"
  />

  <mat-card-content class="row mt-3">
    <span class="d-block col-12">Color: {{ flower?.color }}</span>
    <span class="d-block col-12">Precio: {{ flower?.price }}</span>
  </mat-card-content>
  <mat-card-actions class="d-flex gap-2">
    <button
      type="button"
      mat-flat-button
      color="primary"
      (click)="editFlower()"
    >
      Editar
    </button>
    <button type="button" mat-flat-button color="warn" (click)="deleteFlower()">
      Eliminar
    </button>
  </mat-card-actions>
</mat-card>
```

- **Explicación del HTML:**

  - `<mat-card class="example-card">`: Define una tarjeta de Angular Material.
  - `<mat-card-header>`: Encabezado de la tarjeta que muestra el nombre de la flor.
  - `<img class="flower-card__image" mat-card-image [src]="flower?.imageUrl" alt="Foto de la flor" />`: Imagen de la flor.
  - `<mat-card-content class="row mt-3">`: Contenido de la tarjeta que muestra el color y el precio de la flor.
  - `<mat-card-actions class="d-flex gap-2">`: Acciones de la tarjeta con botones para editar y eliminar la flor.

- Modificar el archivo `./src/app/shared/components/flower-card/flower-card.component.css` con el siguiente contenido:

```css
.flower-card__image {
  object-fit: contain;
  height: 200px;
}
```

- **Explicación del CSS:**
  - `.flower-card__image`: Este estilo asegura que la imagen de la flor se ajuste correctamente dentro de su contenedor, manteniendo la proporción original y evitando que se distorsione.
  - La propiedad `object-fit: contain` hace que la imagen se escale para caber dentro del contenedor sin recortarse, mientras que `height: 200px` establece una altura fija para la imagen.

16. **Página de Listado de Flores**

- Modificar el archivo `./src/app/pages/flowers-list-page/flowers-list-page.component.ts` con el siguiente contenido:

  ```typescript
  import { CommonModule } from "@angular/common";
  import { Component } from "@angular/core";
  import { MatSnackBar } from "@angular/material/snack-bar";
  import { Router, RouterModule } from "@angular/router";
  import { finalize } from "rxjs";
  import { Flower } from "../../shared/model/flower";
  import { FlowersApiService } from "../../shared/services/flowers-api.service";
  import { SharedModule } from "../../shared/shared.module";

  @Component({
    selector: "app-flowers-list-page",
    standalone: true,
    imports: [SharedModule, CommonModule, RouterModule],
    templateUrl: "./flowers-list-page.component.html",
    styleUrl: "./flowers-list-page.component.css",
  })
  export class FlowersListPageComponent {
    isLoading = true;
    flowers: Flower[] = [];

    constructor(
      private flowersApi: FlowersApiService,
      private snackBar: MatSnackBar,
      private router: Router
    ) {
      this.loadFlowers();
    }

    loadFlowers() {
      this.isLoading = true;

      this.flowersApi
        .findAll()
        .pipe(
          finalize(() => {
            this.isLoading = false;
          })
        )
        .subscribe({
          next: (flowers) => {
            this.flowers = flowers;
          },
          error: () => {
            this.flowers = [];
            this.snackBar.open("Error al consultar las flores", "Cerrar", {
              panelClass: "snack-warning",
            });
          },
        });
    }

    editFlower(flower: Flower) {
      this.router.navigate(["flowers", flower.id, "edit"]);
    }

    deleteFlower(flower: Flower) {
      this.isLoading = true;

      this.flowersApi
        .delete(flower.id!)
        .pipe(finalize(() => (this.isLoading = false)))
        .subscribe({
          next: () => {
            this.loadFlowers();

            this.snackBar.open(
              `Flor: ${flower.name} con id: ${flower.id} eliminada con éxito`,
              "Cerrar",
              {
                panelClass: "snack-success",
              }
            );
          },
          error: () => {
            this.snackBar.open("Error al eliminar la flor", "Cerrar", {
              panelClass: "snack-warning",
            });
          },
        });
    }
  }
  ```

- Modificar el archivo `./src/app/pages/flowers-list-page/flowers-list-page.component.html` con el siguiente contenido:

  ```html
  <h1>Listado de flores</h1>

  <section class="flowers-list">
    <div class="row">
      <div class="col-12 col-md-4 col-lg-3" *ngFor="let flower of flowers">
        <app-flower-card
          [flower]="flower"
          (onEdit)="editFlower($event)"
          (onDelete)="deleteFlower($event)"
        ></app-flower-card>
      </div>
    </div>
  </section>
  ```

- Modificar el archivo `./src/app/pages/flowers-list-page/flowers-list-page.component.css` con el siguiente contenido:

  ```css
  .flowers-list {
    display: flex;
    flex-wrap: wrap;
    gap: 16px;
  }
  ```

- Explicación del CSS:

  - Este estilo utiliza `display: flex` para crear un contenedor flexible que permite organizar los elementos secundarios en una fila o columna.
  - La propiedad `flex-wrap: wrap` asegura que los elementos secundarios se ajusten automáticamente a la siguiente línea cuando no hay suficiente espacio en una sola fila.
  - La propiedad `gap: 16px` agrega un espacio uniforme de 16 píxeles entre los elementos secundarios, proporcionando una apariencia ordenada y consistente.

    Esto es útil para mostrar tarjetas de flores en un diseño de cuadrícula adaptable.

17. **Página de Edición de Flores**

- Modificar el archivo `./src/app/pages/flowers-edit-page/flowers-edit-page.component.ts` con el siguiente contenido:

  ```typescript
  import { CommonModule } from "@angular/common";
  import { Component } from "@angular/core";
  import { MatSnackBar } from "@angular/material/snack-bar";
  import { ActivatedRoute } from "@angular/router";
  import { finalize } from "rxjs";
  import { Flower } from "../../shared/model/flower";
  import { FlowersApiService } from "../../shared/services/flowers-api.service";
  import { SharedModule } from "../../shared/shared.module";

  @Component({
    selector: "app-flowers-edit-page",
    standalone: true,
    imports: [SharedModule, CommonModule],
    templateUrl: "./flowers-edit-page.component.html",
    styleUrl: "./flowers-edit-page.component.css",
  })
  export class FlowersEditPageComponent {
    isLoading = true;
    flower?: Flower;
    isEditing = false;

    constructor(
      private flowersApi: FlowersApiService,
      private route: ActivatedRoute,
      private snackBar: MatSnackBar
    ) {
      this.findFlower();
    }

    findFlower() {
      this.isLoading = true;
      const flowerId: number = Number(
        this.route.snapshot.paramMap.get("flowerId")
      );

      this.flowersApi
        .findById(flowerId)
        .pipe(finalize(() => (this.isLoading = false)))
        .subscribe({
          next: (flower) => {
            this.flower = flower;
          },
          error: () => {
            this.flower = undefined;
            this.snackBar.open(
              "Error al cargar información de la flor",
              "Cerrar",
              {
                panelClass: "snack-warning",
              }
            );
          },
        });
    }

    onSubmit(flower: Flower) {
      this.isEditing = true;
      flower.id = this.flower?.id;

      this.flowersApi
        .update(flower)
        .pipe(finalize(() => (this.isEditing = false)))
        .subscribe({
          next: () => {
            this.snackBar.open("Flor actualizada con éxito", "Cerrar", {
              panelClass: "snack-success",
            });
          },
          error: () => {
            this.snackBar.open(
              "Error, la flor no pudo ser actualizada.",
              "Cerrar",
              {
                panelClass: "snack-warning",
              }
            );
          },
        });
    }
  }
  ```

- **Explicación del código:**

  - `isLoading`: Una propiedad booleana que indica si la operación de carga está en curso.
  - `flower`: Una propiedad opcional que contiene la información de la flor a editar.
  - `isEditing`: Una propiedad booleana que indica si la operación de edición está en curso.
  - `constructor`: Inyecta los servicios `FlowersApiService`, `ActivatedRoute` y `MatSnackBar` para manejar las solicitudes HTTP, obtener parámetros de ruta y mostrar notificaciones, respectivamente.
  - `findFlower()`: Este método se llama en el constructor para cargar la información de la flor a editar. Establece `isLoading` en `true` para indicar que la operación está en curso. Luego, llama al método `findById` del servicio `FlowersApiService` para enviar una solicitud HTTP que obtiene la información de la flor. Utiliza el operador `finalize` para establecer `isLoading` en `false` una vez que la solicitud se completa, independientemente de si tuvo éxito o falló. Si la solicitud es exitosa, asigna la información de la flor a la propiedad `flower`. Si la solicitud falla, muestra una notificación de error utilizando `MatSnackBar`.
  - `onSubmit(flower: Flower)`: Este método se llama cuando se envía el formulario. Establece `isEditing` en `true` para indicar que la operación está en curso. Luego, llama al método `update` del servicio `FlowersApiService` para enviar una solicitud HTTP que actualiza la información de la flor. Utiliza el operador `finalize` para establecer `isEditing` en `false` una vez que la solicitud se completa, independientemente de si tuvo éxito o falló. Si la solicitud es exitosa, muestra una notificación de éxito utilizando `MatSnackBar`. Si la solicitud falla, muestra una notificación de error.

- Modificar el archivo `./src/app/pages/flowers-edit-page/flowers-edit-page.component.html` con el siguiente contenido:

  ```html
  <h1>Editar flor</h1>

  <ng-container *ngIf="!isLoading; else loadingContainer">
    <app-flowers-form
      *ngIf="!!flower"
      [isEdit]="true"
      [defaultValue]="flower"
      (onSubmit)="onSubmit($event)"
    ></app-flowers-form>
  </ng-container>

  <ng-template #loadingContainer> Cargando... </ng-template>
  ```

- **Explicación del HTML:**

  - `<h1>Editar flor</h1>`: Título de la página.
  - `<ng-container *ngIf="!isLoading; else loadingContainer">`: Contenedor que muestra el formulario de edición si `isLoading` es `false`.
  - `<app-flowers-form *ngIf="!!flower" [isEdit]="true" [defaultValue]="flower" (onSubmit)="onSubmit($event)"></app-flowers-form>`: Utiliza el componente `FlowersFormComponent` y maneja el evento `onSubmit` para actualizar la información de la flor.
  - `<ng-template #loadingContainer> Cargando... </ng-template>`: Plantilla que muestra un mensaje de carga si `isLoading` es `true`.
