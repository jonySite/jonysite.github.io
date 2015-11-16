# DataLoader Command-Line Interface

@(Data Loader)[Salesforce|数据集成|命令行|批处理|Windows|]

###**Data Loader** <span style='font-size:16px;font-weight:normal;line-height:16px;'>是一个批量数据导入导出的`客户端`应用，使用它可以对SF数据记录进行插入、更新、删除和导出.</span>

#####数据操作
* 当进行数据导入时，DataLoader从文件或者数据库中 读取并加载数据
* 当进行数据导出时，DataLoader输出 CSV文件 或者导入到 数据库 中
#####使用方式
- <del style='color:gray'><i>用户接口GUI</i></del>
- 命令行(Windows only)
#####数据源
+ 数据文件（CSV文件）
+ 数据库（Command only）

#####命令行接口
: **简述**
>当使用`命令行`时，你需要配置**数据源**和**数据映射关系**以及**操作**，以便设置数据并加载程序进行自动处理。
*<span style='color:gray;font-size:14px'> 该配置可以通过DataLoader GUI来进行设置。</span>*


: **1.安装目录文件结构**
``` Bash
fore.com
  └─Data Loader [主目录]
      ├─bin [batch命令文件]
      │  ├─▬▬▬encrypt.bat
      │  └─▬▬▬process.bat
      ├─▬▬▬dataloader-33.0.0-uber.jar
      ├─Java [java工作目录]
      │  ├─bin
      │  │  ├─▬▬▬...
      │  │  ├─▬▬▬java.exe
      │  │  └─▬▬▬...
      │  ├─Data
      │  ├─lib
      │  │  ├─▬▬▬...
      │  │  ├─▬▬▬rt.jar
      │  │  └─▬▬▬...
      │  └─Other
      ├─licenses
      └─samples [配置样本目录]
          ├─conf [配置文件目录]
          │  ├─▬▬▬database-conf.xml
          │  ├─▬▬▬process-conf.xml
          │  └─▬▬▬opportunityInsertMap.sdl
          ├─data [数据源目录]
          │  └─▬▬▬opportunityData.csv
          └─status [日志目录]
              └─▬▬▬success0413060133.csv
```
&nbsp;
:  **2.设计流程**
:  - <span style='font-size:15px;'>配置DataLoader，创建`process-Xxx.bat`批处理文件</span>
:  - <span style='font-size:15px;'>添加任务`脚本`到Windows执行任务计划</span>
:  - <span style='font-size:15px;'>运行batch并备份数据文件，有异常则发送邮件通知</span>
<p>&nbsp;</p>
: **3.用户系统配置**
1. <span style='font-size:15px;'>复制`sample/conf`到主目录`Data Loader`中</span>
2. <span style='font-size:15px;'>创建配置文件`config.properties`并进行编辑如下</span>
```xml
sfdc.endpoint=https://test.salesforce.com
sfdc.username=boxadmin@natr.com.box
sfdc.password=aae9e2c86792c848fd3107685cabd3d6
``` 
&nbsp;
: - &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`sfdc.password`<span style='font-size:15px;'>获取方法为进入`bin`目录中执行以下命令</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style='font-size:15px;'></span>
```bash
bin> encrypt -e password
```
&nbsp;
: **4.批操作核心配置**<br>

: + <span style='font-size:15px;'>打开`process-conf.xml`文件编辑配置</span>
``` xml
<bean id="SObjectOperation"
          class="com.salesforce.dataloader.process.ProcessRunner"
          singleton="false">
        <description>updates or inserts or upsert data into SFDC from csv local file."</description>
        <property name="name" value="SObjectInsert"/>
        <property name="configOverrideMap">
            <map>
                <entry key="sfdc.entity" value="SObject"/>
                <entry key="process.operation" value="Operation"/>
                <entry key="process.mappingFile" value="SObjectOperation.sdl"/>
                <entry key="dataAccess.name" value="SObject_Operation.csv"/>
                <entry key="process.outputSuccess" value="SObject_Operation_Success.csv"/>
                <entry key="process.outputError" value="SObject_Operation_Error.csv"/>
                <!-- <entry key="" value=""/> -->
            </map>
        </property>
    </bean>
```
&nbsp;

