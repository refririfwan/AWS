# Deployment Wordpress in AWS

Belajar Mengembangkan dan Menjalankan Wordpress pada Amazon Web Service (AWS)

Wordpress dengan menggunakan AWS terlebih dahulu perlu untuk membuat server Web dengan EC2 dan Database Amazon RDS.  Server web Apache dengan PHP, dan membuat database MySQL. Server web berjalan pada instance Amazon EC2 menggunakan Amazon Linux, dan database MySQL adalah instance Amazon RDS MySQL DB. Baik instance Amazon EC2 dan instance Amazon RDS DB dijalankan dalam virtual private cloud (VPC) yang didasarkan pada layanan Amazon VPC.

Pada tahap awal diperlukan untuk membuat VPC untuk membangun skema lingkungan jaringan bagi EC2 dan RDS. Berikut terdapat lima langkah unntuk membangun VPC yang akan digunakan :
1. Membuat VPC untuk public dan private subnet
- Buka Amazon VPC console pada https://console.aws.amazon.com/vpc/
- Pada bagian kanan atas dari Amazon VPC console, pilih region untuk membuat VPC kita. Pada contoh ini menggunakan region us-east-1 US East (N. Virginia).
- Pada bagian kiri atas, pilih VPC Dashboard. Untuk memulai pembuatan VPC, pilih Launch VPC Wizard.
- Pada halaman Step 1: Select a VPC Configuration, pilih VPC with Public and Private Subnets, dan kemudian pilih Select.
- Pada Step 2: VPC with Public and Private Subnets, masukkan isian berikut:
• IPv4 CIDR block: 10.0.0.0/16
• IPv6 CIDR block: No IPv6 CIDR Block
• VPC name: wordpress-vpc
• Public subnet's IPv4 CIDR: 10.0.0.0/24
• Availability Zone: us-east-1a
• Public subnet name: wordpress public
• Private subnet's IPv4 CIDR: 10.0.1.0/24
• Availability Zone: us-east-1a
• Private subnet name: wordpress private 1
• Instance type: t2.small
• Key pair name: No key pair
• Service endpoints: Skip this field.
• Enable DNS hostnames: Yes
• Hardware tenancy: Default
- Setelah selesai, pilih Create VPC.

2. Membuat Subnet tambahan
- Buka Amazon VPC console https://console.aws.amazon.com/vpc/
- Untuk menambahkan private subnet kedua pada VPC, pilih VPC Dashboard, pilih Subnets, dan pilih Create subnet.
- Pada halaman Create subnet, masukkan isian berikut:
• Name tag: wordpress private 2
• VPC: Pilih VPC yang dibuat pada step sebelumnya, sebagai contoh: vpc-identifier (10.0.0.0/16) | wordpress-vpc
• Availability Zone: us-east-1b
• IPv4 CIDR block: 10.0.2.0/24
- Ketika selesai, pilih Create. Kemudian pilih Close pada halaman konfirmasi.
- Untuk memastikan bahwa private subnate kedua yang baru saja dibuat menggunakan route table yang sama dengan private subnet pertama, pilih VPC Dashboard, pilih Subnets, dan pilih private subnet pertama yang telah dibuat, yaitu wordpress private 1.
- Di bawah daftar subnets, pilih tab Route Table, dan catat nilai dari Route Table, misal: rtb-98b613fd.
- Pada daftar subnet, hilangkan pilihan (uncheck/deselect) private subnet pertama.
- Pada daftar subnets, pilih private subnet kedua wordpress private 2 dan pilih tab Route Table.
- Jika route table tersebut tidak sama pada private subnet yang pertama, maka pilih Edit route table association. Untuk Route Table ID, pilih route table yang sudah dicatat sebelumnya, misal: rtb-98b613fd. Kemudian unt

