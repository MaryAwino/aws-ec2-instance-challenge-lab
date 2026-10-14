# AWS EC2 Instance Challenge Lab — Step-by-Step Walkthrough

## 1. Project Overview

This walkthrough documents how I deployed a public web server on AWS using Amazon EC2, a custom Virtual Private Cloud (VPC), and Apache HTTP Server.

I configured the network, internet routing, security rules, EC2 instance, and Apache web server. I then created a custom HTML webpage and verified that it was accessible through a browser.

### Objectives

- Build a custom AWS network.
- Configure public subnet connectivity.
- Apply security group rules.
- Launch an Amazon Linux EC2 instance.
- Automate Apache installation using EC2 user data.
- Create and publish a custom HTML webpage.
- Validate the deployment using terminal commands and a web browser.

## 2. Architecture and Resource Names

| Resource | Name or configuration |
|---|---|
| VPC | `Challenge-VPC` |
| VPC CIDR | `10.0.0.0/16` |
| Subnet | `Challenge-Subnet` |
| Subnet CIDR | `10.0.1.0/24` |
| Internet Gateway | `Challenge-IGW` |
| Route table | `Challenge-Route-Table-2` |
| Security group | `Challenge-Web-SG` |
| EC2 instance | `Challenge-Web-Server` |
| Operating system | Amazon Linux 2023 |
| Instance type | `t3.micro` |
| Web server | Apache HTTP Server (`httpd`) |
| Webpage | `/var/www/html/projects.html` |

## 3. Step 1 — Create the VPC

A Virtual Private Cloud (VPC) is a logically isolated network where AWS resources can be deployed.

### Configuration

1. Open the AWS Management Console.
2. Search for and open **VPC**.
3. Select **Your VPCs**.
4. Click **Create VPC**.
5. Choose the option to create a VPC only.
6. Enter the name `Challenge-VPC`.
7. Set the IPv4 CIDR block to `10.0.0.0/16`.
8. Create the VPC.

### Validation

Confirm that `Challenge-VPC` appears in the VPC list and its state is **Available**.

## 4. Step 2 — Create the Subnet

A subnet is a smaller network range within a VPC. The EC2 instance will be deployed into this subnet.

### Configuration

1. In the VPC console, select **Subnets**.
2. Click **Create subnet**.
3. Select `Challenge-VPC`.
4. Enter the subnet name `Challenge-Subnet`.
5. Choose an Availability Zone available in the lab.
6. Set the IPv4 subnet CIDR block to `10.0.1.0/24`.
7. Create the subnet.
8. Select the subnet and open **Actions → Edit subnet settings**.
9. Enable **Auto-assign public IPv4 address** and save.

### Validation

Confirm that the subnet belongs to `Challenge-VPC` and has the expected CIDR block.

## 5. Step 3 — Create the Internet Gateway

An Internet Gateway provides a path between a VPC and the internet when routing and security settings permit the traffic.

### Configuration

1. In the VPC console, select **Internet gateways**.
2. Click **Create internet gateway**.
3. Name it `Challenge-IGW`.
4. Create the gateway.
5. Select it and choose **Actions → Attach to a VPC**.
6. Select `Challenge-VPC`.
7. Attach the Internet Gateway.

### Validation

Confirm that the gateway state is **Attached** and that it is attached to the correct VPC.

## 6. Step 4 — Configure the Route Table

A route table determines where network traffic is directed.

### Configuration

1. In the VPC console, select **Route tables**.
2. Create a route table named `Challenge-Route-Table-2` in `Challenge-VPC` if it does not already exist.
3. Select the route table.
4. Open the **Routes** tab.
5. Choose **Edit routes → Add route**.
6. Set the destination to `0.0.0.0/0`.
7. Select `Challenge-IGW` as the target.
8. Save the routes.
9. Open **Subnet associations**.
10. Choose **Edit subnet associations**.
11. Select `Challenge-Subnet` and save.

### Expected routes

| Destination | Target | Purpose |
|---|---|---|
| `10.0.0.0/16` | `local` | Enables routing within the VPC |
| `0.0.0.0/0` | Internet Gateway | Routes internet-bound IPv4 traffic |

AWS automatically creates the local route for the VPC CIDR.

### Validation

Confirm that the route table has both routes and that `Challenge-Subnet` is explicitly associated with it.

## 7. Step 5 — Create the Security Group

A security group acts as a virtual firewall for the EC2 instance.

### Configuration

1. Open the EC2 console.
2. Select **Network & Security → Security Groups**.
3. Click **Create security group**.
4. Enter the name `Challenge-Web-SG`.
5. Select `Challenge-VPC`.
6. Add the following inbound rules.

| Type | Protocol | Port | Source |
|---|---|---|---|
| SSH | TCP | 22 | My IP |
| HTTP | TCP | 80 | Anywhere IPv4 (`0.0.0.0/0`) |

7. Leave the default outbound rule unless the lab instructions require otherwise.
8. Create the security group.

### Explanation

- **SSH, port 22:** Provides administrative access. Restricting the source to My IP reduces exposure.
- **HTTP, port 80:** Allows browsers to access the hosted webpage.

### Validation

Confirm that both inbound rules are present and the security group belongs to `Challenge-VPC`.

## 8. Step 6 — Launch the EC2 Instance

Amazon EC2 provides virtual servers that can run applications and services.

### Configuration

