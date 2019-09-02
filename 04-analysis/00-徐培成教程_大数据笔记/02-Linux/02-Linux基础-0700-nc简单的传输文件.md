*假设说我们想把本机上的test02.txt文件传输到master上去，如下：*

* 01，首先开启master上到nc，并监听8888端口, 设定接受的数据输出到test02.txt
  
> nc -l 8888 > test02.txt

* 02, 在本机上开启nc，传输test01.txt到master上到8888端口
  
> nc master 8888 < test01.txt

* 03, 传输完成后，master上到nc会自动断开，这个时候文件已经存在

