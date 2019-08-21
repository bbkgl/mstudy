# ti-cutter模型切分简易教程

## 1.为什么要切模型？

因为我们的网络有些层在eve上执行不了。eve上只能执行常规的卷积，全连接和池化。所以要把一些后处理的层从网络中剥离出来放在a15上实现。这样就将模型分成了两部分，前一部分在eve上运行，需要做定点化，后一部分在A15上运行，是浮点的。

## 2.需要准备哪些内容？

- 在自己电脑上或者服务器上git克隆[ti-cutter项目](https://gitlab.momenta.works/xubaobei/ti-cutter)，需要有python2环境

- 打开张宁电脑（windows），确保笔记本不会休眠，且ssh能连接上，测试`ssh zhangning@10.8.118.3`

- `ti-cutter`项目克隆下来后，查看`quant_images`下是否有四个bin格式的文件，这其实是四张量化图像：

  - `quant.bin` ：car的量化图像
  - `quant_joint.bin`：拼接方案car的量化图像
  - `2017-01-20-08-43-47000000.bin`：object的量化图像
  - `object的量化图像`：lane的量化图像

  在切模型的同时，也进行了eve上模型的定点化操作，需要喂一张图进去，决定定点化的范围。

- 需要被切分的模型，也就是打包的tar文件

### 3.操作步骤

1. ssh到44服务器上，并启动docker

   ```shell
   ssh root@172.16.10.44  # 密码：lxjbj2018
   cd /data3/zoujiakun/
   ./docker_run.sh   # docker启动后回到根目录
   ```

2. cd到`ti-cutter`目录下

   ```shell
   cd /data3/zoujiakun/repos/ti-cutter/
   ```

3. 将需要切分的模型放到models文件夹下，然后解压

   ```shell
   cd models
   tar -vxf <**>.tar
   ```

4. 将对应的`net.prototxt、model.caffemodel、config_convert.txt、config_simulate.txt、config_simulate_list.txt`文件拷贝到ti文件夹下

   ```shell
   cd model/<sub_dir>   # sub_dir为对应切分模型的文件夹，如车辆alignment模型就是名字中有car_align字样的文件夹
   cp net.prototxt model.caffemodel config_convert.txt config_simulate.txt config_simulate_list.txt ../../../ti/
   ```

5. 回到`ti-cutter`根目录，将量化图像拷贝到ti文件夹下

   ```shell
   cp quant_images/* ti/
   ```

6. 拼接方案的alignment的模型切分：

   - 进入ti目录下，`cd ti`

   - 将config_convert.txt文件下的inElementType设为0

   - config_simulate.txt文件下的inData设置为quant_joint.bin

   - outData设置为quant_joint_out.bin

   - 然后回到`ti-cutter`根目录，对于当前（2019.07.17）模型结构，执行

     ```shell
     ./cut.py -c s6_b1_eltwise_data --quant_image ./ti/quant_joint.bin
     ```

7. 执行结束后，会在`ti-cutter`目录下，生成一个result_XXXX_XX_XX_XX文件夹，XX是当前的年月日时分秒。

8. 将result_XXXX_XX_XX_XX文件夹下的post.prototxt重命名为net.prototxt，则该文件夹下应该有`backbone.prototxt  model.caffemodel  net.bin  net.prototxt  prm.bin`四个文件

9. 然后找到一个以前的ti模型，以前的ti模型tar包的目录结构应该是

   ```shell
   - ***_ti_XXXXX.tar
   ---config
   ------car.core0.json
   ------car.core1.json
   ------car.core2.json
   ------car.core3.json
   ---model
   ------car.align.v1.190426_core1
   ---------config.json
   ---------model.caffemodel
   ---------net.prototxt
   ---------prm.bin
   ---------net.bin
   ------car.align.v1.190426_core2
   ---------config.json
   ---------model.caffemodel
   ---------net.prototxt
   ---------prm.bin
   ---------net.bin
   ------car.align.v1.190426_core3
   # 同上
   ------car.align.v1.190426_core0
   # 同上
   ```

10. 首先用`backbone.prototxt  model.caffemodel  net.bin  net.prototxt  prm.bin`四个文件覆盖对应的`car.align.v1.190426_core*`(*为1234)下的同名文件，切分的模型对应多个文件夹，就要替换多个文件夹

11. 修改配置文件，在第九点说的tar包目录结构中的json配置文件应该都要修改，一般情况下，根据原x86tar包下的**对应位置和对应模型**的json配置文件去修改ti上tar包对应的配置文件

12. 然后进行打包，打包命令

    ```shell
    tar -cvf <包名>.tar model config
    ```

    