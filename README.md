L3 Switch ile VLAN Yönlendirme ve Firewall Üzerinden NAT Yapılandırması
Günümüzde ağ altyapılarında Layer 3 (L3) switch’ler kullanılarak VLAN’lar arasında routing işlemleri gerçekleştirilebilir. Bu makalede, L3 switch üzerinde VLAN yapılandırması yaparak, firewall entegrasyonu ile birlikte güvenli bir ağ geçidi oluşturma sürecini ele alacağız.


Firewall Nat
1. L3 Switch Üzerinde VLAN ve Routing Yapılandırması
L3 switch üzerinde VLAN yapılandırmasını aşağıdaki adımları takip ederek gerçekleştirebiliriz:

VLAN Oluşturma ve IP Tanımlama
Öncelikle VLAN’ları oluşturuyoruz ve her VLAN’a IP adresi atıyoruz:

ENABLE
CONFIGURE TERMINAL
VLAN 10,11,12,13,14,100

INTERFACE VLAN 10
NO SHUTDOWN
IP ADDRESS 192.168.10.1 255.255.255.0

INTERFACE VLAN 11
NO SHUTDOWN
IP ADDRESS 192.168.11.1 255.255.255.0

INTERFACE VLAN 12
NO SHUTDOWN
IP ADDRESS 192.168.12.1 255.255.255.0

INTERFACE VLAN 13
NO SHUTDOWN
IP ADDRESS 192.168.13.1 255.255.255.0

INTERFACE VLAN 14
NO SHUTDOWN
IP ADDRESS 192.168.14.1 255.255.255.0

INTERFACE VLAN 100
NO SHUTDOWN
IP ADDRESS 192.168.100.1 255.255.255.0

Routing İşleminin Aktif Edilmesi
VLAN’lar arasında iletişimi sağlamak için switch üzerinde IP routing’i etkinleştirmemiz gerekmektedir:

Configure Terminal mod içerisinde yazmalıyız.

IP ROUTING
Switchport Modlarının Tanımlanması
Her VLAN için ilgili portları access moda alıyoruz:

INTERFACE GIGABITETHERNET 0/0
SWITCHPORT MODE ACCESS
SWITCHPORT ACCESS VLAN 10

INTERFACE GIGABITETHERNET 0/1
SWITCHPORT MODE ACCESS
SWITCHPORT ACCESS VLAN 11

INTERFACE  GIGABITETHERNET 0/3
SWITCHPORT MODE ACCESS
SWITCHPORT ACCESS VLAN 12
Firewall tarafı için VLAN yapılandırmasını da şu şekilde yapıyoruz

INTERFACE  GIGABITETHERNET 0/2
SWITCHPORT MODE ACCESS
SWITCHPORT ACCESS VLAN 100
2. FortiGate Firewall Yapılandırması
Firewall cihazının LAN tarafına bakan portunu aşağıdaki gibi yapılandırıyoruz:

CONFIG SYSTEM INTERFACE
EDIT PORT1
SET IP 192.168.100.254 255.255.255.0
END
Bu yapılandırmadan sonra, firewall’un VLAN 10, VLAN 11ve VLAN 12 ağlarını tanımadığını unutmayalım. Bu yüzden firewall üzerinde statik rotalar tanımlamamız gerekmektedir.


CONFIG ROUTER STATIC  
EDIT 1  
SET GATEWAY 192.168.100.1  
SET DISTANCE 10
SET DEVICE PORT1
SET DST 192.168.11.0 255.255.255.0
NEXT
END
Bu işlemler tamamlandıktan sonra firewall arayüzüne 192.168.100.254 adresinden erişim sağlanabilir. Alternatif olarak, Network > Static Routes > Create New sekmesinden VLAN’lara yönelik statik rotaları manuel olarak ekleyebiliriz. Örneğin:



192.168.0.0 255.255.0.0 192.168.100.1
Lan interface den sonra WAN interface yapılandırmasını yapalım. Local yapı dan dolayı ip adresi 192.168.1.110 şeklinde yazarız.

Modeme uygun ip olmalı.


3. Firewall WAN Bağlantısı ve Politikalarının Tanımlanması
LAN yapılandırmasını tamamladıktan sonra, firewall’un WAN tarafına bakan arayüzüne IP adresi atamamız gerekmektedir. Eğer ağımız bir test ortamıysa, ISS tarafından verilen IP yerine yerel bir IP kullanabiliriz.


Firewall Static route yapılandırması.


4. L3 Switch Üzerinde Varsayılan Yönlendirme (Default Route) Tanımlama
Switch üzerindeki networklerin dış dünya ile iletişimini sağlamak için varsayılan yönlendirme ekliyoruz:


IP ROUTE 0.0.0.0 0.0.0.0 192.168.100.254
Bu yapılandırma ile, L3 switch üzerindeki VLAN’lar arası iletişim sağlanırken, internet çıkışı firewall üzerinden gerçekleştirilecektir.

Sonuç: Bu adımları takip ederek, L3 switch ve firewall entegrasyonunu başarılı bir şekilde tamamlamış olursunuz. Switch VLAN’lar arası routing yaparken, firewall güvenlik ve internet erişimini yönetir.

GN3 İLE 192.168.10.10 CİHAZI GOOGLE DNS İLE İLETİŞİM KURABİLİYOR.
