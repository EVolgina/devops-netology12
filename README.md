Узнайте о sparse-файлах (разряженных).
Могут ли файлы, являющиеся жёсткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?
Сделайте vagrant destroy на имеющийся инстанс Ubuntu. Замените содержимое Vagrantfile следующим:
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
            v.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', disk.to_s, '--device', 0, '--type', 'hdd', '--medium', file_to_disk]
        end
    end
    config.vm.network "private_network", type: "dhcp"
end
Эта конфигурация создаст новую виртуальную машину с двумя дополнительными неразмеченными дисками по 2,5 Гб.

Используя fdisk, разбейте первый диск на два раздела: 2 Гб и оставшееся пространство.

Используя sfdisk, перенесите эту таблицу разделов на второй диск.

Соберите mdadm RAID1 на паре разделов 2 Гб.

Соберите mdadm RAID0 на второй паре маленьких разделов.

Создайте два независимых PV на получившихся md-устройствах.

Создайте общую volume-group на этих двух PV.

Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.

Создайте mkfs.ext4 ФС на получившемся LV.

Смонтируйте этот раздел в любую директорию, например, /tmp/new.

Поместите туда тестовый файл, например, wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz.

Прикрепите вывод lsblk.

Протестируйте целостность файла:

root@vagrant:~# gzip -t /tmp/new/test.gz
root@vagrant:~# echo $?
0
Используя pvmove, переместите содержимое PV с RAID0 на RAID1.

Сделайте --fail на устройство в вашем RAID1 md.

Подтвердите выводом dmesg, что RAID1 работает в деградированном состоянии.

Протестируйте целостность файла — он должен быть доступен несмотря на «сбойный» диск:

root@vagrant:~# gzip -t /tmp/new/test.gz
root@vagrant:~# echo $?
0
Погасите тестовый хост — vagrant destroy.
6 mdadm --create --verbose /dev/md1 -l 1 -n 2 /dev/sd{b1,c1}
7 mdadm --create --verbose /dev/md2 -l 0 -n 2 /dev/sd{b2,c2}:
