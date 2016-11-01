### supervisor 托管 tomcat tomcat 日志分割

1、安装 JDK tomcat
   
     下载 jdk tomcat  
     [root@RS1 ~]# rpm -ivh jdk-8u111-linux-x64.rpm
     [root@RS1 jdk1.8.0_111]# pwd 
     /usr/java/jdk1.8.0_111
     解压下载后的 tomcat
     [root@RS1 ~]# tar xf tomcat-8.tar.gz -C /opt
     
2、安装 supervisor 
   
    easy_install
    [root@RS ~]# wget https://bootstrap.pypa.io/ez_setup.py -O - | python
    安装 supervisor
    easy_install supervisor
    生成配置文件
    [root@RS ~]# echo_supervisord_conf > /etc/supervisord.conf
    
3、supervisor 托管 tomcat

 在 supervisor 中生成 tomcat.conf 配置文件
	
	在 /etc/supervisor.d/ 目录中建立 tomcat.conf 文件
	[program:tomcat8]
	command=/opt/apache-tomcat-8.0.38/bin/supervisor-wrapper.sh
	autostart = true
	autorestart = true
	startsecs = 3
	stdout_logfile = /home/logs/tomcat/supervisor_catalina.out
	stderr_logfile = /home/logs/tomcat/catalina.error.out
	
supervisor-wrapper.sh 脚本

    [root@RS1 bin]# cat supervisor-wrapper.sh
	#!/bin/bash
	# Source: http://serverfault.com/questions/425132/controlling-tomcat-with-supervisor
	function shutdown()
	{
    date
    echo "Shutting down Tomcat"
    unset CATALINA_PID # Necessary in some cases
    $CATALINA_HOME/bin/catalina.sh stop
	}

	date
	echo "Starting Tomcat"
	export JAVA_HOME="/usr/java/jdk1.8.0_111"
	export CATALINA_HOME="/opt/apache-tomcat-8.0.38"
	export CATALINA_BASE="/opt/apache-tomcat-8.0.38"
	export CATALINA_PID=/tmp/$$

	. $CATALINA_HOME/bin/catalina.sh start

	# Allow any signal which would kill a process to stop Tomcat
	trap shutdown HUP INT QUIT ABRT KILL ALRM TERM TSTP

	echo "Waiting for `cat $CATALINA_PID`"
	wait `cat $CATALINA_PID`

4、logrotate	

	在 logrotate.d 目录下建立 tomcat8 日志分割文件
	[root@RS1 ~]# vi /etc/logrotate.d/tomcat8
	[root@RS1 ~]# cat /etc/logrotate.d/tomcat8
	/opt/apache-tomcat-8.0.38/logs/catalina.out {
		copytruncate
		daily
   		rotate 7  #日志文件保留7份
		dateext   #日志备份显示时间
		compress   # 压缩备份的日志
		missingok
   	 	size 1M
	}

    


     
