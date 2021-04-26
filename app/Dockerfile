FROM amazonlinux:2.0.20210326.0 as native-builder
RUN yum install -y gcc gcc-c++ libc6-dev  zlib1g-dev curl bash zlib zlib-devel zip unzip gzip wget tar

ENV GRAALVM_VERSION=21.1.0
ENV JDK_VERSION=java11
ENV GRAALVM_FILENAME=graalvm-ce-${JDK_VERSION}-linux-amd64-${GRAALVM_VERSION}.tar.gz
#download graalvm 21.0.0.2 and setup
RUN curl -4 -L https://github.com/graalvm/graalvm-ce-builds/releases/download/vm-${GRAALVM_VERSION}/${GRAALVM_FILENAME} -o /tmp/${GRAALVM_FILENAME}
RUN tar -zxvf /tmp/${GRAALVM_FILENAME} -C /tmp \
    && mv /tmp/graalvm-ce-${JDK_VERSION}-${GRAALVM_VERSION} /opt/graalvm
RUN rm -rf /tmp/*

RUN /opt/graalvm/bin/gu install native-image

#download maven and setup
RUN curl -4 -L https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.8.1/binaries/apache-maven-3.8.1-bin.tar.gz -o /tmp/apache-maven-3.8.1-bin.tar.gz
RUN tar -zxvf /tmp/apache-maven-3.8.1-bin.tar.gz -C /tmp \
    && mv /tmp/apache-maven-3.8.1 /opt/maven
RUN rm -rf /tmp/*

ENV JAVA_HOME=/opt/graalvm
ENV PATH=$JAVA_HOME/bin:/opt/maven/bin:$PATH

WORKDIR "/task"
COPY src src/
COPY pom.xml ./
RUN  mvn -q -Pnative -DskipTests clean package

FROM public.ecr.aws/awsguru/lambda-http-ric:latest as http-ric

FROM public.ecr.aws/lambda/provided:al2
COPY --from=http-ric /opt/bin/lambda-http-ric /opt/bin/lambda-http-ric
COPY --from=native-builder /task/target/petstore-app /var/task/petstore-app

ENTRYPOINT ["/opt/bin/lambda-http-ric"]
# Command can be overwritten by providing a different command in the template directly.
#run linux executable file
CMD ["/var/task/petstore-app"]