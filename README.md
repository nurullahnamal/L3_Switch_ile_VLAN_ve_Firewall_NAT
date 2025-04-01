L3 Switch ile VLAN Yönlendirme ve Firewall Üzerinden NAT Yapılandırması
L3 Switch Üzerinde VLAN Routing ve Firewall Entegrasyonu<br>

Ağ ortamında L3 switch kullanarak routing işlemi yaparken, firewall ile bağlantının doğru yapılandırılması kritik bir adımdır. Bu makalede, L3 switch üzerinde VLAN yapılandırması, routing etkinleştirme ve firewall ile internet erişimi sağlama adımları ele alınacaktır.<br>

L3 Switch Üzerinde VLAN Yapılandırması<br>
 ![image alt](https://github.com/nurullahnamal/L3_Switch_ile_VLAN_ve_Firewall_NAT/blob/main/TOPOLOJI.png)
İlk olarak, VLAN'ları oluşturup, her bir VLAN’a IP atayarak aktif hale getiriyoruz:<br>

enable<br> configure terminal<br> vlan 10,11,12,13,14,100<br> interface vlan 10<br> no shutdown<br> ip address 192.168.10.1 255.255.255.0<br>

interface vlan 11<br> no shutdown<br> ip address 192.168.11.1 255.255.255.0<br>

interface vlan 12<br> no shutdown<br> ip address 192.168.12.1 255.255.255.0<br>

interface vlan 13<br> no shutdown<br> ip address 192.168.13.1 255.255.255.0<br>

interface vlan 14<br> no shutdown<br> ip address 192.168.14.1 255.255.255.0<br>

interface vlan 100<br> no shutdown<br> ip address 192.168.100.1 255.255.255.0<br>

VLAN’ların Access Mode’a Alınması<br>

VLAN’ların iletişim kurabilmesi için uygun portlara atanması gerekmektedir:<br>

interface gigabitethernet 0/0<br> switchport mode access<br> switchport access vlan 10<br>

interface gigabitethernet 0/1<br> switchport mode access<br> switchport access vlan 11<br>

interface gigabitethernet 0/2<br> switchport mode access<br> switchport access vlan 100<br>

Routing’in Etkinleştirilmesi<br>

VLAN’lar arasında iletişimi sağlamak için routing’i aktif hale getiriyoruz:<br>

configure terminal<br> ip routing<br>

Firewall Yapılandırması<br>

FortiGate cihazında LAN tarafına bakan arayüze IP tanımlanması:<br>

config system interface<br> edit port1<br> set ip 192.168.100.254 255.255.255.0<br> end<br>

Bu aşamada, FortiGate varsayılan olarak VLAN 10 ve VLAN 11 ağlarını bilmediği için, bu ağlara ulaşabilmesi adına statik yönlendirme eklenmelidir.<br>

config router static<br> edit 1<br> set gateway 192.168.100.1<br> set distance 10<br> set device port1<br> set dst 192.168.11.0 255.255.255.0<br> set dst 192.168.10.0 255.255.255.0<br> next<br> end<br>
 ![image alt](https://github.com/nurullahnamal/L3_Switch_ile_VLAN_ve_Firewall_NAT/blob/main/firewall%20%20network%20static%20route.png)
Alternatif olarak, tüm 192.168.x.x ağlarını özetleyerek (summary) şu şekilde ekleyebiliriz:<br>
 ![image alt](https://github.com/nurullahnamal/L3_Switch_ile_VLAN_ve_Firewall_NAT/blob/main/static%20yol%20k%C4%B1saltma.png)
192.168.0.0 255.255.0.0 192.168.100.1<br>

Firewall WAN Yapılandırması<br>
 ![image alt](https://github.com/nurullahnamal/L3_Switch_ile_VLAN_ve_Firewall_NAT/blob/main/WAN_GATEWAY.png)
İnternet erişimi sağlamak için FortiGate’in WAN portuna uygun bir IP adresi atanmalıdır. Gerçek bir ISS bağlantısında, ISS tarafından sağlanan IP kullanılmalıdır. Ancak, test ortamında yerel bir IP tanımlanabilir.<br>

Güvenlik Politikaları (IPv4 Policy)<br>
 ![image alt](https://github.com/nurullahnamal/L3_Switch_ile_VLAN_ve_Firewall_NAT/blob/main/GENEL_NET%20POLICY.png)
Varsayılan olarak firewall tüm trafiği kapalı tutar. Bu nedenle, gerekli güvenlik politikalarını tanımlamak gerekmektedir.<br>

config firewall policy<br> edit 1<br> set srcintf port1<br> set dstintf wan1<br> set srcaddr all<br> set dstaddr all<br> set action accept<br> set schedule always<br> set service ALL<br> set logtraffic enable<br> end<br>

L3 Switch Üzerinde Varsayılan Yönlendirme (Default Route)<br>

Tüm internet trafiğini firewall’a yönlendirmek için switch üzerinde varsayılan yönlendirme eklenir:<br>

ip route 0.0.0.0 0.0.0.0 192.168.100.254<br>

Bu yapılandırma ile, L3 switch üzerindeki VLAN'lar arası iletişim sağlanırken, internet çıkışı firewall üzerinden gerçekleştirilecektir.<br>

Sonuç: Bu adımları takip ederek, L3 switch ve firewall entegrasyonunu başarılı bir şekilde tamamlamış olursunuz. Switch VLAN’lar arası routing yaparken, firewall güvenlik ve internet erişimini yönetir.<br>
 ![image alt](https://github.com/nurullahnamal/L3_Switch_ile_VLAN_ve_Firewall_NAT/blob/main/8.8.8.8%20ping%20.gif)
