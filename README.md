[![release-build](https://github.com/KeepCodingCloudDevops6/liberando-productos-carlosfeu/actions/workflows/release.yaml/badge.svg)](https://github.com/KeepCodingCloudDevops6/liberando-productos-carlosfeu/actions/workflows/release.yaml)
[![Test](https://github.com/KeepCodingCloudDevops6/liberando-productos-carlosfeu/actions/workflows/test.yaml/badge.svg)](https://github.com/KeepCodingCloudDevops6/liberando-productos-carlosfeu/actions/workflows/test.yaml)
# keepcoding-devops-liberando-productos-practica-final

## Objetivo

El objetivo es mejorar un proyecto creado previamente para ponerlo en producción, a través de la adicción de una serie de mejoras.

## Proyecto inicial

El proyecto inicial es un servidor que realiza lo siguiente:

- Utiliza [FastAPI](https://fastapi.tiangolo.com/) para levantar un servidor en el puerto `8081` e implementa inicialmente dos endpoints:
  - `/`: Devuelve en formato `JSON` como respuesta `{"health": "ok"}` y un status code 200.
  - `/health`: Devuelve en formato `JSON` como respuesta `{"message":"Hello World"}` y un status code 200.

- Se han implementado tests unitarios para el servidor [FastAPI](https://fastapi.tiangolo.com/)

- Utiliza [prometheus-client](https://github.com/prometheus/client_python) para arrancar un servidor de métricas en el puerto `8000` y poder registrar métricas, siendo inicialmente las siguientes:
  - `Counter('server_requests_total', 'Total number of requests to this webserver')`: Contador que se incrementará cada vez que se haga una llamada a alguno de los endpoints implementados por el servidor (inicialmente `/` y `/health`)
  - `Counter('healthcheck_requests_total', 'Total number of requests to healthcheck')`: Contador que se incrementará cada vez que se haga una llamada al endpoint `/health`.
  - `Counter('main_requests_total', 'Total number of requests to main endpoint')`: Contador que se incrementará cada vez que se haga una llamada al endpoint `/`.

## Software necesario

Es necesario disponer del siguiente software:

- `Python` en versión `3.8.5` o superior, disponible para los diferentes sistemas operativos en la [página oficial de descargas](https://www.python.org/downloads/release/python-385/)

- `virtualenv` para poder instalar las librerías necesarias de Python, se puede instalar a través del siguiente comando:

    ```sh
    pip3 install virtualenv
    ```

    En caso de estar utilizando Linux y el comando anterior diera fallos se debe ejecutar el siguiente comando:

    ```sh
    sudo apt-get update && sudo apt-get install -y python3.8-venv
    ```

- `Docker` para poder arrancar el servidor implementado a través de un contenedor Docker, es posible descargarlo a [través de su página oficial](https://docs.docker.com/get-docker/).

## Ejecución de servidor

### Ejecución directa con Python

1. Instalación de un virtualenv, **realizarlo sólo en caso de no haberlo realizado previamente**:
   1. Obtener la versión actual de Python instalada para crear posteriormente un virtualenv:

        ```sh
        python3 --version
        ```

        El comando anterior mostrará algo como lo mostrado a continuación:ç

        ```sh
            Python 3.8.13
        ```

   2. Crear de virtualenv en la raíz del directorio para poder instalar las librerías necesarias:

       - En caso de en el comando anterior haber obtenido `Python 3.8.*`

            ```sh
            python3.8 -m venv venv
            ```

       - En caso de en el comando anterior haber obtenido `Python 3.9.*`:

           ```sh
           python3.9 -m venv venv
           ```

2. Activar el virtualenv creado en el directorio `venv` en el paso anterior:

     ```sh
     source venv/bin/activate
     ```

3. Instalar las librerías necesarias de Python, recogidas en el fichero `requirements.txt`, **sólo en caso de no haber realizado este paso previamente**. Es posible instalarlas a través del siguiente comando:

    ```sh
    pip3 install -r requirements.txt
    ```

4. Ejecución del código para arrancar el servidor:

    ```sh
    python3 src/app.py
    ```

5. La ejecución del comando anterior debería mostrar algo como lo siguiente:

    ```sh
    [2022-04-16 09:44:22 +0000] [1] [INFO] Running on http://0.0.0.0:8081 (CTRL + C to quit)
    ```

### Ejecución a través de un contenedor Docker

1. Crear una imagen Docker con el código necesario para arrancar el servidor:

    ```sh
    docker build -t simple-server:0.0.1 .
    ```

2. Arrancar la imagen construida en el paso anterior mapeando los puertos utilizados por el servidor de FastAPI y el cliente de prometheus:

    ```sh
    docker run -d -p 8000:8000 -p 8081:8081 --name simple-server simple-server:0.0.1
    ```

3. Obtener los logs del contenedor creado en el paso anterior:

    ```sh
    docker logs -f simple-server
    ```

4. La ejecución del comando anterior debería mostrar algo como lo siguiente:

    ```sh
    [2022-04-16 09:44:22 +0000] [1] [INFO] Running on http://0.0.0.0:8081 (CTRL + C to quit)
    ```

## Comprobación de endpoints de servidor y métricas

Una vez arrancado el servidor, utilizando cualquier de las formas expuestas en los apartados anteriores, es posible probar las funcionalidades implementadas por el servidor:

- Comprobación de servidor FastAPI, a través de llamadas a los diferentes endpoints:

  - Realizar una petición al endpoint `/`

      ```sh
      curl -X 'GET' \
      'http://0.0.0.0:8081/' \
      -H 'accept: application/json'
      ```

      Debería devolver la siguiente respuesta:

      ```json
      {"message":"Hello World"}
      ```

  - Realizar una petición al endpoint `/health`

      ```sh
      curl -X 'GET' \
      'http://0.0.0.0:8081/health' \
      -H 'accept: application/json' -v
      ```

      Debería devolver la siguiente respuesta.

      ```json
      {"health": "ok"}
      ```

- Comprobación de registro de métricas, si se accede a la URL `http://0.0.0.0:8000` se podrán ver todas las métricas con los valores actuales en ese momento:

  - Realizar varias llamadas al endpoint `/` y ver como el contador utilizado para registrar las llamadas a ese endpoint, `main_requests_total` ha aumentado, se debería ver algo como lo mostrado a continuación:

    ```sh
    # TYPE main_requests_total counter
    main_requests_total 4.0
    ```

  - Realizar varias llamadas al endpoint `/health` y ver como el contador utilizado para registrar las llamadas a ese endpoint, `healthcheck_requests_total` ha aumentado, se debería ver algo como lo mostrado a continuación:

    ```sh
    # TYPE healthcheck_requests_total counter
    healthcheck_requests_total 26.0
    ```

  - También se ha credo un contador para el número total de llamadas al servidor `server_requests_total`, por lo que este valor debería ser la suma de los dos anteriores, tal y como se puede ver a continuación:

    ```sh
    # TYPE server_requests_total counter
    server_requests_total 30.0
    ```

## Tests

Se ha implementado tests unitarios para probar el servidor FastAPI, estos están disponibles en el archivo `src/tests/app_test.py`.

Es posible ejecutar los tests de diferentes formas:

- Ejecución de todos los tests:

    ```sh
    pytest
    ```

- Ejecución de todos los tests y mostrar cobertura:

    ```sh
    pytest --cov
    ```

- Ejecución de todos los tests y generación de report de cobertura:

    ```sh
    pytest --cov --cov-report=html
    ```

## Practica a realizar

A partir del ejemplo inicial descrito en los apartados anteriores es necesario realizar una serie de mejoras:

Los requirimientos son los siguientes:

- Añadir por lo menos un nuevo endpoint a los existentes `/` y `/health`, un ejemplo sería `/bye` que devolvería `{"msg": "Bye Bye"}`, para ello será necesario añadirlo en el fichero [src/application/app.py](./src/application/app.py)

- Creación de tests unitarios para el nuevo endpoint añadido, para ello será necesario modificar el [fichero de tests](./src/tests/app_test.py)

- Opcionalmente creación de helm chart para desplegar la aplicación en Kubernetes, se dispone de un ejemplo de ello en el laboratorio realizado en la clase 3

- Creación de pipelines de CI/CD en cualquier plataforma (Github Actions, Jenkins, etc) que cuenten por lo menos con las siguientes fases:

  - Testing: tests unitarios con cobertura. Se dispone de un [ejemplo con Github Actions en el repositorio actual](./.github/workflows/test.yaml)

  - Build & Push: creación de imagen docker y push de la misma a cualquier registry válido que utilice alguna estrategia de release para los tags de las vistas en clase, se recomienda GHCR ya incluido en los repositorios de Github. Se dispone de un [ejemplo con Github Actions en el repositorio actual](./.github/workflows/release.yaml)

- Configuración de monitorización y alertas:

  - Configurar monitorización mediante prometheus en los nuevos endpoints añadidos, por lo menos con la siguiente configuración:
    - Contador cada vez que se pasa por el/los nuevo/s endpoint/s, tal y como se ha realizado para los endpoints implementados inicialmente

  - Desplegar prometheus a través de Kubernetes mediante minikube y configurar alert-manager para por lo menos las siguientes alarmas, tal y como se ha realizado en el laboratorio del día 3 mediante el chart `kube-prometheus-stack`:
    - Uso de CPU de un contenedor mayor al del límite configurado, se puede utilizar como base el ejemplo utilizado en el laboratorio 3 para mandar alarmas cuando el contenedor de la aplicación `fast-api` consumía más del asignado mediante request

  - Las alarmas configuradas deberán tener severity high o critical

  - Crear canal en slack `<nombreAlumno>-prometheus-alarms` y configurar webhook entrante para envío de alertas con alert manager

  - Alert manager estará configurado para lo siguiente:
    - Mandar un mensaje a Slack en el canal configurado en el paso anterior con las alertas con label "severity" y "critical"
    - Deberán enviarse tanto alarmas como recuperación de las mismas
    - Habrá una plantilla configurada para el envío de alarmas

    Para poder comprobar si esta parte funciona se recomienda realizar una prueba de estres, como la realizada en el laboratorio 3 a partir del paso 8.

  - Creación de un dashboard de Grafana, con por lo menos lo siguiente:
    - Número de llamadas a los endpoints
    - Número de veces que la aplicación ha arrancado

## Entregables

Se deberá entregar mediante un repositorio realizado a partir del original lo siguiente:

- Código de la aplicación y los tests modificados
- Ficheros para CI/CD configurados y ejemplos de ejecución válidos
- Ficheros para despliegue y configuración de prometheus de todo lo relacionado con este, así como el dashboard creado exportado a `JSON` para poder reproducirlo
- `README.md` donde se explique como se ha abordado cada uno de los puntos requeridos en el apartado anterior, con ejemplos prácticos y guía para poder reproducir cada uno de ellos


# Realización de la Práctica

## Clonado del repositorio

- Para clonar el repositorio hacemos el siguiente comando:

```bash
git clone https://github.com/KeepCodingCloudDevops6/liberando-productos-carlosfeu.git
```

- El primer paso a realizar es el de añadir al código el nuevo endpoint que queremos configurar. Para ello en `src/application/app.py` añadimos el siguiente código:

```bash
@app.get("/bye")
    async def read_bye():
        """Implement bye endpoint"""
        # Increment counter used for register the total number of calls in the webserver
        REQUESTS.inc()
        # Increment counter used for register the total number of calls in the main endpoint
        BYE_ENDPOINT_REQUESTS.inc()
        return {"msg": "Colorín colorado, este cuento se ha acabado"}
```
- A su vez también es necesario configurar el Counter de dicho endpoint, que es donde vamos a ver el numero de peticiones que se le hacen. Se ve tal que así:

```bash
BYE_ENDPOINT_REQUESTS = Counter('bye_requests_total', 'Total number of requests to bye endpoint')
```
- Una vez configurada el archivo `app.py` vamos a modificar el de test unitarios del endpoint `/bye` que se encuentra en `src/test/app_test.py` para ello añadimos el siguiente código que es el que va a ejecutar `pytest` a la hora de hacer los test:

```bash
@pytest.mark.asyncio
    async def read_bye_test(self):
        """Tests the bye endpoint"""
        response = client.get("bye")

        assert response.status_code == 200
        assert response.json() == {"msg": "Colorín colorado, este cuento se ha acabado"}
```
## Ejecución de los tests

- Para la realización de los test lo primero que debemos hacer es levantar un entorno virtual con `venv`. Para ello necesitamos saber nuestra versión de python. Estando en la ruta `liberando-productos-carlosfeu` lanzamos el siguiente comando

```bash
python3 --version
```
- El comando anterior mostrará algo como lo mostrado a continuación:

```bash
Python 3.10.6
```
- Ahora creamos el entorno virtual en la raíz del directorio para poder instalar las librerías necesarias:

  - En caso de en el comando anterior haber obtenido `python 3.10.*`

  ```bash
  python3.10 -m venv venv
  ```
  - Procederemos a activar el virtualenv creado en el directorio venv en el paso anterior:

  ```bash
  source venv/bin/activate
  ```
- Instalar las librerías necesarias de Python, recogidas en el fichero requirements.txt, sólo en caso de no haber realizado este paso previamente. Es posible instalarlas a través del siguiente comando:

```bash
pip3 install -r requirements.txt
```
- Una vez instaladas las librerias vamos a acceder al directorio `liberando-productos-carlosfeu/src/tests` para poder ejecutar los tests. Es posible ejecutar los test de diferentes maneras:

```bash
pytest
```
- Que nos dará como resultado algo así:

![pytest](https://user-images.githubusercontent.com/102614480/204089088-10157134-841e-4dad-8c9b-9a6d6cbaea12.png)

```bash
pytest --cov
```
- Nos dara algo como esto otro:

![pytest-cov](https://user-images.githubusercontent.com/102614480/204089104-e93ff25b-0a4f-4d51-a215-e461afe58122.png)

```bash
pytest --cov --cov-report=html
```
- veremos esto otro

![pytest-cov-html](https://user-images.githubusercontent.com/102614480/204089119-9aa4dec6-5ee3-47d8-939c-5bfed3983a67.png)

## Pipelines de CI/CD con Github Actions

- Como podemos observar en el código de los pipelines tenemos 2 workflows distintos.
  
  - Uno es el que corre los test unitarios de la aplicación. Se llama `Unit-test`
  - El otro es el crea la imagen y la sube a los registry configurados en este caso DockerHub y GHCR. Al mismo tiempo es el que tiene configurado el semantic-release para la creación de los tags. Se llama `release-build`

- Adjunto capturas de su correcto funcionamiento, se puede revisar el código en el repositorio.

![release-build](https://user-images.githubusercontent.com/102614480/204089448-ea46a6cd-adfb-4ba3-846e-306686872b31.png)

![unit-test](https://user-images.githubusercontent.com/102614480/204089461-a080d9b0-6cc2-4c75-98c9-9ec284b9a6dc.png)

- Adjunto captura de la subida a Dockerhub de la Imagen y la creación de la release y del `Package` en GitHub. 

![docker-hub](https://user-images.githubusercontent.com/102614480/204089469-0e6ee484-ec05-4ee7-af94-75798548e691.png)

![release-package](https://user-images.githubusercontent.com/102614480/204089487-460626cb-0d4b-4524-9667-cee0aa57b5ef.png)

![package-github](https://user-images.githubusercontent.com/102614480/204089524-78eb3856-c3e8-4774-8f8d-8a8230997df2.png)

- Hemos tenido que configurar el archivo `package.json` con nuestras configuraciones para hacer funcionar el semanctic-release.

![package-json](https://user-images.githubusercontent.com/102614480/204089538-b024dad7-6332-4792-8e96-c76765fac4b2.png)

- Esta captura es el workflow de `release-build` poniendole el tag a la release

![tag-release](https://user-images.githubusercontent.com/102614480/204089601-47f7c33c-36b3-4f12-89c0-32a8e88daf65.png)

## Configuración del Alert Manager en Slack

- Vamos a configurar un aviso a través de un alert manager en Slack.

- Para ello hemos debido configurar el webhook en slack de la siguiente manera:

  - En nuestro Slack debemos crear un `Channel` donde queremos que nos llegen las alertas. En este caso lo hemos llamado `practica-sre`. 
  - una vez creado, en `Browse Slack` --> `Apps` tenemos que buscar `Incoming Webhooks` y lo añadiremos.
  - Accedemos a este app y le daremos a `Configuration` y después a `Add to Slack`
  - En este paso tenemos que especificar el canal que queremos recibir las notificaciones para que la url del webhooks lo asocie. Una vez elegido pinchamos en `Add Incoming Webhooks integration` y copiaremos el `Webhook URL`.
  - El archivo donde encontraremos esta configuración del alert manager se llama `custom_values_prometheus.yaml` que se encuentra en la ruta `liberando-productos-carlosfeu/helm/kube-prometheus-stack` 

```bash
  ## Configuration for alertmanager
alertmanager:
  config:
    global:
      resolve_timeout: 5m
    route:
      group_by: ['job']
      group_wait: 30s
      group_interval: 5m
      repeat_interval: 12h
      receiver: 'slack'
      routes:
      - match:
          alertname: Watchdog
        receiver: 'null'
    # This inhibt rule is a hack from: https://stackoverflow.com/questions/54806336/how-to-silence-prometheus-alertmanager-using-config-files/54814033#54814033
    inhibit_rules:
      - target_match_re:
           alertname: '.+Overcommit'
        source_match:
           alertname: 'Watchdog'
        equal: ['prometheus']
    receivers:
    - name: 'null'
    - name: 'slack'
      slack_configs:
      - api_url: '' # <--- AÑADIR EN ESTA LÍNEA EL WEBHOOK CREADO
        send_resolved: true
        channel: '' # <--- AÑADIR EN ESTA LÍNEA EL CANAL
        title: '[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}] Monitoring Event Notification'
        text: |-
          {{ range .Alerts }}
            *Alert:* {{ .Labels.alertname }} - `{{ .Labels.severity }}`
            *Description:* {{ .Annotations.message }}
            *Graph:* <{{ .GeneratorURL }}|:chart_with_upwards_trend:> *Runbook:* <{{ .Annotations.runbook_url }}|:spiral_note_pad:>
            *Details:*
            {{ range .Labels.SortedPairs }} • *{{ .Name }}:* `{{ .Value }}`
            {{ end }}
          {{ end }}
```

## Despliegue de Prometheus y Grafana.

- Vamos a aprovechar el despliegue del fast-api-web, para realizar también el de Prometheus y Grafana.

1. Iniciamos un cluster con minikube para desplegar la aplicación. El comando es el siguiente:

    ```bash
    minikube start --kubernetes-version='v1.21.1' \
        --memory=4096 \
        -p monitoring-demo
    ```
- Accedemos a la siguiente ruta para añadir repos y ejecutar configuraciones de Prometheus `liberando-productos-carlosfeu/helm/kube-prometheus-stack`

2. Añadir el repositorio de helm `prometheus-community` para poder desplegar el chart `kube-prometheus-stack`:

    ```bash
    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm repo update
    ```

3. Desplegar el chart `kube-prometheus-stack` del repositorio de helm añadido en el paso anterior con los valores configurados en el archivo `custom_values_prometheus.yaml` en el namespace `monitoring`:

    ```bash
    helm -n monitoring upgrade --install prometheus prometheus-community/kube-prometheus-stack -f custom_values_prometheus.yaml --create-namespace --wait --version 34.1.1
    ```

4. Realizar split de la terminal o crear una nueva pestaña y ver como se están creando pod en el namespace `monitoring` utilizado para desplegar el stack de prometheus:

    ```bash
    kubectl -n monitoring get po -w
    ```

5. Añadir el repositorio de helm de bitnami para poder desplegar el chart de mongodb empaquetado por esta compañía:

    ```bash
    helm repo add bitnami https://charts.bitnami.com/bitnami
    helm repo update
    ```

6. Instalar el addond `metrics-server` en el perfil utilizado actualmente de minikube:

    ```bash
    minikube addons enable metrics-server -p monitoring-demo
    ```
## Despliegue de Fast-api-web en K8s con helm chart.

- Accedemos a la siguiente ruta para ejecutar el helm de fast-api-webapp `liberando-productos-carlosfeu/helm`

1. Se ha creado un helm chart en la carpeta `fast-api-webapp`, en la cual se han realizado modificaciones respecto a las versiones anteriores para disponer de métricas mediante prometheus. Para desplegarlo es necesario realizar los siguientes pasos:

    1. Descargar las dependencias necesarias, siendo en este caso el chart de mongodb

        ```sh
        helm dep up fast-api-webapp
        ```

    2. Desplegar el helm chart:

        ```sh
        helm -n fast-api upgrade my-app --wait --install --create-namespace fast-api-webapp
        ```

    3. Hacer split de la terminal o crear una nueva pestaña en la misma y observar como se crean los pods en el namespace `fast-api` donde se ha desplegado el web server:

        ```sh
        kubectl -n fast-api get po -w
        ```

    4. Hacer de nuevo split de la terminal o crear una nueva pestaña y obtener los logs del container `wait-mongo` del deployment `my-app-fast-api-webapp` en el namespace `fast-api`, observar como está utilizando ese contenedor para esperar a que MongoDB esté listo:

        ```sh
        kubectl -n fast-api logs -f deployment/my-app-fast-api-webapp -c wait-mongo
        ```

        Una vez se obtenga el mensaje de conexión exitosa a mongo, siendo algo como lo mostrado a continuación, indicará que empezará el contenedor `dfad`:

        ```sh
        Connecting to my-app-mongodb:27017 (10.101.242.27:27017)
        HTTP/1.0 200 OK
        Connection: close
        Content-Type: text/plain
        Content-Length: 85
        ```

    5. Obtener los logs del contenedor `fast-api-webapp` del deployment `my-app-fast-api-webapp` en el namespace `fast-api`, observar como está arrancando el servidor fast-api:

        ```sh
        kubectl -n fast-api logs -f deployment/my-app-fast-api-webapp -c fast-api-webapp
        ```

        Debería obtenerse un resultado similar al siguiente:

        ```sh
        [2022-11-09 11:28:12 +0000] [1] [INFO] Running on http://0.0.0.0:8081 (CTRL + C to quit)
        ```

2. Abrir una nueva pestaña en la terminal y realizar un port-forward del puerto `http-web` del servicio de Grafana al puerto 3000 de la máquina:

    ```sh
    kubectl -n monitoring port-forward svc/prometheus-grafana 3000:http-web
    ```

3. Abrir otra pestaña en la terminal y realizar un port-forward del servicio de Prometheus al puerto 9090 de la máquina:

    ```sh
    kubectl -n monitoring port-forward svc/prometheus-kube-prometheus-prometheus 9090:9090
    ```

4. Abrir una nueva pestaña en la terminal y realizar un port-forward al `Service` creado para nuestro servidor:

    ```sh
    kubectl -n fast-api port-forward svc/my-app-fast-api-webapp 8081:8081
    ```
5. Una vez tenemos esto desplegado podemos hacer el siguiente comando por consola, que es un Curl al nuevo endpoint creado:

    ```bash
    curl -X 'GET' \
      'http://localhost:8081/bye' \
      -H 'accept: application/json'
    ```
  - Nos va a devolver este mensaje: `{"msg":"Colorín colorado, este cuento se ha acabado"}`

  ![curl-bye](https://user-images.githubusercontent.com/102614480/204089682-285b05a9-3107-43d7-950c-52793c286ae2.png)

6. Si accedemos a `http://localhost:8081/docs` veremos que a través de Swagger UI podemos ver el servidor de FastAPI con los endpoints creados en el código y la posibilidad de hacer peticiones a cada uno de ellos.

![swagger-fastapi](https://user-images.githubusercontent.com/102614480/204089625-98ef8369-623f-429c-ad3e-837e8a2b2aef.png) 

- En caso que queramos acceder directamente a un endpoint deberemos escribir en la barra del navegador `http://localhost:8081` para ver el mensaje raiz, `http://localhost:8081/health` para ver el estado del healtcheck o por último, `http://localhost:8081/bye` para ber el mensaje configurado en ese endpoint.

![msg-bye](https://user-images.githubusercontent.com/102614480/204089674-af3ca205-d5a6-4db1-8cd0-89629bac20d2.png)

7. Para acceder a Prometheus una vez levantado deberemos ir a `http://localhost:9090` y al no tener que autenticarse accederemos directamente. Ya que por defecto no hace falta.

8. Para acceder a Grafana deberemos ir a `http://localhost:3000` y para logearse deberemos escribir `admin` en el usuario y `prom-operator` en la contraseña.

  - Una vez dentro de la interfaz gŕafica de grafana en el panel izquierdo encontraremos un icono donde pone Dashboards y dentro `browse` deberemos meternos ahí, y un pop-up arriba a la derecha encontraremos el botón de `import`. Añadiremos el `custom_dashboard.json` que se encuentra en el repo en la ruta `liberando-productos-carlosfeu/helm/kube-prometheus-stack`. Cuando lo importemos, accederemos a el y veremos esto que se muestra a continuación:

![dashboard-grafana](https://user-images.githubusercontent.com/102614480/204089636-d942d9e6-0e24-490a-9ceb-41635ab72932.png)

- Lo que vemos es el dashboard de grafana donde solo hay un pod levantado y el consumo de CPU está estable.

## Prueba de Estrés

9. Obtener el pod creado en el paso 1 para poder lanzar posteriormente un comando de prueba de extres, así como seleccionarlo en el menú desplegable del panel de grafana:

   ```sh
   export POD_NAME=$(kubectl get pods --namespace fast-api -l "app.kubernetes.io/name=fast-api-webapp,app.kubernetes.io/instance=my-app" -o jsonpath="{.items[0].metadata.name}")
   echo $POD_NAME
   ```

   Se debería obtener un resultado similar al siguiente:

   ```sh
   my-app-fast-api-webapp-585bf945cc-lvvpv
   ```

10. Utilizar el resultado obtenido en el paso anterior para seleccionar en el dashboard creado de grafana para seleccionar el pod del que obtener información, seleccionando este a través del menú desplegable de nombre `pod`.

11. Acceder mediante una shell interactiva al contenedor `fast-api-webapp` del pod obtenido en el paso anterior:

    ```sh
    kubectl -n fast-api exec --stdin --tty $POD_NAME -c fast-api-webapp -- /bin/sh
    ```

12. Dentro de la shell en la que se ha accedido en el paso anterior instalar y utilizar los siguientes comandos para descargar un proyecto de github que realizará pruebas de extress:

    1. Instalar los binarios necesarios en el pod

        ```sh
        apk update && apk add git go
        ```

    2. Descargar el repositorio de github y acceder a la carpeta de este, donde se realizará la compilación del proyecot:

        ```sh
        git clone https://github.com/jaeg/NodeWrecker.git
        cd NodeWrecker
        go build -o extress main.go
        ```

    3. Ejecución del binario obtenido de la compilación del paso anterior que realizará una prueba de extress dentro del pod:

        ```sh
        ./extress -abuse-memory -escalate -max-duration 10000000
        ```

13. Abrir una nueva pestaña en la terminal y ver como evoluciona el HPA creado para la aplicación web:

    ```sh
    kubectl -n fast-api get hpa -w
    ```

  - Veremos como el autoescaler hace su función levantando replicas y la prueba de estrés también hace su función ya que el consumo de CPU esta disparado. Al igual que veremos las consultas a los endpoints.

  ![grafana-setres](https://user-images.githubusercontent.com/102614480/204091770-86f5919f-2296-41b3-bf24-797dcfdf5ff9.png)

14. Se debería recibir una notificación como la siguiente en el canal de Slack configurado para el envío de notificaciones sobre alarmas. Cuando el Horizontal Pod Autoscaler escalará el número de pods para mitigar este pico y por lo tanto pasado un tiempo debería recibirse una notificación de que la alarma se ha mitigado, tal y como se puede ver en la siguiente captura.

![slack](https://user-images.githubusercontent.com/102614480/204089655-8ffba0e0-44e3-49f3-9c57-a11dccfe6971.png)


16. Parar el cluster de minikube creado para esta parte del laboratorio:

    ```sh
    minikube -p monitoring-demo stop
    ```


