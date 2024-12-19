DRAM
* DRAM （动态随机存储器）通过电容存储的，需要定期刷新电容
* SDRAM  同步DRAM
* DDRRAM 相对SDRAM速度提升
* 还有DDR3和DDR4
SRAM（静态随机存储器），以锁存器存储数据，不需要定期刷新充电，同样分为同步（SSRAM）和异步
### 0.1.1 DRAM与SRAM的应用场合[](https://doc.embedfire.com/mcu/stm32/f103badao/std/zh/latest/book/storage.html#dramsram "永久链接至标题")

对比DRAM与SRAM的结构，可知DRAM的结构简单得多，所以生产相同容量的存储器，DRAM的成本要更低，且集成度更高。 而DRAM中的电容结构则决定了它的存取速度不如SRAM，特性对比见表 [DRAM与SRAM对比](https://doc.embedfire.com/mcu/stm32/f103badao/std/zh/latest/book/storage.html#id7)。

![[../_resources/内存与存储器/42b10ffeb15e503a4bd448aaa121cd7d_MD5.png]]
### 0.1.2 存储器
rom（read only memory） 
* mask ROM 生产后不可修改
* OPTROM 一次性可编程存储器
* EPROM 可重复擦鞋存储器 使用特定设备可以擦除和写入
* EEPROM 电擦写存储器 可重复擦写
flash存储器 它的容量一般比EEPROM大得多， 且在擦除时，一般以多个字节为单位。如有的FLASH存储器以4096个字节为扇区，最小的擦除单位为一个扇区。根据存储单元电路的不同， FLASH存储器又分为NOR FLASH和NAND FLASH

[[../_resources/内存与存储器/dbf49f531216649aa7c40ddd7d819f5a_MD5.jpeg|Open: Pasted image 20241121225529.png]]
![[../_resources/内存与存储器/dbf49f531216649aa7c40ddd7d819f5a_MD5.jpeg]]
所以在实际应用场合中，SRAM一般只用于CPU内部的高速缓存(Cache)，而外部扩展的内存一般使用DRAM。 在STM32系统的控制器中，只有STM32F429型号或更高级的芯片才支持扩展SDRAM，
其它型NOR与NAND的共性是在数据写入前都需要有擦除操作，而擦除操作一般是以“扇区/块”为单位的。号如STM32F1、STM32F2及STM32F407等型号只能扩展SRAM。

NOR与NAND flash
NOR 地址线与数据线分开 读取数据会快一些

NAND 地址线与数据线共用 存储比较大 没有挂在到地址总线中，作为系统的启动盘需要将数据写入sram中 寿命长

空气质量检测仪 外置 flash W25Q64   

W25Q64（NAND flash） 的内存空间结构：一页 256 字节，4K(4096 字节)为一个扇区，16 个扇区为 1 块（64K），容量为 8M 字节,共有 128 个块,2048 个扇区

大小64Mbit/8=8M

### 0.1.3 I2C 读取EEPROM

i2c配置
* I2C传输速率、I2C模式、占空比、本设备地址、从设备地址、使能应答信号、地址位数（7位或者10位）
[[EEPROM AT24C02操作说明]]
