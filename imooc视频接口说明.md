**```/video/upload```：上传视频接口**

参数说明：

```java
userId：用户ID
bgmId：背景音乐id
videoSeconds：背景音乐播放长度
videoWidth：视频宽度
videoHeight：视频高度
desc：视频描述（非必须，可为空）
```

1. 判断userId是否为空，若为空返回错误信息："用户id不能为空..."（```StringUtils.isBlank(userId)```）

2. 获取小程序端上传的视频文件名称(```file.getOriginalFilename```)（例：abc.mp4），并截取出文件名（```fileName.split```）

3. 定义小程序上传视频文件的本地存储位置和视频封面截图存储位置，并创建文件夹（```outFile.getParentFile().mkdirs()```）

4. 定义视频文件和视频截图保存到数据库的相对路径和文件本地最终保存路径

   ```java
   String uploadPathDB = "/" + userId + "/video";
   String coverPathDB = "/" + userId + "/video";
   finalVideoPath = FILE_SPACE + uploadPathDB + "/" + fileName;
   ```

5. 将小程序上传的视频文件复制到本地文件（```IOUtils.copy(inputStream, fileOutputStream)```）

6. 判断bgmId是否为空，不为空则从数据库中查询该bgmId对应的记录(```表bgm```)获取到bgm在本地的路径，利用ffmpeg将背景音乐和上传的视频文件合成一个新的带有背景音乐的视频

   ```java
   实际上是调用命令行，例如：
   ffmpeg.exe -i 苏州大裤衩.mp4 -i bgm.mp3 -t 7 -y 新的视频.mp4
   ```

7. 利用ffmpeg从视频文件截图，默认截取第一秒图像

   ```java
   ffmpeg -ss 00:00:01 -y -i test.mp4 -vframes 1 image.jpg
   ```

8. 保存视频信息到数据库(表video)

   ```java
   若小程序端请求的videoWidth为空，则默认为200 
   if (StringUtils.isEmpty(videoWidth) ||"0".equals(videoWidth)){ videoWidth = "200";}
   若小程序端请求的videoHeight为空，则默认为400
   if (StringUtils.isEmpty(videoHeight) || "0".equals(videoHeight)){ videoHeight= "400";}
   ```

   

**```/video/uploadCover```上传封面接口**

**```/video/showAll```他人主页（我的tab）、我的主页（我的tab）、小程序主页和搜索查询视频列表**

参数说明

```java
userId: 用户id 
isSaveRecord（此参数主要用于热词搜索时）:
1 - 需要保存
0 - 不需要保存 ，或者为空的时候
page：当前页（默认从第一页开始查询）
pageSize：每页数据量（默认为5条数据）
```

1. 小程序端首页加载视频时传入page和isSaveRecord 从数据库中获取全部视频数据，利用分页组件```PageHelper```将数据返回给小程序

   ```java
   这里查询用到了mybatis操作数据库
     <select id="queryAllVideos" resultMap="BaseResultMap" parameterType="String">
     select v.*,u.face_image as face_image,u.nickname as nickname from videos v
     left join users u on u.id = v.user_id
     where 1 = 1 <if test=" videoDesc != null and videoDesc != '' "> and v.video_desc like '%${videoDesc}%' </if><if test=" userId != null and userId != '' ">  and v.user_id = #{userId}</if>and v.status = 1 order by v.create_time desc </select>
   语句解释：
<if test=" videoDesc != null and videoDesc != '' "> and v.video_desc like '%${videoDesc}%' </if>   
   如果外部传入的videoDesc（视频描述信息，表中字段为video_desc）不为空，则根据videoDesc值从表videos模糊搜索数据（v.video_desc like '%${videoDesc}%'）
   <if test=" userId != null and userId != '' ">  and v.user_id = #{userId}</if>
   如果外部传入的userId（用户ID，表中字段为user_id）不为空，则根据userId值从表videos搜索数据（v.user_id = #{userId}）
   查询的逻辑：
   首页默认为获取表中全部视频数据
   视频搜索页默认根据videoDesc（视频描述信息，表中字段为video_desc）获取视频数据
   ```
   

