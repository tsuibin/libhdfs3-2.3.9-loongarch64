# libhdfs3 2.3.9 (doris-thirdparty fork) on loongarch64

## 唯一的架构改动：软件 CRC32C 回退

上游（含 Doris fork）的 CRC32C 走 x86 SSE4.2 硬件指令，loongarch64 没有对应内建。
必须加一条**标量 CRC32C 软件实现回退**，否则编译期就报找不到 SSE 内建。
这也是本包移植中唯一真正需要写 loongarch 代码的地方。

## 构建（cmake）

    cmake -B build -DCMAKE_INSTALL_PREFIX=$P -DCMAKE_BUILD_TYPE=RelWithDebInfo \
          -DBUILD_SHARED_LIBS=OFF -DBUILD_STATIC_LIBS=ON -DBUILD_TEST=OFF \
          -DENABLE_BOOST=OFF -DENABLE_SSE=OFF -DENABLE_FRAME_POINTER=ON \
          -DCMAKE_CXX_FLAGS="-include cstdint -std=gnu++17" .
    cmake --build build -j4 && cmake --install build

关键开关：`-DENABLE_SSE=OFF`（关掉 x86 CRC32C 路径，走新增的标量回退）、
`-DENABLE_BOOST=OFF`、`-include cstdint`（GCC 16 不再隐式包含）。
产物 `installed/lib/libhdfs3.a`。

## 头文件

Doris 用 `#include <hdfs/hdfs.h>`，需从源码 `src/client/hdfs.h` 拷到 `installed/include/hdfs/`。

## 验证

已链接进 `doris_be`，0 编译错误、0 undefined symbol。
