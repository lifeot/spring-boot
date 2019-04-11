FROM openjdk:8-jdk-alpine
RUN apk add --update \
    curl \
    && rm -rf /var/cache/apk/*
VOLUME /tmp
ARG DEPENDENCY=build/dependency
COPY ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY ${DEPENDENCY}/META-INF /app/META-INF
COPY ${DEPENDENCY}/BOOT-INF/classes /app
HEALTHCHECK --interval=10s --timeout=5s --retries=3 CMD curl --fail http://localhost:8080/health || exit 1
ENTRYPOINT ["java","-cp","app:app/lib/*","sample.web.ui.SampleWebUiApplication"]