1. Open **EC2 → Instances**.
2. Click **Launch instances**.
3. Set the instance name to `Challenge-Web-Server`.
4. Choose **Amazon Linux 2023**.
5. Select instance type `t3.micro`.
6. Create and download a new key pair named `challenge-web-key.pem` if prompted by the lab.
7. Under network settings, choose:
   - VPC: `Challenge-VPC`
   - Subnet: `Challenge-Subnet`
   - Auto-assign public IP: Enabled
   - Security group: `Challenge-Web-SG`
8. Configure the root EBS volume as **gp2**, as required by the lab.
9. Expand **Advanced details** and locate **User data**.
10. Enter the startup script from the next section.
11. Launch the instance.

### Validation

Wait until the instance is **Running** and its status checks pass.

Keep the private key secure. Never upload the `.pem` file to GitHub.

## 9. Step 7 — Configure EC2 User Data

EC2 user data runs a startup script to automate initial instance configuration.

The following script was used in the lab:

```bash
#!/bin/bash
dnf install -y httpd
systemctl enable httpd
systemctl start httpd
chmod 777 /var/www/html
```

### Explanation

- `dnf install -y httpd` installs Apache HTTP Server.
- `systemctl enable httpd` configures Apache to start automatically at boot.
- `systemctl start httpd` starts Apache.
- `chmod 777 /var/www/html` gives all users read, write, and execute permissions on the directory.

**Security improvement:** The final command is intentionally documented as used in this lab, but `777` permissions are overly permissive for production. A production deployment should use appropriate ownership and restricted permissions instead.

### Validation

Review the EC2 system log and look for evidence that the `httpd` package was installed.

## 10. Step 8 — Connect to the Instance

EC2 Instance Connect provides browser-based access to supported EC2 instances.

### Configuration

1. In **EC2 → Instances**, select `Challenge-Web-Server`.
2. Click **Connect**.
3. Choose **EC2 Instance Connect**.
4. Click **Connect**.

### Validation

Confirm that the terminal opens and displays an Amazon Linux shell prompt.

## 11. Step 9 — Verify Apache

Run the following command in the EC2 terminal:

```bash
sudo systemctl status httpd
```

The expected status includes:

```text
Active: active (running)
```

This confirms that Apache is running.

Press `q` to exit the status view.

## 12. Step 10 — Create the Project Webpage

The webpage was created at `/var/www/html/projects.html`.

### Create the file

Run:

```bash
sudo nano /var/www/html/projects.html
```

Enter the following HTML:

```html
<!DOCTYPE html>
<html>
<body>
    <h1>Mary Awino's re/Start Project Work</h1>
    <p>EC2 Instance Challenge Lab</p>
</body>
</html>
```

Save the file in Nano using **Ctrl+O**, press **Enter**, then exit using **Ctrl+X**.

### Verify the file

Run:

```bash
cat /var/www/html/projects.html
```

Confirm that the HTML content appears in the terminal.

## 13. Step 11 — Test the Webpage

1. Return to **EC2 → Instances**.
2. Select `Challenge-Web-Server`.
3. Copy its public IPv4 address.
4. Open a browser and navigate to:

   `http://YOUR-PUBLIC-IP/projects.html`

5. Replace `YOUR-PUBLIC-IP` with the actual public IPv4 address.

### Expected result

The browser displays:

**Mary Awino's re/Start Project Work**

EC2 Instance Challenge Lab

This verifies that the webpage is served through Apache over HTTP.

## 14. Troubleshooting Notes

### The webpage does not load

Check the following:

- The EC2 instance is running.
- The instance has a public IPv4 address.
- `Challenge-Subnet` has auto-assign public IPv4 enabled.
- `Challenge-Route-Table-2` is associated with the subnet.
- The route `0.0.0.0/0` targets `Challenge-IGW`.
- `Challenge-IGW` is attached to `Challenge-VPC`.
- The security group allows inbound TCP port 80.
- Apache is running.

### Apache is not running

Check:

```bash
sudo systemctl status httpd
```

If necessary, start the service:

```bash
sudo systemctl start httpd
```

Review the EC2 system log for installation errors.

### The HTML file is missing

Check:

```bash
ls -l /var/www/html/
```

Confirm that the file is named `projects.html` and is located in `/var/www/html/`.

## 15. Evidence and Screenshots

Screenshots are stored separately in the repository's `screenshots/` directory.

They document:

- Apache installation in the EC2 system log.
- EC2 instance configuration.
- VPC and subnet configuration.
- Internet Gateway and route table configuration.
- Security group inbound rules.
- Apache service status.
- The final webpage in a browser.

Refer to the [`../screenshots/`](../screenshots/) directory for the visual evidence.

## 16. Key Learnings

Through this lab, I gained hands-on experience with:

- AWS VPC and subnet configuration.
- Internet Gateway and route table setup.
- Public IPv4 networking.
- Security group rules and network access.
- EC2 instance provisioning.
- Linux service management.
- Startup automation using user data.
- Apache web server configuration.
- HTML file creation and browser-based validation.
- Basic troubleshooting of cloud-hosted applications.

## 17. Cleanup and Cost Awareness

When the lab is complete, follow the lab provider's instructions to end the lab.

For resources in a personal AWS account, remove resources that are no longer needed to avoid ongoing charges. Terminating an instance alone does not necessarily delete every associated resource.

## Conclusion

This challenge provided practical experience building a public web server on AWS, from networking and security configuration to application deployment and validation.

The project strengthened my understanding of AWS infrastructure, Linux administration, web hosting, and basic cloud troubleshooting.

**Program:** AWS re/Start  
**Project:** EC2 Instance Challenge Lab
