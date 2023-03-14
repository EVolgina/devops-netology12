# 1.Узнайте о sparse-файлах (разряженных).
ответ: Разрежённый файл (англ. sparse file) — файл, в котором последовательности нулевых байтов заменены на информацию\
об этих последовательностях (список дыр)
# 2. Могут ли файлы, являющиеся жёсткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?
ответ: Не могут, так как ссылки будут на один и тот же inode, где хранятся права доступа и прочие атрибуты файла.

Установила все дополнительные утилиты\
![ustanvka](https://github.com/EVolgina/devops-netology12/blob/main/ustanovka.PNG)
# 3. Сделайте vagrant destroy на имеющийся инстанс Ubuntu. Замените содержимое Vagrantfile следующим:
path_to_disk_folder = './disks'
host_params = {
    'disk_size' => 2560,
    'disks'=>[1, 2],
    'cpus'=>2,
    'memory'=>2048,
    'hostname'=>'sysadm-fs',
    'vm_name'=>'sysadm-fs'
}
Vagrant.configure("2") do |config|
    config.vm.box = "bento/ubuntu-20.04"
    config.vm.hostname=host_params['hostname']
    config.vm.provider :virtualbox do |v|

        v.name=host_params['vm_name']
        v.cpus=host_params['cpus']
        v.memory=host_params['memory']

        host_params['disks'].each do |disk|
            file_to_disk=path_to_disk_folder+'/disk'+disk.to_s+'.vdi'
            unless File.exist?(file_to_disk)
                v.customize ['createmedium', '--filename', file_to_disk, '--size', host_params['disk_size']]
            end
            v.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', disk.to_s, '--device',
            0, '--type', 'hdd', '--medium', file_to_disk]
        end
    end
    config.vm.network "private_network", type: "dhcp"
end\
Эта конфигурация создаст новую виртуальную машину с двумя дополнительными неразмеченными дисками по 2,5 Гб.

# 4. Используя fdisk, разбейте первый диск на два раздела: 2 Гб и оставшееся пространство.
![fdisk](https://github.com/EVolgina/devops-netology12/blob/main/fdisk.PNG)
![fdisk1](https://github.com/EVolgina/devops-netology12/blob/main/fdisk1.PNG)

# 5. Используя sfdisk, перенесите эту таблицу разделов на второй диск.
![sfdisk](https://github.com/EVolgina/devops-netology12/blob/main/zad5.PNG)
# 6. Соберите mdadm RAID1 на паре разделов 2 Гб.
![raid1](https://github.com/EVolgina/devops-netology12/blob/main/zad6.PNG)
# 7.Соберите mdadm RAID0 на второй паре маленьких разделов.
![raid0](https://github.com/EVolgina/devops-netology12/blob/main/zad7.PNG)
![cat](https://github.com/EVolgina/devops-netology12/blob/main/7%20cat.PNG)
# 8.Создайте два независимых PV на получившихся md-устройствах.
Проверим физические тома LVM командой pvscan\
Инициализируем тома для работы LVM командой pvcreate\
Снова проверяем pvscan
![raid1](https://github.com/EVolgina/devops-netology12/blob/main/zd8.PNG)
# 9.Создайте общую volume-group на этих двух PV.
Физические разделы уже сделаны, делаем из них группу томов и проверяем что получилось
![pv](https://github.com/EVolgina/devops-netology12/blob/main/zd9.PNG)
![pv1](https://github.com/EVolgina/devops-netology12/blob/main/zd91.PNG)
# 10.Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.
При помощи lvcreate создаем и проверяем что получилось
![pv2](https://github.com/EVolgina/devops-netology12/blob/main/zd10.PNG)
![pv3](https://github.com/EVolgina/devops-netology12/blob/main/zd101.PNG)
# 11.Создайте mkfs.ext4 ФС на получившемся LV.
Создаем и проверяем
![pv4](https://github.com/EVolgina/devops-netology12/blob/main/zd11.PNG)
# 12.Смонтируйте этот раздел в любую директорию, например, /tmp/new.
Создаем папку и монтируем раздел в директорию for_logical_volume1\
Проверяем\ 
![pv5](https://github.com/EVolgina/devops-netology12/blob/main/zd12.PNG)
# 13.Поместите туда тестовый файл, например, wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz.
![pv6](https://github.com/EVolgina/devops-netology12/blob/main/zd13.PNG)
# 14.Прикрепите вывод lsblk.
![lsblk](https://github.com/EVolgina/devops-netology12/blob/main/zd14.PNG)
# 15. Протестируйте целостность файла:
root@vagrant:~# gzip -t /tmp/new/test.gz
root@vagrant:~# echo $?
0
![lsblk1](https://github.com/EVolgina/devops-netology12/blob/main/zd15.PNG)
# 16.Используя pvmove, переместите содержимое PV с RAID0 на RAID1.
![lsblk2](https://github.com/EVolgina/devops-netology12/blob/main/zd16.PNG)
# 17.Сделайте --fail на устройство в вашем RAID1 md.
![lsblk3](https://github.com/EVolgina/devops-netology12/blob/main/zd17.PNG)
# 18.Подтвердите выводом dmesg, что RAID1 работает в деградированном состоянии.
![lsblk31](https://github.com/EVolgina/devops-netology12/blob/main/zd18.PNG)
# 19.Протестируйте целостность файла — он должен быть доступен несмотря на «сбойный» диск:
root@vagrant:~# gzip -t /tmp/new/test.gz
root@vagrant:~# echo $?
0
![lsblk33]()
# 20. Погасите тестовый хост — vagrant destroy.
6 mdadm --create --verbose /dev/md1 -l 1 -n 2 /dev/sd{b1,c1}
7 mdadm --create --verbose /dev/md2 -l 0 -n 2 /dev/sd{b2,c2}:
![lsblk32]()
