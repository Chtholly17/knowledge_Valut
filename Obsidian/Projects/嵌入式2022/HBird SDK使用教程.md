## E203相关资料
* Soc外设包括：PLIC、CLINT、LCLKGEN、HCLKGEN、GPIO、SPI、I2C(还有部分未列出)
	* PLIC：平台级别中断控制器，标准系统中断控制器，用于多个外部中断源的优先级仲裁
	* CLINT：产生计时器中断和软件中断
	* LCLKGEN：为常开域生成时钟（通常为低速的使用时钟）
	* HCLKGEN：生成高速时钟
	* GPIO：General Purpose IO，用于管理系统IO
	* SPI：Serial peripheral interface（串行外设接口）
	* I2C：集成电路互联总线
## 基于SDK的软件开发
* 程序存放在ITCM中，并从ITCM中直接执行![[Pasted image 20221024143306.png]]