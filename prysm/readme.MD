Wanted to build a docker image for https://github.com/prysmaticlabs/prysm and build from master.

Started with the interop.Dockerfile but made lots of changes

docker build .

# bazel run //beacon-chain -- --datadir=/tmp/prysm-data
