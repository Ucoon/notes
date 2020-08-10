**结合毕设项目，简要叙述一下springboot是怎么工作的：**

Spring Boot 是Spring MVC 的简化版本。是在 Spring MVC 的基础上实现了自动配置，简化了开发人员开发过程。

Spring MVC 是通过一个叫 `DispatcherServlet` 前端控制器的来拦截请求的。而在 Spring Boot 中 使用自动配置把 `DispatcherServlet` 前端控制器自动配置到框架中。

例如，以解析```/wxLogin```这个请求

![](https://www.cnblogs.com/images/cnblogs_com/fishpro/1453719/o_restful2.png)

1. `DispatcherServlet` 控制器拦截请求 `/wxLogin`

2. `servlet` 决定使用哪个 `handler` 处理

3. Spring 检测哪个控制器匹配 `/wxLogin`，Spring 从 @RquestMapping 中查找出需要的信息

4. Spring 找到正确的`WXLoginController`类后，开始执行 对应的wxLogin方法

   ```java
   1.从参数对象中获取到微信code，微信昵称，微信头像信息
   2.判断微信code是否为空，如果为空，则返回错误信息"wechat code can not be null or empty"；不为空则去请求微信接口，获取openid
   3.判断微信接口返回的openId是否为空，如果为空，则返回错误信息"wechat request error"；不为空则根据openId从user表获取用户信息
   4.判断获取到的用户信息usersResult是否为null，如果为null，则新建user对象，保存openId、微信昵称，微信头像信息到数据库中，并往redis写入token信息，有效期为半小时（redis.set(USER_REDIS_SESSION + ":" + userModel.getId(), uniqueToken, 1000 * 60 * 30)；
   5.如果不为null，则直接往redis写入token信息，有效期为半小时（redis.set(USER_REDIS_SESSION + ":" + userModel.getId(), uniqueToken, 1000 * 60 * 30)；
   ```

6. 需要返回 Json 格式数据给小程序端

**SSM框架采用什么模式，一般分为哪些层，作用是什么：**

SSM框架，是Spring + Spring MVC + MyBatis的缩写；它采用了MVC（Model View Controller）模式，一般分为三层：模型(model)－视图(view)－控制器(controller)，三者的关系是视图层（View层）提交请求到控制器（Controller层），控制器（Controller层）根据url调用模型层获取数据，并将其返回给视图层，所以它的作用就是实现了页面展示与业务逻辑向分离，这也是解耦的重要实现方式。

**解释一下pom.xml的各个节点的作用**

1. project：pom文件的顶级节点

2. parent：父项目的坐标。如果项目中没有规定某个元素的值，那么父项目中的对应值即为项目的默认值。
   ​         坐标包括group ID，artifact ID和 version

3. artifactId：构件的标识符

4. groupId：项目的全球唯一标识符，通常使用全限定的包名区分该项目和其他项目，并且构建时生成的路径也是由此生成

5. version：一个项目的特定版本

   **由groupId、artifactId和version唯一的确定了一个项目坐标**

6. modelVersion：指定 pom.xml 符合哪个版本的描述符。maven 2 和 3 只能为 4.0.0。

7. packaging：项目产生的构件类型

8. name：项目的名称

9. dependencies：项目引入插件所需要的额外依赖

10. plugins：使用的插件列表

11. plugin：包含描述插件所需要的信息

12. build：构建配置

13. properties：属性列表

14. project.build.sourceEncoding：项目统一字符集编码

**解释一下vo和mapper包大概的功能**

```java
VO(View Object)：显示层对象，通常是Web向模板渲染引擎层传输的对象；即对于一个WEB页面，用一个VO对象对应整个界面的值。
```

结合实际代码，vo包在imooc-videos-dev-pojo工程下

```com.imooc.pojo.vo.UsersVO```是对应```wxlogin```接口返回给小程序端的对象；

```com.imooc.pojo.vo.VideosVO```是返回给小程序端首页视频列表的对象；

```com.imooc.pojo.vo.CommentsVO```是返回给小程序端视频页面的评论功能

```com.imooc.pojo.vo.PublisherVideo```是返回给小程序端视频页面的对象

**mapper包**

mapper包在imooc-videos-dev-mini-api工程下的resources的mapper文件夹

```java
所有的mapper.xml都有CRUD作用
额外的：
    CommentsMapperCustom.xml：queryComments 根据videoId查找相应评论数据
    SearchRecordsMapper.xml：getHotwords 返回热词（content）列表
    UsersMapper.xml：
    	addReceiveLikeCount：根据userId累加视频点赞数量字段receive_like_counts
    	reduceReceiveLikeCount：根据userId累减视频点赞数量字段receive_like_counts
    	addFansCount：根据userId累加粉丝数量字段fans_counts
    	reduceFansCount：根据userId累减粉丝数量字段fans_counts
    	addFollersCount：根据userId累加关注者数量字段follow_counts
    	reduceFollersCount：根据userId累减关注者数量字段follow_counts
    VideosMapperCustom.xml
    	queryAllVideos：根据videoDesc（视频描述信息）和userId（用户ID）获取视频列表信息
    	queryMyFollowVideos：根据userId查询我关注的人发的视频
    	queryMyLikeVideos：根据userId查询我喜欢的视频
    	addVideoLikeCount：根据videoId累加视频收到点赞数量字段like_counts
    	reduceVideoLikeCount：根据videoId累减视频收到点赞数量字段like_counts
```





**mybatis如何简化或缩短你的开发时间**

```java
原生的jdbc操作存在大量的重复性代码（如注册驱动，创建连接，创建statement，结果集检测等），而MyBatis通过XML或者注解的方式将要执行的sql语句配置起来，并通过java对象和sql语句映射成最终执行的sql语句。最终由MyBatis框架执行sql，并将结果映射成java对象并返回。
```

**你的前后台是采取什么样的方法进行数据传输？结合代码**

**你的图片上传是怎么实现的**

**项目说明**

```imooc-videos-dev```下各个子工程的作用

```java
imooc-videos-dev-common:主要是封装了一些工具类，比如Http请求（HttpClientUtil），Redis操作实现类（RedisOperator），分页封装类（PagedResult），FFmpegexe视频合成的工具类（MergeVideoMp3），返回json格式数据封装类（IMoocJSONResult），获取视频截图工具类（FetchVideoCover）
imooc-videos-dev-mini-api:controller层，包含各个接口
imooc-videos-dev-pojo:主要是对应数据库各张表的表结构的实体类
imooc-videos-dev-service:数据库的实现类，主要是获取数据库的数据
mybatis-generatorConfig:逆向生成pojo和mapper.xml
```

## SpringMVC的注解说明

1. ```@RestController```：Spring4之后新加入的注解，原来返回json需要@ResponseBody和@Controller配合。即@RestController是@ResponseBody和@Controller的组合注解。

2. ```@Api```：用在请求的类上，表示对类的说明

   1. tags="说明该类的作用，可以在UI界面上看到的注解"
   2. value="该参数没什么意义，在UI界面上也看不到，所以不需要配置"

3. ```@RequestMapping```：配置url映射

   @RequestMapping此注解即可以作用在控制器（Controller类）的某个方法上，也可以作用在此控制器类（Controller类）上。

   当控制器在类级别上添加@RequestMapping注解时，这个注解会应用到控制器的所有处理器方法上。

4. ```@ApiOperation```：用在请求的方法上，说明方法的用途、作用

   1. value="说明方法的用途、作用"
   2.  notes="方法的备注说明"

5. ```@ApiImplicitParams```：用在请求的方法上，表示一组参数说明

6.  ```@ApiImplicitParam```：用在```@ApiImplicitParams```注解中，指定一个请求参数的各个方面

      	1. name：参数名
    	2. value：参数的汉字说明、解释
    	3.  required：参数是否必须传
    	4.  dataType：参数类型，默认String   
    	5.  paramType：参数放在哪个地方

   ​            · header --> 请求参数的获取：@RequestHeader

   ​            · query --> 请求参数的获取：@RequestParam

   ​            · path（用于restful接口）--> 请求参数的获取：@PathVariable

   ​            · body（不常用）

   ​            · form（不常用）    

7. ```@ApiParam```用于方法，参数，字段说明；
    表示对参数的添加元数据（说明或是否必填等）

8. ```@PostMapping```：只处理post类型的http请求

