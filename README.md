# spring-boot-crud-operations-api-docker-testing

![RestApi](https://github.com/user-attachments/assets/4c9c8ca7-12c2-4a4c-bd8a-5715caf8ed32)


### Dockerfile

```properties
# Stage 1 - Build the Jar using maven
# pull maven and jdk-17 image
FROM maven:3.8.3-openjdk-17 AS builder

# Create a folder where the app code will be stored
WORKDIR /project

# Copy the source code from your Host machine to your container
COPY . /project

# Create Jar file
RUN mvn clean install -DskipTests=true

# Stage 2- execute JAR file from the above stage
# pull jdk-small size
FROM openjdk:17-alpine

# Create a new folder where the app code will be stored
WORKDIR /app

# Copy the app jar file and rename the jar file name
COPY --from=builder /project/target/*.jar /app/spring-app.jar

# Run the  application when the container starts
CMD ["java", "-jar", "spring-app.jar"]
```


### docker-compose 

```properties
version: "3.8"

services:

  nginx:
    container_name: nginx_cont
    build:
      context: ./nginx
    image: nginx
    ports:
      - "80:80"
    networks:
      - api-network
    restart: always
    depends_on:
      - crud-api

  mysql:
    image: mysql:latest
    container_name: mysql
    environment:
      MYSQL_ROOT_PASSWORD: test
      MYSQL_DATABASE: crudapi
    ports:
      - "3306:3306"
    volumes:
      - ./api-data:/var/lib/mysql
    networks:
      - api-network
    healthcheck:
      test: ["CMD","mysqladmin","ping","-h","localhost","-uroot","-ptest"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
    restart: always

  crud-api:
    build: .
    container_name: crud-api-v1
    env_file:
      - ".env"
    ports:
      - "8080:8080"
    networks:
      - api-network
    depends_on:
      - mysql
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:8080 || exit 1"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
    restart: always

volumes:
  api-data:

networks:
  api-network:
    name: api-network
    driver: bridge
```

### Dockerfile (Ngnix)

```properties
# pull nginx image
FROM nginx:1.23.3-alpine

COPY ./default.conf /etc/nginx/conf.d/default.conf
```

### default.config (Ngnix config)

```properties
server {

    listen 80;          # Nginx server default port
    server_name localhost;      # DNS server name (www.abc.com)

    location / {
        proxy_pass http://crud-api:8080;        # running host and post no
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
