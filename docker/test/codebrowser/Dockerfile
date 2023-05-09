# rebuild in #33610
# docker build --network=host -t clickhouse/codebrowser .
# docker run --volume=path_to_repo:/repo_folder --volume=path_to_result:/test_output clickhouse/codebrowser
ARG FROM_TAG=latest
FROM clickhouse/binary-builder:$FROM_TAG

# ARG for quick switch to a given ubuntu mirror
ARG apt_archive="http://archive.ubuntu.com"
RUN sed -i "s|http://archive.ubuntu.com|$apt_archive|g" /etc/apt/sources.list

RUN apt-get update && apt-get --yes --allow-unauthenticated install libclang-${LLVM_VERSION}-dev libmlir-${LLVM_VERSION}-dev

# libclang-15-dev does not contain proper symlink:
#
# This is what cmake will search for:
#
#     # readlink -f /usr/lib/llvm-15/lib/libclang-15.so.1
#     /usr/lib/x86_64-linux-gnu/libclang-15.so.1
#
# This is what exists:
#
#     # ls -l /usr/lib/x86_64-linux-gnu/libclang-15*
#     lrwxrwxrwx 1 root root       16 Sep  5 13:31 /usr/lib/x86_64-linux-gnu/libclang-15.so -> libclang-15.so.1
#     lrwxrwxrwx 1 root root       21 Sep  5 13:31 /usr/lib/x86_64-linux-gnu/libclang-15.so.15 -> libclang-15.so.15.0.0
#     -rw-r--r-- 1 root root 31835760 Sep  5 13:31 /usr/lib/x86_64-linux-gnu/libclang-15.so.15.0.0
#
ARG TARGETARCH
RUN arch=${TARGETARCH:-amd64} \
    && case $arch in \
        amd64) rarch=x86_64 ;; \
        arm64) rarch=aarch64 ;; \
        *) exit 1 ;; \
    esac \
    && ln -rsf /usr/lib/$rarch-linux-gnu/libclang-15.so.15 /usr/lib/$rarch-linux-gnu/libclang-15.so.1

# repo versions doesn't work correctly with C++17
# also we push reports to s3, so we add index.html to subfolder urls
# https://github.com/ClickHouse-Extras/woboq_codebrowser/commit/37e15eaf377b920acb0b48dbe82471be9203f76b
RUN git clone --depth=1 https://github.com/ClickHouse/woboq_codebrowser /woboq_codebrowser \
  && cd /woboq_codebrowser \
  && cmake . -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_CXX_COMPILER=clang\+\+-${LLVM_VERSION} -DCMAKE_C_COMPILER=clang-${LLVM_VERSION} \
  && ninja

COPY build.sh /
CMD ["bash", "-c", "/build.sh 2>&1"]
