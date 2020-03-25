-   Giới thiệu:
    -   Dm-writeboost là công cụ hỗ trợ SSD-caching (dùng SSD làm caching để write xuống HDD).
    -   Cơ chế dm-writeboost kiểm soát ba lớp khác nhau
        -   Bộ đệm RAM (rambuf), bộ nhớ đệm thiết bị (cache_dev, ví dụ SSD) và thiết bị sao lưu (back_dev, ví dụ: HDD).
        -   Tất cả dữ liệu được lưu trữ đầu tiên trong bộ đệm RAM và khi bộ đệm RAM đầy, dm-writeboost thêm khối siêu dữ liệu (có tổng kiểm tra) vào bộ đệm RAM để tạo Log.
        -   Nhật ký được ghi vào cache_dev theo tuần tự bởi background thread và sau đó được write xuống lại backing device.
    -   Đỉểm khác nhau với  vs dm-cache or bcache
        -   Vì theo cơ chế  log, nên file log sẽ chứa meta data cho 127 lần write xuống device, khác với các cơ chế  caching khác là sẽ ghi liên tục sẽ làm giảm tuổi thọ của SSD, chúng ta sẽ giảm thiểu được rủi ro đĩa bị hư bất chợt
-   Mô hình:

                    -------             	    -------
        Data    --> | SSD | --dm-writeboost-->  | HDD | <-- Access from another process
                    -------             	    -------

-   Cài đặt các package cần thiết:
    -   `sudo yum install -y epel-release`
    -   `sudo curl -sL -o /etc/yum.repos.d/khara-dm-writeboost.repo https://copr.fedoraproject.org/coprs/khara/dm-writeboost/repo/epel-7/khara-dm-writeboost-epel-7.repo`
    -   `sudo yum install -y dm-writeboost-dkms dm-writeboost-tools writeboost`
-   Cắm đĩa HDD và SSD
-   Check đĩa using:

        [root@localhost lib]# lsblk
        NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
        sda                         8:0    0 465.8G  0 disk
        ├─sda1                      8:1    0   200M  0 part /boot/efi
        ├─sda2                      8:2    0     1G  0 part /boot
        └─sda3                      8:3    0 464.6G  0 part
        ├─centos_votranchi-root     253:0  0    50G  0 lvm  /
        ├─centos_votranchi-swap     253:1  0   7.8G  0 lvm  [SWAP]
        └─centos_votranchi-home     253:2  0 406.8G  0 lvm  /home
        sdb                         8:0    0 465.8G  0 disk  <---- Giả sử đây là đống đĩa HDD mới cắm vào
        sdc                         8:0    0 465.8G  0 disk  <---- Còn đây là đống SSD

-  Dùng fdisk để tạo phân vùng cho ssd và hdd
    -   SSD:
        -   Chỉ lấy 1 phần để làm cache (1% của dung lượng cache là memory cần để writeboost thực hiện)
        -   VD: 800GB memory cần 8G RAM
    -   HDD: Lấy full

            [root@localhost lib]# lsblk
            NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
            sda                         8:0    0 465.8G  0 disk
            ├─sda1                      8:1    0   200M  0 part /boot/efi
            ├─sda2                      8:2    0     1G  0 part /boot
            └─sda3                      8:3    0 464.6G  0 part
            ├─centos_votranchi-root     253:0  0    50G  0 lvm  /
            ├─centos_votranchi-swap     253:1  0   7.8G  0 lvm  [SWAP]
            └─centos_votranchi-home     253:2  0 406.8G  0 lvm  /home
            sdb                         8:0    0 465.8G  0 disk
            ├─sdb1                      8:1    0 465.8G  0 part <--- Thực tế có thể hơi khác
            sda                         8:0    0 465.8G  0 disk
            ├─sdc1                      8:1    0 465.8G  0 part <--- Thực tế có thể hơi khác

-   Để xài được dm-writeboost 2 phân vùng này cần là phân vùng LVVM
-   Tạo phân vùng LVVM:
    -   SSĐ:
        -   vgcreate ssddisk /dev/sdc1
        -   lvcreate --name ssd --size 5G sdddisk &lt;----- Lấy size phân vùng vừa tạo
    -   HDD:
        -   vgcreate hdddisk /dev/sdb1
        -   lvcreate --name hdd --size 5G hdddisk &lt;----- Lấy size phân vùng vừa tạo

-   Dùng lệnh lsblk check:

        [root@localhost lib]# lsblk
        NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
        sda                         8:0    0 465.8G  0 disk
        ├─sda1                      8:1    0   200M  0 part /boot/efi
        ├─sda2                      8:2    0     1G  0 part /boot
        └─sda3                      8:3    0 464.6G  0 part
        ├─centos_votranchi-root     253:0  0    50G  0 lvm  /
        ├─centos_votranchi-swap     253:1  0   7.8G  0 lvm  [SWAP]
        └─centos_votranchi-home     253:2  0 406.8G  0 lvm  /home
        sdb                         8:0    0 465.8G  0 disk
        ├─sdb1                      8:1    0 465.8G  0 part <--- Thực tế có thể hơi khác
        ├─hdddisk-hdd 253:0    0    465.8G 0 lvm
        sda                         8:0    0 465.8G  0 disk
        ├─sdc1                      8:1    0 465.8G  0 part <--- Thực tế có thể hơi khác
        ├─ssddisk-sdd 253:0    0    465.8G 0 lvm

-   Cuối cùng sẽ là lệnh tạo phân vùng của dm-writeboost

        wbcreate --reformat --writeback_threshold=80 wbdev /dev/mapper/hdddisk-hdd /dev/mapper/ssddisk-ssd

-   [writeback threshold note](https://www.youtube.com/watch?v=xU0ICkgTLTo)

-   Check xem có tạo được phân vùng này chưa

        [root@localhost lib]# dmsetup table
        wbdev ..... <---- ở đây
        centos_votranchi-home: 0 853139456 linear 8:3 16254976
        centos_votranchi-swap: 0 16252928 linear 8:3 2048
        centos_votranchi-root: 0 104857600 linear 8:3 869394432

-   `mount /dev/mapper/wbdev /mnt/wbdev/` &lt;--- Thư mục mong muốn chứa video recording , ...Log

-   Troubershooting:

-   Restart server
    -   Chạy lại lệnh `wbcreate wbdev /dev/mapper/hdddisk-hdd /dev/mapper/ssddisk-ssd` nhưng không có `--reformat`
-   Remove a WB device
    -   ``` wbremove wbdev ``` removes a WB device after flushing data in RAM buffer and then writing back all cache blocks. This is the way Dmirty Smirnov's writeboost script suggests. (Recommended)
    -   ``` wbremove wbdev --nowriteback ``` remove a WB device without writing back all cache blocks.
