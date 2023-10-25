<p align="center">
  <img src="https://user-images.githubusercontent.com/115700386/232264116-5cef4e89-92c9-4548-b392-fc82e02747e3.png" width="600px">
</p>

<h1 align="center">TUTORIAL BYPASS DPI KOMINFO MENGGUNAKAN OPENWRT</h1>

<p align="center">
    <b>Indonesia</b> | <a href="openwrt-tutorial.en.md">English</a>
</p>

<p align="center">
  <b><sup>Following this instruction is in your own risk. I'm not responsible for content you are trying to access after following this tutorial or the damage you done during the installation process. Please use it wisely and follow the instruction carefully</sup></b>
</p>

### Langkah Persiapan Instalasi
1. Pertama-tama, kita login ke OpenWRT kita melalui SSH sebagai admin dengan mengetikan IP dari OpenWRT kita
2. Setelah login, jalankan perintah ```opkg update``` di terminal dan tunggu sampai selesai<br>
3. Setelah itu, jalankan perintah ```opkg install iptables ip6tables git git-http nano``` untuk menginstall package yang digunakan untuk proses installasi Zapret
4. Pergi ke file tmp dengan mengetikan ```cd /tmp``` di Terminal<br>
5. Jalankan perintah ```git clone https://github.com/bol-van/zapret.git``` dan tunggu sampai selesai<br>

### Bypass DNS Nasional
Karena Kominfo menerapkan peraturan <a href="https://cms.dailysocial.id/wp-content/uploads/2015/05/slack_for_ios_upload_1024.png">DNS Nasional</a> yang dimana setiap ISP wajib membelokan Port 53 ke server ISP masing-masing dan probing tehnik bypassnya Zapret akan menggunakan resolve DNS dari OpenWRT kita, kita harus membypass DNS ISP terlebih dahulu sebelum melakukan proses installasi Zapret

