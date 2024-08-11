# Docker Clustered Web Architecture


![Untitled__1_-removebg-preview](https://github.com/user-attachments/assets/0d5cf3a9-88ba-47f2-902f-6753fa91eab9)


Sistem, PHP tabanlı web uygulamalarını barındırmak üzere üç ayrı konteynerden oluşur. Her bir PHP konteyneri, Apache web sunucusunu ve PHP’yi çalıştırır. Bu konteynerler, /var/www/html/ dizininde saklanan PHP dosyalarını sunar. Her konteyner, belirli bir port üzerinden erişilebilir. Örneğin, ilk konteyner (web1) 80 numaralı portu 3180 numaralı port ile eşleştirirken, diğer konteynerler farklı portlarla yapılandırılmıştır.

HAProxy, bu üç konteyner arasında yük dengelemesi yapmak için kullanılır. HAProxy, gelen HTTP isteklerini alır ve bu istekleri arka planda çalışan PHP konteynerlerine dağıtır. Bu dağıtım işlemi, HAProxy'nin yük dengeleme algoritmalarına dayanır; bu örnekte, istekler roundrobin (dönüşümlü) yöntemiyle dağıtılır. HAProxy, aynı zamanda SSL/TLS desteği sağlayabilir ve HTTP trafiğini dinleyerek istekleri uygun konteynere yönlendirir.

Bu mimari, yüksek erişilebilirlik ve yük dengelemesi sağlayarak, web uygulamanızın daha hızlı ve daha güvenilir bir şekilde çalışmasını mümkün kılar. PHP konteynerlerinin birbirinden bağımsız olması, herhangi bir konteynerin başarısız olması durumunda diğerlerinin hizmet vermeye devam etmesini sağlar. HAProxy’nin yük dengeleme özelliği ise trafik yoğunluğunu verimli bir şekilde dağıtarak sistem performansını optimize eder.

Test sağlamanız açısından isterseniz https://docker.amigdala.net.tr web sitesine giriş yapabilirsiniz.


## Kurulum

Bu yapılandırma şunları içerir:
- **3 PHP Konteyneri**: Apache ve PHP desteği ile çalışır.
- **1 HAProxy Konteyneri**: HTTP isteklerini PHP konteynerlerine dağıtır.

### Dizin Yapısı

Home dizini altında  `web` ve `haproxy` adlı iki klasör oluşturalım. HaProxy klasörü altında bir tane `haproxy.cfg` dosyası oluşturup repo üzerindeki `haproxy.cfg` dosyasının içeriğini yapıştıralım.

### haproxy.cfg dosya içeriği:

```
global
  log /dev/log local0
  log /dev/log local1 notice
  chroot /var/lib/haproxy
  user haproxy
  group haproxy
  daemon

defaults
  log global
  mode http
  option httplog
  option dontlognull
  timeout connect 5000
  timeout client 50000
  timeout server 50000

frontend web
  bind *:80
  default_backend web_servers

backend web_servers
    balance roundrobin
    server web1 web1:80 check
    server web2 web2:80 check
    server web3 web3:80 check
```


## Compose

Repo üzerindeki `docker-compose.yml` dosyasını  indirip  çalıştırınız.

```bash
root@ip-172-31-34-227:/home/web# docker-compose up -d
[+] Running 4/0
 ✔ Container haproxy  Running      0.0s 
 ✔ Container web1     Running      0.0s 
 ✔ Container web2     Running      0.0s 
 ✔ Container web3     Running      0.0s 
root@ip-172-31-34-227:/home/web# 
```

IP adresinize giderek `IT's Work` yazısını görebilirsiniz. İçeriği değiştirmek için daha önce oluşturmuş olduğumuz Home dizini altındaki web dizinin altında `index.php` dosyasını oluşturup repo üzerindeki `index.php` dosyasının içeriğini kopyalayıp oluşturduğumuz dosyaya yapıştırıyoruz.


![image](https://github.com/user-attachments/assets/dde3cb83-0ea0-41fd-a097-8e0943b75e38)


Her F5 attığnız da Container ID değerinin değiştiğini göreceksiniz.



## SSL 

Şimdi bu yapımız için bir tane SSL sertifikası kuracağız. Sunucumuza Cerbotu kurarak başlıyoruz.

```
sudo apt install certbot
```

Certbot'u kullanarak SSL sertifikası oluşturmak için aşağıdaki komutu yazıyoruz. Komutu yazmadan önce HaProxy containerını durdurmamız gerekiyor. `docker container rm haproxy -f` yazarak kaldırabilirsiniz.

```
sudo certbot certonly --standalone -d domain.com -d www.domain.com
```

Burada, domain.com ve www.domain.com kendi domain adresinle değiştirilmeli. Bu komut, bir SSL sertifikası oluşturur ve sertifika dosyalarını /etc/letsencrypt/live/domain.com/ dizininde saklar.


### Sertifikayı HAProxy İçin Hazırlama

Let's Encrypt, sertifikaları .pem formatında sağlar. Bu dosyaları HAProxy için tek bir .pem dosyasında birleştirmen gerekiyor:

```
sudo cat /etc/letsencrypt/live/domain.com/fullchain.pem /etc/letsencrypt/live/domain.com/privkey.pem > /etc/letsencrypt/live/domain.com/haproxy.pem
```

### Docker'da HAProxy İçin Sertifikayı Ayarlama

Docker Compose dosyanda sertifikayı HAProxy konteynerine mount etmeniz gerekiyor:

```
version: '3'

services:
  web1:
    image: php:7.4-apache
    container_name: web1
    volumes:
      - /home/web:/var/www/html/
    ports:
      - "3180:80"
    networks:
      - webnet

  web2:
    image: php:7.4-apache
    container_name: web2
    volumes:
      - /home/web:/var/www/html/
    ports:
      - "3181:80"
    networks:
      - webnet

  web3:
    image: php:7.4-apache
    container_name: web3
    volumes:
      - /home/web:/var/www/html/
    ports:
      - "3281:80"
    networks:
      - webnet

  web4:
    image: php:7.4-apache
    container_name: web4
    volumes:
      - /home/web:/var/www/html/
    ports:
      - "3282:80"
    networks:
      - webnet

  haproxy:
    image: haproxy:2.3
    container_name: haproxy
    ports:
      - "80:80"
      - "443:443"
      - "8080:8080"
    volumes:
      - /home/haproxy/haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg:ro
      - /etc/letsencrypt/live/domain.com/haproxy.pem:/etc/letsencrypt/live/domain.com/haproxy.pem:ro
    sysctls:
      - net.ipv4.ip_unprivileged_port_start=0
    networks:
      - webnet

networks:
  webnet:

```

### HAProxy Konfigürasyonunu Güncelleme

haproxy.cfg dosyasını aşağıdaki gibi düzenleyiniz:

```
frontend web
  bind *:80
  bind *:443 ssl crt /etc/letsencrypt/live/domain.com/haproxy.pem
  redirect scheme https if !{ ssl_fc }
  default_backend web_servers

backend web_servers
    balance roundrobin
    server web1 web1:80 check
    server web2 web2:80 check
    server web3 web3:80 check
    server web4 web4:80 check

```

### Docker Compose'u Yeniden Başlatma

Yapılandırmayı tamamladıktan sonra Docker Compose'u yeniden başlat:

```
docker-compose down
docker-compose up -d
```


Başarılı bir şekilde SSL kurulumunu gerçekleştirdik.

![image](https://github.com/user-attachments/assets/ef348b3d-645e-43aa-b3a4-71e6d097ca7f)



---------------------------------------------------------------

Okuduğunuz için teşekkürler.




