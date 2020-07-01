# ssm整合
1. 导入jar包：
   - spring
   - springMVC
   - mybatis
   - 第三方支持：log4j，pageHelper，aspectJ，jackson，jstl
2. 搭建springMVC
   1. web.xml:
      1. DispatcherServlet
      2. HiddenHttpMethodFilter
      3. characterEncodingFilter
    2. springMVC.xml
       1. 扫描控制层组件
       2. 视图解析器
       3. Default Servlet
       4. MVC驱动
       5. (可选)MultipartResolver 拦截器
3. 整合springMVC和spring 
    1. web.xml:
        1. contextLoaderListener
        2. context-param
    2. spring.xml
        1. 扫描组件(排除控制层)
        2.  事务管理器
4. 搭建mybatis
    1. 核心配置文件
    2. mapper接口和mapper.xml映射文件
5. spring整合mybatis
    1. spring.xml
        1. properties文件的引入
        2. DataSource数据源配置
        3. 事务管理器
        4. 开启事务驱动
        5. SqlSessionFactoryBean：管理sqlSession
        6. MapperScannerConfigurer 
6. REST CRUD
    1. 查询+分页
    2. 修改(form)