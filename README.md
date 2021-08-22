# Docker images

## Image preparation

* [Oracle Database 11g Release 2 (11.2.0.2) Express Edition (XE)](https://github.com/oracle/docker-images/tree/main/OracleDatabase/SingleInstance/dockerfiles).
  _Prerequisite: `docker` and Oracle binaries._

  You will have to provide the installation binaries of Oracle Database.
  The needed file is
  named `oracle-xe-11.2.0-1.0.x86_64.rpm.zip` and should be put in the folder `11.2.0.2`. Then run:
  ```
  $ ./buildContainerImage.sh -v 11.2.0.2 -x
  ```
  The script builds the image `oracle/database:11.2.0.2-xe`.

* [Sitmun Backend Core](https://github.com/sitmun/sitmun-backend-core).
  _Prerequisite: `Java 11`._

  You must run:
  ```
  $ git clone https://github.com/sitmun/sitmun-backend-core.git
  $ cd sitmun-backend-core
  $ ./gradlew build -x test
  ```
  Then copy the JAR file at `build/libs` to `src/main/docker/backend/sitmun-backend-core.jar`.

* **Administration application**
  _Prerequisite: `npm`._

  You must run:
  ```
  $ git clone https://github.com/sitmun/sitmun-admin-app.git
  $ cd sitmun-admin-app
  ```

  Then update `src/environment/environment.testdeployment.ts` to:
  ```
  export const environment = {
    production: false,
    apiBaseURL: '/sitmun',
  };
  ```

  Then edit `.npmrc` and comment the line
  ```
  #registry=https://npm.pkg.github.com/sitmun
  ```
  Then run:
  ```
  $ npm ci
  $ npm run build -- --configuration=testdeployment
  ```
  Next, copy the contents of `dist/admin-app` to Backend Core `admin/html`


## Postgres-backed run

In a terminal you can run the development profile (with test data, admin:admin as user):

```
$ docker compose -f docker-compose-dev-postgres-9.3.yml up -d
```
The bootstrap is quick. Next, you can open your browser and navigate to localhost:8000 to the **Sitmun Admin App**.

In a terminal you can run the production profile (without test data, admin:admin as user):

```
$ docker compose -f docker-compose-dev-postgres-9.3.yml up -d
```

The bootstrap is quick. Next, you can open your browser and navigate to localhost:9000 to the **Sitmun Admin App**.

## Oracle-backed run

Oracle image has a slow start, so the run must be done in three stages.:

```
$ docker-compose -f docker-compose-dev-oracle-11.yml up -d oracle
```

Use a console provided by a docker tool to monitor this container. When the log console outputs:

```
#########################
DATABASE IS READY TO USE!
#########################
```

we can proceed with the next step. Run:

```
$ docker-compose -f docker-compose-dev-oracle.yml up -d backend
```

Monitor this container and wait until a message similar to:

```
Started Application in nnn seconds (JVM running for mmm)
```

Finally, we can run the front:

```
$ docker-compose -f docker-compose-dev-oracle.yml up -d admin
```

Admin is fast, so you can open your browser and navigate to localhost:8000 to the **Sitmun Admin App**.
