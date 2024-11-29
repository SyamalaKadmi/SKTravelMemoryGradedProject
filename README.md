# SKTravelMemoryGradedProject
Graded Project on Travel Memory Application Deployment


# Travel Memory Application Deployment (MERN Stack)

## Overview
The Travel Memory application is built using the MERN stack (MongoDB, Express.js, React.js, Node.js). It consists of a backend connected to a MongoDB database and a React-based frontend. This guide explains how to deploy the application on AWS with scalability and domain configuration.

---

## 1. MongoDB Configuration
1. Create a MongoDB account and cluster.
2. Save the database user password.
3. Open MongoDB Compass:
   - Add a new connection using the connection string (replace `<password>` with your password).
   - Copy the connection string for backend configuration.

---

## 2. Backend Configuration
1. Fork the repository: [TravelMemory GitHub](https://github.com/UnpredictablePrashant/TravelMemory).
2. Set up an **EC2 t3.micro instance**:
   - Add inbound rules for **ports 3000 and 3001**.
   - Connect to the instance via SSH:
     ```bash
     ssh -i "SKMern.pem" ec2-user@<EC2-public-IP>
     ```
   - Install required packages:
     ```bash
     sudo yum install -y nodejs git nginx
     sudo systemctl start nginx
     sudo systemctl enable nginx
     ```
   - Verify Nginx installation: Open `http://<EC2-public-IP>` in a browser.

3. Clone the repository:
   ```bash
   git clone https://github.com/SyamalaKadmi/TravelMemory.git
   cd TravelMemory/backend
   ```
4. Create `.env` file in the backend directory:
   ```env
   MONGO_URI=mongodb+srv://<username>:<password>@<cluster>.mongodb.net/travelmemory
   PORT=3001
   ```
5. Start the backend:
   ```bash
   node index.js &
   ```

---

## 3. Frontend Configuration
1. Navigate to the frontend folder and create `.env`:
   ```bash
   cd TravelMemory/frontend
   sudo vi .env
   ```
   Add the backend URL:
   ```env
   REACT_APP_BACKEND_URL=http://localhost:3001/
   ```
2. Install dependencies and start the frontend:
   ```bash
   npm install
   npm start &
   ```
3. Access the frontend: `http://<EC2-public-IP>:3000`.

---

## 4. Backend-Frontend Connection (Nginx Reverse Proxy)
1. Configure Nginx as a reverse proxy:
   ```bash
   cd /etc/nginx/conf.d/
   sudo nano reverse-proxy.conf
   ```
2. Add the following configuration:
   ```nginx
   server {
       listen 80;
       server_name <your-domain>;

       # Frontend
       location / {
           proxy_pass http://localhost:3000;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }

       # Backend
       location /trip/ {
           proxy_pass http://localhost:3001;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }
   }
   ```
3. Save the file and reload Nginx:
   ```bash
   sudo systemctl reload nginx
   ```

---

## 5. Scaling the Application
1. Create an AMI of the configured EC2 instance.
2. Set up **Elastic Load Balancer (ALB)**:
   - Distribute traffic across multiple EC2 instances.
   - Use the ALB DNS name for application access.
3. Configure **Auto Scaling Group (ASG)** to scale instances automatically.

---

## 6. Domain Setup with Cloudflare
1. Purchase a domain from GoDaddy or Namecheap.
2. Configure the domain in Cloudflare:
   - **A Record**:
     - Points to the EC2 instance IP for the frontend.
   - **CNAME Record**:
     - Points to the ALB DNS name.
3. Verify connectivity by navigating to your domain.
