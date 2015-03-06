title: log4j常见配置
date: 2015-01-15 15:27:41
categories: java
tags: log4j配置
---

##xml类型的配置
Log4j配置文件实现了输出到控制台、文件、固定大小文件、发送日志邮件、输出到数据库日志表、自定义标签等全套功能。 

    <?xml version="1.0" encoding="UTF-8" ?>  
    <!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">  
      
    <log4j:configuration>  
      
     <!--自定义输出到控制台-->
     <appender name="CONSOLE" class="org.apache.log4j.ConsoleAppender">
      <layout class="org.apache.log4j.PatternLayout">  
       <param name="ConversionPattern"  
        value="%d - %c -%-4r [%t] %-5p %x - %m%n" />  
      </layout>  
        
      <!--限制输出级别-->  
      <filter class="org.apache.log4j.varia.LevelRangeFilter">  
       <param name="LevelMax" value="ERROR"/>  
       <param name="LevelMin" value="TRACE"/>  
      </filter>  
     </appender>  
       
     <!--输出到文件-->
     <appender name="FILE" class="org.apache.log4j.FileAppender">  
      <param name="File" value="log4j.log"/>  
      <layout class="org.apache.log4j.PatternLayout">  
       <param name="ConversionPattern"  
        value="%d - %c -%-4r [%t] %-5p %x - %m%n" />  
      </layout>  
     </appender>    
     
     <!--输出到数据库-->
     <appender name="DATABASE" class="org.apache.log4j.jdbc.JDBCAppender">  
      <param name="URL" value="jdbc:oracle:thin:@192.168.0.59:1521:oanet"/>  
      <param name="driver" value="oracle.jdbc.driver.OracleDriver"/>  
      <param name="user" value="user"/>  
      <param name="password" value="password"/>      
      <layout class="org.apache.log4j.PatternLayout">  
       <param name="ConversionPattern" 
       value="INSERT INTO hdczoa.LOG4J(stamp,thread,info_level,class,message) VALUES ('%d', '%t', '%p', '%c', %m)" />  
      </layout>  
     </appender>  
       
     <!-- 发邮件（只有ERROR时才会发送！） -->  
     <appender name="MAIL"  class="org.apache.log4j.net.SMTPAppender">  
      <param name="threshold" value="debug" />  
      <!-- 日志的错误级别  
       <param name="threshold" value="fatal"/>  
      -->  
      <!-- 缓存文件大小，日志达到512K时发送Email -->  
      <param name="BufferSize" value="512" /><!-- 单位K -->  
      <param name="From" value="test@163.com" />  
      <param name="SMTPHost" value="smtp.163.com" />  
      <param name="Subject" value="juyee-log4jMessage" />  
      <!--收件人-->
      <param name="To" value="reciever@163.com" />
      <!--发件人登陆邮箱服务器-->
      <param name="SMTPUsername" value="user" />  
      <param name="SMTPPassword" value="password" />  
      <layout class="org.apache.log4j.PatternLayout">  
       <param name="ConversionPattern"  
        value="%-d{yyyy-MM-dd HH:mm:ss.SSS} [%p]-[%c] %m%n" />  
      </layout>  
     </appender>  
      
     <appender name="ASYNC" class="org.apache.log4j.AsyncAppender">  
      <param name="BufferSize" value="256" />  
      <appender-ref ref="DATABASE" />  
     </appender>  
       
     <!--通过<logger></logger>的定义可以将各个包中的类日志输出到不同的日志文件中-->  
     <logger name="packageName" additivity="false">     
            <level value="WARN" />     
            <appender-ref ref="CONSOLE" />     
        </logger>  
      
     <!--通过<category></category>的定义可以将各个包中的类日志输出到不同的日志文件中-->  
     <category name="com.litt3">     
        <level value="DEBUG" />   
           <appender-ref ref="CONSOLE" />  
           <appender-ref ref="MAIL" />  
      </category>  
       
     <root>  
      <priority value="debug" />  
      <appender-ref ref="CONSOLE" />  
      <appender-ref ref="FILE" />  
     </root>  
      
      
    </log4j:configuration>  