3. Membuat VPC Security Group untuk Web Server Public
- Buka Amazon VPC Console https://console.aws.amazon.com/vpc/
- Pilih VPC Dashboard, pilih Security Groups, dan pilih Create security group.
- Pada halaman Create security group, masukkan isian berikut:
• Security group name: wordpress-securitygroup
• Description: wordpress Security Group
• VPC: Pilih VPC yang telah dibuat sebelumnya, contoh: vpc-identifier (10.0.0.0/16) | wordpress-vpc
- Untuk membuat security group, pilih Create. Kemudian pilih Close pada halaman konfirmasi.
Perhatikan security group ID yang dibuat karena akan dipakai pada tahap berikutnya.
- Kemudian tentukan alamat IP yang akan kita gunakan untuk mengakses instance di dalam VPC. Untuk menentukan alamat IP public kita, kita dapat menggunakan layanan yang ada pada https://checkip.amazonaws.com/. Contoh IP public kita misalkan 203.0.113.25/32.
Jika kita mengakses internet menggunakan ISP atau berada di belakang firewall tanpa alamat IP static, kita perlu menentukan range alamat IP yang digunakan pada komputer klien.
- Buka Amazon VPC console https://console.aws.amazon.com/vpc/
- Pilih VPC Dashboard, kemudian pilih Security Groups, dan pilih wordpress-securitygroup yang telah dibuat.
- Pada daftar pilihan security groups, pilih tab Inbound Rules, dan pilih Edit rules.
- Pada halaman Edit inbound rules, pilih Add Rule.
- Masukkan isian berikut pada inbound rule baru untuk mengijinkan secure shell (SSH) mengakses EC2 instance kita. Jika kita menjalankan ini, kita dapat mengakses instance EC2 untuk menginstal web server dan utility lain, dan upload konten ke dalam web server
• Type: SSH
• Source: Alamat IP atau range pada step 5, contoh: 203.0.113.25/32.
- Setelah isian selesai, pilih Add rule.
- Set nilai berikut pada inbound rule yang baru dibuat untuk mengijinkan akses HTTP ke web server kita.
• Type: HTTP
• Source: 0.0.0.0/0.
- Untuk menyimpan settings, pilih Save rules. Kemudian pilih Close pada halaman konfirmasi.

4. Membuat VPC Security Group untuk Private Amazon RDS DB Instance.
- Buka Amazon VPC console https://console.aws.amazon.com/vpc/
- Pilih VPC Dashboard, pilih Security Groups, dan kemudian pilih Create security group.
- Pada halaman Create security group, masukkan isian sebagai berikut:
• Security group name: wordpress-db-securitygroup
• Description: wordpress DB Instance Security Group
• VPC: Pilih VPC yang telah dibuat pada step sebelumnya, misal: vpc-identifier (10.0.0.0/16) | wordpress-vpc
- Untuk membuat security group, pilih Create. Kemudian pilih Close pada halaman konfirmasi.
- Langkah selanjutnya adalah menambahkan inbound rules kepada security group yang baru saja dibuat. Untuk itu, buka VPC console pada https://console.aws.amazon.com/vpc/.
- Pilih VPC Dashboard, kemudian Security Groups, dan kemudian pilih security group wordpress-db-securitygroup yang telah dibuat.
- Pada daftar security groups, pilih tab Inbound Rules dan kemudian pilih Edit rules.
- Pada halaman Edit inbound rules, pilih Add Rule.
- Masukkan isian berikut pada inbound rule yang akan dibuat untuk mengijinkan akses MySQL pada port 3306 dari EC2 instance. Jika kita melakukan ini, maka web server dapat mengakses database.
- Untuk menyimpan setting, pilih Save rules. Kemudian pilih Close pada halaman konfirmasi.

5. Membuat DB Subnet Group
- Buka Amaon RDS console https://console.aws.amazon.com/vpc/
- Pada panel navigasi, pilih Subnet groups.
- Pilih Create DB Subnet Group.
- Pada halaman Create DB Subnet Group, masukkan isian berikut pada Subnet group details:
• Name: wordpress-db-subnet-group
• Description: wordpress DB Subnet Group
• VPC: wordpress-vpc (vpc-identifier)
- Pada bagian Add subnets, pilih Add all the subnets related to this VPC.
- Pilih Create.

Setelah selesai mempersiapkan lingkungan VPC, tahap selanjutnya akan menyiapkan web server dan database mysql. 

