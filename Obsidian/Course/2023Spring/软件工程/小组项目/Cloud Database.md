## Init and Management
- env: Huawei Cloud MySql database
	- **intranet ip: 192.168.0.4
	- **public ip: **120.46.191.251
	- **port: 3306
	- **passwd for root: 1277411655Wjc
	- console: https://console.huaweicloud.com/
- connetion
	- official reference: 
		- all connection method: https://support.huaweicloud.com/qs-rds/rds_02_0060.html
		- connect by intranet: https://support.huaweicloud.com/qs-rds/zh-cn_topic_apply_for_rds.html
		- connect by DAS: https://support.huaweicloud.com/qs-rds/rds_02_0007.htm
	- Connect with DAS(Data Admin Service): **officially recommaned and easy**
		- open database management page, and click this button ![[Screenshot 2023-04-02 at 19.50.17.png]]
		- login with above passwd for root
- database management:
	- Datagrip or IDEA: input the public net address and port![[Pasted image 20230403115523.png]]
	- DAS: 
		- user name: 15076587331
		- passwd: jl010625
		- detail: click here to enter the entity management page![[Screenshot 2023-04-02 at 20.27.57.png]]
		- add database: click the button and select a name. I set the **utf8** as the character set![[Screenshot 2023-04-02 at 19.52.52 1.png]]
		- there are a series of operation after creating the table, contain all common table management operations![[Pasted image 20230402195604.png]]
		- here I created the first table for **user, character set is also utf8**![[Pasted image 20230402201857.png]]
## Usage in Develpoment
- IDEA project settings
	- common ![[Pasted image 20230402203445.png]]
	- dependencies![[Pasted image 20230402203515.png]]
	- **add new dependency for jdbc**
		- reference: https://blog.csdn.net/BingTaiLi/article/details/109842838
- link the database to a dymanic public net ip, and add this ip to security group![[Screenshot 2023-04-03 at 11.33.08.png]]
- modified the application.properties to reset the data source
```java
spring.datasource.url=jdbc:mysql://120.46.191.251:3306/user?serverTimezone=UTC  
spring.datasource.username=root  
spring.datasource.password=1277411655Wjc  
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```
- write class UserController![[Pasted image 20230402220040.png]]
- connection successed: ![[Pasted image 20230403114947.png]]
- there is no table in management page(user table should be there)
	- fix: change the datasource properties![[Pasted image 20230403115211.png]]
	- reference: https://blog.csdn.net/Yanzudada/article/details/117714441
