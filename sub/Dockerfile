#############################
#     设置公共的变量         #
#############################
FROM alpine:latest as base
# 作者描述信息
MAINTAINER danxiaonuo
# 时区设置
ARG TZ=Asia/Shanghai
ENV TZ=$TZ


# 镜像变量
ARG DOCKER_IMAGE=danxiaonuo/subconverter
ENV DOCKER_IMAGE=$DOCKER_IMAGE
ARG DOCKER_IMAGE_OS=subconverter
ENV DOCKER_IMAGE_OS=$DOCKER_IMAGE_OS
ARG DOCKER_IMAGE_TAG=latest
ENV DOCKER_IMAGE_TAG=$DOCKER_IMAGE_TAG
ARG BUILD_DATE
ENV BUILD_DATE=$BUILD_DATE
ARG VCS_REF
ENV VCS_REF=$VCS_REF


# ##############################################################################

# ***** 设置变量 *****

# dumb-init
# https://github.com/Yelp/dumb-init
ARG DUMBINIT_VERSION=1.2.2
ENV DUMBINIT_VERSION=$DUMBINIT_VERSION


####################################
#       构建subconverter            #
####################################
FROM base AS builder

# http://label-schema.org/rc1/
LABEL maintainer="danxiaonuo <danxiaonuo@danxiaonuo.me>" \
      org.label-schema.build-date=$BUILD_DATE \
      org.label-schema.name="$DOCKER_IMAGE" \
      org.label-schema.schema-version="1.0" \
      org.label-schema.url="https://github.com/$DOCKER_IMAGE" \
      org.label-schema.vcs-ref=$VCS_REF \
      org.label-schema.vcs-url="https://github.com/$DOCKER_IMAGE" \
      versions.dumb-init=${DUMBINIT_VERSION}

# 构建安装依赖
ARG BUILD_DEPS="\
      wget \
      tzdata"
ENV BUILD_DEPS=$BUILD_DEPS


# 修改源地址
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories
# ***** 安装相关依赖并更新系统软件 *****
# ***** 安装依赖 *****
RUN set -eux \
   # 更新源地址
   && apk update \
   # 更新系统并更新系统软件
   && apk upgrade && apk upgrade \
   && apk add -U --update --virtual .$BUILD_DEPS \
   # 更新时区
   && ln -sf /usr/share/zoneinfo/${TZ} /etc/localtime \
   # 更新时间
   && echo ${TZ} > /etc/timezone
# ##############################################################################

# 安装dumb-init
RUN set -eux \
    && wget --no-check-certificate https://github.com/Yelp/dumb-init/releases/download/v${DUMBINIT_VERSION}/dumb-init_${DUMBINIT_VERSION}_x86_64 -O /usr/bin/dumb-init \
    && chmod +x /usr/bin/dumb-init

# 工作目录
WORKDIR /base
	
# 安装subconverter
RUN set -eux \
    && wget --no-check-certificate -P /base https://github.com/tindy2013/subconverter/releases/latest/download/subconverter_linux64.tar.gz -O /base/subconverter_linux64.tar.gz \
    && tar xzf subconverter_linux64.tar.gz \
    && rm -rf subconverter_linux64.tar.gz \
    && apk del .$BUILD_DEPS
	

# 容器信号处理
STOPSIGNAL SIGQUIT

# 入口
ENTRYPOINT ["dumb-init"]

# 启动命令
CMD ["./subconverter/subconverter"]
