<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
</head>
<body>
<main>

<!-- ============ TITLE & TAGLINE ============ -->
<h1>📘 AWS VPC – Secure Web Hosting with Private EC2 & Bastion</h1>
<p class="note"><strong>TL;DR&nbsp;—</strong> A production-ready VPC architecture that keeps compute, data and cache nodes <em>private</em>, funnels administration through a Bastion host, and lets application servers reach the internet via highly-available NAT Gateways.</p>

<!-- ============ OVERVIEW ============ -->
<h2>📌 Overview</h2>
<p>
  This project illustrates how to design and deploy a two-AZ, high-availability network in AWS that:
</p>
<ul>
  <li>Isolates <strong>application EC2 instances</strong>, <strong>MySQL&nbsp;RDS</strong> in <em>private subnets</em>.</li>
  <li>Provides controlled SSH administration through a hardened <strong>Bastion Host</strong> in the public tier.</li>
  <li>Enables outbound traffic (package updates, external APIs) via redundant <strong>NAT Gateways</strong>.</li>
  <li>Uses a single <strong>Internet Gateway</strong> to expose only what is necessary (e.g., an ALB in front of your app, or future services).</li>
</ul>

<!-- ============ ARCHITECTURE DIAGRAM ============ -->
<h2>🏗️ Architecture Diagram</h2>
<div class="diagram">
  <!-- Place your screenshot in docs/architecture.png or adjust the path -->
  <img src="docs/architecture.png" alt="VPC Architecture diagram with two AZs, NAT gateways, Bastion host, MySQL, Cache, and private EC2 instances">
</div>

<!-- ============ TECH STACK ============ -->
<h2>🧰 Tech Stack Used</h2>
<table>
  <thead>
    <tr><th>Category</th><th>Technology</th><th>Description</th></tr>
  </thead>
  <tbody>
    <tr><td>Backend</td><td><code>Flask</code></td><td>Lightweight Python web framework powering the e-commerce app</td></tr>
    <tr><td>Database</td><td><code>MySQL (RDS)</code></td><td>Relational database hosted in a private subnet using Amazon RDS</td></tr>
    <tr><td>DevOps</td><td><code>Docker</code></td><td>Containerizes the Flask app for consistent deployment</td></tr>
    <tr><td>App Hosting</td><td><code>Gunicorn</code></td><td>WSGI server used for serving the Flask app in production</td></tr>
    <tr><td>Frontend</td><td><code>HTML + Jinja2</code></td><td>Flask's templating engine to render dynamic content</td></tr>
    <tr><td>Search</td><td><code>SQLite (local)</code></td><td>Used optionally for fast local prototyping/testing before RDS integration</td></tr>
    <tr><td>Networking</td><td><code>VPC</code></td><td>Virtual network to logically isolate resources in the cloud</td></tr>
    <tr><td>Security</td><td><code>Security Groups</code></td><td>Firewall rules to control traffic between EC2, RDS, Bastion</td></tr>
    <tr><td>Internet Access</td><td><code>NAT Gateway</code></td><td>Provides internet access to private subnet resources</td></tr>
    <tr><td>SSH Access</td><td><code>Bastion Host</code></td><td>Jump server to SSH into private EC2 instances securely</td></tr>
    <tr><td>Cache</td><td><code>ElastiCache (Redis or Memcached)</code></td><td>Used for session or catalog caching in private subnet</td></tr>
    <tr><td>Container Base</td><td><code>python:3.8-slim</code></td><td>Lightweight image to keep container size small and efficient</td></tr>
  </tbody>
</table>

<!-- ============ COMPONENTS ============ -->
<h2>🔧 Components</h2>

<table>
<thead>
<tr><th>Layer</th><th>Resource</th><th>Description</th></tr>
</thead>
<tbody>
<tr><td rowspan="2"><span class="badge">Public&nbsp;Subnet</span></td><td>Bastion Host<br><code>t3.micro (ssh-only)</code></td><td>Single entry point for administration; locked to specific IPs via Security Group rules.</td></tr>
<tr><td>2 × NAT Gateway</td><td>One per AZ to provide outbound internet for private instances; elastic IP attached.</td></tr>
<tr><td rowspan="3"><span class="badge">Private&nbsp;Subnet</span></td><td>EC2 App Servers</td><td>Dockerised Flask app (<code>souravangre/flask-fitstore</code>) responding on port 4000.</td></tr>
<tr><td>MySQL RDS</td><td>Managed relational database (single-AZ or Multi-AZ, depending on budget).</td></tr>
<tr><td>Cache Node</td><td>Elasticache Redis/Memcached for low-latency session &amp; catalogue caching.</td></tr>
</tbody>
</table>

