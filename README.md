<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>AWS VPC – Private EC2 Hosting with Bastion & NAT Gateways</title>

  <!-- ============ Simple Styling ============ -->
  <style>
    :root {
      --blue: #0d6efd;
      --dark: #212529;
      --light: #f8f9fa;
    }
    * { box-sizing: border-box; }
    body {
      font-family: "Segoe UI", Roboto, Arial, sans-serif;
      margin: 0;
      background: var(--light);
      color: var(--dark);
      line-height: 1.55;
      padding: 2.5rem 1rem 4rem;
    }
    main { max-width: 900px; margin: auto; }
    h1, h2, h3 { color: var(--blue); font-weight: 600; }
    h1 { margin-bottom: .2em; }
    h2 { margin-top: 2.2rem; border-bottom: 2px solid #dee2e6; padding-bottom: .35rem; }
    p, li { font-size: 1.05rem; }
    ul { margin-left: 1.2em; }
    li { margin: .35em 0; }
    pre, code { background: #e9ecef; font-family: Consolas, monospace; padding: .2rem .4rem; border-radius: 4px; }
    pre { display: block; padding: 1rem; overflow-x: auto; }
    table { width: 100%; border-collapse: collapse; font-size: .95rem; }
    th, td { padding: .5rem .65rem; border: 1px solid #ced4da; }
    th { background: #e2e6ea; text-align: left; }
    .badge { background: var(--blue); color: #fff; padding: .18rem .55rem; border-radius: .25rem; font-size: .8rem; }
    .diagram { border: 1px solid #ced4da; border-radius: 6px; overflow: hidden; box-shadow: 0 2px 8px rgba(0,0,0,.06); margin: 1rem 0 2rem; }
    img { width: 100%; height: auto; display: block; }
    .note { background: #e7f5ff; border-left: 4px solid #0d6efd; padding: .8rem 1rem; margin: 1.5rem 0; }
    a { color: var(--blue); text-decoration: none; }
    a:hover { text-decoration: underline; }
  </style>
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
  <li>Isolates <strong>application EC2 instances</strong>, <strong>MySQL&nbsp;RDS</strong>, and an in-memory <strong>cache node</strong> in <em>private subnets</em>.</li>
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
