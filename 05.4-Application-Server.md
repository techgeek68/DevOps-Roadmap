# Application Servers & Apache Tomcat

---

**What is an Application Server?**
- An application server is a software platform providing runtime services for dynamic application code (e.g., Java Servlets, JSP, WebSocket endpoints, REST APIs). It abstracts:
  - Business logic execution (transactions, security integration)
  - Resource management (thread pools, connection pools, clustering hooks)
  - Standard APIs (Servlet, JSP, WebSocket, Expression Language; in full Java/Jakarta EE servers: JPA, JMS, CDI, JTA)
  - Deployment model (WAR, EAR, exploded or packaged)

Examples (lightweight vs. full profile):
- Lightweight / Servlet containers: Apache Tomcat, Jetty, Undertow
- Full Jakarta EE / Enterprise: WildFly, Payara, WebLogic, WebSphere

**Web Server vs. Application Server**

| Feature        | Web Server (HTTPD, Nginx)                        | Application Server (Tomcat, WildFly)                |
|----------------|--------------------------------------------------|-----------------------------------------------------|
| Primary Role   | Static content, reverse proxy, TLS termination   | Dynamic request execution (Servelts/JSP), APIs      |
| Execution Model| File streaming                                   | Managed components (Servlet lifecycle)              |
| Protocols      | HTTP/HTTPS (+ optional FastCGI, gRPC proxy)      | HTTP/HTTPS + implementation of Jakarta EE APIs      |
| Caching        | Strong static cache, edge friendliness           | Caches objects in heap (depends on app design)      |
| Scaling        | Horizontal (stateless)                           | Horizontal + session handling (sticky or replicated)|
| Use Case       | Front tier, static, proxy, load balancing        | Backend dynamic logic                               |

> Web servers specialise in efficient HTTP request handling and static file delivery; application servers host and execute code that renders dynamic responses.

---

**Apache Tomcat**

**What is Tomcat?**
  - Open-source Java Servlet/JSP container
  - Implements: Servlet, JSP, WebSocket, EL, annotations
  - Not a full Jakarta EE stack (no built-in EJB/JPA/JMS/JTA); integrate frameworks (Spring, Hibernate) at app layer

**Core Internal Components**
  - Coyote: Protocol handler (HTTP/1.1, HTTP/2 with upgrade, AJP if enabled)
  - Catalina: Servlet container (deployment, lifecycle, mapping, filters)
  - Jasper: JSP compiler (JSP→Java servlet class→bytecode)
  - Realm: Authentication/authorisation integration (MemoryRealm, JDBCRealm, JNDIRealm)
  - Connectors: Network endpoints (HTTP, HTTPS, AJP)
  - Executor: Shared thread pool (optional) for connectors & servlet threads

**Request Flow**
```
Client → Connector (Coyote) → Catalina Mapping → Filter Chain → Servlet/JSP → Response Commit → Output via Coyote
```

 - Tomcat Request Handling:
```
+-------------------------------+      +-------------------------------+      +--------------------------------+
|      Client (Browser)         |<---->|  Coyote (HTTP connector)      |<---->|   Servlet engine (Catalina)    |
|  HTTP request                 |      |  Acceptor threads             |      |  Servlets/JSP processing       |
|  HTTP response                |      |  Worker threads               |      |                                |
|                               |      |  SSL/TLS encryption (if HTTPS)|      |                                |
+-------------------------------+      +-------------------------------+      +--------------------------------+
                                                                                 |                           |
                                                                                 |                           |
                                                                                 v                           v
                                                                                +-------------------------------+
                                                                                | Jasper                        |
                                                                                | (JSP engine, if JSP)          |
                                                                                | JSP compilation (if JSP file) |
                                                                                | and Java bytecode delivery    |
                                                                                | for Catalina processing       |
                                                                                +-------------------------------+
```
  - Client (Browser): sends HTTP request, receives HTTP response.
  - Coyote (HTTP Connector): Listens for incoming connections, SSL/TLS support, and hands requests to Catalina.
  - Catalina (Servlet Engine): manages Servlet lifecycle; delegates JSPs to Jasper.
  - Jasper (JSP Engine): compiles JSPs to Servlets (bytecode) returned to Catalina.

