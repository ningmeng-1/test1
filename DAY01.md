## 项目-学子商城

### 1. 分析

该项目中的数据大致有：用户、收货地址、商品、商品类别、订单、购物车、收藏……

在这些数据中，相对比较好处理的应该是用户数据，而且，用户数据也应该是最优先处理的，否则，后续的某些数据或功能是无法完成的！

大致的处理顺序：用户 > 收货地址 > 商品 > 商品类别 > 购物车 > 订单 > 收藏

然后，需要为每种数据的处理，划分出对应的功能，以用户数据为例，应该有注册、登录、修改密码、修改个人信息等功能。当划出功能时，通常先处理增加，再处理查询，然后处理删除，最后处理修改。

而对应的具体某个功能的开发，应该是先开发持久层，再开发业务层，再开发控制器层，和前端页面。

### 2. 用户-注册

#### 2.1. 用户-注册-持久层

**创建项目**

创建项目，Group Id为`cn.tedu.store`，Artifact Id为`TeduStore`。

关于各配置文件：

- 【web.xml】配置`DispatcherServlet`，是SpringMVC的核心组件，用于接收所有请求，然后分发到各控制器，在配置时，必须确定初始化时加载的Spring配置文件是哪些！配置`CharacterEncodingFilter`，即字符编码过滤器，用于设置接收请求参数时的编码、响应的编码，必须确定所使用的编码。

- 【spring-mvc.xml】应该配置**组件扫描**，扫描目标是控制器类所在的包，虽然该功能可以配置为根级包，但是，不推荐这样做！关于**视图解析器**，使用的`InternalResourceViewResolver`，需要配置前缀和后缀，本次应该把前缀配置为`/`，而不是`/WEB-INF/`，主要是考虑到有HTML、CSS、JS文件，这些文件如果存放在`/WEB-INF/`下是无法被用户访问的！关于**拦截器**，本项目也需要使用登录拦截器，所以，把此前项目中的拦截器类复制到当前项目，并调整配置，主要是拦截器类所在的包！关于**注解驱动**，是固定的配置，每个项目中都添加即可。

- 【spring-service.xml】应该配置组件扫描，同上！

- 【db.properties】重点检查`url`中的数据库名称、`password`的值！

- 【spring-dao.xml】关于加载`db.properties`，通常是固定做法，即使更换项目也无须修改！关于`BasicDataSource`，也是固定做法，无须修改！
关于`MapperScannerConfigurer`，需要检查持久层接口所在的包！关于`SqlSessionFactoryBean`，需要检查持久层映射的XML文件所在的文件夹！

- 【mappers/UserMapper.xml】由于目前是新建的项目，所以，删除整个`<mapper>`节点

**创建数据库**

	CREATE DATABASE tedu_store;

**创建用户数据表**

	CREATE TABLE t_user (
		id INT AUTO_INCREMENT,
		username VARCHAR(16) UNIQUE NOT NULL,
		password CHAR(32) NOT NULL,
		email VARCHAR(32),
		phone VARCHAR(20),
		gender INT,
		avatar VARCHAR(50),
		salt CHAR(36),
		status INT,
		is_delete INT,
		created_user VARCHAR(16),
		created_time DATETIME,
		modified_user VARCHAR(16),
		modified_time DATETIME,
		PRIMARY KEY(id)
	) DEFAULT CHARSET=UTF8;

**创建实体类**

通常，每张数据表都应该有完全对应的实体类，即从属性的数量、数据类型都与数据表中的字段保持一致，当然，在数据库中，是不区分大小写的，所以，字段名中可能包含下划线`_`，而Java代码中，类的属性名称应该没有下划线，所以，名称可能不完全一致。

则创建`cn.tedu.store.entity.User`：

	public class User {

		private Integer id;
		private String username;
		private String password;
		private String email;
		private String phone;
		private Integer gender;
		private String avatar;
		private Integer status;
		private Integer isDelete;
		private String createdUser;
		private Date createdTime;
		private String modifiedUser;
		private Date modifiedTime;

		// 生成所有属性的SET/GET方法
	}

**注：关于实体类，还应该有一些其它操作，由于后续会对数据表作调整，该实体类也会跟着调整，所以，其它操作暂时不处理。**

**分析功能**

关于用户注册，最核心的是向数据表中插入数据，同时，为了保证业务的完整性，还应该有根据用户名查询信息的功能，以便于检查用户名是否被占用，保证“用户名唯一”。

**开发持久层接口**

创建`cn.tedu.store.mapper.UserMapper`接口文件，并添加抽象方法：

	Integer insert(User user);

	User getUserByUsername(String username);

**开发持久层映射**

编辑`src/main/resources/mappers/UserMapper.xml`映射文件，配置以上2个抽象方法对象的节点：

	<!-- 插入用户数据 -->
	<!-- Integer insert(User user) -->
	<insert id="insert"
		parameterType="cn.tedu.store.entity.User"
		useGeneratedKeys="true"
		keyProperty="id">
		INSERT INTO t_user (
			id,
			username,
			password,
			email,
			phone,
			gender,
			avatar,
			status,
			is_delete,
			created_user,
			created_time,
			modified_user,
			modified_time
		) VALUES (
			#{id},
			#{username},
			#{password},
			#{email},
			#{phone},
			#{gender},
			#{avatar},
			#{status},
			#{isDelete},
			#{createdUser},
			#{createdTime},
			#{modifiedUser},
			#{modifiedTime}
		)
	</insert>

	<!-- 根据用户名查询用户数据 -->
	<!-- User getUserByUsername(String username) -->
	<select id="getUserByUsername"
		resultType="cn.tedu.store.entity.User">
		SELECT 
			id,
			username,
			password,
			email,
			phone,
			gender,
			avatar,
			status,
			is_delete AS isDelete
		FROM 
			t_user 
		WHERE 
			username=#{username}
	</select>

为了便于检查问题，应该随时写完随时测试，即在`src\test\java`下创建类执行以上2个功能的单元测试。

#### 2.2. 用户-注册-业务层

创建`cn.tedu.store.service.IUserService`接口，并在接口中声明抽象方法：

	User reg(User user);

	Integer insert(User user);

	User getUserByUsername(String username);

创建`cn.tedu.store.service.impl.UserServiceImpl`类，实现`IUserService`接口，并且使用`@Service("userService")`对类进行注解。

由于需要通过持久层来实现功能，所以，声明全局变量`@Autowired private UserMapper userMapper;`，关于`reg()`方法：

	public User reg(User user) {
		// 根据用户名查询用户信息
		// 判断是否查询到数据
		// 否：用户名可用，执行注册
		// 是：用户名已经被占用，抛出UsernameConflictException
		return null;
	}

由于需要抛出异常，所以，创建`cn.tedu.store.service.ex.ServiceException`，继承自`RuntimeException`，然后创建`cn.tedu.store.service.ex.UsernameConflictException`，继承自`ServiceException`。


**通常，在业务层中的方法应该是通俗易懂、简单易用的！**

#### 2.3. 用户-注册-控制器层

#### 2.4. 用户-注册-前端页面


验证消息

X ------> R --> R -----------------------> Y
"hello"x10000							"?????"
消息摘要	111								消息摘要 111

哈希算法									哈希算法
消息摘要	111								消息摘要 112

SHA-2	MD5

"hello" ----> "123456" --> 存"123456"

同样的原数据，经过摘要算法，得到的摘要信息一定是相同的！

不一样的原数据，可能得到一样的摘要！











