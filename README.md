# 🔥 Project 2 — Azure Two-Tier Application

![Azure](https://img.shields.io/badge/Microsoft%20Azure-Cloud-0078D4?logo=microsoftazure&logoColor=white)
![Ubuntu](https://img.shields.io/badge/OS-Ubuntu-E95420?logo=ubuntu&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)
![Flask](https://img.shields.io/badge/Flask-3.1-000000?logo=flask&logoColor=white)
![Gunicorn](https://img.shields.io/badge/Gunicorn-26.1-499848)
![Azure VNet](https://img.shields.io/badge/Azure-Virtual%20Network-0078D4)
![Status](https://img.shields.io/badge/Status-Completed-success)

> A hands-on Azure cloud engineering project demonstrating a secure two-tier application architecture using separate frontend and backend subnets, Network Security Groups, private IP communication, NAT Gateway, Gunicorn, and systemd.

---

# 🚀 Project Overview

This project demonstrates the deployment of a **two-tier web application architecture on Microsoft Azure**.

The architecture consists of:

1. A **public frontend tier**
2. A **private backend tier**

The frontend VM is placed inside a dedicated frontend subnet and has a public IP address so users can access the application from the internet.

The backend VM is placed inside a separate backend subnet and **does not have a public IP address**.

The frontend communicates with the backend using the backend VM's **private IP address** over TCP port `8080`.

A **NAT Gateway** is associated with the backend subnet to provide outbound internet connectivity to the private backend without exposing the backend to unsolicited inbound internet traffic.

The Flask applications are served using **Gunicorn** and managed using **systemd**.

This means the applications continue running after SSH sessions are closed and automatically start again after VM reboots.

---

# 🎯 Project Objectives

The primary objectives of this project were to:

- Create an Azure Virtual Network
- Create separate frontend and backend subnets
- Deploy a Linux VM in each subnet
- Assign a public IP only to the frontend VM
- Keep the backend VM private
- Configure Network Security Groups
- Allow public access to the frontend
- Allow frontend-to-backend communication over TCP port `8080`
- Prevent direct internet access to the backend
- Configure a NAT Gateway for backend outbound connectivity
- Build a Flask frontend application
- Build a Flask backend API
- Use Gunicorn to serve the Flask applications
- Use systemd to manage the applications
- Test communication between the two application tiers
- Test application persistence after SSH sessions are closed
- Test automatic application startup after VM reboots

---

# 🏗️ Architecture

The final architecture separates the application into two logical tiers.

```text
                         INTERNET
                            │
                            │ HTTP :8000
                            ▼
                 ┌─────────────────────┐
                 │     FRONTEND VM     │
                 │                     │
                 │ Public IP           │
                 │ 4.222.218.179       │
                 │                     │
                 │ Private IP          │
                 │ 10.0.1.4            │
                 └──────────┬──────────┘
                            │
                            │ TCP :8080
                            │ Private IP
                            ▼
                 ┌─────────────────────┐
                 │     BACKEND VM      │
                 │                     │
                 │ Private IP          │
                 │ 10.0.2.4            │
                 │                     │
                 │ Public IP: None     │
                 └──────────┬──────────┘
                            │
                            │ Outbound
                            ▼
                    ┌──────────────┐
                    │ NAT Gateway  │
                    └──────┬───────┘
                           │
                           ▼
                        INTERNET
```

## Architecture Diagram

<img width="1536" height="1024" alt="ChatGPT Image Aug 21, 2026, 04_25_01 AM" src="https://github.com/user-attachments/assets/90d22e3d-8f09-49e7-8e77-c9da8ae7f9cb" />

---

# ☁️ Azure Architecture

The Azure environment consists of a single Virtual Network with two separate subnets.

```text
two-tier-vnet
│
├── frontend-subnet
│   └── frontend-vm
│
└── backend-subnet
    └── backend-vm
```

This separation provides a basic network boundary between the public-facing frontend and the private backend.

---

# 🌐 Network Design

## Virtual Network

The Virtual Network created for the project is:

```text
Name: two-tier-vnet
```

The VNet contains two subnets.

## Frontend Subnet

```text
Name: frontend-subnet
Address Range: 10.0.1.0/24
```

The frontend VM is deployed inside this subnet.

## Backend Subnet

```text
Name: backend-subnet
Address Range: 10.0.2.0/24
```

The backend VM is deployed inside this subnet.

---

# 🖥️ Frontend Tier

The frontend VM is the public-facing component of the application.

```text
VM Name: frontend-vm

Private IP:
10.0.1.4

Public IP:
4.222.218.179

Subnet:
frontend-subnet
```

The frontend hosts the Flask web application and listens on TCP port `8000`.

The frontend is intentionally the only application tier directly accessible from the public internet.

---

## 📸 Frontend VM

<img width="1919" height="959" alt="Screenshot 2026-08-19 084927" src="https://github.com/user-attachments/assets/55bd5d11-745c-473f-a7b7-251dce91865f" />


---

# 🖥️ Backend Tier

The backend VM is deployed inside the private backend subnet.

```text
VM Name: backend-vm

Private IP:
10.0.2.4

Public IP:
None

Subnet:
backend-subnet
```

The backend hosts the Flask API and listens on TCP port `8080`.

The backend does not have a direct public IP address.

This means the backend cannot be directly accessed from the public internet.

---

## 📸 Backend VM

<img width="1919" height="959" alt="Screenshot 2026-08-19 085005" src="https://github.com/user-attachments/assets/70832dfb-9a87-45ab-88f6-3818b7f7cd0c" />


---

# 🔐 Network Security Groups

Network Security Groups were used to control traffic entering the frontend and backend tiers.

The project uses separate security controls for each tier:

```text
Frontend NSG
     │
     └── Controls frontend traffic


Backend NSG
     │
     └── Controls backend traffic
```

This allows each tier to have different security rules based on its role.

---

# 🛡️ Frontend NSG

The frontend requires public access because it provides the user-facing web application.

The required application traffic is:

```text
Internet
   │
   │ TCP :8000
   ▼
Frontend VM
```

SSH access was also enabled for administration during the project.

```text
Internet
   │
   │ TCP :22
   ▼
Frontend VM
```

The frontend NSG therefore allows the required public application traffic and administrative SSH access.

---

## 📸 Frontend NSG

<img width="1919" height="960" alt="Screenshot 2026-08-19 084720" src="https://github.com/user-attachments/assets/383793ae-0d3a-4d53-923a-52d66012ef1a" />

---

# 🔒 Backend NSG

The backend has a more restrictive security configuration.

The backend application listens on:

```text
TCP :8080
```

However, the backend should only receive application traffic from the frontend tier.

The intended traffic flow is:

```text
Frontend Subnet
10.0.1.0/24
      │
      │ TCP :8080
      ▼
Backend Subnet
10.0.2.0/24
```

Direct public access is not permitted.

```text
Internet
    │
    │ TCP :8080
    ▼
Backend VM

Result: DENIED
```

This prevents users from bypassing the frontend and directly accessing the backend API.

---

## 📸 Backend NSG

<img width="1919" height="969" alt="Screenshot 2026-08-19 084819" src="https://github.com/user-attachments/assets/aa5c93ae-d97c-47ea-a197-364a106d554e" />

---

# 🌍 NAT Gateway

The backend VM does not have a public IP address.

However, the backend still needs outbound internet connectivity for activities such as:

- Updating Ubuntu
- Installing packages
- Installing Python dependencies
- Downloading application dependencies
- Accessing external services when required

To provide this connectivity, a NAT Gateway was created and associated with the backend subnet.

```text
Backend VM
10.0.2.4
     │
     ▼
backend-subnet
     │
     ▼
NAT Gateway
     │
     ▼
Internet
```

The NAT Gateway provides **outbound connectivity** without assigning a public IP directly to the backend VM.

---

## 📸 NAT Gateway Association

The NAT Gateway was associated with `backend-subnet`.

---

# 🧩 Application Architecture

The application consists of two Flask applications.

```text
                   INTERNET
                      │
                      ▼
              ┌───────────────┐
              │   FRONTEND    │
              │    Flask      │
              │    :8000      │
              └───────┬───────┘
                      │
                      │ HTTP :8080
                      ▼
              ┌───────────────┐
              │    BACKEND    │
              │    Flask      │
              │    :8080      │
              └───────┬───────┘
                      │
                      ▼
                NAT Gateway
                      │
                      ▼
                   INTERNET
```

---

# 🌐 Frontend Application

The frontend application runs on the frontend VM.

```text
Frontend VM
10.0.1.4
```

Gunicorn listens on:

```text
0.0.0.0:8000
```

The frontend provides the user-facing web interface.

When a user visits the frontend, the frontend makes a request to the backend API using the backend's private IP address.

---

# ⚙️ Backend Application

The backend application runs on:

```text
Backend VM
10.0.2.4
```

Gunicorn listens on:

```text
0.0.0.0:8080
```

The backend exposes an API endpoint:

```text
/api/status
```

The endpoint returns:

```json
{
  "message": "Hello from the backend!",
  "status": "success"
}
```

---

# 🔗 Frontend-to-Backend Communication

The frontend communicates with the backend using the backend's private IP address.

```text
Frontend VM
10.0.1.4
     │
     │ TCP :8080
     │
     ▼
Backend VM
10.0.2.4
```

The communication occurs over the Azure Virtual Network.

The backend does not need a public IP address for the frontend to communicate with it.

---

## 🧪 Connectivity Test

From the frontend VM, the backend API was tested with:

```bash
curl http://10.0.2.4:8080/api/status
```

The backend successfully returned:

```json
{
  "message": "Hello from the backend!",
  "status": "success"
}
```
---

# 🚫 Backend Internet Isolation

The backend VM has no public IP address.

```text
Backend Private IP:
10.0.2.4

Public IP:
None
```

Therefore, a public internet user cannot directly connect to the backend application.

The intended access pattern is:

```text
Internet
   │
   ▼
Frontend
   │
   │ Private Network
   │ TCP :8080
   ▼
Backend
```

Instead of:

```text
Internet
   │
   │ Direct Access
   ▼
Backend
```

This architecture reduces the public attack surface by keeping the backend private.

---

# 🐍 Flask

Python Flask was used to build both application tiers.

The backend exposes an API endpoint that returns a JSON response.

A simplified version of the backend application is:

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route("/api/status")
def status():
    return jsonify({
        "message": "Hello from the backend!",
        "status": "success"
    })
```

The frontend application consumes the backend response and displays it to the user.

---

# 🦄 Gunicorn

Initially, the Flask applications were started manually using:

```bash
python app.py
```

This was useful during development and connectivity testing.

However, running Flask directly in the terminal creates a problem.

If the SSH session is closed, the foreground application process can stop.

To create a more persistent deployment, Gunicorn was introduced.

The final application process is:

```text
Flask Application
       │
       ▼
    Gunicorn
       │
       ▼
     systemd
```

Gunicorn version used:

```text
26.1.0
```

---

# ⚙️ systemd

Linux `systemd` was used to manage both applications as persistent services.

This solved the problem of having to keep an SSH terminal open.

The final architecture is:

```text
systemd
   │
   ▼
Gunicorn
   │
   ▼
Flask
```

The services were configured to:

- Start automatically
- Run independently of SSH sessions
- Restart when necessary
- Start automatically after VM reboot
- Allow service status to be checked using `systemctl`

---

# 🖥️ Backend systemd Service

The backend service is:

```text
backend-app.service
```

The service runs Gunicorn from the backend virtual environment.

The backend application is exposed internally on:

```text
0.0.0.0:8080
```

The service was enabled using:

```bash
sudo systemctl enable backend-app
```

The service can be checked using:

```bash
sudo systemctl status backend-app
```

Expected status:

```text
Active: active (running)
```
---

# 🖥️ Frontend systemd Service

The frontend service is:

```text
frontend-app.service
```

The service runs Gunicorn from the frontend virtual environment.

The frontend application is exposed on:

```text
0.0.0.0:8000
```

The service was enabled using:

```bash
sudo systemctl enable frontend-app
```

The service can be checked using:

```bash
sudo systemctl status frontend-app
```

Expected status:

```text
Active: active (running)
```

---

# 🔄 Application Persistence

One of the major lessons from this project was understanding the difference between running an application manually and running it as a system service.

## ❌ Initial Approach

The application was initially started with:

```bash
python app.py
```

This meant the application was attached to the terminal session.

The architecture was effectively:

```text
SSH Terminal
     │
     ▼
python app.py
     │
     ▼
Flask
```

Closing the terminal could stop the application.

---

## ✅ Final Approach

The final deployment uses:

```text
systemd
   │
   ▼
Gunicorn
   │
   ▼
Flask
```

The SSH session is no longer responsible for keeping the application alive.

Therefore:

```text
SSH Session
     │
     X
   CLOSED
     │
     ▼
systemd
     │
     ▼
Gunicorn
     │
     ▼
Flask
     │
     ▼
Application remains available
```

---

# 🔁 Reboot Testing

A major requirement of the final deployment was proving that the applications automatically restart after a VM reboot.

Both the backend and frontend VMs were rebooted independently.

---

# 🔁 Backend Reboot Test

Before rebooting the backend, the service was configured to start automatically.

The backend VM was rebooted.

After the VM came back online, the service was checked using:

```bash
sudo systemctl status backend-app
```

The service returned:

```text
Active: active (running)
```

The backend API was then tested again from the frontend VM.

```bash
curl http://10.0.2.4:8080/api/status
```

The API successfully returned:

```json
{
  "message": "Hello from the backend!",
  "status": "success"
}
```

This confirmed that the backend application automatically started after the VM reboot.

---

# 🔁 Frontend Reboot Test

The frontend VM was also rebooted.

After the reboot, the frontend service was checked using:

```bash
sudo systemctl status frontend-app
```

The service returned:

```text
Active: active (running)
```

The application was then accessed through the frontend public IP.

The frontend successfully communicated with the backend after the reboot.

---

# 🌎 Final Application Test

The final application was accessed through:

```text
http://4.222.218.179:8000
```

The application displayed:

```text
Azure Two-Tier Application

Backend Response

Status: success

Hello from the backend!
```

This confirms the complete application flow:

```text
Internet
   │
   ▼
Frontend Public IP
   │
   ▼
Frontend VM
   │
   ▼
Frontend Flask Application
   │
   │ Private TCP :8080
   ▼
Backend VM
   │
   ▼
Backend Flask API
   │
   ▼
Successful Response
```

---

## 📸 Final Application

<img width="1919" height="1079" alt="Screenshot 2026-08-19 085739" src="https://github.com/user-attachments/assets/0f815d5e-a1e9-4e8a-9d34-0264cceaae5b" />

---

# 🧪 Testing Summary

| Test | Expected Result | Result |
|---|---|---|
| Internet → Frontend | Allowed | ✅ Passed |
| Internet → Backend | Blocked | ✅ Passed |
| Frontend → Backend | Allowed | ✅ Passed |
| Backend → Internet | Allowed through NAT Gateway | ✅ Passed |
| Backend has no public IP | Private | ✅ Passed |
| Backend survives SSH disconnect | Application remains running | ✅ Passed |
| Frontend survives SSH disconnect | Application remains running | ✅ Passed |
| Backend survives VM reboot | Service starts automatically | ✅ Passed |
| Frontend survives VM reboot | Service starts automatically | ✅ Passed |
| Frontend application loads | Successful | ✅ Passed |
| Backend API responds | Successful | ✅ Passed |

---

# 🔐 Security Design

The project follows a basic defense-in-depth approach.

## Public Frontend

The frontend is intentionally public because users need access to the application.

```text
Internet
   │
   ▼
Frontend VM
```

## Private Backend

The backend is intentionally private.

```text
Internet
   │
   X
   │
Backend VM
```

The backend is only intended to receive application traffic from the frontend.

```text
Frontend
   │
   │ TCP :8080
   ▼
Backend
```

## Outbound Backend Connectivity

The backend can initiate outbound connections through the NAT Gateway.

```text
Backend
   │
   ▼
NAT Gateway
   │
   ▼
Internet
```

This provides a useful distinction between:

- Inbound internet exposure
- Private application communication
- Outbound internet connectivity

---

# 🧠 Key Cloud Engineering Concepts Learned

## 1. Network Segmentation

Separating frontend and backend resources into different subnets creates a logical security boundary.

```text
Frontend Subnet
10.0.1.0/24

Backend Subnet
10.0.2.0/24
```

---

## 2. Private IP Communication

The frontend can communicate with the backend using:

```text
10.0.2.4:8080
```

without requiring the backend to have a public IP.

---

## 3. Network Security Groups

NSGs provide traffic filtering at the network level.

The frontend and backend have different traffic requirements, so separate NSG configurations were used.

---

## 4. NAT Gateway

A NAT Gateway allows private resources to access the internet outbound without exposing them to direct inbound internet connections.

This was particularly useful for the backend VM when installing packages and updating Ubuntu.

---

## 5. Gunicorn

Gunicorn provides a production-oriented WSGI application server for the Flask applications.

The final application stack is:

```text
Gunicorn
    │
    ▼
Flask
```

---

## 6. systemd

systemd provides service management for the applications.

Instead of depending on an SSH session:

```text
SSH
 │
 └── python app.py
```

the final architecture uses:

```text
systemd
   │
   └── Gunicorn
          │
          └── Flask
```

---

## 7. Service Persistence

The applications were tested after closing SSH sessions and rebooting the VMs.

Both applications automatically returned because their systemd services were enabled.

---

# 🧩 Challenges and Lessons Learned

## Challenge 1 — Understanding SSH and Application Processes

Initially, the Flask applications were started directly from an SSH terminal.

This led to an important lesson:

> SSH provides access to the VM, but it should not be responsible for keeping an application running.

The application process needs to be managed independently.

---

## Challenge 2 — Flask Development Server vs Gunicorn

Flask's built-in development server was useful for initial testing but was not appropriate as the final application process.

Gunicorn was introduced as the application server.

The final architecture became:

```text
systemd
   ↓
Gunicorn
   ↓
Flask
```

---

## Challenge 3 — Private Backend Connectivity

The backend did not have a public IP.

The frontend therefore had to communicate with:

```text
10.0.2.4
```

over the private Azure network.

This demonstrated how application tiers can communicate privately without exposing every component to the internet.

---

## Challenge 4 — Backend Outbound Connectivity

The private backend initially experienced connectivity issues when attempting to access external package repositories.

A NAT Gateway was implemented and associated with the backend subnet.

This provided the backend with outbound internet access while keeping it private from inbound internet traffic.

---

## Challenge 5 — Automatic Startup

Manually starting applications with:

```bash
python app.py
```

was not sufficient for a persistent cloud deployment.

Using systemd solved this problem by allowing the applications to automatically start after VM reboots.

---

# 📊 Final Traffic Model

The final traffic model can be summarized as:

```text
                    ┌──────────────┐
                    │   INTERNET   │
                    └──────┬───────┘
                           │
                           │ TCP :8000
                           ▼
                    ┌──────────────┐
                    │  FRONTEND    │
                    │  10.0.1.4    │
                    └──────┬───────┘
                           │
                           │ TCP :8080
                           │ Private
                           ▼
                    ┌──────────────┐
                    │   BACKEND    │
                    │  10.0.2.4    │
                    └──────┬───────┘
                           │
                           │ Outbound
                           ▼
                    ┌──────────────┐
                    │ NAT GATEWAY  │
                    └──────┬───────┘
                           │
                           ▼
                        INTERNET
```

---

# 📋 Resource Summary

| Component | Configuration |
|---|---|
| Virtual Network | `two-tier-vnet` |
| Frontend Subnet | `frontend-subnet` |
| Frontend CIDR | `10.0.1.0/24` |
| Backend Subnet | `backend-subnet` |
| Backend CIDR | `10.0.2.0/24` |
| Frontend VM | `frontend-vm` |
| Frontend Private IP | `10.0.1.4` |
| Frontend Public IP | `4.222.218.179` |
| Frontend Port | `8000` |
| Backend VM | `backend-vm` |
| Backend Private IP | `10.0.2.4` |
| Backend Public IP | None |
| Backend Port | `8080` |
| NAT Gateway | `backend-nat-gateway` |
| Frontend Service | `frontend-app.service` |
| Backend Service | `backend-app.service` |
| Application Framework | Flask |
| Application Server | Gunicorn |
| Python Version | `3.12.3` |
| Gunicorn Version | `26.1.0` |
| Operating System | Ubuntu Linux |

---

# 🛠️ Technologies Used

## Cloud

- Microsoft Azure
- Azure Virtual Network
- Azure Virtual Machines
- Azure Network Security Groups
- Azure NAT Gateway
- Azure Public IP
- Azure Network Interfaces
- Azure Storage Account

## Operating System

- Ubuntu Linux

## Application

- Python 3.12
- Flask 3.1.3
- Gunicorn 26.1.0

## Networking

- IPv4
- TCP
- Private IP addressing
- Public IP addressing
- VNet subnetting
- Network Security Groups
- NAT Gateway

## Linux Administration

- SSH
- systemd
- systemctl
- curl
- pip
- Python virtual environments

---

# ✅ Project Status

## Completed

The Azure two-tier application has been successfully deployed and tested.

The final environment demonstrates:

- [x] Azure Virtual Network
- [x] Frontend subnet
- [x] Backend subnet
- [x] Frontend Linux VM
- [x] Backend Linux VM
- [x] Frontend public IP
- [x] Backend private IP
- [x] Network Security Groups
- [x] Private frontend-to-backend communication
- [x] Backend internet isolation
- [x] NAT Gateway
- [x] Backend outbound internet connectivity
- [x] Flask frontend
- [x] Flask backend
- [x] Gunicorn
- [x] systemd
- [x] Persistent application services
- [x] Backend reboot test
- [x] Frontend reboot test
- [x] End-to-end application test

---

# 🎓 Conclusion

This project provided hands-on experience designing and deploying a two-tier application architecture in Microsoft Azure.

The most important architectural principle demonstrated was the separation of **public-facing and private application components**.

The frontend is exposed to users through a public IP address, while the backend remains private and communicates with the frontend through the Azure Virtual Network.

The NAT Gateway provides outbound connectivity for the backend without exposing it to inbound internet traffic.

Finally, Gunicorn and systemd transformed the Flask applications from simple processes running inside SSH sessions into persistent Linux services capable of automatically starting after VM reboots.

The final architecture demonstrates practical knowledge of:

```text
Azure Networking
        +
Network Security
        +
Linux Administration
        +
Python Application Deployment
        +
Private/Public Network Design
        +
NAT Gateway
        +
Gunicorn
        +
systemd
        +
Application Testing
```

---

# 👨‍💻 Author

**Martins**
---

## ⭐ If you found this project useful

Feel free to explore the repository and follow the progression of my cloud engineering projects as I continue building practical Azure infrastructure and application deployments.