1. Membuat RDS DB Instance
- Masuk ke dalam Amazon RDS Console https://console.aws.amazon.com/rds/
- Pada bagian kanan atas, pilih AWS Region di mana DB instance akan dibuat. Contoh berikut menggunakan region US east (Virginia).
- Pada panel navigasi pilih Databases. Jika panel navigasi tertutup, pilih menu icon pada pojok kiri atas untuk membukanya.
- Pilih Create database untuk membuat halaman Select engine.
- Pada halaman Select engine, pilih MySQL dan kemudian pilih Next.
- Pada halaman Choose use case, pilih Dev/Test – MySQL, dan kemudian pilih Next.
- Pada halaman Setting, masukkan isian berikut:
• DB instance identifier: wordpress-db-instance
• Master username: wordpress_user
• Master password: Choose a password.
• Confirm password: Retype the password.
• DB instance class : Burstable classes (includes t classes) pilih db.t2.small
• Storage type: General Purpose (SSD)
• Allocated storage: 20 GiB
• Multi-AZ deployment: No
- Pengaturan Connectivity
• Virtual Private Cloud (VPC): Pilih VPC yang sudah ada baik public ataupun private subnets, seperti wordpress-vpc (vpc-identifier) pada bagian sebelumnya.
• Subnet group: DB subnet group untuk VPC, seperti pada wordpress berikutnya adalah
wordpress-db-subnet-group.
• Public accessibility: No
• Availability zone: No Preference
• VPC security groups: Pilih VPC security group yang telah dikonfigurasi untuk akses private, seperti pada langkah berikutnya yaitu wordpress-db-securitygroup. Hapus security groups yang lain, seperti default security groups dengan cara memilih tanda silang X pada bagian security groups yang akan dihapus.
• Database name: wordpress
Biarkan pilihan lain sesuai dengan isian default.

2. Membuat EC2 Instance
- Untuk membuat instance, akses AWS Management Console pada https://console.aws.amazon.com/ec2/
- Pilih EC2 Dashboard, dan kemudian pilih Launch Instance
- Pilih Amazon Linux AMI
- Pilih tipe instance t2.small, seperti pada gambar berikut, dan kemudian pilih Next: Configure Instance Details.
- Pada halaman Configure Instance Details, seperti pada gambar, isikan nilai berikut dan biarkan lainnya tanpa diubah (default).
• Network: Pilih VPC yang memiliki subnet public dan private yang kita pilih untuk DB
instance, yaitu: wordpress-vpc (vpc-identifier).
• Subnet: Pilih subnet public yang sudah dibuat sebelumnya, yaitu: subnet-identifier | wordpress public | us-east-1a.
• Auto-assign Public IP: pilih Enable.
- Berikutnya pilih Next: Add Storage.
- Pada halaman Add Storage, biarkan nilainya tanpa perlu diubah (default) dan kemudian pilih Next: Add Tags.
- Pada halaman Add Tags, seperti pada gambar, pilih Add Tag, kemudian masukkan Name pada bagian Key dan masukkan wordpress-web-server untuk Value-nya.
- Selanjutnya pilih Next: Configure Security Group.
- Pada halaman Configure Security Group, seperti pada gambar, pilih Select an existing security group, dan kemudian pilih security group yang sudah ada, seperti wordpress-securitygroup. Security group yang dipilih harus memilki inbound rules untuk akses SSH dan HTTP.
- Pilih Review and Launch.
- Pada halaman Review Instance Launch, seperti gambar, verifikasi setting yang dibuat dan kemudian pilih Launch.
- Pada halaman Select an existing key pair or create a new key pair, seperti pada gambar, pilih Create a new key pair dan set Key pair name dengan isian wordpress-key-pair. Pilih Download Key Pair dan kemudian simpan file key pair pada komputer lokal. Gunakan key pair ini saat ingin mengakses EC2 instance.
- Untuk menjalankan EC2 instance, pilih Launch Instance. Pada halaman Launch Status, sepertipada gambar, perhatikan identifier dari EC2 instance baru yang dibuat, seperti: i-0288d65fd4470b6a9.
- Untuk menemukan instance, pilih View Instances.
- Tunggu sampai Instance Status untuk instance berubah menjadi running sebelum melanjutkan tutorial berikutnya.

3. Instal Apache Web Server dan PHP serta konfigurasi wordpress
- Untuk mengakses EC2 instance yang dibuat, ikuti langkah-langkah pada Connect to Your Linux
Instance https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstances.html
- Untuk mendapatkan bug fixes terkini dan security updates, lakukan update software pada EC2 instance dengan menjalankan perintah berikut.

[ec2-user ~]$ sudo yum update -y

- Setelah update selesai, install Apache web server dengan PHP dengan menggunakan perintah yum install, yang akan menginstal software packages dan dependencies yang dibutuhkan secara bersamaan.

[ec2-user ~]$ sudo yum install -y httpd24 php56 php56-mysqlnd

