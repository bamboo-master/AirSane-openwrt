Первоисточник: https://github.com/polikasov/AirSane-openwrt </br>
и https://github.com/cmangla/AirSane-openwrt</br>


Новая инструкция, без докера:
</br>
1. Открываем PowerShell от администратора и вводим 
`wsl --install` - так мы установим сусбсистему Linux, заодно последнюю версию Ubuntu. Процесс не быстрый и потребует перезагрузки. Можно добыть Ubuntu и другими путями VMWare, Vbox, ПК...
2. Когда имеется установленный Ubuntu нужно установить немало пакетов необходимых для SDK, вот примерный список, может даже избыточный:

`sudo apt update && sudo apt upgrade -y` пока что обновимся

Сами пакеты.
`sudo apt install build-essential libncursesw5-dev libssl-dev rsync wget unzip python3 asciidoc bash bc binutils bzip2 fastjar flex gawk gcc genisoimage gettext intltool jikespg libgtk2.0-dev libncurses5-dev mercurial patch perl-modules python2.7-dev ruby sdcc util-linux xsltproc subversion git g++ zlib1g-dev make cmake -y`

Теперь нужно найти и загрузить SDK для вашей архитектуры, также стоит обратить внимание на версию OpenWrt.
</br>

4. Заходитм в домашний каталог `cd ~`
  
Вам придется потрудится чтобы понять какой у вас чип, как определил я? <br/>Все очень просто! делаю `opkg update` на роутере и первой строкой получаю: 

`Downloading http://downloads.openwrt.org/releases/19.07.8/targets/brcm47xx/legacy/packages/Packages.gz` <br/>         И тут мы сразу видим то что нам нужно.<br/> А именно "19.07.8",  "brcm47xx", "legacy". 

5. Теперь я могу загрузить нужный архив. 

`wget https://archive.openwrt.org/releases/19.07.8/targets/brcm47xx/legacy/openwrt-sdk-19.07.8-brcm47xx-legacy_gcc-7.5.0_musl.Linux-x86_64.tar.xz` 

Как вы видите есть сайт [https://archive.openwrt.org/](https://archive.openwrt.org/releases/) где можно ходить по каталогам и выбрать то что нужно.

6. Распакуем `tar -xvf  openwrt-sdk*` и зайдем в папку `cd openwrt-sdk*/` 

Пропишем репозиторий</br>

`echo "src-git airsaned https://github.com/bamboo-master/AirSane-openwrt.git" >> feeds.conf.default` 

обновим "feeds"

`./scripts/feeds update base packages airsaned && make defconfig && ./scripts/feeds install airsaned`

соберем

`make package/airsaned/compile V=s -j5` <br/> где j5 это 4 ядра, соответственно если у вас 2 ядра -пишем j3 итд.

Собирается все относительно быстро и без ошибок, пакеты лежат по пути `/openwrt-sdk-19.07.8-brcm47xx-legacy_gcc-7.5.0_musl.Linux-x86_64/bin/packages/mipsel_mips32/airsaned` это примерный путь, версии могут быть другие, вместо mipsel_mips32 может быть тоже что-то другое.

В проводнике Windows 10 есть пингвин и там все наши файлы, а копировать на сам роутер можно через [WinSCP](https://winscp.net/eng/download.php).

Устанавливать `opkg install ./airsaned*` при условии нахождения в этой же папке.

После установки нужно создать правило для файрвола на порт `8090 tcp` 

`/etc/init.d/airsaned enable` автозагрузка

`/etc/init.d/airsaned start` старт

если это метод не работает, то можно прописать в вайле `/etc/rc.local`, соответственно `airsaned`

Через браузер сканирует нормально `IP_Роутера:8090` 

Windows теперь находит ваш сканер, предлгает скачать сканер от windows, тут важно чтобы microsoft store был на [месте](https://4pda.to/forum/index.php?showtopic=926185). Проверял на Windows 10/11, древний сканер Mustek600CU все отлично работает.
Android PrintHand не видит, расхваленый [ими](https://github.com/SimulPiscator/AirSane) [Mopria Scan](https://play.google.com/store/apps/details?id=org.mopria.scan.application&hl) тоже не видит, но можно добавить вручную ip адрес и порт 8090, с таким же успехом можно использовать [sanedroid](https://play.google.com/store/apps/details?id=com.sane.droid&hl). MacOS, iOS, Linux не проверял. 

Если у вас старая Windows (8.1/8/7/Vista/XP)  установите [sanetwain](https://sanetwain.ozuzo.net/downloads/setup137.exe) в связке с [naps2](https://www.naps2.com/download) </br> Либо сканируйте через браузер, никакой eSCL работать не будет.

Некоторые пакеты выложу [тут](https://github.com/bamboo-master/AirSane-openwrt/tree/master/packages).
