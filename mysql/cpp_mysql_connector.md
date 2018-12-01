##C++连接mysql
1. 下载mysql-connector-c++
````
wget https://cdn.mysql.com//Downloads/Connector-C++/mysql-connector-c++-8.0.13-src.tar.gz
````
2. 解压
````
tar -zxf mysql-connector-c++-8.0.13-src.tar.gz
````
3. 依赖
CMake (2.8.12 or higher), C++ compiler that supports C++11.
MySQL Client Library
Boost C++ Libraries
SSL Support openssl

4. 编译
````
cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql_cpp_connector -DWITH_BOOST=/usr/local/boost_1_59_0 -DCMAKE_BUILD_TYPE=Debug -DMYSQL_DIR=/usr/local/mysql
````