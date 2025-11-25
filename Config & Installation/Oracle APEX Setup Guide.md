# 🚀 Oracle XE + APEX + ORDS + Tomcat Setup

A complete installation and configuration guide for Oracle XE Database 21.3, Oracle APEX 23.2, Oracle REST Data Services (ORDS), and Apache Tomcat 9 on Windows.

---

## ✨ Features

- **Oracle XE 21.3** – Free, lightweight Oracle Database
- **APEX 23.2** – Low-code application development platform
- **ORDS 22.1** – RESTful services for Oracle Database
- **Tomcat 9** – Java servlet container for ORDS deployment
- **Complete Integration** – Full stack setup with proper configuration
- **Production Ready** – Security best practices included

---

## 📋 Prerequisites

- **OS:** Windows 10/11 (64-bit)
- **RAM:** Minimum 4GB (8GB recommended)
- **Disk Space:** At least 10GB free
- **Admin Rights:** Administrator privileges required
- **Browser:** Modern web browser (Chrome, Firefox, Edge)

---

## 🔧 Installation Steps

### 1️⃣ Install Oracle XE Database 21.3

**Download Oracle XE:**
```
https://download.oracle.com/otn-pub/otn_software/db-express/OracleXE213_Win64.zip
```

- Extract and run the installer
- Note your installation directory (e.g., `E:\Oracle\DB`)
- Set a strong password for SYS and SYSTEM users during installation

---

### 2️⃣ Download Oracle APEX 23.2

**Download APEX:**
```
https://download.oracle.com/otn_software/apex/apex_23.2.zip
```

- Extract to a directory (e.g., `E:\Oracle\apex\apex`)

---

### 3️⃣ Connect to Database

Open Command Prompt as Administrator:

```cmd
cd E:\Oracle\apex\apex
sqlplus / as sysdba
```

Switch to the pluggable database:
```sql
ALTER SESSION SET CONTAINER=XEPDB1;
```

---

### 4️⃣ Create Tablespace for APEX

Create dedicated tablespaces for APEX:

```sql
CREATE TABLESPACE appx 
DATAFILE 'E:\Oracle\DB\oradata\XE\XEPDB1\apex.dbf' 
SIZE 2000M 
EXTENT MANAGEMENT LOCAL 
SEGMENT SPACE MANAGEMENT AUTO;

CREATE TEMPORARY TABLESPACE temp0_02 
TEMPFILE 'E:\Oracle\DB\oradata\XE\XEPDB1\temp.dbf' 
SIZE 2000M 
AUTOEXTEND ON;
```

> 💡 **Tip:** Adjust paths according to your Oracle installation directory.

---

### 5️⃣ Install APEX

Run the APEX installation script:

```sql
@apexins.sql appx appx temp0_02 /i/
```

⏱️ **Installation takes 15-30 minutes.** Wait for completion, then exit:
```sql
EXIT;
```

---

### 6️⃣ Configure APEX Admin Account

Reconnect and set up the APEX admin workspace:

```sql
sqlplus / as sysdba
ALTER SESSION SET CONTAINER=XEPDB1;
@apxchpwd.sql
```

**Enter the following when prompted:**
- **Username:** `ADMIN`
- **Email:** `admin@yourcompany.com`
- **Password:** Choose a strong password (e.g., `Apex@SecurePass2024!`)

---

### 7️⃣ Configure REST Services

```sql
@apex_rest_config.sql
```

Follow the prompts to configure REST API passwords.

---

### 8️⃣ Unlock Required Database Users

```sql
-- Unlock SYS user
ALTER USER SYS IDENTIFIED BY Next#2024 ACCOUNT UNLOCK;
ALTER USER SYS IDENTIFIED BY Next#2024 ACCOUNT UNLOCK CONTAINER=ALL;
ALTER USER ANONYMOUS IDENTIFIED BY Next#2024 ACCOUNT UNLOCK;

-- Switch to XEPDB1
ALTER SESSION SET CONTAINER=XEPDB1;

-- Unlock APEX users
ALTER USER APEX_LISTENER IDENTIFIED BY Next#2024 ACCOUNT UNLOCK;
ALTER USER APEX_PUBLIC_USER IDENTIFIED BY Next#2024 ACCOUNT UNLOCK;
ALTER USER APEX_REST_PUBLIC_USER IDENTIFIED BY Next#2024 ACCOUNT UNLOCK;
```

