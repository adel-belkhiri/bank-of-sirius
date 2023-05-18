# Copyright 2021 F5
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

ARG ARCH=amd64

FROM $ARCH/nginx:1.18.0

# COPY --from=ghcr.io/nginxinc/amd64/ngx_otel_module:linux-libc-nginx-1.21.6 /usr/lib/nginx/modules/otel_ngx_module.so /usr/lib/nginx/modules/otel_ngx_module.so

# Install build dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
	git wget pkg-config build-essential \
	autoconf libtool \
	libpcre3-dev libssl-dev \
	libxml2-dev libxslt1-dev \
	protobuf-compiler libcurl4-openssl-dev \
	cmake clang libc++-dev \
	libpopt-dev uuid-dev automake flex \
	gobjc++ bison make libc6 libc6-dev libglib2.0-0 libglib2.0-dev

# Install lttng-ust v2.13.*
WORKDIR /tmp
RUN wget https://lttng.org/files/urcu/userspace-rcu-latest-0.13.tar.bz2 && \
	tar -xf userspace-rcu-latest-0.13.tar.bz2 && \
	cd userspace-rcu-0.13.* && \
	./configure --libdir=/usr/local/lib && \
	make && \
	make install && \
	ldconfig

WORKDIR /tmp
RUN wget https://lttng.org/files/lttng-ust/lttng-ust-latest-2.13.tar.bz2 && \
	tar -xf lttng-ust-latest-2.13.tar.bz2 && \
	cd lttng-ust-2.13.* && \
	./configure --disable-numa --libdir=/usr/local/lib \
        LDFLAGS='-L/usr/local/lib -L/usr/lib' && \
	make && \
	make install && \
	make clean && \
	ldconfig

# Install gRPC and its dependencies
WORKDIR /tmp
RUN git clone -b v1.48.0 https://github.com/grpc/grpc.git
RUN cd grpc && \
    git submodule update --init && \
    mkdir -p cmake/build && \
    cd cmake/build && \
    cmake ../.. && \
    make -j4 && \
    make install && \
	ldconfig && \
	make clean

WORKDIR /tmp
# Clone the OpenTelemetry C++ SDK and build it
RUN git clone --single-branch -b v1.6.0 https://github.com/open-telemetry/opentelemetry-cpp.git
COPY ./patch/lttng-instrumentation.patch  opentelemetry-cpp/
RUN cd opentelemetry-cpp \
	&& git apply  ./lttng-instrumentation.patch \
    && mkdir build \
    && cd build \
    && cmake -DCMAKE_BUILD_TYPE=release \
       -DBUILD_TESTING=OFF \
       -DWITH_EXAMPLES=OFF \
       -DCMAKE_POSITION_INDEPENDENT_CODE=ON \
       -DWITH_ABSEIL=ON \
       -DWITH_OTLP=ON \
       ..  \
    && make -j4 \
    && make install \
	&& make clean

WORKDIR /tmp
# Clone the OpenTelemetry instrumentation modules from GitHub and build the Nginx module
RUN git clone --single-branch https://github.com/open-telemetry/opentelemetry-cpp-contrib.git
RUN cd opentelemetry-cpp-contrib/instrumentation/nginx \
    && mkdir build \
    && cd build \
    && cmake -DgRPC_BUILD_GRPC_NODE_PLUGIN=OFF \
                        -DgRPC_BUILD_GRPC_OBJECTIVE_C_PLUGIN=OFF \
                        -DgRPC_BUILD_GRPC_PHP_PLUGIN=OFF \
                        -DgRPC_BUILD_GRPC_PYTHON_PLUGIN=OFF \
                        -DgRPC_BUILD_GRPC_RUBY_PLUGIN=OFF \
                       ..\
    && make -j4 \
    && make install \
	&& make clean

#ENV LTTNG_UST_DEBUG 1

COPY etc/nginx/nginx.conf /etc/nginx/nginx.conf
COPY etc/nginx/otel-nginx.toml /etc/nginx/otel-nginx.toml
COPY etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf
