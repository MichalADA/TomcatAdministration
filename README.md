# Apache Tomcat Installation and Configuration on Ubuntu

This guide outlines the steps to install and configure Apache Tomcat on Ubuntu with Java 17, set it up as a system service, and enable HTTPS with a self-signed certificate.

---

## **1. Install Java 17**

### Update the package manager and install Java 17:
```bash
sudo apt update
sudo apt install openjdk-17-jdk -y
```

### Verify the Java installation:
```bash
java -version
```

### Configure the default Java version (if multiple versions are installed):
```bash
sudo update-alternatives --config java
```

### Set the `JAVA_HOME` environment variable:
```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
```

### Make the `JAVA_HOME` variable permanent:
```bash
echo 'export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64' >> ~/.bashrc
source ~/.bashrc
```

### Verify the `JAVA_HOME` variable:
```bash
echo $JAVA_HOME
```

---

## **2. Install Apache Tomcat**

### Download Apache Tomcat:
```bash
wget https://dlcdn.apache.org/tomcat/tomcat-11/v11.0.2/bin/apache-tomcat-11.0.2.tar.gz
```

### Extract the archive:
```bash
tar -xvzf apache-tomcat-11.0.2.tar.gz
```

### Move Tomcat to the `/opt` directory:
```bash
sudo mv apache-tomcat-11.0.2 /opt/tomcat
```

### Set ownership of the Tomcat directory:
```bash
sudo chown -R $USER:$USER /opt/tomcat
```

### Start Tomcat:
```bash
/opt/tomcat/bin/startup.sh
```

---

## **3. Set Up Tomcat as a System Service**

### Create a systemd service file:
```bash
sudo nano /etc/systemd/system/tomcat.service
```

### Add the following content to the service file:
```ini
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
Environment=CATALINA_HOME=/opt/tomcat
Environment=CATALINA_BASE=/opt/tomcat
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

User=tomcat
Group=tomcat
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
```

### Reload systemd and enable the service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable tomcat
```

### Start the Tomcat service:
```bash
sudo systemctl start tomcat
```

### Check the status of the service:
```bash
sudo systemctl status tomcat
```

---

## **4. Enable HTTPS with a Self-Signed Certificate**

### Generate a self-signed certificate:
```bash
keytool -genkey -alias tomcat -keyalg RSA -keystore /opt/tomcat/conf/keystore.jks -keysize 2048
```

- Provide the required details (e.g., password, organization name, etc.) during the certificate generation.

### Configure Tomcat for HTTPS:
1. Open the `server.xml` file:
   ```bash
   sudo nano /opt/tomcat/conf/server.xml
   ```

2. Locate the `<Connector>` element for HTTPS and uncomment or modify it as follows:
   ```xml
   <Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
              maxThreads="200" SSLEnabled="true">
       <SSLHostConfig>
           <Certificate certificateKeystoreFile="conf/keystore.jks"
                        type="RSA" certificateKeystorePassword="your_keystore_password"/>
       </SSLHostConfig>
   </Connector>
   ```

3. Save and close the file.

### Restart Tomcat to apply the changes:
```bash
sudo systemctl restart tomcat
```

### Test HTTPS:
Open your browser and navigate to:
```
https://your-ec2-public-ip:8443
```

---

## **5. Additional Commands**

### Stop Tomcat:
```bash
sudo systemctl stop tomcat
```

### Restart Tomcat:
```bash
sudo systemctl restart tomcat
```

### Check Logs:
```bash
sudo tail -f /opt/tomcat/logs/catalina.out
```

---

Tomcat is now fully installed and configured to run as a system service with HTTPS support!
