# Docker Clustered Web Architecture

Sistem, PHP tabanlı web uygulamalarını barındırmak üzere üç ayrı konteynerden oluşur. Her bir PHP konteyneri, Apache web sunucusunu ve PHP’yi çalıştırır. Bu konteynerler, /var/www/html/ dizininde saklanan PHP dosyalarını sunar. Her konteyner, belirli bir port üzerinden erişilebilir. Örneğin, ilk konteyner (web1) 80 numaralı portu 3180 numaralı port ile eşleştirirken, diğer konteynerler farklı portlarla yapılandırılmıştır.

HAProxy, bu üç konteyner arasında yük dengelemesi yapmak için kullanılır. HAProxy, gelen HTTP isteklerini alır ve bu istekleri arka planda çalışan PHP konteynerlerine dağıtır. Bu dağıtım işlemi, HAProxy'nin yük dengeleme algoritmalarına dayanır; bu örnekte, istekler roundrobin (dönüşümlü) yöntemiyle dağıtılır. HAProxy, aynı zamanda SSL/TLS desteği sağlayabilir ve HTTP trafiğini dinleyerek istekleri uygun konteynere yönlendirir.

Bu mimari, yüksek erişilebilirlik ve yük dengelemesi sağlayarak, web uygulamanızın daha hızlı ve daha güvenilir bir şekilde çalışmasını mümkün kılar. PHP konteynerlerinin birbirinden bağımsız olması, herhangi bir konteynerin başarısız olması durumunda diğerlerinin hizmet vermeye devam etmesini sağlar. HAProxy’nin yük dengeleme özelliği ise trafik yoğunluğunu verimli bir şekilde dağıtarak sistem performansını optimize eder.


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

IP adresinize giderek `IT's Work` yazısını görebilirsiniz. İçeriği değiştirmek için daha önce oluşturmuş olduğumuz Home dizini altındaki web dizinin altında `index.php` dosyasını oluşturup aşağıdaki kodu yazıyoruz.

```
<?php
// Hostname ve IP adresini al
$hostname = gethostname();
$ip_address = getenv('SERVER_ADDR'); 
// Verileri ekrana yazdır
echo "<h1>Container Bilgileri</h1>";
echo "<p><strong>Hostname:</strong> $hostname</p>";
echo "<p><strong>IP Adresi:</strong> $ip_address</p>";
?>
```

![image](https://github.com/user-attachments/assets/815cf816-0931-4223-995f-2d1bbbf7c81c)


Her F5 attığnız da Container ID değerinin değiştiğini göreceksiniz.



---------------------------------------------------------------

Okuduğunuz için teşekkürler.




