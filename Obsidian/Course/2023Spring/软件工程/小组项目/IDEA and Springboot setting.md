- IDEA ultimize for MAC installation:
- Maven installation and setting:  [[https://blog.csdn.net/mjshuang/article/details/91448545]]
- Install JND for am1 slicon(arm 64 artitechure ) https://blog.csdn.net/q863672107/article/details/125453718
## Springboot helloworld
### Setting History
#### Wrong type selecting
- IDEA -> new project -> Spring Initializr
- set java version as 8![[Pasted image 20230401215641.png]]
- Spring Boot Version: 3.0.5
	- select spring web suit
- could not run demo after input the code![[Pasted image 20230401220449.png]]
- we need to set **IDEA run configration** before running, but I can not find the class that I just created![[Pasted image 20230401221640.png]]
- we need to set project structure manually
[[https://blog.csdn.net/yyj108317/article/details/89182332]]
- problem occured: cannot resolve the symbol 'springframework'![[Pasted image 20230401222141.png]]
#### **Appropriate way: use maven as porject type**
- a project created by using spring boot initializer ![[Pasted image 20230402144643.png]]
	- a main class: create automaticlly using project name({name}+ Application)
	- a test class: ({name}+ ApplicationTests)
	- application.properties: set project properties(such as the port and so on)![[Pasted image 20230402150311.png]]
	- pom.xml: import compoments according the tools we selected when creating the project
#### JAVA VERSION ERROR
- frist: try to run the application without the controller
	- an error occured: **java version probelm**![[Pasted image 20230402151506.png]]
	- **worked solution**: [[https://stackoverflow.com/questions/27037657/stop-intellij-idea-to-switch-java-language-level-every-time-the-pom-is-reloaded/27037879#27037879]]
		- change the setting in project structure-> language level, set it to 8 ![[Pasted image 20230402151623.png]]
		- change in pom.xml: set java version to 8![[Pasted image 20230402151743.png]]
		- settings-> Compiler-> Java Compiler: set **project bytecode version and Target bytecode version** to 8![[Pasted image 20230402152030.png]]
		- settings-> maven -> importing: JDK for impoter, set it to Azul Zulu 1.8![[Pasted image 20230402152256.png]]
#### Import Error
- after solved previous error, an new error occured![[Pasted image 20230402152531.png]]
- It seems that the reason for series error is that I choosed wrong springboot version when creating the project, **this explains why previous java version error occured when I choose the Java version as 8**  ![[Pasted image 20230402152824.png]][[https://stackoverflow.com/questions/73679679/java-cannot-access-org-springframework-boot-springapplication-bad-class-file]]
-  Trying to recreate the project with approriate springboot version![[Pasted image 20230402153012.png]]
#### package does not exists error
- **reason: do not change the file structure!! 
- after creating a project with springboot version 2.7.11-SNAPSHOT, a new error occured![[Pasted image 20230402153732.png]]
- reason: forget to set maven version for this project![[Pasted image 20230402154428.png]]
- next import error in test class
![[Pasted image 20230402154623.png]]
#### create a controller class
- this step reference to[[https://www.jetbrains.com/idea/guide/tutorials/your-first-spring-application/creating-spring-controller/]]
- create a new java class, **in the same floder as the start class**(name it as controller for recognition)![[Pasted image 20230402163421.png]]
- add a class level annotation of `@RestController`
	- This annotation means this class will be picked up by the component scan, this class will therefor be available from the application context
- create a method that will tell Spring that if we go the root of our webserver, we would like to see the string(Hello world)
	- using @RequestMapping, to create a mapping towards the root of our web server
	- @RequestMapping("/test")  ,then the word will be show when we open localhost:{port}/test
```java
@RestController  
public class DemoController {  
	@RequestMapping("/")  
	public String test(){  
		return "hello world";  
	}  
}
```
### overall steps
- create a spring boot project, the setting is as follows![[Screenshot 2023-04-02 at 16.42.33.png]] ![[Screenshot 2023-04-02 at 16.47.15.png]]
- create a controller class, refer to selection: **create a controller class**