: + <span style='font-size:15px;'>字段列映射配置</span>
<span style='font-size:14px;'>复制`conf`中文件`opportunityInsertMap.sdl`编辑并进行重命名</span>

: | <span style='font-size:14px;'>数据流向</span>|<span style='font-size:14px;'>配置原则 </span>|
| :-------- | --------:|
| <span style='font-size:14px;'>数据流向为CSV到SFDC</span>| <span style='font-size:14px;'>csv列字段名=SFDC字段API名</span>|
|<span style='font-size:14px;'>数据流向从SFDC到CSV</span>|<span style='font-size:14px;'>csv列字段名=SFDC字段API名</span>|
<i style='font-size:14px;color:gray;'>*该配置中勿要出现中文字符,该配置为增量配置</i>
:  **5.邮件发送配置**

:  > <span style='font-size:15px;'>邮件是使用SendEmail程序对其命令行进行操作，详见[sendEmail](http://caspian.dotconf.net/menu/Software/SendEmail/)官网</span>

:  - <span style='font-size:15px;'>邮件**模板**配置</span>
<span style='font-size:15px;'>在上述邮件程序目录中的文件`mail`为邮件模板，使用文本编辑器打开即可修改（注，其中的${}部分请勿修改），注意格式必须符合HTML规范</span>

:  -  <span style='font-size:15px;'>邮件**程序**配置</span>
<span style='font-size:15px;'>邮件调用程序为邮件命令目录下的`sendMail.bat`文件，使用文本编辑器进行编辑，如下</span>
``` bash
sendEmail -o message-charset="utf-8" ^
		  -o tls=yes ^
		  -o message-content-type=html ^
		  -o message-header="X-Priority: 1" ^
		  -f "DataLoader<your_mail_account@163.com>" ^
		  -t jony.fang@celnet.com.cn ^
		  -s smtp.163.com ^
		  -o username=your_mail_account@163.com ^
		  -o password=mailPassword ^
		  -u "DataLoader数据导入异常" ^
		  -o message-file=%1 ^
		  -a %2 %3 %4
```
&nbsp;

: <span style='font-size:15px;'>其中，主要配置参数为</span>
: <span style='font-size:15px;'>`-f "DataLoader<发送方邮箱地址>"` &nbsp;&nbsp;此处填入发送方邮箱地址</span>
: <span style='font-size:15px;'>`-t [接收方邮箱地址]` &nbsp;&nbsp;此处填写接收方邮箱地址</span>
: <span style='font-size:15px;'>`-s [发送方邮箱服务器地址]` &nbsp;&nbsp;此处填入发送方邮箱服务器地址 (<i style='color:gray;'>如163邮箱服务器地址为smtp.163.com</i>)</span>
: <span style='font-size:15px;'>`-o username=[发送方邮箱地址] `<br>`-o password=[发送方邮箱密码]`</span>
: <span style='font-size:15px;'>上面两处为发送方邮箱用户名和密码</span>
: <span style='font-size:15px;color:gray;'>
*如果需要发送多个用户，请进入**sendEmail官网**查看用法
*在任务批处理文件中，调用该sendMail程序用法如下
`sendMail.bat mail Insert.csv Error.csv log.txt`
其中该命名的四个参数依次为发送的邮件文件、数据源，错误日志文件和控制台日志文件
</span>
#&nbsp;
:  **6.批处理文件配置**
> 批处理文件对应每个SObject每一种操作，如Account的insert（导入）对应着一个bat文件，在`Windows`的 **任务计划**中对应一个任务 

: - <span style='font-size:15px；'>创建一个batch文件，编写程序如下：</span>

``` batch
    @echo off
    
    REM 配置该任务的工作目录，如果不存在则创建
    set WORK_DIR=..\data\work\Bonus_Bal__c_Insert
    if exist %WORK_DIR% ( 
    echo "Directory is exist" 
     ) else ( 
    	echo "make the directory"
    	md %WORK_DIR%
     )
     
    REM set process name
    set PROCESS_OPTION=process.name=Bonus_Bal__c_Insert
    
    REM 执行数据操作
    ..\Java\bin\java.exe -cp ..\dataloader-uber.jar -Dsalesforce.config.dir=../conf com.salesforce.dataloader.process.ProcessRunner %PROCESS_OPTION% > %WORK_DIR%/log.txt
    
    REM 设置备份文件夹名称
    set bakDir=%date:~0,4%%date:~5,2%%date:~8,2%%time:~0,2%%time:~3,2%%time:~6,2%
    ::pause
    
    REM 创建备份文件夹
    mkdir ..\backup\%bakDir%-Bonus_Bal__c_Insert
    
    REM 复制源数据文件和日志文件到备份文件夹下
    @copy ..\data\Bonus_Bal__c_Insert.csv ..\backup\%bakDir%-Bonus_Bal__c_Insert
    @copy ..\data\work\Bonus_Bal__c_Insert ..\backup\%bakDir%-Bonus_Bal__c_Insert
    
    REM 分析错误1 by log file
    findstr /C:"doneSuccess" ..\backup\%bakDir%-Bonus_Bal__c_Insert\log.txt
    set isSuccess = %errorlevel%
    findstr /C:"The operation has fully completed." ..\backup\%bakDir%-Bonus_Bal__c_Insert\log.txt
    set isSuccess2 = %errorlevel%
    if isSuccess EQU 0 if isSuccess2 EQU 0(::sucess then get get related info
    	echo "处理完成……"
    	goto csvAnalysis
    ) else (
    	goto sendMail
    )
    
    :csvAnalysis
    REM find isError by csv log file
    for /f %%a in (' find /c /v "" ^<..\data\work\Bonus_Bal__c_Insert\Bonus_Bal__c_Insert_Error.csv ') do (set n=%%a ) 
    IF %n% GTR 1 (goto sendMail) ELSE goto end
    
    REM 发送异常邮件通知
    :sendMail
    REM 复制邮件模板
    @copy ..\mail\mail ..\backup\%bakDir%-Bonus_Bal__c_Insert
    REM 生成邮件
    "..\Java\bin\java" -classpath ..\bin EmailGen ^
    ..\backup\%bakDir%-Bonus_Bal__c_Insert\mail ^
    ..\backup\%bakDir%-Bonus_Bal__c_Insert\mail2 ^
    用户 "Insert" Bonus_Bal__c ^
    %bakDir:~0,4%年%bakDir:~4,2%月%bakDir:~6,2%号^&nbsp;%bakDir:~8,2%时%bakDir:~10,2%分%bakDir:~12,2%秒 ^
    Bonus_Bal__c_Insert.csv ^
    Bonus_Bal__c_Insert_Error.csv  ^
    log.txt  ^
    %bakDir%-Bonus_Bal__c_Insert
    REM delete
    del ..\backup\%bakDir%-Bonus_Bal__c_Insert\mail
    ren ..\backup\%bakDir%-Bonus_Bal__c_Insert\mail2 mail
    
    REM 发送邮件通知
    cd ..
    bin\sendMail.bat ^
    backup/%bakDir%-Bonus_Bal__c_Insert/mail ^
    backup/%bakDir%-Bonus_Bal__c_Insert/Bonus_Bal__c_Insert.csv ^
    backup/%bakDir%-Bonus_Bal__c_Insert/Bonus_Bal__c_Insert_Error.csv ^
    backup/%bakDir%-Bonus_Bal__c_Insert/log.txt
    
    exit
    
    :end
    echo data process done
    exit
```
&nbsp;
:  **7.添加batch到任务**
: 
<span style='font-size:15px;'>在`Windows`中的**任务计划程序**中设定执行脚本（processXXX.bat）和执行频率</span>
![任务执行计划程序](http://markdown.tw/images/208x128.png)

---------
<i style='color:gray;font-size:12px;float:right;'>Powered by celnet</i>




