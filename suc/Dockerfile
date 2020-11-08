##########################################
#            设置公共的变量               #
##########################################
# 指定构建的基础镜像
FROM node:lts-alpine AS dependencies

# 作者描述信息
MAINTAINER danxiaonuo
# 时区设置
ARG TZ=Asia/Shanghai
ENV TZ=$TZ

# 镜像变量
ARG DOCKER_IMAGE=danxiaonuo/sub-web
ENV DOCKER_IMAGE=$DOCKER_IMAGE
ARG DOCKER_IMAGE_OS=node
ENV DOCKER_IMAGE_OS=$DOCKER_IMAGE_OS
ARG DOCKER_IMAGE_TAG=lts-alpine
ENV DOCKER_IMAGE_TAG=$DOCKER_IMAGE_TAG
ARG BUILD_DATE
ENV BUILD_DATE=$BUILD_DATE
ARG VCS_REF
ENV VCS_REF=$VCS_REF

ARG BUILD_DEPS="\
      tzdata \
      curl \
      git \
      wget \
      bash"
ENV BUILD_DEPS=$BUILD_DEPS

# dumb-init
# https://github.com/Yelp/dumb-init
ARG DUMBINIT_VERSION=1.2.2
ENV DUMBINIT_VERSION=$DUMBINIT_VERSION

# 修改源地址
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories
# ***** 安装依赖 *****
RUN set -eux \
   # 更新源地址
   && apk update \
   # 更新系统并更新系统软件
   && apk upgrade && apk upgrade \
   && apk add -U --update $BUILD_DEPS \
   # 更新时区
   && ln -sf /usr/share/zoneinfo/${TZ} /etc/localtime \
   # 更新时间
   && echo ${TZ} > /etc/timezone


# 克隆源码运行安装
RUN set -eux \
    && git clone --progress https://github.com/CareyWang/sub-web.git /sub-web

# 拷贝配置文件	
COPY conf/sub-web/.env /sub-web/.env
COPY conf/sub-web/Subconverter.vue /sub-web/src/views/Subconverter.vue

# 工作目录
WORKDIR /app

RUN set -eux \
    && cp /sub-web/package.json /app \
    && yarn install

# ##############################################################################
    
##########################################
#            构建运行环境                 #
##########################################
FROM dependencies AS build
WORKDIR /app
RUN set -eux \
    && cp -rf /sub-web/. /app \
    && yarn build

# ##############################################################################


##########################################
#         构建基础镜像                    #
##########################################
# 
# 指定创建的基础镜像
FROM danxiaonuo/nginx:latest

# http://label-schema.org/rc1/
LABEL maintainer="danxiaonuo <danxiaonuo@danxiaonuo.me>" \
      org.label-schema.build-date=$BUILD_DATE \
      org.label-schema.name="$DOCKER_IMAGE" \
      org.label-schema.schema-version="1.0" \
      org.label-schema.url="https://github.com/$DOCKER_IMAGE" \
      org.label-schema.vcs-ref=$VCS_REF \
      org.label-schema.vcs-url="https://github.com/$DOCKER_IMAGE" \
      versions.dumb-init=${DUMBINIT_VERSION}


# 工作目录
WORKDIR /www

# 拷贝资源文件
COPY --from=build /app/dist /www
COPY conf/sub-web/vhost/default.conf /data/nginx/conf/vhost/default.conf

# 容器信号处理
STOPSIGNAL SIGQUIT

# 入口
ENTRYPOINT ["dumb-init"]

# 启动命令
CMD ["nginx", "-g", "daemon off;"]
