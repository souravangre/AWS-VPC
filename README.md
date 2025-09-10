<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
</head>
<body>
<main>

<!-- ============ TITLE & TAGLINE ============ -->
<h1> AWS VPC – Secure Web Hosting with Private EC2 & Bastion</h1>
<p class="note">—</strong> A production-ready VPC architecture that keeps compute, data nodes <em>private</em>, funnels administration through a Bastion host, and lets application servers reach the internet via highly-available NAT Gateways.</p>

<!-- ============ OVERVIEW ============ -->
<h2>📌 Overview</h2>
<p>
  This project illustrates how to design and deploy a two-AZ, high-availability network in AWS that:
</p>
<ul>
  <li>Deploys <strong>application EC2 instances</strong> in <em>public subnets</em> for direct ALB access.</li>
  <li>Isolates <strong>MySQL RDS</strong> (Primary + Standby) in <em>private subnets</em> across AZs.</li>
  <li>Includes a <strong>Private EC2 instance</strong> for backend tasks or app connections.</li>
  <li>Provides controlled SSH access through a hardened <strong>Bastion Host</strong>.</li>
  <li>Enables outbound access via redundant <strong>NAT Gateways</strong>.</li>
  <li>Uses an <strong>Application Load Balancer</strong> to handle traffic from the Internet Gateway.</li>
</ul>

<!-- ============ ARCHITECTURE DIAGRAM ============ -->
<h2>🏗️ Architecture Diagram</h2>
<div class="diagram">
  <img src="https://github.com/user-attachments/assets/9ddba0da-0e86-4c00-ace3-a36fa58af44b"  alt="AWS VPC Architecture Diagram">
</div>

<h2>🧰 AWS Services Used</h2>
<table>
  <thead>
    <tr>
      <th>Service</th>
      <th>Purpose</th>
    </tr>
  </thead>
  <tbody>
    <tr><td><strong>Amazon VPC</strong></td><td>Custom VPC with 2 public and 2 private subnets across AZs.</td></tr>
    <tr><td><strong>EC2</strong></td><td>Instances for apps (public), backend (private), and Bastion host.</td></tr>
    <tr><td><strong>RDS</strong></td><td>MySQL DB hosted in private subnets with Multi-AZ setup.</td></tr>
    <tr><td><strong>ALB</strong></td><td>Routes HTTP traffic from users to public EC2 app instances.</td></tr>
    <tr><td><strong>Bastion Host</strong></td><td>Secure entry point for SSH into private resources.</td></tr>
    <tr><td><strong>Security Groups</strong></td><td>Restrict traffic to/from EC2, RDS, ALB.</td></tr>
    <tr><td><strong>Route Tables</strong></td><td>Separate tables for public/private subnets.</td></tr>
    <tr><td><strong>Internet Gateway</strong></td><td>Internet access for public subnets.</td></tr>
    <tr><td><strong>NAT Gateway</strong></td><td>Outbound access for private subnet instances.</td></tr>
  </tbody>
</table>

<!-- ============ CIDR & ROUTING ============ -->
<h2>📀 CIDR Blocks & Routing</h2>
<ul>
  <li><strong>VPC CIDR</strong> – <code>172.16.0.0/16</code></li>
  <li><strong>Subnets</strong>:
    <ul>
      <li>Public AZ-A: <code>172.16.0.0/24</code></li>
      <li>Private AZ-A: <code>172.16.1.0/24</code></li>
      <li>Public AZ-B: <code>172.16.2.0/24</code></li>
      <li>Private AZ-B: <code>172.16.3.0/24</code></li>
    </ul>
  </li>
  <li><strong>Routing</strong>:
    <ul>
      <li>Public RT: <code>0.0.0.0/0 → IGW</code></li>
      <li>Private RTs: <code>0.0.0.0/0 → NAT GW (per AZ)</code></li>
    </ul>
  </li>
</ul>

<!-- ============ SECURITY ============ -->
<h2>🔒 Security Groups</h2>
<ul>
  <li><code>sg_bastion</code>: SSH from admin IPs.</li>
  <li><code>sg_app</code>: HTTP from ALB only.</li>
  <li><code>sg_alb</code>: HTTP (80) from 0.0.0.0/0.</li>
  <li><code>sg_rds</code>: MySQL (3306) from private EC2 SG only.</li>
</ul>

<!-- ============ DEPLOYMENT STEPS ============ -->
<h2>🚀 Deployment Steps</h2>
<ol>
  <li>Create VPC and subnets (4 total: 2 public, 2 private)</li>
  <li>Attach Internet Gateway</li>
  <li>Set up route tables with correct associations</li>
  <li>Provision NAT Gateways in public subnets</li>
  <li>Launch EC2: Bastion (public), App (public), Private EC2 (private)</li>
  <li>Configure RDS in Multi-AZ mode inside private subnets</li>
  <li>Set up Security Groups and Key Pairs</li>
  <li>Create ALB targeting both public EC2 app instances</li>
  <li>Test via ALB DNS (should route to your apps)</li>
</ol>

<!-- ============ FUTURE ============ -->
<h2>📈 Future Work</h2>
<ul>
  <li>Attach ACM SSL to ALB for HTTPS</li>
  <li>Use CloudWatch + CloudTrail for monitoring</li>
  <li>Add Auto Scaling Group for public app EC2s</li>
  <li>Write Infrastructure as Code (IaC) using Terraform or CDK</li>
</ul>

<h2>🤝 Contact</h2>
<p>
  Built by <a href="https://github.com/souravangre" target="_blank">@souravangre</a>
</p>

</main>
</body>
</html>
