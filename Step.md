# deploy_flask_to_aws_ec2
This is a repository that will show how can a python flask app be deployed on an cloud via AWS EC2 machine

Step by Step Procedure:

(1) Launch an EC2 Machine and use RHEL as OS
sudo su -

(3) Install the latest python version
yum install python3.11 -y

(4) Install pip
yum install python3.11-pip -y

(5) Install git
yum install git -y

(6) Clone the project to the desired directory
git clone https://github.com/EngineerMarion29/flaskapp_emailautom.git 
(7) Go to the project dir
cd flaskapp_emailautom

(8) Install virtualenc
pip3.11 install virtualenv

(9) Enable virtualenv on the project dir
virtualenv .

(10) Activate virtual env
source bin/activate

(9) Install dependencies
pip3.11 install -r requirements.txt (Command to list down all requirements: pip freeze > requirements.txt)

(10) Install gunicorn
pip3.11 install gunicorn

(11) Configure gunicorn daemon

vi /etc/systemd/system/gunicorn.service
[Unit]
Description=Gunicorn daemon to serve my flaskapp 
After=network.target
[Service]
User=root
Group=root
WorkingDirectory=/root/flaskapp_emailautom
ExecStart=/root/flaskapp_emailautom/bin/gunicorn --bind unix:main.sock -m 666 main:app
[Install]
WantedBy=multi-user.target


(12) Start and enable gunicorn.service

systemctl start gunicorn.service
systemctl enable gunicorn.service

##Note: If error 203 / permission denied, occurs, try these:
sudo cp /root/flaskapp_emailautom/bin/gunicorn /usr/local/bin/
sudo chmod +x /usr/local/bin/gunicorn

(13) Install, start and enable nginx

yum install nginx
systemctl start nginx
systemctl enable nginx

(14) Configure nginx

vi /etc/nginx/nginx.conf
##First line
user root;

##Under http {
    server {
        listen       80;
        listen       [::]:80;
        server_name <aws public dns>;
        root         /usr/share/nginx/html;
        location / {
        proxy_pass http://unix: /root/flaskapp_emailautom/main.sock;
        }
        
##Note: If error in loading the actual webpage occurs, try this:

sudo cat /var/log/audit/audit.log | grep nginx | grep denied | audit2allow -M mynginx
semodule -i mynginx.pp
