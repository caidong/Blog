1. 下载源码文件

   上 https://go.dev/dl/ 找到合适的安装版本，

   wget https://go.dev/dl/go1.17.5.linux-arm64.tar.gz

2. 安装包解压

   tar -zxvf   *.tar.gz

3. 配置环境变量

   在 /etc/profile 文件末尾追加

   export GOROOT=/go
   export GOPATH=/goProject
   export GOBIN=$GOPATH/bin
   export PATH=$PATH:$GOROOT/bin
   export PATH=$PATH:$GOPATH/bin