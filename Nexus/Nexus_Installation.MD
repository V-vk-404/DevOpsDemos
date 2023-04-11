# Nexus Installation
Nexus is one a artifact repository which helps to store your build outcomes.  

### Follow this article in **[Youtube](https://www.youtube.com/watch?v=83AGz9huJGo)
### Prerequisites

1. EC2 instance with java 

### Implementation steps 
Install Java
You can install it with the following command:
```sh 
apt-get install openjdk-8-jdk -y
```

Once Java is installed, you can verify the installed version of Java with the following command:
```sh 
java -version
```

Download and setup nexus stable version
```sh 
cd /opt
wget https://download.sonatype.com/nexus/3/nexus-3.51.0-01-unix.tar.gz
tar -zxvf  nexus-3.51.0-01-unix.tar.gz
mv /opt/nexus-3.51.0-01 /opt/nexus
```

As a good security practice, it is not advised to run nexus service as root. so create new user called nexus and grant sudo access to manage nexus services 
```sh 
useradd -M -d /opt/nexus -s /bin/bash -r nexus
# visudo \\ nexus   ALL=(ALL)       NOPASSWD: ALL
# echo "nexus ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/nexus
sudo chown -R nexus:nexus /opt/nexus
```
Next, edit the nexus.vmoptions configuration file and define max memory size:

```sh 
nano /opt/nexus/bin/nexus.vmoptions
```

Set Java max memory size and replaced "../sonatype-work" with "./sonatype-work":
```sh 
-Xms1024m
-Xmx1024m
-XX:MaxDirectMemorySize=1024m

-XX:LogFile=./sonatype-work/nexus3/log/jvm.log
-XX:-OmitStackTraceInFastThrow
-Djava.net.preferIPv4Stack=true
-Dkaraf.home=.
-Dkaraf.base=.
-Dkaraf.etc=etc/karaf
-Djava.util.logging.config.file=/etc/karaf/java.util.logging.properties
-Dkaraf.data=./sonatype-work/nexus3
-Dkaraf.log=./sonatype-work/nexus3/log
-Djava.io.tmpdir=./sonatype-work/nexus3/tmp
```

Open /opt/nexus/bin/nexus.rc file, uncomment run_as_user parameter and set it as following.
```sh 
vi /opt/nexus/bin/nexus.rc
run_as_user="nexus" (file shold have only this line)
```

start the Nexxus service with the following command:
```sh
sudo -u nexus /opt/nexus/bin/nexus start
```
Next, verify the Nexus with the following command:
```sh
tail -f /opt/nexus/sonatype-work/nexus3/log/nexus.log
```
You should get the following output:
```sh 
2021-02-23 12:20:51,839+0000 INFO  [jetty-main-1]  *SYSTEM com.softwarementors.extjs.djn.servlet.DirectJNgineServlet - Servlet GLOBAL configuration: registryConfiguratorClass=
2021-02-23 12:20:51,853+0000 INFO  [jetty-main-1]  *SYSTEM com.softwarementors.extjs.djn.jscodegen.CodeFileGenerator - Creating source files for APIs...
2021-02-23 12:20:52,582+0000 INFO  [jetty-main-1]  *SYSTEM org.sonatype.nexus.siesta.SiestaServlet - JAX-RS RuntimeDelegate: org.sonatype.nexus.siesta.internal.resteasy.SisuResteasyProviderFactory@649a69ca
2021-02-23 12:20:52,611+0000 INFO  [jetty-main-1]  *SYSTEM org.jboss.resteasy.plugins.validation.i18n - RESTEASY008550: Unable to find CDI supporting ValidatorFactory. Using default ValidatorFactory
2021-02-23 12:20:53,811+0000 INFO  [jetty-main-1]  *SYSTEM org.sonatype.nexus.siesta.SiestaServlet - Initialized
2021-02-23 12:20:53,817+0000 INFO  [jetty-main-1]  *SYSTEM org.sonatype.nexus.repository.httpbridge.internal.ViewServlet - Initialized
2021-02-23 12:20:53,852+0000 INFO  [jetty-main-1]  *SYSTEM org.eclipse.jetty.server.handler.ContextHandler - Started o.e.j.w.WebAppContext@7a65358d{Sonatype Nexus,/,file:///opt/nexus/public/,AVAILABLE}
2021-02-23 12:20:53,883+0000 INFO  [jetty-main-1]  *SYSTEM org.eclipse.jetty.server.AbstractConnector - Started ServerConnector@764a4935{HTTP/1.1, (http/1.1)}{0.0.0.0:8081}
2021-02-23 12:20:53,884+0000 INFO  [jetty-main-1]  *SYSTEM org.eclipse.jetty.server.Server - Started @37529ms
2021-02-23 12:20:53,884+0000 INFO  [jetty-main-1]  *SYSTEM org.sonatype.nexus.bootstrap.jetty.JettyServer - 
-------------------------------------------------

Started Sonatype Nexus OSS 3.29.2-02

-------------------------------------------------
```
At this point, Nexus is started and listening on port 8081. You can check it with the following command:
```sh 
ss -altnp | grep 8081
```
Next, stop the Nexus service with the following command:
```sh 
/opt/nexus/bin/nexus stop
```

Create a Systemd Service File for Nexus

Next, you will need to create a systemd service file to manage the Nexus service. You can create it with the following command:
```sh 
nano /etc/systemd/system/nexus.service
```

Add the following lines:
```sh 
[Unit]
Description=nexus service
After=network.target

[Service]
Type=forking
LimitNOFILE=65536
ExecStart=/opt/nexus/bin/nexus start
ExecStop=/opt/nexus/bin/nexus stop
User=nexus
Restart=on-abort

[Install]
WantedBy=multi-user.target
```

Save and close the file then reload the systemd daemon with the following command:
```sh
systemctl daemon-reload
```
Next, start the Nexus service and enable it to start at system reboot with the following command:
```sh
systemctl start nexus
systemctl enable nexus
```
You can also verify the status of the Nexus service using the following command:
```sh
systemctl status nexus
```

Login nexus server from browser on port 8081
Use default credentials to login

username : admin
password : admin123

http://<Nexus_server>:8081



### Troubleshooting

service is not starting?
 - make sure you are trying to start service with nexus user. 
- check java installation

Unable to access nexus URL?
- make sure port 8081 is opened in security group. 

### Next Steps
- [x] [Configure Users & Groups in Jenkins](https://youtu.be/jZOqcB32dYM)
- [x] [Secure your Jenkins Server](https://youtu.be/19FmJumnkDc)
- [x] [Jenkins Plugin Installation](https://youtu.be/p_PqPBbjaZ4)
- [x] [Jenkins Master-Slave Configuration](https://youtu.be/hwrYURP4O2k)