---

**Environment & Prerequisites**

| Item            | Recommendation |
|-----------------|----------------|
| OS              | RHEL / CentOS / Rocky / AlmaLinux / Debian / Ubuntu |
| Java Version    | Use latest LTS (e.g., OpenJDK 21) |
| User Separation | Run Tomcat as non-root (e.g., `tomcat` user) |
| Firewall        | Restrict inbound to required ports only |
| SELinux/AppArmor| Configure policies rather than disabling |


- Set hostname:
```
 sudo hostnamectl set-hostname tomcatserver.devopsclass.com
 sudo exec bash
```

- Configure network:
```
  nmtui
```

- Install Java (RHEL family):
```
# Install Java 17 first (Debian/Ubuntu)
sudo apt update
sudo apt install -y openjdk-17-jdk

# Or on RHEL/CentOS:
sudo dnf install java-*-openjdk-devel
```

- Verify:
```
java -version
```

- Add variable to `sudo vim .bashrc` or system profile:
```
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk
export CATALINA_HOME=/opt/tomcat
export PATH=$PATH:$CATALINA_HOME/bin
```

---

**Install Tomcat**:

| Methods         | Pros | Cons |
|----------------|------|------|
| tar.gz manual  | Full control, multiple versions side-by-side | Manual updates, no automatic service management |
| OS packages    | Easy updates via package manager | Might lag latest upstream, opinionated paths |
| Container image| Fast portability, immutable layering | Requires container orchestration hygiene |
| Configuration mgmt (Ansible) | Repeatable, versionable | Setup complexity |


**Transfer a previously downloaded archive:**
```
  scp <apache-tomcat.tar.gz> user_name@<IP_Addr>:/opt
```

