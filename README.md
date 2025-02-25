Step 1: Install Required Dependencies
Update system packages and install OpenJDK, unzip, and required utilities:

bash
Copy
Edit
sudo apt update -y
sudo apt install -y openjdk-17-jdk unzip
Step 2: Install PostgreSQL
Add PostgreSQL repository:

bash
Copy
Edit
echo "deb https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" | sudo tee /etc/apt/sources.list.d/pgdg.list
Import PostgreSQL signing key:

bash
Copy
Edit
wget -qO - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
Update package lists and install PostgreSQL:

bash
Copy
Edit
sudo apt update -y
sudo apt install -y postgresql-15
Start and enable PostgreSQL service:

bash
Copy
Edit
sudo systemctl enable postgresql
sudo systemctl start postgresql
Step 3: Configure PostgreSQL for SonarQube
Switch to the PostgreSQL user:

bash
Copy
Edit
sudo -i -u postgres
Create a SonarQube database user with a password:

bash
Copy
Edit
createuser sonar
psql -c "ALTER USER sonar WITH ENCRYPTED PASSWORD 'Password';"
Create the SonarQube database and grant privileges:

bash
Copy
Edit
createdb -O sonar sonarqube
Exit PostgreSQL:

bash
Copy
Edit
exit
Step 4: Install SonarQube
Download SonarQube:

bash
Copy
Edit
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.6.1.59531.zip
Extract SonarQube:

bash
Copy
Edit
unzip sonarqube-9.6.1.59531.zip
Move SonarQube to /opt:

bash
Copy
Edit
sudo mv sonarqube-9.6.1.59531 /opt/sonarqube
Step 5: Create a SonarQube User
Create a new system user:

bash
Copy
Edit
sudo useradd -m -d /opt/sonarqube -r -s /bin/bash sonarqube
Change ownership of SonarQube files:

bash
Copy
Edit
sudo chown -R sonarqube:sonarqube /opt/sonarqube
sudo chmod -R 775 /opt/sonarqube
Step 6: Configure SonarQube
Open the SonarQube configuration file:

bash
Copy
Edit
sudo nano /opt/sonarqube/conf/sonar.properties
Modify the database settings:

ini
Copy
Edit
sonar.jdbc.username=sonar
sonar.jdbc.password=Password
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
Modify the SonarQube web server port (optional, default is 9000):

ini
Copy
Edit
sonar.web.port=9000
Save the file (CTRL + X, then Y, then ENTER).

Step 7: Create a Systemd Service for SonarQube
Create a new systemd service file:

bash
Copy
Edit
sudo nano /etc/systemd/system/sonarqube.service
Add the following content:

ini
Copy
Edit
[Unit]
Description=SonarQube service
After=network.target postgresql.service

[Service]
Type=forking
User=sonarqube
Group=sonarqube
ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
Restart=always
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
Save the file (CTRL + X, then Y, then ENTER).

Reload systemd and enable the SonarQube service:

bash
Copy
Edit
sudo systemctl daemon-reload
sudo systemctl enable sonarqube
Start SonarQube:

bash
Copy
Edit
sudo systemctl start sonarqube
Step 8: Access SonarQube Web Interface
Open a browser and go to:

cpp
Copy
Edit
http://<your-server-ip>:9000
Login with default credentials:

pgsql
Copy
Edit
Username: admin
Password: admin
Change the default password after logging in.

Step 9: Check SonarQube Logs (if needed)
To verify if SonarQube is running properly:

bash
Copy
Edit
sudo journalctl -u sonarqube -f
Step 10: (Optional) Open Firewall Port
If UFW is enabled, allow port 9000:

bash
Copy
Edit
sudo ufw allow 9000/tcp
sudo ufw reload
Conclusion
You have successfully installed and configured SonarQube on Ubuntu without Ansible. 🎉

Would you like a script to automate this manual process? 🚀



Better Alternative: Modify Instead of Append
If you want to modify existing values instead of just adding duplicates, use sed:

bash
Copy
Edit
sudo sed -i 's/^sonar.jdbc.username=.*/sonar.jdbc.username=sonar/' /opt/sonarqube/conf/sonar.properties
sudo sed -i 's/^sonar.jdbc.password=.*/sonar.jdbc.password=Password/' /opt/sonarqube/conf/sonar.properties
sudo sed -i 's|^sonar.jdbc.url=.*|sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube|' /opt/sonarqube/conf/sonar.properties
sudo sed -i 's/^sonar.web.port=.*/sonar.web.port=9000/' /opt/sonarqube/conf/sonar.properties
sudo sed -i '/^sonar.web.host=/d' /opt/sonarqube/conf/sonar.properties
echo "sonar.web.host=0.0.0.0" | sudo tee -a /opt/sonarqube/conf/sonar.properties > /dev/null
This approach ensures existing values are replaced, avoiding duplicates.

Let me know which method you prefer! 🚀

3️⃣ Check Database Connection from SonarQube
Run this inside psql to check if SonarQube has created tables:

bash
Copy
Edit
sudo -u postgres psql -d sonarqube -c "\dt"
✅ If you see a list of tables, SonarQube is connected to PostgreSQL.
❌ If empty, SonarQube is not connected.
