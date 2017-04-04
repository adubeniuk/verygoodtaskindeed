FROM maven:latest
ENV PGHOST postgres-postgresql
RUN git clone https://github.com/thoersch/spring-boot-rest-api-seed.git && cd spring-boot-rest-api-seed && sed -i -- "s/localhost/$PGHOST/g" pom.xml && sed -i -- "s/127.0.0.1/$PGHOST/g" src/main/resources/config/local.yml && mvn clean install
EXPOSE 8080
CMD  java -jar spring-boot-rest-api-seed/target/spring-boot-rest-api-seed-1.0-SNAPSHOT.jar | tee -a /var/log/java/app.log
