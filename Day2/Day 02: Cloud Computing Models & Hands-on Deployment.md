# Day 02: Cloud Computing Models & Hands-on Deployment
## 📌 Table of Contents

1. [Overview of Cloud Computing Models](#1-overview-of-cloud-computing-models)

2. [Cloud Service Models (IaaS, PaaS, SaaS)](#2-cloud-service-models-iaas-paas-saas)

   * [Real-World Analogy: Pizza as a Service](#real-world-analogy-pizza-as-a-service)

3. [Cloud Deployment Models](#3-cloud-deployment-models)

4. [Practical Understanding: Deploying a Web Server](#4-practical-understanding-deploying-a-web-server)

5. [Key Takeaways for Quick Revision](#5-key-takeaways-for-quick-revision)

## 1. Overview of Cloud Computing Models

Cloud computing relies on two main categories to define how resources are built, managed, and accessed:

```
                  ┌─────────────────────────────┐
                  │   Cloud Computing Models    │
                  └──────────────┬──────────────┘
                                 │
         ┌───────────────────────┴───────────────────────┐
         ▼                                               ▼
┌──────────────────┐                           ┌──────────────────┐
│  Service Models  │                           │ Deployment Models│
├──────────────────┤                           ├──────────────────┤
│ 1. IaaS          │                           │ 1. Public Cloud  │
│ 2. PaaS          │                           │ 2. Private Cloud │
│ 3. SaaS          │                           │ 3. Hybrid Cloud  │
└──────────────────┘                           └──────────────────┘

```

## 2. Cloud Service Models (IaaS, PaaS, SaaS)

Service models define **how much control you have** versus **how much the provider manages for you**.

### 🛠️ 1. Infrastructure as a Service (IaaS)

* **Definition:** The cloud provider gives you access to basic compute, networking, and storage resources. You have maximum control over the operating system, server configurations, runtime environment, and security settings.

* **Who uses it:** System Administrators, Cloud Architects, and DevOps Engineers.

* **Real-World Analogy:** **Renting an Empty House** 🏠

  * The landlord gives you the bare structure (walls, roof, power lines).

  * You bring your own furniture, paint the walls, choose the layout, and manage security.

* **Examples:** AWS (Amazon EC2, Amazon EBS), Microsoft Azure VMs, Google Compute Engine (GCE).

### 💻 2. Platform as a Service (PaaS)

* **Definition:** The provider manages the underlying infrastructure, operating system, patch updates, and runtime engine. You only provide your code and data.

* **Who uses it:** Software Developers.

* **Real-World Analogy:** **Renting a Fully Furnished Apartment** 🛋️

  * The apartment is pre-fitted with furniture, utilities, and maintainers.

  * You just walk in with your clothes and start living without worrying about repairs or buying items.

* **Examples:** AWS Elastic Beanstalk, Heroku, Google App Engine.

### 🌐 3. Software as a Service (SaaS)

* **Definition:** A complete, fully managed software application accessible directly over the web via a browser or mobile app. You do not manage any hardware, OS, or code base.

* **Who uses it:** End users and consumers.

* **Real-World Analogy:** **Ordering Takeout / Dining at a Restaurant** 🍕

  * You do not buy raw ingredients, cook, or clean dishes.

  * You simply sit down, consume the meal, and pay for what you ate.

* **Examples:** Google Workspace (Gmail, Drive), Microsoft 365, Netflix, Zoom, Salesforce.

### 🍕 Real-World Analogy: Pizza as a Service

| **Task / Component** | **Traditional On-Premises** | **IaaS** | **PaaS** | **SaaS** | 
| **Dining Table & Drinks** | You Manage | You Manage | You Manage | Provider Manages | 
| **Oven & Gas** | You Manage | You Manage | Provider Manages | Provider Manages | 
| **Pizza Dough & Toppings** | You Manage | You Manage | Provider Manages | Provider Manages | 
| **Kitchen Infrastructure** | You Manage | Provider Manages | Provider Manages | Provider Manages | 
| **Analogy Equivalent** | Made at home from scratch | Rent a kitchen setup | Pizza delivery to door | Eat at a Restaurant | 

## 3. Cloud Deployment Models

Deployment models specify **where** your cloud environment is physically and logically hosted and **who** can access it.

```
       PUBLIC CLOUD                   PRIVATE CLOUD                   HYBRID CLOUD
   ┌───────────────────┐          ┌───────────────────┐          ┌───────────────────┐
   │ Shared Infra      │          │ Dedicated Infra   │          │ On-Premises Data  │
   │ Access: Internet  │          │ Access: Private   │          │ Center + AWS      │
   │ Ex: AWS, Azure    │          │ Ex: On-Premises   │          │ Connected via DirectConnect│
   └───────────────────┘          └───────────────────┘          └───────────────────┘

```

### 1. Public Cloud

* **Definition:** Computing resources are owned and operated by a third-party cloud provider and shared among multiple customers (*tenants*) over the public internet.

* **Key Advantages:** Cost-effective, zero hardware maintenance, unlimited scalability.

* **Example Use Case:** Hosting a public web server or blog on AWS EC2.

### 2. Private Cloud

* **Definition:** Cloud infrastructure dedicated exclusively to a single business or enterprise. It can be physically located on-site or hosted by a third-party provider.

* **Key Advantages:** Maximum security, strict compliance, full isolation.

* **Example Use Case:** Financial banking applications handling sensitive customer data.

### 3. Hybrid Cloud

* **Definition:** Combines public and private clouds, allowing data and applications to be shared between them securely over encrypted connections.

* **Key Advantages:** Flexibility—keep sensitive workloads on private servers while offloading high-traffic web traffic to the public cloud.

* **Example Use Case:** A bank keeping core customer balances on a private database while hosting their mobile application frontend on AWS.

## 4. Practical Understanding: Deploying a Web Server

Today's practical objective is to run a live web server on AWS cloud.

### 🛠️️ What happens behind the scenes during deployment?

1. **Select Service Model (IaaS - Amazon EC2):**

   * Launch a virtual server instance (e.g., Ubuntu Linux).

2. **Configure Networking & Security:**

   * Open Network Ports: Allow **HTTP (Port 80)** traffic so users can access your website over the browser, and **SSH (Port 22)** for administrative remote login.

3. **Install Application Stack:**

   * Connect to the virtual server via SSH.

   * Install a web server software like **Apache (httpd)** or **Nginx**.

4. **Go Live:**

   * Copy your HTML code into the server directory.

   * Access the live site using the server's **Public IP address**.

## 5. Key Takeaways for Quick Revision

* **IaaS:** You manage OS & applications; Provider manages physical hardware (e.g., AWS EC2).

* **PaaS:** You manage code/data; Provider manages OS & infrastructure (e.g., AWS Elastic Beanstalk).

* **SaaS:** Fully managed product ready to use (e.g., Gmail, Office 365).

* **Public Cloud:** Shared infrastructure over the open internet.

* **Private Cloud:** Dedicated infrastructure for a single tenant.

* **Hybrid Cloud:** Combination of private on-premise infrastructure and public cloud services.

```

```
