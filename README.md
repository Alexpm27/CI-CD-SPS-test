# practica-CI-CD-SPS-V2

### Descripción
Este proyecto es una API Java que implementa un pipeline de CI/CD utilizando GitHub Actions. El objetivo es automatizar la construcción, prueba y despliegue de la aplicación, asegurando calidad y eficiencia en el proceso de desarrollo. La aplicación se construye con Maven y los artefactos generados se almacenan en GitHub Packages. Además, el pipeline está diseñado para simular un entorno de despliegue, permitiendo una fácil transición a plataformas basadas en contenedores como Kubernetes o ECS en el futuro.

### Flujo del pipeline de CI/CD
Se ejecuta el pipeline al crear o actualizar un `pull request` en la rama `main` o si el `pull request` se acepta a la rama `main`
```pipeline.yml
on:
  push:
    branches:
      - main
    pull_request:
      branches:
        - main

jobs:
  build:
    runs-on: ubuntu-latest
```
###### Pasos:
- Checkout del repositorio: Se clona el código fuente.
```pipeline.yml
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4
```
- Configuración del JDK: Se configura Java 17.
```pipeline.yml
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: maven
```
- `Caché` de dependencias Maven: Esto optimiza el tiempo de ejecución al almacenar dependencias de Maven, evitando que se descarguen en cada ejecución.
```pipeline.yml
      - name: Cache Maven dependencies
        uses: actions/cache@v3
        with:
          path: ~/.m2/repository
          key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
          restore-keys: |
            ${{ runner.os }}-maven-
```
- Construcción y verificación: Se ejecuta mvn verify para compilar y probar la aplicación.
```pipeline.yml
      - name: Build and Verify with Maven
        run: mvn verify
```
- Validación de `merge`: Si las pruebas son exitosas, se permite hacer merge del pull request.
```pipeline.yml
      - name: Merge PR if successful
        if: success() && github.event_name == 'pull_request'
        run: echo "Successful PR build, ready to merge..."
```
[![Screenshot-2024-06-29-at-2-58-28-a-m.png](https://i.postimg.cc/zGW9Pd2Q/Screenshot-2024-06-29-at-2-58-28-a-m.png)](https://postimg.cc/LJ9CJVpk)

o de lo contrario no debería permitirlo.

[![Screenshot-2024-06-29-at-2-57-27-a-m.png](https://i.postimg.cc/RV3rwC74/Screenshot-2024-06-29-at-2-57-27-a-m.png)](https://postimg.cc/tYHmjjvM)

###### Si el pull request se acepta a la rama `main` (commit).
- Carga de artefactos: Se suben los artefactos generados a GitHub Packages.
```pipeline.yml
      - name: Upload Artifact
        if: success() && github.event_name == 'push'
        uses: actions/upload-artifact@v3
        with:
          name: api-artifact
          path: target/*.jar

      - name: Publish to GitHub Packages
        if: success() && github.event_name == 'push'
        env:
          GITHUB_TOKEN: ${{ secrets.GH_TOKEN }}
        run: mvn deploy -Dregistry=https://maven.pkg.github.com/Alexpm27 -Dtoken=$GITHUB_TOKEN
```
[![Screenshot-2024-06-29-at-4-16-33-a-m.png](https://i.postimg.cc/j2f72HQ5/Screenshot-2024-06-29-at-4-16-33-a-m.png)](https://postimg.cc/64931GPJ)
- Despliegue simulado: Es posible desplegar automáticamente el artefacto guardado en GitHub Packages
```pipeline.yml
      - name: Download Artifact
        if: success() && github.event_name == 'push'
        uses: actions/download-artifact@v2
        with:
          name: api-artifact
          path: deploy

      - name: Deployment Artifact
        if: success() && github.event_name == 'push'
        run: |
          echo "Despliegue en curso..."
          # Comandos para desplegar en el entorno deseado (ejemplo: Kubernetes, ECS, etc.)
```

## Para adaptar el pipeline y la estructura del proyecto a una plataforma basada en contenedores como Kubernetes, ECS, o App Runner, se podrían considerar las siguientes modificaciones:
- Dockerfile:
    - Agregar un Dockerfile en la raíz del proyecto para definir cómo construir la imagen del contenedor de la aplicación.

- GitHub Actions Workflow:
    - Build and Push Docker Image: Añadir pasos al workflow para construir la imagen Docker y publicarla en un registro de contenedores como GitHub Container Registry o Amazon ECR.
    - Despliegue en Plataforma de Contenedores: Para Kubernetes, ECS u otras plataformas de contenedores, definir pasos específicos para actualizar los despliegues o servicios después de haber subido la imagen.
    - Pruebas Post-Despliegue: Hacer ejecución de pruebas automáticas después de desplegar la imagen para validar que todo funcione correctamente en el entorno.

- Monitoreo y Logs:
    - Integrar herramientas de monitoreo y recopilación de logs para el despliegue en la plataforma de contenedores, asegurando visibilidad y diagnóstico continuo del rendimiento de la aplicación.


## Pasos y/o herramientas para entender las plantillas de Cloudformation y evaluar que ajustes se tendrían que realizar a la etapa de despliegue

###### Revisión de la Documentación de AWS:
Se inicia consultando la documentación oficial de AWS sobre ECS y Fargate para comprender los requisitos y las mejores prácticas para el despliegue de contenedores en estas plataformas.

###### Análisis de las Plantillas de CloudFormation:
Posteriormente, se procede a descargar las plantillas de CloudFormation proporcionadas por AWS desde el repositorio. Luego, se examinan los recursos definidos en estas plantillas, como clústeres ECS, definiciones de tarea, servicios y roles IAM.

###### Identificación de Parámetros y Variables:
Dentro de las plantillas de CloudFormation, se identifican los parámetros y variables utilizados, que pueden incluir configuraciones específicas del entorno como nombres de clústeres, configuraciones de red y políticas de IAM.

### Herramientas para evaluar y ajustar las plantillas de CloudFormation:
###### AWS CloudFormation Designer:
Herramienta gráfica intuitiva, para visualizar y editar las plantillas de CloudFormation de manera eficiente. Con esta herramienta, se pueden arrastrar y soltar componentes, ver relaciones entre recursos y validar la estructura general de las plantillas antes de la implementación.

###### AWS CLI y AWS SDKs:
Estas herramientas permiten interactuar con las plantillas de CloudFormation a través de comandos y scripts, facilitando la implementación, actualización y validación de stacks de forma repetible y controlada.

###### Pruebas en Entornos de Desarrollo:
Se implementan las plantillas de CloudFormation en entornos de desarrollo. Observar cómo se comportan los despliegues en un entorno controlado permite realizar ajustes según sea necesario.

### Consideraciones adicionales:
###### Seguridad y Accesos IAM:
Es fundamental asegurarse de que los roles IAM utilizados en las plantillas de CloudFormation tengan los permisos adecuados para interactuar con los servicios de AWS utilizados en el entorno ECS con Fargate.

###### Monitoreo y Logging:
Se integran herramientas de monitoreo y registro, como Amazon CloudWatch, para supervisar el rendimiento y el comportamiento de los contenedores desplegados en ECS con Fargate. Configurar alertas y métricas relevantes proporciona visibilidad continua del entorno y permite una respuesta rápida a posibles problemas operativos.
