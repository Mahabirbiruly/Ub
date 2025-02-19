# Ubuntu Desktop

> Repo'yu RC için katkıda bulunan [Mehmet](https://github.com/mehmet0150)'e teşekkürler <3

#

> Aşağıdaki komutlar ile Ubuntu sunucunuza desktop özelliği kazandırabilir.

> Ardından windows RDP veya başka bir 

#

> Öncelikle root klasöründe yeni bir klasör oluşturalım:

```ruby
mkdir ubuntu
```

> Klasöre girelim ve aşağıdaki komutu çalıştıralım.

```ruby
cd ubuntu
nano ub.sh
```
Açılan dosyaya aşağıdaki kod bloğunun en üstünde değişiklik yaparak yapıştıralım. Yeni bir kullanıcı adı ve şifre oluşturacağız. Komutların üzerinde ne işe yaradığı yazıyor. Şifre ve kullanıcı adı için şu karakterleri kullanmayın: ; | & $ ` \ ' “ < > ( ) * ? [ ] { } # ~ % !

```ruby
#!/bin/bash

# Uzak masaüstü için kullanıcı ve şifre değişkenlerini tanımlayın
# Root kullanmayın
USER="KULLANICI ADI"
PASSWORD="ŞİFRE"

# Paket listesini günceller
apt update && apt upgrade -y

# GNOME Masaüstünü yükler
sudo apt install -y ubuntu-desktop

# Uzak masaüstü sunucusunu (xrdp) kurar
sudo apt install -y xrdp

# USER kullanıcısını şifreyle ekler
sudo useradd -m -s /bin/bash $USER
echo "$USER:$PASSWORD" | sudo chpasswd

# USER kullanıcısını yönetim hakları için sudo grubuna ekler
sudo usermod -aG sudo $USER

# xrdp'yi GNOME masaüstünü kullanacak şekilde yapılandırır
echo "gnome-session" > ~/.xsession

# xrdp hizmetini yeniden başlatır
sudo systemctl restart xrdp

# Başlangıçta xrdp'yi etkinleştirir
sudo systemctl enable xrdp



```

``` CTRL + X ``` tuşlarına ve ```Y ``` tuşuna basalım, ardından ```ENTER``` tuşuna basalım.

Dosyayı kullanılabilir hale getirelim.

```ruby
chmod +x ub.sh
```

Şimdi scripti çalıştıralım.

```ruby
./ub.sh
```

Tüm kurulumlar yapıldıktan sonra ubuntuda kurulan rclient'lerin dashboard'da görünmesini engelleyen hatayı düzeltelim.

```ruby
nano VPSFix.sh
```
Aşağıdaki scripti olduğu gibi yapıştıralım.

```ruby
#!/bin/bash

# Version 1.1

# Hizmet dosyası yolunu tanımlar
SERVICE_FILE="/etc/systemd/system/default-interface-config.service"

# Komut dosyasının root ayrıcalıklarıyla çalıştırılıp çalıştırılmadığını kontrol eder
if [ "$EUID" -ne 0 ]; then
  echo "Lutfen root olarak çaliştirin."
  exit 1
fi

# Zaten yüklü değilse ethtool'u yükler
if ! command -v ethtool &> /dev/null; then
  echo "ethtool bulunamadi, yukleniyor..."
  apt-get update
  apt-get install -y ethtool
fi

# Varsayılan ağ arayüzünü alır
DEFAULT_INTERFACE=$(ip route show default | awk '/default/ {print $5}')

# systemd hizmet dosyasını oluşturur
cat <<EOL > $SERVICE_FILE
[Unit]
Description=Configure default network interface
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/sbin/ethtool -s $DEFAULT_INTERFACE speed 1000 duplex full autoneg off
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
EOL

# Yeni hizmeti tanımak için systemd'yi yeniden yükler
systemctl daemon-reload

# Hizmetin önyüklemede başlatılmasını sağlar
systemctl enable default-interface-config.service

# Starts the service immediately
systemctl start default-interface-config.service

echo "Varsayilan arayuz yapilandirmasi hizmeti $DEFAULT_INTERFACE arayuzune yuklendi ve baslatildi."
```

``` CTRL + X ``` tuşlarına ve ```Y ``` tuşuna basalım, ardından ```ENTER``` tuşuna basalım.
Dosyayı kullanılabilir hale getirelim.

```ruby
chmod +x VPSFix.sh
```

Şimdi scripti çalıştıralım.

```ruby
./VPSFix.sh
```

Kurulum bu kadar. Şimdi Windows uzak Masaüstü programı ile sunucumuca bağlanalım. Sunucumuzun IP adresini giriyoruz.

![image](https://github.com/ruesandora/Rivalz/assets/101149671/f7d00889-b44e-48bb-b98f-cab38b3cc7a8)


Script içine yazdığımız kullanıcı adı ve şifresini giriyoruz.

![image](https://github.com/ruesandora/Rivalz/assets/101149671/75ad52bd-2bfe-427a-9204-b7844b8a4219)

Herşey doğru bir şekilde tamamlandıysa bizi ubuntu masaüstü karşılayacak. Sol üstte ```Activities``` sekmesine basıyoruz. Eğer rıvalz dışında chrome üzerinden eklenti kuracaksanız (nodepay vb.) ```Firefox``` ile chrome indirip eklentileri yükleyebilirsiniz. Rivalz için okla gösterilen yere basıyoruz ve açılan ekranda ```Documents``` klasöründe rclient bizi bekliyoruz. Çift tıklayarak çalıştırabilirsiniz.

![image](https://github.com/ruesandora/Rivalz/assets/101149671/6c0939f7-a592-46a2-81c7-c376b2fa2a73)

**```NOT 1:``` Sunucunuzun özelliklerine göre masaüstündeki tıklamalar biraz geç algılanabilir, lütfen sabırlı olun.**

**```NOT 2:``` Masaüstünde yaptığınız tüm işlemler sunucunun tamamında etkili olabileceği ve içerisinde çalışan diğer nodelara etki edebileceği için lütfen sadece gerekli işlemleri yapın.**

**```NOT 3:``` Kurulumla ilgili tüm sorumluluk kuran kişiye aittir. Oluşabilecek herhangi bir hatadan veya kayıptan kuran kişi tamamen kendisi sorumludur.**

**```NOT 4:``` Kurulumdan sonra uzak masaüstüne bağlanırken sorun yaşayanlar aşağıdaki kodu deneyebilirler.(Login failed ve siyah ekranda kalma problemleri)**
```ruby
sysctl -a | grep disable_ipv6
```
Yukarıdaki kodun çıktısında **net.ipv6.conf.lo.disable_ipv6** değeri **1** olarak dönüyorsa aşağıdaki kodu çalıştırın.
```ruby
sysctl -w net.ipv6.conf.lo.disable_ipv6=0
```
Şimdi bağlanmayı tekrar deneyebilirsiniz.
