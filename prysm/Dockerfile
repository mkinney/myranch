FROM ubuntu:bionic

RUN apt-get -y update

# install pre-reqs
RUN apt-get install -y git pkg-config zip g++ zlib1g-dev unzip python3 wget software-properties-common 

# install go
RUN add-apt-repository -y ppa:longsleep/golang-backports
RUN apt-get -y update
RUN apt-get install -y golang-go

# install bazel
RUN wget https://github.com/bazelbuild/bazel/releases/download/1.1.0/bazel-1.1.0-installer-linux-x86_64.sh
RUN chmod +x ./bazel-1.1.0-installer-linux-x86_64.sh
RUN ./bazel-1.1.0-installer-linux-x86_64.sh

RUN git clone https://github.com/prysmaticlabs/prysm.git workspace

WORKDIR /workspace

# Build binaries for minimal configuration.
RUN bazel build --define ssz=minimal \
  //beacon-chain \
  //validator \
  //tools/interop/convert-keys

#FROM gcr.io/whiteblock/base:ubuntu1804
#
#COPY --from=builder /workspace/bazel-bin/beacon-chain/linux_amd64_stripped/beacon-chain .
#COPY --from=builder /workspace/bazel-bin/validator/linux_amd64_pure_stripped/validator .
#COPY --from=builder /workspace/bazel-bin/tools/interop/convert-keys/linux_amd64_stripped/convert-keys .
#
#RUN mkdir /launch
#
#COPY scripts/interop_start.sh /launch/start.sh
#
#ENTRYPOINT ["start.sh"]
