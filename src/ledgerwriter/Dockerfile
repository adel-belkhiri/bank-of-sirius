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

FROM bos-java

LABEL org.opencontainers.image.source=https://github.com/nginxinc/bank-of-sirius

ENV APP_NAME ledger-writer

COPY target/ledgerwriter.jar /app/libs/ledgerwriter.jar

#ENV LTTNG_UST_DEBUG 1

CMD ["java", "-Dlogging.config=/app/log4j2.xml", "-Djava.library.path=/usr/lib/x86_64-linux-gnu/jni", "-jar", "/app/libs/ledgerwriter.jar"]