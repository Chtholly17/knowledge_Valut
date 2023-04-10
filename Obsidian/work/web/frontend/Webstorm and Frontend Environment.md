## set up for Vue
- reference:
	- https://blog.csdn.net/L812273396/article/details/117988207
	- https://juejin.cn/post/7039581214272913415
- install node.js(npm is concluded in latest node.js)
	- check:![[Pasted image 20230406150224.png]]
	- what is node.js: a JavaScript environment based on Chrome V8
	- what is npm: Node.js Package Manager
- install cnpm(chinese npm, accelerate npm in china)
```bash
npm install -g cnpm --registry=https://registry.npm.taobao.org
cnpm - v
```
![[Pasted image 20230406150404.png]]
![[Pasted image 20230406150510.png]]
- install webpack
	- what is webpack: https://zhuanlan.zhihu.com/p/30981251
```bash
cnpm install webpack -g
```
- install vue-cli
	- what is vue and vue-cil: https://blog.csdn.net/weixin_46392266/article/details/122342711?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0-122342711-blog-121642395.235^v27^pc_relevant_recovery_v2&spm=1001.2101.3001.4242.1&utm_relevant_index=2
```bash
npm install --global vue-cli
vue -V
```
![[Pasted image 20230406151130.png]]

- HelloWorld: 
	- create a vue project in webstorm and run directly![[Pasted image 20230406152314.png]]
	- webStrom select **Vue 3 as default( not the vue 2.9.x we installed before)**