### jdbc存在的问题

1. 数据库配置信息存在硬编码问题
2. 频繁创建释放数据库连接
3. sql语句、设置参数、获取结果集参数均存在硬编码问题
4. 需要手动封装返回结果集，较为繁琐

### 解决思路
1. 配置信息提取到配置文件
2. 连接池
3. 反射、内省

### 方案
**使用端（具体项目，引入自定义框架jar包）**
1. 为了解决硬编码问题，引入配置文件，配置信息和sql信息分别创建配置文件
   - sqlMapConfig.xml：存放数据库配置
   - mapper.xml：存放sql配置信息
   
**自定义持久层框架（本质是对JDBC代码进行封装）**
1. 加载配置文件：根据配置文件路径，加载配置文件为字节流，存储在内存中
   - 创建Resource类，方法：inputStream getResourceAsStream(String path)
2. 创建两个javaBean：存放的就是配置文件解析出来的内容
   - Configuration(核心配置类)：存放的就是sqlMapConfig.xml解析出来的内容
   - MappedStatement(映射配置类)：存放的就是mapper.xml解析出来的内容
3. 解析xml配置文件
   - 使用dom4j解析配置文件，将内容解析到对应容器对象中
4. 创建sqlSessionFactory接口及实现类DefaultSqlSessionFactory（工厂模式）
   - 生产sqlSession(会话对象)，方法：openSession()
5. 创建sqlSession接口及实现类DefaultSession
   - 定义对数据库crud操作：
     - select()
     - selectOne()
     - update()
     - delete
6. 创建Executor接口及实现类SimpleExecutor(对第五步直接执行JDBC的操作进行进一步封装)
   - query(Configuration,MappedStatement,Object...params)，执行的就是JDBC代码

### 实现
**使用端**
1. sql的唯一标识：用namespace.id来组成：statementId

**自定义持久层框架**
1. 一个select标签就是一个MappedStatement对象
2. 因为要使用反射或内省去封装返回结果集，所以在使用resultType时，实体类字段和sql字段要一一对应（这也是为什么大部分时候sql字段使用别名的原因）
