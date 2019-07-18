# spring-cloud-sofa
 背景

公司没有自研框架的弊端

       1 技术形成闭环
   
    公司发展业务线，新增一条业务线，新接到一个项目，招聘一个高级工程师，然后在这个高级工程师的基础上
    组建了一只团队。这个团队所使用的技术栈不同于公司其他业务线和项目,当这个高级工程师离职以后，基本情况
    就是无法维护不可控
    
     2 直接使用开源框架导致公司业务线，项目做技术升级改造基本变为不可能的事情
     
     公司的业务线，项目如果直接使用开源框架，并且业务中存在大量的三方包。当这个开源框架技术落后。
     存在问题。需要替换其他技术的时候，改造成本太大。
       
    3  盈利效果低
        
        公司的业务线，和项目 如何才能盈利，快，并且投入人员少。处于业务线和项目位置的时候，开发
        最专注的点应该是业务，一些通用的技术难点如果都放在业务线上和项目上去做，时间成本太大
        而且给业务方很不好的体验
   
   
   
公司自研框架的好处 
  
  盈利角度

    1  代码利用得到大量的提升
   
       相似的功能 不存在重复开发的作用。业务上如果碰到相似的应用场景直接复用
     
     2   降低高级工程师的数量
      
        拓展业务线时，基础的架构和技术难点性问题由公司自研框架解决，并且新招入的员工水平能力不用太高
        基础研发部的人员并且为他们进行基础框架培训，让他们快速上手。以解决业务为出发点
       
  项目质量角度
    
      1 基础框架负责与第三方框架进行对接工作，当第三方框架存在问题，或需要更换的时候。无须牵动业务部门
      
      2 代码质量得到提高，公司各个业务线的代码风格一致，技术栈相同，便于维护
      
  提高公司形象
  
        1 大型的技术公司都有自研的技术框架，自研技术框架在公司拓展业务线，提升公司在客户心中的形象
        
        上有很大的帮助
        
从构建一个  开发平台业务线为例子，讲如何从该框架去拓展一条业务线
    
    
  




       


   
  
  


总体模块说明
       
       spring-cloud-sofa-center        ---------            注册中心       
       
       spring-cloud-sofa-gate          ---------            网关 
       
       spring-cloud-sofa-tracer        ---------            分布式链路追踪 
       
       spring-cloud-sofa-tracer        ---------            分布式框架鉴权
       
       spring-cloud-sofa-drm           ------------         分布式DRM开关
       
       spring-cloud-sofa-file          ------------         分布式文件系统
       
       spring-cloud-sofa-message       ------------          scokcet服务器
       
       spring-cloud-sofa-tari          ------------          缓存服务
       
       spring-cloud-sofa-sidecar      ---------------        第三方接入系统
       
       spring-cloud-sofa-todo        ---------------         代办中心
       
       spring-cloud-sofa-common      ----------------        基础组件基础包
       

spring-cloud-sofa-center 

     1 分布式系统的注册中心，用于存储各个服务的基本信息   技术底层使用的是spring-cloud-eurka。

     2  注册中心部署上分为单节点部署，集群部署俩种方式。这两种方式不同之处，对于框架使用者来说只是编写配置文件

        2.1  单节点部署方式配置文件编写详解
            
            server:
            
              port: 8761 (指定注册中心的端口号，用于其他系统连接时使用)
            
            eureka:
            
              instance:
            
                hostname: localhost (配置当前服务器ip地址)
            
              client:
            
                    registerWithEureka: false  (是否将注册中心也进行注册)
            
                    fetchRegistry: false       （是否获取注册中心的注册信息）
            
                    service-url:                （注册中心的连接地址）
            
                        defaultZone :  http://${eureka.instance.hostname}:${server.port}/eureka/
          2.2 集群配置
          
             此处待补充
             
    3  启动注册中心               
    
         3.1 开发环境
    
             启动 CenterApplication main 方法即可启动注册中心
    
         3.2 正式环境
    
              打成jar包后，java -jar 即可启动
                  
             
   spring-cloud-sofa-gate
   
     分布式系统的网关，提供对外系统的服务能力，目前底层使用的框架为 spring-cloud-gateway.网关核心能力是提供外部系统调用能力，
  
     网关状态码已经进行明确的规定，如下图
       
      code（返回码）	msg（返回码描述）	sub_code（明细返回码）	sub_msg（明细返回码描述）	解决方案
        
        10000            服务调用成功
                      
       20000           	服务不可用
       
       20001           	授权权限不足
       
       40001           缺少必选参数
       
       40004           业务处理失败
       
       40006           权限不足	
       
     网关使用的的路由转发策略是
         Path Route  Predicate Factory，配置的原则和nginx 的负载均衡配置相同
         spring:
           cloud:
             gateway:
               routes:
                 - id:  spring-cloud-sofa-auth        服务名称，注册在中心的服务
                   uri: lb://spring-cloud-sofa-auth   固定格式 lb://服务名称，注册在注册中心的服务
                   predicates:  
                     - Path= /api/auth/**             何种url会被转发
         
 spring-cloud-drm
        
        分布式系统的开关，主要是用于对分布式系统的一些场景可以进行动态的开启，和关闭
        
        使用的时候，在drm的地方注册开关code,开关名称，开关状态 0 为关闭，1为开启
        
        并且提供了一个fegin 的接口，参数为开关code ,返回值为 true 开关开启， false 开关关闭
 
spring-cloud-sofa-api      
        
        服务化接口的定义位置，A系统想调用C系统的服务，只需要C系统在此处定义自己的接口调用格式。A系统引入此某块
        
        并调用C系统定义的接口 即实现 服务化接口的调用过程       