> ⚠️ **Security Note:** Replace `Next#2024` with your own secure password!

---

### 9️⃣ Install Java JDK 17

**Download JDK 17:**
```
https://download.oracle.com/java/17/latest/jdk-17_windows-x64_bin.exe
```

- Install to default location: `C:\Program Files\Java\jdk-17`
- Verify installation:

```cmd
java -version
```

Expected output: `java version "17.x.x"`

---

### 🔟 Install Apache Tomcat 9

**Download Tomcat 9:**
```
https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.84/bin/apache-tomcat-9.0.84.exe
```

- Install to: `E:\Oracle\Tomcat 9.0` (or your preferred location)
- Choose to install as Windows service for auto-start

---

### 1️⃣1️⃣ Install and Configure ORDS

#### A. Download ORDS

```
https://download.oracle.com/otn_software/java/ords/ords-22.1.0.105.1723.zip
```

#### B. Create Directory Structure

```cmd
mkdir E:\Oracle\ORDS
mkdir E:\Oracle\ORDS\Config
```

- Extract `ords-22.1.0.105.1723.zip` to `E:\Oracle\ORDS`

#### C. Copy APEX Static Files

1. Navigate to: `E:\Oracle\apex\apex\images`
2. Copy the entire `images` folder
3. Paste to: `E:\Oracle\Tomcat 9.0\webapps`
4. **Rename** `images` → `i`

> 📁 Final path should be: `E:\Oracle\Tomcat 9.0\webapps\i`

#### D. Run ORDS Installation

```cmd
cd C:\Program Files\Java\jdk-17\bin
java.exe -jar E:\Oracle\ORDS\ords.war install
```

#### E. ORDS Configuration Wizard

Answer the prompts as follows:

**Database Connection:**
```
Location to store configuration: E:\Oracle\ORDS\Config
Name of the database server [localhost]: localhost
Database listen port [1521]: 1521
Database service name: XEPDB1
```

**ORDS Schema Setup:**
```
Enter 1 if you want to verify/install Oracle REST Data Services schema: 1
Database password for ORDS_PUBLIC_USER: [Your password]
Database password for SYS AS SYSDBA: [Your SYS password]
Confirm password: [Confirm]
```

**Tablespace Configuration:**
```
Default tablespace for ORDS_METADATA [SYSAUX]: [Press Enter]
Temporary tablespace for ORDS_METADATA [TEMP]: [Press Enter]
Default tablespace for ORDS_PUBLIC_USER [USERS]: [Press Enter]
Temporary tablespace for ORDS_PUBLIC_USER [TEMP]: [Press Enter]
```

**APEX Integration:**
```
If using Oracle Application Express: 1
PL/SQL Gateway database user name: APEX_PUBLIC_USER
Database password for APEX_PUBLIC_USER: [Your password]
Confirm password: [Confirm]
```

**RESTful Services Users:**
```
Enter 1 to specify passwords for RESTful Services: 1
Database password for APEX_LISTENER: [Your password]
Confirm password: [Confirm]
Database password for APEX_REST_PUBLIC_USER: [Your password]
Confirm password: [Confirm]
```

**Startup Mode:**
```
Enter 1 for standalone mode or 2 to exit: 2
```

---

### 1️⃣2️⃣ Deploy ORDS to Tomcat

1. Copy `E:\Oracle\ORDS\ords.war` → `E:\Oracle\Tomcat 9.0\webapps\`
2. Start Tomcat service (Windows Services or `startup.bat`)
3. Wait for WAR file to deploy (~30 seconds)

---

## 🌐 Accessing APEX

### Login to APEX Workspace

1. **Open your browser:** `http://localhost:8080/ords`
2. **Click:** "Workspace Login"
3. **Enter credentials:**
   - **Workspace:** `INTERNAL`
   - **Username:** `ADMIN`
   - **Password:** [Your configured password]

### Create Application Workspace

1. Log in as `ADMIN`
2. Navigate to: **Manage Workspaces** → **Create Workspace**
3. Follow the wizard to set up your development workspace

---

## 🛠️ Useful Commands

