FROM eclipse-temurin:17-jre

LABEL maintainer="teamup"

ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/${TZ} /etc/localtime && \
    echo ${TZ} > /etc/timezone

WORKDIR /app

COPY teamup-1.0.0.jar app.jar
COPY application.yml config/application.yml

EXPOSE 8080

ENTRYPOINT ["sh", "-c", "java -Djava.security.egd=file:/dev/./urandom -jar app.jar --spring.config.additional-location=file:/app/config/"]
