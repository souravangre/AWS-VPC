<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
</head>
<body>
<main>

<!-- ============ TITLE & TAGLINE ============ -->
<h1>📘 AWS VPC – Secure Web Hosting with Private EC2 & Bastion</h1>
<p class="note">—</strong> A production-ready VPC architecture that keeps compute, data nodes <em>private</em>, funnels administration through a Bastion host, and lets application servers reach the internet via highly-available NAT Gateways.</p>

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

<h2>🧰 AWS Services Used</h2>
<table>
  <thead>
    <tr>
      <th>Service</th>
      <th>Purpose</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Amazon VPC</strong></td>
      <td>Custom Virtual Private Cloud with public and private subnets across multiple availability zones.</td>
    </tr>
    <tr>
      <td><strong>EC2</strong></td>
      <td>Virtual machines used to host the Bastion Host and the Private Web Server.</td>
    </tr>
    <tr>
      <td><strong>Bastion Host</strong></td>
      <td>Provides secure SSH access to instances in the private subnet.</td>
    </tr>
    <tr>
      <td><strong>Application Load Balancer (ALB)</strong></td>
      <td>Internet-facing ALB routes external traffic to the private web server.</td>
    </tr>
    <tr>
      <td><strong>Security Groups</strong></td>
      <td>Controls inbound/outbound traffic for EC2 instances and ALB.</td>
    </tr>
    <tr>
      <td><strong>Route Tables</strong></td>
      <td>Manages routing between public/private subnets and the internet.</td>
    </tr>
    <tr>
      <td><strong>Internet Gateway</strong></td>
      <td>Allows resources in public subnet to access the internet.</td>
    </tr>
    <tr>
      <td><strong>NAT Gateway</strong></td>
      <td>Enables private instances to access the internet (for updates, wget, etc.) securely.</td>
    </tr>
    <tr>
      <td><strong>MySQL on EC2</strong></td>
      <td>Relational database hosted securely in the private subnet.</td>
    </tr>
    <tr>
      <td><strong>ElastiCache (optional)</strong></td>
      <td>Managed caching service for fast data retrieval.</td>
    </tr>
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
      <li><code>sg_elb</code>: HTTP/HTTPS from Anywhere to access the site on private EC2.</li>
    </ul>
  </li>
  <li><strong>NACLs</strong> left at default (stateful SGs are primary guard-rails).</li>
</ul>

<!-- ============ DEPLOYMENT ============ -->
<!-- ============ DEPLOYMENT STEPS ============ -->
<h2>🚀 Deployment Steps</h2>

<ol>
  <li><strong>Create the VPC</strong>
    <ul>
      <li>CIDR block&nbsp;– <code>172.16.0.0/16</code></li>
    </ul>
  </li>

  <li><strong>Create Four Subnets</strong>
    <ul>
      <li>Public&nbsp;AZ-A&nbsp;→ <code>172.16.0.0/24</code></li>
      <li>Private&nbsp;AZ-A&nbsp;→ <code>172.16.1.0/24</code></li>
      <li>Public&nbsp;AZ-B&nbsp;→ <code>172.16.2.0/24</code></li>
      <li>Private&nbsp;AZ-B&nbsp;→ <code>172.16.3.0/24</code></li>
    </ul>
  </li>

  <li><strong>Attach an Internet Gateway</strong>
    <ul>
      <li>Create IGW → <em>Actions → Attach to VPC</em></li>
    </ul>
  </li>

  <li><strong>Configure the <em>Public</em> Route Table</strong>
    <ul>
      <li>Create RT, associate with VPC</li>
      <li><em>Subnet associations</em> → tick both public subnets</li>
      <li><em>Routes</em> → <code>0.0.0.0/0 → igw-xxxxxxxx</code></li>
    </ul>
  </li>

  <li><strong>Provision NAT Gateways (HA)</strong>
    <ul>
      <li>Create two Elastic IPs (one per AZ)</li>
      <li>Create NAT GW in each public subnet, attach its EIP</li>
    </ul>
  </li>

  <li><strong>Configure the <em>Private</em> Route Table</strong>
    <ul>
      <li>Create RT, associate with VPC</li>
      <li><em>Subnet associations</em> → tick both private subnets</li>
      <li><em>Routes</em> → <code>0.0.0.0/0 → nat-gw-az-a</code> (for AZ-A RT) and<br>
          &nbsp;&nbsp;&nbsp;<code>0.0.0.0/0 → nat-gw-az-b</code> (for AZ-B RT)</li>
    </ul>
  </li>

  <li><strong>Launch Bastion Host</strong>
    <ul>
      <li>Key pair + <code>sg_bastion</code> (SSH from your IP)</li>
      <li>Place in <em>public</em> subnet AZ-A (or AZ-B)</li>
    </ul>
  </li>

  <li><strong>Launch Private Web Server</strong>
    <ul>
      <li>AMI: Ubuntu 22.04 LTS (example)</li>
      <li>Subnet: <em>private</em> (no public IP)</li>
      <li>Security Group <code>sg_app</code> – allow port 80/4000 from ALB SG</li>
      <li>SSH in via Bastion → install packages &amp; deploy site</li>
    </ul>
  </li>

  <li><strong>Set Up the Application Load Balancer</strong>
    <ol type="a">
      <li><em>Target Group</em>
        <ul>
          <li>Type = Instance, Protocol = HTTP:80, VPC = your VPC</li>
          <li>Register the private web server</li>
        </ul>
      </li>
      <li><em>ALB</em>
        <ul>
          <li>Scheme = Internet-facing</li>
          <li>Subnets = both public subnets</li>
          <li>Security Group <code>sg_alb</code> – allow HTTP 80 from 0.0.0.0/0</li>
          <li>Listener 80 → forward to the target group</li>
        </ul>
      </li>
    </ol>
  </li>

  <li><strong>Smoke-Test the Stack</strong>
    <ul>
      <li>Browser → <code>http://&lt;ALB-DNS&gt;</code> should load your site 🎉</li>
      <li>Optional: SSH tunnel via Bastion to verify direct app port</li>
    </ul>
  </li>
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