Catatan:
Jika ada error No package package-name available, itu berarti instance tidak dibuat menggunakan Amazon Linux AMI (mungkin kita menggunakan Amazon Linux 2 AMI). Agar dapat melihat versi Amazon Linux yang digunakan menggunakan perintah berikut.

cat /etc/system-release

4. Jalankan web server dengan menggunakan perintah berikut.

[ec2-user ~]$ sudo service httpd start

Agar dapat mengetes apakah web server telah diinstal dan berjalan dengan memasukkan nama public DNS dari EC2 instance, contoh: http://ec2-42-8-168-21.us-west-1.compute.amazonaws.com. Jika web server sudah jalan, maka akan melihat halaman tes Apache.
Apabila tidak muncul, verifikasi apakah inbound rules untuk VPC security group mengikutkan allow akses HTTP (port 80) untuk IP yang digunakan saat mengakses web server.
- Lakukan konfigurasi web server agar berjalan setiap kali sistem melakukan booting.

[ec2-user ~]$ sudo chkconfig httpd on

- Sebelum konfigutas permission file pada apache terlebih dahulu dsiapkan project wordpress dengan perintah 

[ec2-user ~]$ wget https://wordpress.org/latest.zip

- Setelah file terunduk lalu ekstrak file tersebut dengan perintah 

[ec2-user ~]$ unzip latest.zip 

- Untuk tahap selanjutnya pindahkan folder hasil ekstrak ke folder apache di /var/www/ dengan perintah 

[ec2-user ~]$ sudo mv wordpress/ /var/www/

- Untuk mengeset permission pada Apache web server, lakukan penambahan group dengan nama www.

[ec2-user ~]$ sudo groupadd www

- Tambahkan ec2-user ke dalam grup www.

[ec2-user ~]$ sudo usermod -a -G www ec2-user

- Untuk me-refresh permissions dan memasukkan ke dalam grup www, lakukan logout dari sistem.

[ec2-user ~]$ exit

- Log back in kembali dan verifikasi apakah group www ada.

[ec2-user ~]$ groups
ec2-user wheel www

- Ubah group ownership dari direktori /var/www dan isinya ke dalam grup www.

[ec2-user ~]$ sudo chown -R root:www /var/www

- Ubah permission dari direktori /var/www dan subdirektori-nya untuk menambahkan permission write dan set group ID pada subdirektori yang dibuat di kemudian hari.

[ec2-user ~]$ sudo chmod 2775 /var/www
[ec2-user ~]$ find /var/www -type d -exec sudo chmod 2775 {} +

- Ubah permission secara rekursif pada file-file di dalam direktori /var/www dan subdirektori
untuk menambahkan permission write pada grup.

[ec2-user ~]$ find /var/www -type f -exec sudo chmod 0664 {} +

- Setelah itu lakukan perubahan default document root apache ke direktori wordpress dengan merubah konfigurasi pada http.conf. Perintah untuk mengedit file konfigurasi sebagai berikut : 

[ec2-user ~] sudo nano /etc/httpd/conf/httpd.conf

- lalu rubah pada bagian /var/www/html menjadi  /var/www/wordpress

- Setelah dirubah restart service apache dengan perintah

[ec2-user ~] sudo service httpd restart

- Akses kembali web server apache melalui browser seperti percobaanya sebelumnya. Selanjutnya konfigurasi wordpress melalui halaman website.

- klik lets go dan isikan database RDS yang di konfigurasi sebelumnya. lalu setelah itu bila tidak ada file wp-config maka buat file tersebut serta copy kan kode program php yang di sediakan di halaman web. perintah membuat file wp-config sebagai berikut :

ec2-user ~] sudo nano /var/www/wordpress/wp-config.php

- Setelah itu masuk tahapan konfigutasi sebelum instalasi wordpress sebagai berikut

• site title = wordpress
• username = wordpressuser
• password = wordpressadmin
• email = refririfwan@yahoo.com

Note : isian diatas dapat diisi sesuai dengan yang diingikan.

- Setelah sukses maka bisa langsung login menggunakan username dan password yang telah dibuat.

- Enjoying Customize the new website.

Terimakasih, Semoga bermanfaat :D

Sumber Referensi : 
1. https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/TUT_WebAppWithRDS.html 
2. https://wordpress.org/latest.zip
3. https://wordpress.org/support/article/how-to-install-wordpress/

# ^-^