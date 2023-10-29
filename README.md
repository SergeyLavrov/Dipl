# Дипломпаня работа по курсу Системный администратор Linux

В файле main.tf прописан процер разворачивания виртуальных машин, в частности посмотрим на код описания первой виртуальной машины для сервера nginx:

```
resource "yandex_compute_instance" "nginxserver1" {
  name = "nginxserver1"
  zone = "ru-central1-a"

scheduling_policy {
    preemptible = true
  }
  
  resources{
    cores = 2
    core_fraction = 20
    memory = 4
  }

  boot_disk{
    initialize_params {
      image_id = var.image_id
      size = 10
      description = "boot disk for nginx_server1"
    }
  }
  network_interface {
    subnet_id = yandex_vpc_subnet.subnet1.id
    nat = true
  }
  metadata = {
    user-data = "${file("./meta.txt")}"
  }
  provisioner "remote-exec" {
    inline = [
      "apt-get -qq install python -y",
    ]

    connection {
      host        = "${self.ipv4_address}"
      type        = "ssh"
      user        = "sergey"
      private_key = "${file("./meta.txt")}"
    }
  }

  provisioner "local-exec" {
    environment {
      PUBLIC_IP  = "${self.ipv4_address}"
      PRIVATE_IP = "${self.ipv4_address_private}"
    }

    working_dir = "../Ansible/"
    command     = "ansible-playbook -u sergey --private-key ${var.ssh_key_private} nginx.yml -i ${self.ipv4_address},"
  }
}
```

в данном куске кода описано какие ресурсы должна создать Terraform на облаке Яндекса, а так же что должно произойти после разворачивания инфраструктуры, в частности в ниже приведённом куске кода описано что после развёртывания должен запускается Ansible и выполнять копирования конфигураций сервисов.

```
  provisioner "remote-exec" {
    inline = [
      "apt-get -qq install python -y",
    ]

    connection {
      host        = "${self.ipv4_address}"
      type        = "ssh"
      user        = "sergey"
      private_key = "${file("./meta.txt")}"
    }
  }

  provisioner "local-exec" {
    environment {
      PUBLIC_IP  = "${self.ipv4_address}"
      PRIVATE_IP = "${self.ipv4_address_private}"
    }

    working_dir = "../Ansible/"
    command     = "ansible-playbook -u sergey --private-key ${var.ssh_key_private} nginx.yml -i ${self.ipv4_address},"
  }
```

Все переменные вынесены в отдельный файл  :  variable.tf
Данные для авторизации , хранятся в meta.txt

Все преднастроенные конфигурации лежат в папке ../Ansible/