**tar.gz Manual Install**
  - Visit [Apache Tomcat](https://tomcat.apache.org/) to find a download link.
    
<img width="1446" height="567" alt="Screenshot 2025-11-20 at 12 48 16 PM" src="https://github.com/user-attachments/assets/8fc7e1c4-c8fb-4904-bc85-36db28486fa5" />

```
# Download Tomcat
cd /tmp
sudo wget https://dlcdn.apache.org/tomcat/tomcat-11/v11.0.14/bin/apache-tomcat-11.0.14.tar.gz

# Extract
sudo tar -xzf apache-tomcat-11.0.14.tar.gz

# Move to /opt
sudo mv apache-tomcat-11.0.14 /opt/tomcat

# Back to home directory
cd
```
```
sudo groupadd tomcat
sudo useradd -g tomcat -d /opt/tomcat -s /sbin/nologin tomcat
sudo chown -R tomcat:tomcat /opt/tomcat
```

  - Directory Layout
```
sudo tree /opt/tomcat
```
```
/opt/tomcat            (CATALINA_HOME)
/opt/tomcat/conf       (server.xml, web.xml)
/opt/tomcat/logs
/opt/tomcat/webapps    (deployed WARs)
/opt/tomcat/temp
/opt/tomcat/work       (JSP compilation)
/opt/tomcat/lib        (shared libs if needed)
```
> Avoid modifying default `webapps` for production; deploy only required WARs. Remove sample apps (docs, examples, manager) when not needed.

---

**Running Tomcat**
- Start Tomcat:
```
sudo /opt/tomcat/bin/startup.sh
```

- Check port (default 8080):
```
  netstat -tnl | grep 8080
```

<img width="988" height="237" alt="Screenshot 2025-11-20 at 1 20 56 PM" src="https://github.com/user-attachments/assets/00ac68c2-cdb5-40ba-bfa6-d45e125839ab" />


- Stop Tomcat:
```
sudo /opt/tomcat/bin/shutdown.sh
```


**Make it persistent**

1. rc.local
   rc.local is a traditional Linux script that runs at the end of the boot process, once most other system services have been launched. A straightforward way for system administrators to add startup tasks; however, on modern systems, this facility is often replaced by systemd services.
> Note: rc.local must exist and be executable.

```
vim /etc/rc.d/rc.local
```

- Add full path to startup script:
```
sudo /opt/bin/startup.sh
```

<img width="743" height="268" alt="Screenshot 2025-11-21 at 7 47 33 AM" src="https://github.com/user-attachments/assets/836b8ea6-aada-463b-9f03-f8ffb05331ab" />

- Permissions:
```
sudo chmod u+x /etc/rc.d/rc.local
```

2. Systemd Service (**Preferred over rc.local**)
   
  - Stop any manually started Tomcat
```
sudo -u tomcat /opt/tomcat/bin/catalina.sh stop || true

# If still running:
ps -ef | grep java | grep tomcat
sudo pkill -u tomcat java || true
```

  - Ensure temp directory clean:
```
sudo rm -f /opt/tomcat/temp/tomcat.pid
sudo mkdir -p /opt/tomcat/temp
sudo chown tomcat:tomcat /opt/tomcat/temp
```

  - Apply SELinux context to the entire Tomcat tree
```
sudo semanage fcontext -a -t bin_t "/opt/tomcat/bin(/.*)?"
sudo semanage fcontext -a -t usr_t "/opt/tomcat/(conf|lib|webapps|logs|temp|work)(/.*)?"
sudo restorecon -Rv /opt/tomcat
```
  
  - Create an environment file
```
sudo tee /etc/default/tomcat <<EOF
JAVA_HOME=${JAVA_HOME}
CATALINA_HOME=/opt/tomcat
CATALINA_BASE=/opt/tomcat
CATALINA_PID=/opt/tomcat/temp/tomcat.pid
CATALINA_OPTS=-Xms256m -Xmx512m -XX:+UseG1GC -Djava.security.egd=file:/dev/urandom
EOF
```

  - Create a minimal unit file
```
sudo tee /etc/systemd/system/tomcat.service <<'EOF'
[Unit]
Description=Apache Tomcat 11 Servlet Container
After=network.target

[Service]
Type=forking
User=tomcat
Group=tomcat
EnvironmentFile=-/etc/default/tomcat

# Pre-step: ensure temp directory exists
ExecStartPre=/usr/bin/mkdir -p /opt/tomcat/temp
ExecStartPre=/usr/bin/chown tomcat:tomcat /opt/tomcat/temp

ExecStart=/opt/tomcat/bin/catalina.sh start
ExecStop=/opt/tomcat/bin/catalina.sh stop

Restart=on-failure
RestartSec=5
UMask=0027
LimitNOFILE=65536
PrivateTmp=true
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target
EOF
```

- Check syntax, reload service, enable & start Tomcat as service:
```
sudo systemd-analyze verify /etc/systemd/system/tomcat.service      #Check syntax
```
```
sudo systemctl daemon-reload
```
```
sudo systemctl enable tomcat
```
```
sudo systemctl start tomcat
```
```
sudo systemctl status tomcat
```

<img width="1294" height="493" alt="Screenshot 2025-11-24 at 7 57 07 AM" src="https://github.com/user-attachments/assets/bee80e0a-8715-4093-b000-5ef2dac9771d" />


---

**Changing Tomcat default Port**
  - Check if the desired port (e.g., 5080) is free:
```
netstat -tnl | grep 5080
```

- Edit:
```
sudo vim $CATALINA_HOME/conf/server.xml
```

- Modify Connector:
```
<Connector port="5080" protocol="HTTP/1.1"
connectionTimeout="20000"
redirectPort="8443" />
```
<img width="753" height="146" alt="Screenshot 2025-11-21 at 8 08 02 AM" src="https://github.com/user-attachments/assets/f92dfb60-49f3-459f-b837-7fc5e9afbebe" />

---

**Secure Remote Access to Manager/Host-Manager**
  - By default, access is restricted to localhost via RemoteAddrValve.
  - Allow a specific IP (Example 10.10.5.162,10.10.5.157)
```
        allow="127\.\d+\.\d+\.\d+|::1|10\.10\.5\.162|10\.10\.5\.157" />
```
  - Allow a subnet (192.168.1.x and 10.10.5.x )
> 192.168.1.0-255
```    allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1|192\.168\.1\.\d+|192\.168\.5\.\d+"
```

  - Update Host Manager context.xml
```
sudo vim $CATALINA_HOME/webapps/host-manager/META-INF/context.xml
```
```
<Context antiResourceLocking="false" privileged="true" >
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1|10\.10\.5\.\d+" />
</Context>
```
 
  - Update Manager App context.xml
```
sudo vim $CATALINA_HOME/webapps/manager/META-INF/context.xml
```
> Note: Commenting the Valve removes IP restriction, but is a security risk:
```
<Context antiResourceLocking="false" privileged="true" >
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1|10\.10\.5\.\d+" />
  <Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.CsrfPreventionFilter\$LruCache(?:\$1)?|java\.util\.(?:Linked)?HashMap"/>
</Context>
```

---

**Create Roles and Users (tomcat-users.xml)**

 - Edit
```
sudo vim $CATALINA_HOME/conf/tomcat-users.xml
```

  - Add roles:
```
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<role rolename="manager-jmx"/>
<role rolename="manager-status"/>
```
Add a user and assign roles:
```
<user username="admin" password="devops"
      roles="manager-gui,manager-script,manager-jmx,manager-status"/>
```
<img width="839" height="181" alt="Screenshot 2025-11-24 at 8 46 00 AM" src="https://github.com/user-attachments/assets/e2e1d15d-aa1e-44a7-904b-0e808f489ec2" />


Reload Tomcat to apply:
```
sudo systemctl restart tomcat
```

---

**Set firewall:**

```
sudo firewall-cmd --list-all

sudo firewall-cmd --permanent --add-port=8080/tcp

sudo firewall-cmd --permanent --add-port=80/tcp

sudo firewall-cmd --reload
```

---

**Verify:**
 - Access remotely (use your server IP and port):
```
http://<server_ip>:8080
```

 - Expected landing page text:
```
If you're seeing this, you've successfully installed Tomcat. Congratulations!
```
<img width="984" height="862" alt="Screenshot 2025-11-24 at 8 28 40 AM" src="https://github.com/user-attachments/assets/e7851142-8c9b-4ccf-917d-737b3368e160" />


 - Open the Manager App and log in with the configured credentials(Server with GUI is preferred):

<img width="1052" height="256" alt="Screenshot 2025-11-24 at 9 43 58 AM" src="https://github.com/user-attachments/assets/16ba0da2-d895-4535-af77-701d9fbf9bfc" />


<img width="829" height="335" alt="Screenshot 2025-11-24 at 11 00 45 AM" src="https://github.com/user-attachments/assets/af2e74d5-3031-45cc-999c-9b19fcaef27a" />

```
http://10.10.5.157:5080/manager/html
```
<img width="1469" height="918" alt="Screenshot 2025-11-24 at 11 01 15 AM" src="https://github.com/user-attachments/assets/b8de7e63-f4af-4f8d-9e37-5610cd21fd70" />



---

**Host-Manager vs Manager context.xml**

- host-manager/META-INF/context.xml
  - Purpose: Configure Host Manager (manage virtual hosts).
    - Scope: Server-wide host administration.

- manager/META-INF/context.xml
  - Purpose: Configure Manager App (manage webapps).
    - Scope: Deploy/undeploy/start/stop applications; runtime controls.
  
---

**Security Hardening**

| Area | Action |
|------|--------|
| Default Apps | Remove `/webapps/docs`, `/webapps/examples` |
| Manager Access | Restrict by IP + secure credentials; consider reverse proxy + HTTP auth |
| TLS | Use reverse proxy (Nginx/HAProxy) or Tomcat native APR/OpenSSL for performance |
| Headers | Add security headers (CSP, HSTS, X-Frame-Options) via Filters or proxy |
| JMX | Bind JMX to localhost; secure with password file + SSL or disable remote unless needed |
| Users | Manage roles in `tomcat-users.xml` or prefer external realm (LDAP/OIDC) |
| Logging | Rotate logs (logrotate config) to prevent disk exhaustion |
| File Permissions | App user read-only on binaries, write only to `logs/`, `temp/`, `work/` |
| Updates | Track CVEs (Tomcat, Java) and patch regularly |
| AJP Connector | Disable unless required (ghostcat vulnerability history) |
| Session Security | Use secure cookies (`secure`, `httponly`), set session timeout appropriately |

Example `tomcat-users.xml` entries (minimal):
```
<role rolename="manager-status"/>
<user username="statususer" password="StrongPass123!" roles="manager-status"/>
```
(Restrict full GUI admin to the internal network only.)

---

**Performance & Tuning**

| Parameter | Description | Strategy |
|-----------|-------------|----------|
| maxThreads | Concurrent request worker threads | Start ~200; adjust with throughput testing |
| acceptCount | Queue length when threads exhausted | Ensure > typical spikes (e.g., 200–500) |
| connectionTimeout | Socket timeout (ms) | Tune (20000 default); reduce for APIs with quick responses |
| Executor | Shared thread pool across connectors | Enables central control; configure min/max |
| JVM Heap | `-Xms` / `-Xmx` sizing | Base on app profile; monitor GC pauses |
| GC Strategy | HotSpot G1 / ZGC for large heaps | Choose with load testing |
| Compression | `compression="on"` for text responses | Watch CPU overhead; exclude binary formats |
| KeepAlive | Controls re-use of connections | Improves efficiency for HTTP/1.1 clients |
| HTTP/2 | Enables multiplexing | Requires appropriate connector & Java version |

Add to `setenv.sh` (create in `$CATALINA_HOME/bin`):
```
export JAVA_OPTS="-Xms512m -Xmx1024m -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/opt/tomcat/logs"
```

---

**Monitoring**
  - JMX (local + Prometheus JMX exporter)
  - Access logs (latency patterns)
  - Heap & GC metrics (VisualVM, JMC, jstat)
  - Thread dumps for diagnosing stalls (`jstack <pid>`)
  - APM integration (e.g., OpenTelemetry Java agent)

Prometheus JMX exporter config example snippet:
```
rules:
  - pattern: 'java.lang<type=Memory><HeapMemoryUsage>(.*)'
```

---

**Deployment & Lifecycle**

| Step | Action |
|------|--------|
| Build | Produce WAR via Maven/Gradle |
| Scan | SAST/DAST security scanning (dependency check) |
| Stage | Copy WAR to `webapps/` or use `/manager/text/deploy` endpoint |
| Validate | Health endpoint check (`/health` or custom servlet) |
| Traffic Shift | Use reverse proxy or load balancer to drain old instance before update |
| Rollback | Keep previous WAR version; name war with version stamp (e.g., `app-1.4.2.war`) |

CLI Manager (script role required):
```
  curl -u admin:devops "http://10.10.5.4:5080/manager/text/list"
  curl -u admin:devops "http://10.10.5.4:5080/manager/text/deploy?              path=/app&war=file:/opt/releases/app-1.4.2.war"
  curl -u admin:devops "http://10.10.5.4:5080/manager/text/undeploy?path=/app"
```

Zero-Downtime Strategy:
  - Use blue/green: `app-v1` and `app-v2` context paths behind proxy rewriting
  - Or predeploy to a new instance & deregister the old from the load balancer

---

**Logging & Diagnostics**
- Default Logs:
  - `catalina.out` (stdout/stderr; rotate externally)
  - `localhost_access_log*.txt`
  - `localhost.yyyy-MM-dd.log`
  - `manager.yyyy-MM-dd.log`

Rotation via logrotate example (`/etc/logrotate.d/tomcat`):
```
/opt/tomcat/logs/*.log {
  daily
  rotate 14
  compress
  missingok
  copytruncate
  notifempty
}
```

Thread Dump (investigate slowness):
```
jstack $(pgrep -f 'org.apache.catalina.startup.Bootstrap') > /tmp/tdump.txt
```

Heap Dump (on OOM automatically with JVM flag):
Analyse with Eclipse MAT or VisualVM.

---

**Reverse Proxy Integration**
  - Nginx sample:
```
server {
  listen 80;
  server_name app.example.com;

  location / {
    proxy_pass http://127.0.0.1:5080;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $remote_addr;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-Port $server_port;
  }
}
```

Enable secure cookies & correct redirect logic: set
```
proxy_set_header X-Forwarded-Proto $scheme;
```
Then in `server.xml` inside `<Engine>` or `<Host>` optionally configure `RemoteIpValve`:
```
<Valve className="org.apache.catalina.valves.RemoteIpValve"
       internalProxies="127\.0\.0\.1"
       remoteIpHeader="X-Forwarded-For"
       protocolHeader="X-Forwarded-Proto" />
```

---

**Session Management & Scalability**

| Strategy | Description | Trade-offs |
|----------|-------------|------------|
| Sticky Sessions | Load balancer routes user to same node | Simplicity; single-node failure risk for session |
| Session Replication | Delta or full replication between nodes | Higher memory/network overhead |
| External Store | Redis / Memcached via custom manager | Decouples state; added latency |
| Stateless (Token) | JWT/OAuth tokens; store minimal server state | Requires redesign of legacy session apps |

For scaling horizontally, prefer an external session store or a stateless design.

---

**Comparison: Tomcat vs Alternatives**

| Feature         | Tomcat | Jetty | Undertow | Full EE (WildFly) |
|-----------------|--------|-------|----------|-------------------|
| Footprint       | Small  | Smaller | Very small | Larger |
| Async Support   | Yes (Servlet 3.x) | Strong | Strong | Yes |
| WebSocket       | Yes    | Yes   | Yes      | Yes |
| HTTP/2          | Yes (config) | Yes | Yes | Yes |
| Jakarta EE Full | No     | No    | No       | Yes |
| Embedded Mode   | Limited (via Spring Boot) | Native | Native | Not typical |
| Startup Speed   | Fast   | Faster | Very fast | Moderate |

---

**Examples**
  - Enhanced `setenv.sh`
    `$CATALINA_HOME/bin/setenv.sh`:
```
#!/bin/bash
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk
export CATALINA_PID="$CATALINA_HOME/temp/tomcat.pid"
export JAVA_OPTS="-Xms1024m -Xmx1536m \
  -XX:+UseG1GC -XX:MaxGCPauseMillis=200 \
  -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=$CATALINA_HOME/logs \
  -Dfile.encoding=UTF-8 \
  -Djava.security.egd=file:/dev/./urandom \
  -Dcom.sun.management.jmxremote \
  -Dcom.sun.management.jmxremote.local.only=true \
  -Dlog4j2.formatMsgNoLookups=true"
```

Make executable:
```
chmod +x $CATALINA_HOME/bin/setenv.sh
```
    
**- Hardened `server.xml` Snippet**
```
<Server port="8005" shutdown="SHUTDOWN">
  <Service name="Catalina">
    <Executor name="tomcatThreadPool" namePrefix="catalina-exec-" maxThreads="300" minSpareThreads="50"/>

    <Connector port="5080" protocol="HTTP/1.1"
               connectionTimeout="15000"
               executor="tomcatThreadPool"
               maxKeepAliveRequests="1000"
               compression="on"
               compressableMimeType="text/html,text/xml,text/plain,text/css,application/json,application/javascript"
               redirectPort="8443" />

    <!-- HTTPS via reverse proxy preferred; direct connector optional -->
    <!--
    <Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
               SSLEnabled="true" maxThreads="200"
               keystoreFile="/opt/tomcat/ssl/keystore.jks"
               keystorePass="changeit" scheme="https" secure="true"
               clientAuth="false" sslProtocol="TLSv1.3" />
    -->

    <Engine name="Catalina" defaultHost="localhost">
      <Valve className="org.apache.catalina.valves.RemoteIpValve"
             internalProxies="127\.0\.0\.1"
             protocolHeader="X-Forwarded-Proto"
             remoteIpHeader="X-Forwarded-For" />

      <Host name="localhost" appBase="webapps" unpackWARs="true" autoDeploy="false">
        <!-- Session cookie config -->
        <Context useHttpOnly="true" sessionCookieName="JSESSIONID" />
      </Host>
    </Engine>
  </Service>
</Server>
```

---

**Test Questions**
  1. Difference between Coyote and Catalina?
  2. Why avoid running Tomcat as root?
  3. What parameters affect concurrent request throughput?
  4. Two strategies for zero-downtime deployment?
  5. Why remove `examples` and `docs` from production?
  6. Best way to manage SSL: reverse proxy or direct connector? Why?
  7. How to investigate memory leak symptoms?
  8. Session management options—trade-offs?
  9. Purpose of `RemoteIpValve` behind a proxy?
  10. Why use an Executor thread pool instead of connector-specific threads?

---
