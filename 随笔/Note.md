1. Servlet类原理：
   1. 接收http请求
   2. 分发请求给真正业务处理类
   3. 响应处理结果
2. 源码解析
   1. DispatcherServlet:doservice
3. @RequestMapping
   1. ![image-20181217222458899](/Users/changbingbing/Library/Application Support/typora-user-images/image-20181217222458899.png)
   2. @RestController修饰类，那么该类中所有方法都
4. 

![image-20181219210156156](/Users/changbingbing/Library/Application Support/typora-user-images/image-20181219210156156.png)

![image-20181219210432904](/Users/changbingbing/Library/Application Support/typora-user-images/image-20181219210432904.png)



![image-20181219215407968](/Users/changbingbing/Library/Application Support/typora-user-images/image-20181219215407968.png)

1. ![image-20181219220613440](/Users/changbingbing/Library/Application Support/typora-user-images/image-20181219220613440.png)
2. ![image-20181219221031509](/Users/changbingbing/Library/Application Support/typora-user-images/image-20181219221031509.png)
3. 
4. clean tomcat:run
5. ![image-20181221212425667](/Users/changbingbing/Library/Application Support/typora-user-images/image-20181221212425667.png)

get请求，走上面queryItemById;post请求，就走saveItem