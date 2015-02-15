# vagrant-mariadb-galera
Lab สำหรับศึกษา mariadb galera โดยใช้ vagrant เป็นตัวสร้าง virtual machine จำนวน 3 เครื่องและใช้ shell script ในการ provision

สิ่งที่ต้องการ
1. VirutalBox
2. Vagrant
3. git

สิ่งที่จะได้
- Ubuntu\Trusty64 3 server

ขั้นตอนหลังจาก provision แล้ว
ให้ทำการ copy จากเครื่อง mm01
```shell
/etc/mysql/debian.cnf
````
ไปยังเครื่อง mm02 และ mm03

ทำการ stop mysql ทุกเครื่องด้วยคำสั่ง
sudo service mysql stop

และที่เครื่อง mm01 ให้พิมพ์
```shell
sudo service mysql start --wsrep-new-cluster
```
จากนั้นให้ start mysql ที่เครื่อง mm02, mm03 
```shell
sudo service mysql start
```

ที่เครื่อง mm01 ตรวจสอบได้โดย
```shell
mysql -uroot -ppassword -e "SHOW STATUS LIKE 'wsrep_cluster_size%';"
```
จะเห็นจำนวน member ภายใน cluster ที่ join เข้ามาได้