<!-- ============ CIDR & ROUTING ============ -->
<h2>📐 CIDR Blocks &amp; Routing</h2>
<ul>
  <li><strong>VPC CIDR</strong> – <code>172.16.0.0/16</code></li>
  <li><strong>Subnets</strong> – split /24 per AZ:
    <ul>
      <li>AZ 1 Public: <code>172.16.0.0/24</code></li>
      <li>AZ 1 Private: <code>172.16.1.0/24</code></li>
      <li>AZ 2 Public: <code>172.16.2.0/24</code></li>
      <li>AZ 2 Private: <code>172.16.3.0/24</code></li>
    </ul>
  </li>
  <li><strong>Route Tables</strong>:
    <ul>
      <li>Public RT → <code>0.0.0.0/0</code> ➜ <em>Internet Gateway</em></li>
      <li>Private RTs → <code>0.0.0.0/0</code> ➜ <em>AZ-local NAT Gateway</em></li>
    </ul>
  </li>
</ul>

<!-- ============ SECURITY ============ -->
<h2>🔒 Security Highlights</h2>
<ul>
  <li><strong>Security Groups</strong>
    <ul>
      <li><code>sg_bastion</code>: SSH (22) from admin IPs → Bastion.</li>
      <li><code>sg_app</code>: HTTP/HTTPS from ALB or internal LB only → App EC2.</li>
      <li><code>sg_db</code>: MySQL (3306) from <code>sg_app</code> only.</li>
      <li><code>sg_cache</code>: Redis/Memcached port from <code>sg_app</code>.</li>
    </ul>
  </li>
  <li><strong>NACLs</strong> left at default (stateful SGs are primary guard-rails).</li>
  <li><strong>IAM roles</strong> grant minimal S3/CloudWatch permissions to instances.</li>
</ul>

<!-- ============ DEPLOYMENT ============ -->
<h2>🚀 Deployment Steps (Manual)</h2>
<ol>
  <li>Create the VPC and subnets (or import via Terraform).</li>
  <li>Attach an Internet Gateway and set up route tables.</li>
  <li>Spin up NAT Gateways, allocate Elastic IPs.</li>
  <li>Launch the Bastion EC2 in a public subnet.<br>
      <code>ssh -i Bastion.pem ubuntu@&lt;Public-IP&gt;</code></li>
  <li>Launch private EC2 app servers with the Docker image:<br>
<pre><code>docker pull souravangre/flask-fitstore
docker run -d -p 4000:4000 --restart unless-stopped \
  --name fitstore souravangre/flask-fitstore</code></pre></li>
  <li>Optionally attach an Application Load Balancer in front of <code>sg_app</code> for public traffic.</li>
</ol>

<!-- ============ LOCAL TEST VIA SSH TUNNEL ============ -->
<h2>🛠️ Quick Local Test (SSH Tunnel)</h2>
<pre><code># Forward local port 8080 ➜ private EC2:4000 through Bastion
ssh -i Bastion.pem -L 8080:172.16.1.123:4000 ubuntu@&lt;Bastion-IP&gt;
# Then open: http://localhost:8080
</code></pre>

<!-- ============ FUTURE WORK ============ -->
<h2>📈 Future Enhancements</h2>
<ul>
  <li>Replace manual steps with <strong>Terraform IaC</strong>.</li>
  <li>Add an <strong>ACM-issued SSL cert</strong> on the ALB (HTTPS).</li>
  <li><strong>Auto Scaling Group</strong> for stateless application layer.</li>
  <li>Use <strong>RDS Multi-AZ</strong> and <strong>ElastiCache Cluster Mode</strong> for HA.</li>
  <li>Integrate <strong>CloudWatch Alarms</strong> &amp; <strong>WAF</strong>.</li>
</ul>

<!-- ============ LICENSE & CONTACT ============ -->
<h2>📜 License</h2>
<p>MIT – free to use, modify, and distribute.</p>

<h2>🤝 Contact</h2>
<p>
  Built with ❤️ by <a href="https://github.com/souravangre" target="_blank">@souravangre</a>.  
  Feel free to open issues or PRs!
</p>

</main>
</body>
</html>