### Database Operations

```sql
-- Connect to database
sqlplus sys/[password]@localhost:1521/XEPDB1 as sysdba

-- Check listener status
lsnrctl status

-- Switch to pluggable database
ALTER SESSION SET CONTAINER=XEPDB1;
```

### Tomcat Operations

```cmd
# In Tomcat bin directory
cd E:\Oracle\Tomcat 9.0\bin

# Start Tomcat
startup.bat

# Stop Tomcat
shutdown.bat
```

### Service Management

```cmd
# Start Oracle service
net start OracleServiceXE

# Stop Oracle service
net stop OracleServiceXE
```

---

## 🐛 Troubleshooting

### Issue: Cannot Connect to Database

**Solution:**
- Verify Oracle XE service is running in Windows Services
- Check listener: `lsnrctl status`
- Ensure port 1521 is not blocked by firewall

### Issue: ORDS Returns 404 Error

**Solution:**
- Verify `ords.war` exists in `webapps` folder
- Check Tomcat logs: `E:\Oracle\Tomcat 9.0\logs\catalina.out`
- Restart Tomcat service

### Issue: Images Not Loading in APEX

**Solution:**
- Verify `i` folder exists in: `E:\Oracle\Tomcat 9.0\webapps\i`
- Check folder contains APEX static files
- Clear browser cache

### Issue: Authentication Failures

**Solution:**
- Verify all passwords match configured values
- Check user accounts are unlocked:
```sql
SELECT username, account_status FROM dba_users 
WHERE username IN ('APEX_PUBLIC_USER', 'APEX_LISTENER', 'APEX_REST_PUBLIC_USER');
```

### Issue: Port 8080 Already in Use

**Solution:**
- Change Tomcat port in: `E:\Oracle\Tomcat 9.0\conf\server.xml`
- Look for `<Connector port="8080"` and change to `8081` or another port

---

## 🔒 Security Best Practices

✅ **Change Default Passwords**
- Use strong passwords (12+ characters, mixed case, numbers, special chars)
- Never use default passwords in production

✅ **Network Security**
- Restrict database listener to localhost if not needed externally
- Use firewall rules to limit access

✅ **HTTPS Configuration**
- Enable SSL/TLS for production deployments
- Use valid certificates

✅ **Regular Updates**
- Keep APEX, ORDS, and Tomcat updated
- Monitor Oracle security advisories

✅ **Audit Logging**
- Enable database audit logs
- Monitor application access logs

✅ **Backup Strategy**
- Regular database backups
- Export APEX applications regularly

---

## 📚 Additional Resources

- [Oracle APEX Documentation](https://docs.oracle.com/en/database/oracle/apex/)
- [ORDS Installation Guide](https://docs.oracle.com/en/database/oracle/oracle-rest-data-services/)
- [Oracle XE Documentation](https://docs.oracle.com/en/database/oracle/oracle-database/21/xeinw/)
- [Apache Tomcat Documentation](https://tomcat.apache.org/tomcat-9.0-doc/)
- [APEX Community Forum](https://community.oracle.com/mosc/categories/apex)

---

## ⚙️ System Architecture

```
┌─────────────────┐
│   Web Browser   │
└────────┬────────┘
         │ HTTP :8080
         ▼
┌─────────────────┐
│  Apache Tomcat  │
│   (ords.war)    │
└────────┬────────┘
         │ JDBC
         ▼
┌─────────────────┐
│   Oracle XE     │
│   Database      │
│   (XEPDB1)      │
└─────────────────┘
```

---

## 📄 License

This guide is provided for educational purposes. Oracle products are subject to Oracle's licensing terms and conditions.

---

## 🙌 Contributing

Contributions are welcome! Please feel free to:
- Submit issues for problems or questions
- Create pull requests for improvements
- Share your setup experiences

---

## 📞 Support

For issues related to:
- **Oracle Products:** [Oracle Support](https://support.oracle.com/)
- **APEX Community:** [forums.oracle.com/apex](https://forums.oracle.com/apex)
- **This Guide:** Open an issue in this repository

---

**Last Updated:** November 2024  
**Tested On:** Windows 10/11, Oracle XE 21.3, APEX 23.2, ORDS 22.1, Tomcat 9.0.84
