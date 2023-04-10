- SDK的使用，安装相关依赖包并建立helloworld示例项目
点击build进行编译![[Pasted image 20221121214046.png]]
- 如何获取Verilog代码（可以喂给soc系统进行调试）
[Hbirdv2的仿真_全国大学生集成电路创新创业大赛_RISC-V论坛讨论_RISC-V MCU中文社区 (riscv-mcu.com)](https://www.riscv-mcu.com/community-topic-id-623.html)
![[Pasted image 20221121214137.png]]
```
riscv-nuclei-elf-objcopy -O verilog "${BuildArtifactFileBaseName}.elf" "${BuildArtifactFileBaseName}.verilog";sed -i 's/@800/@000/g' "${BuildArtifactFileBaseName}.verilog"; sed -i 's/@00002FB8/@00002000/g' "${BuildArtifactFileBaseName}.verilog";
```
[技术分享--利用 NucleiStudio IDE 和 vivado 进行软硬件联合仿真_全国大学生集成电路创新创业大赛_RISC-V论坛讨论_RISC-V MCU中文社区 (rvmcu.com)](https://www.rvmcu.com/community-topic-id-386.html)