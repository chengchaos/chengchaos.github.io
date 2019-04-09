$ sudo yum install autoconf automake libtool curl make gcc-c++ unzip


https://github.com/google/protobuf/releases/tag/v2.5.0


cd protobuf-2.5.0
./autogen.sh
./configure
make
make check
make install
ldconfig


http://googletest.googlecode.com/files/gtest-1.5.0.tar.bz2


https://github.com/google/googletest/archive/release-1.5.0.tar.gz

g++ -std=c++11 -isystem ./include -pthread path/to/your_test.cc libgtest.a \
    -o your_test


在github上下载的google protobuf 2.5.0分支编译问题

由于没有configure文件，需要使用autogen生成，autogen依赖gtest所以

wget https://codeload.github.com/google/protobuf/zip/v2.5.0
mv v2.5.0 protobuf_2.5.0.zip
unzip protobuf_2.5.0.zip
cd protobuf-2.5.0
修改autogen.sh，注释curl http://googletest.googlecode.com/files/gtest-1.5.0.tar.bz2 | tar jx
改为：
    wget https://codeload.github.com/google/googletest/zip/release-1.5.0
    unzip release-1.5.0
    mv googletest-release-1.5.0 gtest
./autogen.sh
./configure
make