**/video/showMyFollow 我关注的人发的视频**（用于小程序他人主页和我的主页）

参数说明：

```java
userId：用户ID
page：当前页（默认从第一页开始查询）
```

1. 小程序端查看他人主页和我的主页时点击关注```Tab```时传入userId和page，根据userId从数据库中获取到全部视频数据，利用分页组件```PageHelper```将数据返回给小程序

   ```java
   这里查询涉及多表查询
   <select id="queryMyFollowVideos" resultMap="BaseResultMap" parameterType="String">
   		select v.*,u.face_image as face_image,u.nickname as nickname from videos v 
   		left join users u on v.user_id = u.id
   		where 
   			v.user_id in (select uf.user_id from users_fans uf where uf.fan_id = #{userId})
   			and v.status = 1
   			order by v.create_time desc
   	</select>
    查询的逻辑：先看where语句：先从表users_fans根据userId（外部传入的userId,关注者，表中字段为 fan_id )获取到被关注者userId(表中字段为user_Id)，然后再用被关注者userId(表中字段为user_Id)从表videos查询获取视频数据
   ```

**/video/showMyLike 我收藏(点赞)过的视频列表** （用于小程序他人主页和我的主页）

参数说明

```java
userId：用户ID
page：当前页（默认从第一页开始查询）
pageSize：每页数据量（默认为6条数据）
```

1. 小程序端查看他人主页和我的主页时点击收藏```Tab```时传入userId、page、pageSize，根据userId从数据库中获取到全部视频数据，利用分页组件```PageHelper```将数据返回给小程序

   ```java
   这里查询涉及多表查询	
   <select id="queryMyLikeVideos" resultMap="BaseResultMap" parameterType="String">
   		select v.*,u.face_image as face_image,u.nickname as nickname from videos v 
   		left join users u on v.user_id = u.id
   		where 
   			v.id in (select ulv.video_id from users_like_videos ulv where ulv.user_id = #{userId})
   			and v.status = 1
   			order by v.create_time desc
   	</select>
    查询的逻辑：先看where语句：先从表users_like_videos根据userId（外部传入的userId,用户ID，表中字段为 user_Id )获取到对应userId的videoId(表中字段为video_id)，然后再用video_id从表videos(表中字段为id，主键)查询获取视频数据
   ```

**/video/hot 搜索页展示搜索热点**

无需参数

1. 小程序端请求该接口，从数据库中获取热词记录返回给小程序端

   ```java
   select content from search_records group by content order by count(content) desc
   查询的逻辑：从表search_records获取content字段，返回全部记录
   ```

**/video/userLike 用户点赞**

参数说明

```java
userId：用户ID
videoId：视频ID
videoCreaterId：视频上传者ID
```

1. 将userId和videoId插入到表users_like_videos中

   ```usersLikeVideosMapper.insert(ulv);```

2. 更新数据库中字段：

   1. 根据videoId将表videos的字段like_counts（视频喜欢数量）累加

   ```java
   update videos set like_counts=like_counts+1 where id=#{videoId}
   ```

   2. 根据videoCreaterId将表users的字段receive_like_counts（受喜欢数量）累加

   ```java
   update users set receive_like_counts=ifnull(receive_like_counts,0)+1 where id=#{userId}
   ```

**/video/userUnLike 用户取消点赞**

参数说明：

```java
userId：用户ID
videoId：视频ID
videoCreaterId：视频上传者ID
```

1. 根据userId和videoId删除表users_like_videos中对应的数据

   ```usersLikeVideosMapper.deleteByExample(example);```

2. 更新数据库中字段：

   1. 根据videoId将表videos的字段like_counts（视频喜欢数量）累减

   ```java
   update videos set like_counts=like_counts-1 where id=#{videoId}
   ```

   2. 根据videoCreaterId将表users的字段receive_like_counts（受喜欢数量）累减

   ```java
   update users set receive_like_counts=ifnull(receive_like_counts,0)-1 where id=#{userId}
   ```

**/video/saveComment 评论**

参数说明：

