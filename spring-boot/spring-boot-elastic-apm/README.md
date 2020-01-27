## Run ELK stack (ES, Kibana, APM)
```bash
docker-compose up
```

## Build 
```bash
mvn clean package
```

## Run from terminal 
```bash
java -Delastic.apm.server_urls=http://localhost:8200 -Delastic.apm.service_name=spring-boot-elastic-apm -Delastic.apm.disable_instrumentations="" -Delastic.apm.application_packages=com.afrunt -Delastic.apm.environment=local  -javaagent:elastic-apm-agent.jar -jar ./target/spring-boot-elastic-apm-1.0.jar
```

## Kibana
Go to [Kibana UI](http://localhost:5601)

## Run from Intellij IDEA
Don't forget to add VM options
![IDEA Run Configuration](./static/idea-run-configuratio.png)

### Call http endpoints
From browser
* [http://localhost:8080/hello](http://localhost:8080/hello)
* [http://localhost:8080/hello-apm](http://localhost:8080/hello-apm)

From terminal
```bash
curl http://localhost:8080/hello
```

```bash
curl http://localhost:8080/hello-apm
```