- Restful: **GET  POST PUT DELETE**
- recieve data from frontend: 
	- mapping to a web page 
	- set parameter, Springboot will rec it from frontend automaticlly
## layers
- controller
- service
- DAO(data access object)
	- insert interface: read from entity class automaticlly![[Pasted image 20230403172341.png]]
	- Entity class: related to a table in database(such as User class)
		- It is strongly recommondated to **name the attribute according to colomn name of the table**. thus mybatis can fill the data automaticlly
## bugs
- Error attempting to get column 'name' from result set
	- Entity must have a No parameter construction method![[Pasted image 20230403170917.png]]