--- 

##属性文件配置
Log4j配置文件实现了输出到控制台、文件、固定大小文件、发送日志邮件、输出到数据库日志表、自定义标签等全套功能。 

    log4j.rootLogger=DEBUG,console,dailyFile,im 
    log4j.additivity.org.apache=true 
### 控制台(console) 
    log4j.appender.console=org.apache.log4j.ConsoleAppender 
    log4j.appender.console.Threshold=DEBUG 
    log4j.appender.console.ImmediateFlush=true 
    log4j.appender.console.Target=System.err 
    log4j.appender.console.layout=org.apache.log4j.PatternLayout 
    log4j.appender.console.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n 

### 日志文件(logFile) 
    log4j.appender.logFile=org.apache.log4j.FileAppender 
    log4j.appender.logFile.Threshold=DEBUG 
    log4j.appender.logFile.ImmediateFlush=true 
    log4j.appender.logFile.Append=true 
    log4j.appender.logFile.File=log.log4j 
    log4j.appender.logFile.layout=org.apache.log4j.PatternLayout 
    log4j.appender.logFile.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n 
### 指定大小文件(rollingFile) 
`文件大小到达指定尺寸的时候产生一个新的文件`

    log4j.appender.rollingFile=org.apache.log4j.RollingFileAppender 
    log4j.appender.rollingFile.Threshold=DEBUG 
    log4j.appender.rollingFile.ImmediateFlush=true 
    log4j.appender.rollingFile.Append=true 
    log4j.appender.rollingFile.File=log.log4j 
    #文件达到多大开始重新开始记录新文件
    log4j.appender.rollingFile.MaxFileSize=200KB 
    log4j.appender.rollingFile.MaxBackupIndex=50 
    log4j.appender.rollingFile.layout=org.apache.log4j.PatternLayout 
    log4j.appender.rollingFile.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n 
    
### 每天产生一个文件(dailyFile) 
    log4j.appender.dailyFile=org.apache.log4j.DailyRollingFileAppender 
    log4j.appender.dailyFile.Threshold=DEBUG 
    log4j.appender.dailyFile.ImmediateFlush=true 
    log4j.appender.dailyFile.Append=true 
    log4j.appender.dailyFile.File=log.log4j 
    log4j.appender.dailyFile.DatePattern='.'yyyy-MM-dd 
    log4j.appender.dailyFile.layout=org.apache.log4j.PatternLayout 
    log4j.appender.dailyFile.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n 

### 发送日志到指定邮件 
    log4j.appender.mail=org.apache.log4j.net.SMTPAppender 
    log4j.appender.mail.Threshold=FATAL 
    log4j.appender.mail.BufferSize=10 
    log4j.appender.mail.From = xxx@mail.com 
    log4j.appender.mail.SMTPHost=mail.com 
    log4j.appender.mail.Subject=Log4J Message 
    log4j.appender.mail.To= xxx@mail.com 
    log4j.appender.mail.layout=org.apache.log4j.PatternLayout 
    log4j.appender.mail.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n 
### 应用于数据库 
    log4j.appender.database=org.apache.log4j.jdbc.JDBCAppender 
    log4j.appender.database.URL=jdbc:mysql://localhost:3306/test 
    log4j.appender.database.driver=com.mysql.jdbc.Driver 
    log4j.appender.database.user=root 
    log4j.appender.database.password= 
    log4j.appender.database.sql=INSERT INTO LOG4J (Message) VALUES('=[%-5p] %d(%r) --> [%t] %l: %m %x %n') 
    log4j.appender.database.layout=org.apache.log4j.PatternLayout 
    log4j.appender.database.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n 

### 自定义Appender 
    log4j.appender.im = net.cybercorlin.util.logger.appender.IMAppender 
    log4j.appender.im.host = mail.cybercorlin.net 
    log4j.appender.im.username = username 
    log4j.appender.im.password = password 
    log4j.appender.im.recipient = 
    log4j.appender.im.layout=org.apache.log4j.PatternLayout 
    log4j.appender.im.layout.ConversionPattern=[%-5p] %d(%r) --> [%t] %l: %m %x %n 
