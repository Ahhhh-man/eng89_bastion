# EC2 app w/ db

### 1) Create VPC
- Name it whatever
- IPv4 CIDR block is `10.207.0.0/16`

### 2) Create Internet Gateway (IG)
- Name it and click create.
- Now attach VPC via actions.

### 3) Create Subnets
- for **public** subnet use IPv4 CIDR block `10.207.1.0/24`
- for **private** subnet use IPv4 CIDR block `10.207.2.0/24`
- for **bastion** subnet use IPv4 CIDR block `10.207.3.0/24`

### 4) Create Route Table
- Name it and click create.
- click edit route table.
	- Add 0.0.0.0/0 and the internet gateway (igw-...)

### 5) Edit subnet route table association
- for each of the subnets (public, private and bastion), click edit route table association and add the new route table.

### 6) Create Security Groups
Create a security group with the following rules for app, db and bastion and ensure you **connect to VPC**:
- **APP** Inbound Rules
	- SSH, 22, *your-ip*/32
	- HTTP, 80, 0.0.0.0/0
- **APP** Outbound Rules
	- Dont need to add anything here

- **DB** Inbound Rules
	- TCP, 27017, sg-of-app
	- SSH, 22, 10.207.3.0/24

- **DB** Outbound Rules
	- Dont need to add anything here

- **BASTION** Inbound Rules
	- SSH, 22, 0.0.0.0/0

- **BASTION** Outbound Rules
	- Dont need to add anything here

### 7) Create NaCl for public
With the following Rules
- Inbound
	- 100, HTTP, 0.0.0.0/0
	- 110, SSH, MyIP/32
	- 120, TCP, 1024-65535, 0.0.0.0/0
- Outbound
	- 100, HTTP, 0.0.0.0/0
	- 110, TCP, 27017, 10.207.2.0/24
	- 120, TCP, 1024-65535, 0.0.0.0/0

Now click edit subnet assaciation and add the public SG

### 8) Create NaCl for private
With the following Rules
- Inbound
	- 100, 27017, 10.207.1.0/24
	- 110, TCP, 1024-65535, 0.0.0.0/0
	- 120, SSH,  10.207.3.0/24
- Outbound
	- 100, HTTP, 0.0.0.0/0
	- 110, TCP, 1024-65535, 10.207.1.0/24
	- 120, TCP, 1024-65535, 10.207.3.0/24

Now click edit subnet assaciation and add the private SG

### 9) Create NaCl for bastion
With the following Rules
- Inbound
	- 100, SSH, 0.0.0.0/0
	- 110, TCP, 1024-65535, 10.207.2.0/24
- Outbound
	- 100, SSH, 10.207.2.0/24
	- 110, TCP, 1024-65535, 0.0.0.0/0

Now click edit subnet assaciation and add the bastionSG

### 10) Create Instances
- for app (using ami if possible)
- for db (using ami if possible)
	> Remark: Ensure IPv4 is disabled
- for bastion

### 11) Resolve Host (optional)
- `sudo nano /etc/hosts` then add `10.xxx.xx.xx ip-10-xxx-xx-xxx` as the second loc

### 12) Add Reverse Proxy
- `cd /etc/nginx/sites-available`
- `sudo nano default`
- ```server {
    listen 80;
    server_name _;
    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}```

### 13) Add DB_HOST to app instance
- `export DB_HOST=mongodb://10.207.2.234:27017/posts`

### 14) check on browser 
