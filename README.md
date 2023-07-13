Отказоустойчивость в облаке. Васёв А.В.

## Задание 1
### Возьмите за основу задание 1 из модуля 7.3 «Подъём инфраструктуры в Яндекс Облаке».

Сделайте terraform playbook, который:

    1. создаст 2 идентичные виртуальные машины. Используйте аргумент count для создания таких ресурсов;
    2. создаст таргет-группу. Поместите в неё созданные на шаге 1 виртуальные машины;
    3. создаст сетевой балансировщик нагрузки, который слушает на порту 80, отправляет трафик на порт 80 виртуальных машин и http healthcheck на порт 80 виртуальных машин.

## Задание 2*
### Nginx нужно будет поставить тоже автоматизированно. Для этого вам подложить файл установки Nginx в user-data-ключ метадаты виртуальной машины.

```java
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
}
provider "yandex" {
  token = var.yc_iam_token
  cloud_id = "b1gneo9mi2c02taf7erq"
  folder_id = "b1gneo9mi2c02taf7erq"
  zone = "ru-central1-b"
}
resource "yandex_compute_instance" "vm" {
count = 2
name = "vm${count.index}"
resources {
    core_fraction = 20
    cores = 2
    memory = 2
}
boot_disk {
    initialize_params {
      image_id = "fd85f37uh98ldl1omk30"
      size = 5
    }
  }
metadata = {
user-data = file("./metadata.yaml")
}
network_interface {
    subnet_id = yandex_vpc_subnet.subnet-1.id
    nat       = true
}
placement_policy {
   placement_group_id = "${yandex_compute_placement_group.group1.id}"
  }
}
resource "yandex_lb_target_group" "netology-tr" {
name      = "netologytr"
  target {
    subnet_id = yandex_vpc_subnet.subnet-1.id
    address   = yandex_compute_instance.vm[0].network_interface.0.ip_address
  }
  target {
    subnet_id = yandex_vpc_subnet.subnet-1.id
    address   = yandex_compute_instance.vm[1].network_interface.0.ip_address
  }
}
resource "yandex_lb_network_load_balancer" "netology-bl" {
  name = "netologybl"
  listener {
    name = "netology-listener"
    port = 80
    external_address_spec {
      ip_version = "ipv4"
    }
  }
  attached_target_group {
    target_group_id = yandex_lb_target_group.netology-tr.id
    healthcheck {
      name = "http"
      http_options {
        port = 80
        path = "/"
      }
    }
  }
}
resource "yandex_vpc_network" "network-1" {
name = "network1"
}
resource "yandex_vpc_subnet" "subnet-1" {
name = "subnet1"
  zone = "ru-central1-b"
  v4_cidr_blocks = ["192.168.10.0/24"]
  network_id = "${yandex_vpc_network.network-1.id}"
}
resource "yandex_compute_placement_group" "group1" {
  name = "netology-test1"
}
resource "yandex_compute_snapshot" "snapshot-1" {
  name           = "snapshot"
  source_disk_id = "${yandex_compute_instance.vm[1].boot_disk[0].disk_id}"
}
```

![alt text](https://github.com/rus42/ResiliencyInTheCloud/blob/main/Task_1.1.png)

![alt text](https://github.com/rus42/ResiliencyInTheCloud/blob/main/Task_1.2.png)

![alt text](https://github.com/rus42/ResiliencyInTheCloud/blob/main/Task_1.3.png)

![alt text](https://github.com/rus42/ResiliencyInTheCloud/blob/main/Task_1.4.png)

![alt text](https://github.com/rus42/ResiliencyInTheCloud/blob/main/Task_1.5.png)