<b>Ada 4 cara untuk bypass:</b><br>
- <b>Menggunakan host BebasID</b><br>
   - Buatlah file bernama bebasid di system dengan mengetikan ```touch /etc/bebasid``` di terminal
   - Buka filenya dengan mengetikan ```nano /etc/bebasid```
   - Silahkan copy isi hosts <a href="https://raw.githubusercontent.com/bebasid/bebasid/master/releases/hosts" target="_blank">BebasID</a> kedalam file yang kita buat tadi lalu save
   - Login ke OpenWRT melalui Web dengan mengetikan IP dari OpenWRT, lalu pergi ke <b>Network >> DHCP & DNS</b><br>
     ![image](https://user-images.githubusercontent.com/115700386/232265676-e1c5f8a7-e7ec-47e8-afe2-b703ee64e48f.png)
   - Pergi ke bagian <b>Resolv and Hosts Files</b>, tambahkan `/etc/bebasid` dibagian <b>Additional hosts files
</b> dan klik + seperti dibawah ini:<br>
     ![image](https://user-images.githubusercontent.com/115700386/232265727-0596f6b2-5e58-4cdd-bb24-302a44f76162.png)
   - Klik Save & Apply 
   - Untuk mengecek apakah Host BebasID sudah terpasang dengan benar, jalankan `nslookup lamanlabuh.aduankonten.id` di terminal OpenWRT<br>
     <p align="center"><img src="https://user-images.githubusercontent.com/115700386/232265834-d88744e5-bb59-462f-82e9-20c24434a6b3.png"><br>
     <b><sup>Jika hasilnya seperti diatas ini, maka konfigurasi host BebasID telah berhasil</sup></b></p><br>
- <b>Menggunakan DNS dengan port selain 53</b><br>
   - Login ke OpenWRT
   - Pergi ke <b>Network >> Interfaces</b> dan Edit WAN (atau apapun interface sumber internet kalian)
     ![image](https://user-images.githubusercontent.com/115700386/232383389-329fceba-d178-4ca7-88b7-448c7c5dcc19.png)
   - Pergi ke <b>Advanced Settings</b> dan uncheck opsi `Use DNS servers advertised by peer`
     ![image](https://user-images.githubusercontent.com/115700386/232383541-96bba9e0-712a-415f-bdde-ffcdc3a6408c.png)
   - Setting DNS ke 127.0.0.1 dan klik +
   - Lalu Save dan Apply
   - Setelah itu, pergi ke <b>Network >> DHCP and DNS</b><br>
     ![image](https://user-images.githubusercontent.com/115700386/232383622-711dde04-c1b9-4099-8101-3234084c22fc.png)
   - Dibagian DNS Forwarding, silahkan isi DNS dan alt-portnya dengan format `IP#PORT`
     Contohnya seperti ini:<br>
     ![image](https://user-images.githubusercontent.com/115700386/232384543-1a87981d-2186-45d1-b056-9b2a5ed146c9.png)<br>
     <sup>Contoh menggunakan DNS dari BebasID dengan alt-port 1753</sup><br>
     <p align="center"><sup><b>Untuk pengguna Moratel/Oxygen, jangan menggunakan alt-port 5353 karena Moratel memblokir port tersebut. Gunakan DNS yang mempunyai alternatif port selain 5353 jika anda memakainya</b></sup></p>
   - Lalu klik + dan Save & Apply
- <b>Menggunakan DNS-over-TLS (Stubby)</b><br>
   - Sebelum anda menggunakan DoT di OpenWRT, pastikan port 853 tidak terblokir oleh ISP
   - Cek dengan menjalankan perintah `curl -v portquiz.net:853` di terminal<br>
     <p align="center">
     <img src="https://user-images.githubusercontent.com/115700386/232675750-9c30e527-7ce9-4f04-acfc-db092894a346.png"><br>
     <sup>Pastikan muncul tulisan <b>`Port test successful!`</b><br>Jika tidak, silahkan pakai cara lain seperti memakai hosts, alt port, dan DoH</p>
   - Jika tes berhasil, jalankan command `opkg update` di terminal
   - Lalu jalankan command `opkg install stubby` dan tunggu sampai selesai<br>
     ![image](https://user-images.githubusercontent.com/115700386/232675427-8aa0b7b5-e498-4e9c-8c6d-7c8d40f4e505.png)<br>
   - Jalankan command `nano /etc/stubby/stubby.yml` untuk mengedit config Stubby
   - Catat port yang digunakan<br>
     ![image](https://user-images.githubusercontent.com/115700386/232676399-e662ed62-eb88-42c7-a278-aa6ee7a32136.png)<br>
      <sup>Nanti akan digunakan di konfigurasi DNS</sup>
   - Jika anda mau mengganti atau DNS provider default (Cloudflare 1.1.1.1), silahkan ganti bagian `address-data:` dan `tls_auth_name:`<br>
     ![image](https://user-images.githubusercontent.com/115700386/232677110-6173b9cc-2568-4d9a-800f-8488e8fd0435.png)<br>
     Seperti Contoh, untuk mengganti ke DNS-over-TLS milik BebasID:<br>
     ![image](https://user-images.githubusercontent.com/115700386/232692104-7ca5eda9-1ad4-4d27-9e8d-ec2f3c2e379e.png)<br>
   - Save hasilnya lalu jalankan perintah `nano /etc/config/stubby`
   - Ubah `option manual '0'` menjadi `option manual '1'` lalu save<br>
     ![image](https://user-images.githubusercontent.com/115700386/232694974-294f09e0-0f3f-434a-af28-777f009c3593.png)
   - Jalankan perintah `service stubby restart` dan `service stubby enable`
   - Setelah itu, login ke OpenWRT melalui Web Interface
   - Pergi ke <b>Network >> Interfaces</b> dan Edit WAN (atau apapun interface sumber internet kalian)
     ![image](https://user-images.githubusercontent.com/115700386/232383389-329fceba-d178-4ca7-88b7-448c7c5dcc19.png)
   - Pergi ke <b>Advanced Settings</b> dan uncheck opsi `Use DNS servers advertised by peer`
     ![image](https://user-images.githubusercontent.com/115700386/232383541-96bba9e0-712a-415f-bdde-ffcdc3a6408c.png)
   - Setting DNS ke 127.0.0.1 dan klik +
   - Lalu Save dan Apply
   - Pergi ke <b>Network >> DHCP and DNS</b><br>
     ![image](https://user-images.githubusercontent.com/115700386/232383622-711dde04-c1b9-4099-8101-3234084c22fc.png)
   - Dibagian DNS Forwarding, silahkan isi DNS dengan hasil settingan tadi `127.0.0.1#5453`
     ![image](https://user-images.githubusercontent.com/115700386/232693231-9a1f0c02-df3e-4383-9fc3-a830945fb1bf.png)
  - Klik + dan <b>Save and Apply</b>
  - Check dengan melakukan nslookup ke domain yang diblokir Kominfo (Mis: `nslookup reddit.com`) Pastikan tidak muncul IP internet positif<br>
    ![image](https://user-images.githubusercontent.com/115700386/232694308-b934f8e4-31d3-4922-99be-30b4ca1adaa5.png)<br>   
- <b>Menggunakan DNS-over-HTTPS</b><br>
<p align="center"><b>( TO BE CONTINUED... )</b></p>

### Installasi Zapret
1.  Setelah selesai menjalankan perintah <i>git clone</i> di terminal dan membypass DNS Nasional / DNS ISP, silahkan navigasi ke /tmp/zapret dengan mengetikan ```cd /tmp/zapret``` di terminal<br>
2.  Jalankan ```./install-easy.sh``` di Terminal
3.  Jika muncul pesan 
    ```
    easy install is supported only from default location : /opt/zapret 
    currently its run from /tmp/zapret
    do you want the installer to copy it for you (default : N) (Y/N) ?
    ```
    Silahkan ketik `Y` dan Enter
4.  Untuk Firewall, pilih iptables dengan mengetik 1 dan enter<br>
    ![image](https://user-images.githubusercontent.com/115700386/232266676-b901a3a2-3cf1-48d1-87ee-bbce9d8e0721.png)<br>
5.  Untuk enable IPv6 support, silahkan pilih `Y` untuk jaga-jaga<br>
    ![image](https://user-images.githubusercontent.com/115700386/232266756-e24ba6de-df68-4b65-bce7-e39c3e8669b3.png)<br>
6.  Untuk Mode, silahkan pilih `3` dan enter<br>
    ![image](https://user-images.githubusercontent.com/115700386/232266796-e218738c-8399-4469-93c4-b81146730fdc.png)<br>
7.  Pastikan enable HTTP support, HTTPS support di enablekan dengan pilih `Y` pada saat proses instalasi <br>
    ![image](https://user-images.githubusercontent.com/115700386/232266856-6abfb4da-e52a-41a9-a720-ae71e2ed293a.png)<br>
8.  Setelah itu klik Enter sampai selesai
9.  Hapus folder Zapret di /tmp untuk menyimpan memory dengan cara pergi ke `cd /tmp` dan jalankan `rm zapret -r`<br>

### Konfigurasi Zapret
1.  Pergi ke folder `cd /opt/zapret/` dan jalankan script `./install_bin.sh`
2.  Saat proses sudah berhasil, silahkan jalankan `./blockpage.sh` untuk mencari setting Zapret yang optimal untuk ISP anda
3.  Jika muncul pesan:
    ```
    specify domain(s) to test. multiple domains are space separated.
    domain(s) (default: rutracker.org) :
    ```
    Silahkan isi dengan domain yang diblokir Kominfo (Misalkan: `reddit.com`, `vimeo.com`, `omegle.com`, dll)
4.  Jika muncul presan `ip protocol version`, sesuaikan dengan jaringan anda
    - Misalkan jaringan anda hanya support IPv4, maka ketik `4` dan enter
    - Tetapi, jika Jaringan anda support IPv4 dan IPv6, maka ketik `46` dan enter
5.  Klik Enter sampai pesan `how many times to repeat each test (default: 1)`. Ketik `2` dan Enter
6.  Setelah itu, akan muncul pesan berbunyi `do all test despite of result?`. Ketik Y dan Enter
7.  Tunggu hingga Zapret menemukan settingan optimal untuk ISP anda 
8.  Jika sudah selesai, akan muncul seperti ini:<br>
    ![image](https://user-images.githubusercontent.com/115700386/232272140-d9b62495-493e-4170-93d9-c8b70eae965a.png)<br>
    Catat hasilnya
9.  Setelah itu, stop service Zapret di OpenWRT dengan mengetikan `service zapret stop`
10. Edit Config nya dengan mengetikan `nano /opt/zapret/config`
11. Cari bagian ini di confignya dan replace dengan config yang sesuai pada gambar yang anda dapat tadi
    ```
    #NFQWS_OPT_DESYNC_HTTP=
    #NFQWS_OPT_DESYNC_HTTPS=
    #NFQWS_OPT_DESYNC_HTTP6=
    #NFQWS_OPT_DESYNC_HTTPS6=
    ```
    Hilangkan # pada NFQWS<br>
    Untuk curl_test_https_tls12, isi di bagian HTTPS dan HTTPS6 (Tulis setelah huruf nfqws di hasil tadi)<br>
    Dan, untuk curl_test_http, isi di bagian HTTP dan HTTP6 (Tulis setelah huruf nfqws di hasil tadi)<br><br>
    <b>Sebagai Contoh:</b> (Jangan ikuti ini melainkan sesuaikan dengan apa yang anda dapat)
    ```
    NFQWS_OPT_DESYNC_HTTP="--hostcase"
    NFQWS_OPT_DESYNC_HTTPS="--dpi-desync=split2"
    NFQWS_OPT_DESYNC_HTTP6="--hostcase"
    NFQWS_OPT_DESYNC_HTTPS6="--dpi-desync=split2"
    ```
12. Lalu, save hasilnya dan nyalakan Zapret dengan mengetikan `service zapret start`
13. Jangan lupa enable iptables dan zapret dengan mengetikan `service zapret enable` dan `service iptables enable` agar servicenya berjalan saat startup

## Masalah pada aplikasi banking (Credit to One for the step)
Beberapa bank akan menolak request anda jika anda mengaktifkan Zapret di Router OpenWRT karena itu, kita harus buat whitelist untuk beberapa situs bank

1. Pergi ke folder `/opt/zapret` lalu ketik perintah `nano whitelist.txt`
2. Lalu isikan:
    ```
    bankbjb.co.id
    bankbsi.co.id
    bankmandiri.co.id
    bca.co.id
    bi.go.id
    blubybcadigital.id
    bni.co.id
    bri.co.id
    btn.co.id
    cimbniaga.co.id
    danamon.co.id
    hanabank.co.id
    hsbc.co.id
    jago.com
    klikbca.com
    maybank.co.id
    permatabank.com
    permatanet.com
    sc.com
    ```
      <sup><b>(Silahkan tambahkan jika kurang)</b></sup><br>
3. Lalu Save dan jalankan command `chmod 755 whitelist.txt` di terminal
4. Edit config Zapret dengan mengetikan `nano config`
5. Cari line bertuliskan `NFQWS_OPT_DESYNC` dan tambah ` --hostlist-exclude=/opt/zapret/whitelist.txt` disetiap akhir line sebelum `"`
   - Sebagai contoh, Konfigurasi Zapret kita seperti ini:
     ```
     # CHOOSE NFQWS DAEMON OPTIONS for DPI desync mode. run "nfq/nfqws --help" for option list
     DESYNC_MARK=0x40000000
     NFQWS_OPT_DESYNC="--dpi-desync=fake --dpi-desync-ttl=0 --dpi-desync-ttl6=0 --dpi-desync-fooling=badsum"
     NFQWS_OPT_DESYNC_HTTP="--hostcase"
     NFQWS_OPT_DESYNC_HTTPS="--dpi-desync=split2"
     NFQWS_OPT_DESYNC_HTTP6="--hostcase"
     NFQWS_OPT_DESYNC_HTTPS6="--dpi-desync=split2"
     ```
   - Yang kita harus lakukan adalah menambah `--hostlist-exclude=/opt/zapret/whitelist.txt` dibelakang sehingga menjadi seperti ini:
     ```
     # CHOOSE NFQWS DAEMON OPTIONS for DPI desync mode. run "nfq/nfqws --help" for option list
     DESYNC_MARK=0x40000000
     NFQWS_OPT_DESYNC="--dpi-desync=fake --dpi-desync-ttl=0 --dpi-desync-ttl6=0 --dpi-desync-fooling=badsum --hostlist-exclude=/opt/zapret/whitelist.txt"
     NFQWS_OPT_DESYNC_HTTP="--hostcase --hostlist-exclude=/opt/zapret/whitelist.txt"
     NFQWS_OPT_DESYNC_HTTPS="--dpi-desync=split2 --hostlist-exclude=/opt/zapret/whitelist.txt"
     NFQWS_OPT_DESYNC_HTTP6="--hostcase --hostlist-exclude=/opt/zapret/whitelist.txt"
     NFQWS_OPT_DESYNC_HTTPS6="--dpi-desync=split2 --hostlist-exclude=/opt/zapret/whitelist.txt"
     ```
6. Save dan restart Zapret dengan mengetikan `service zapret restart`
