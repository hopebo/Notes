# 安装 mold 加速链接

[github: rui314/mold](https://github.com/rui314/mold)

## 编译

mold 编译需要 gcc 10 以上

```bash
git clone git@github.com:rui314/mold.git
mkdir mold/build
cd mold/build
git checkout v2.1.0

# 已经安装好gcc-g++，cmake等，剩余可能所需依赖
sudo yum install -y openssl-devel zlib-devel
sudo yum install -y glibc-static file libstdc++-static diffutils util-linux

cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_CXX_COMPILER=c++ -DCMAKE_C_COMPILER=gcc -DCMAKE_INSTALL_PREFIX=/usr/local ..
cmake --build . -j $(nproc)
sudo cmake --build . --target install
```

## 使用方法

- 第一种：在编译的 flag 里指定 mold 路径

`EXPORT CXXFLAGS="-B/usr/local/libexec/mold"`

- 第二种：通过 mold 命令来编译

`mold -run make <make-options-if-any>`
