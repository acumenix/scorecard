# Copyright 2021 Security Scorecard Authors
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

FROM golang@sha256:3c4de86eec9cbc619cdd72424abd88326ffcf5d813a8338a7743c55e5898734f AS base
WORKDIR /src
ENV CGO_ENABLED=0
COPY go.* ./
RUN go mod download
COPY . ./

FROM base AS shuffle
ARG TARGETOS
ARG TARGETARCH
RUN CGO_ENABLED=0 make build-shuffler
RUN ./cron/shuffle/shuffle 2500 cron/data/projects.csv cron/data/projects.release.csv

FROM base AS controller
ARG TARGETOS
ARG TARGETARCH
RUN CGO_ENABLED=0 make build-controller

FROM gcr.io/distroless/base:nonroot@sha256:764b74b1789b4ec9044e6f20bb938d077fe8c7bcf9e4d3767eebb440c5d76f11
COPY ./cron/data/projects*csv cron/data/
COPY --from=shuffle /src/cron/data/projects.release.csv cron/data/projects.release.csv
COPY --from=controller /src/cron/controller/controller cron/controller/controller
ENTRYPOINT ["cron/controller/controller"]
