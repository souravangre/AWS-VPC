<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>VPC-Based Secure Web Hosting with Private EC2 & ALB</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      line-height: 1.6;
      background-color: #f8f9fa;
      padding: 20px;
      color: #333;
    }
    h1 {
      color: #005792;
    }
    h2 {
      color: #003f5c;
      border-bottom: 2px solid #ccc;
      padding-bottom: 5px;
    }
    ul {
      list-style-type: square;
      margin-left: 20px;
    }
    .tagline {
      background: #e8f0fe;
      padding: 10px 15px;
      border-left: 4px solid #1976d2;
      font-style: italic;
      margin-bottom: 20px;
    }
    code {
      background: #eee;
      padding: 2px 4px;
      font-family: monospace;
      border-radius: 4px;
    }
  </style>
</head>
<body>

  <h1>📘 VPC-Based Secure Web Hosting with Private EC2 & ALB</h1>

  <div class="tagline">
    This project demonstrates how to securely host a website on a private EC2 instance in a private subnet using an ALB and Bastion Host.
  </div>

  <h2>📌 Overview</h2>
  <p>This setup includes:</p>
  <ul>
    <li><strong>Bastion Host</strong> for SSH access</li>
    <li><strong>Internet-facing Application Load Balancer (ALB)</strong> to serve public traffic</li>
    <li><strong>Custom VPC</strong> setup with public and private subnets</li>
    <li><strong>Apache web server</strong> installed and configured on Ubuntu</li>
    <li>Website securely hosted in a private subnet, accessible only via ALB</li>
  </ul>

  <h2>🧰 Technologies Used</h2>
  <ul>
    <li>AWS EC2</li>
    <li>AWS VPC (Custom)</li>
    <li>Subnets (Public & Private)</li>
    <li>Internet Gateway & Routing Tables</li>
    <li>Application Load Balancer</li>
    <li>Security Groups</li>
    <li>Bastion Host for SSH Access</li>
    <li>Apache Web Server on Ubuntu</li>
    <li>Linux Tools: <code>scp</code>, <code>ssh</code>, <code>wget</code>, <code>netstat</code></li>
  </ul>

</body>
</html>
