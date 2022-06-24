minio-cluster
=========

Роль для развертывания кластера S3-совместимого хранилища MINIO, добавление новых узлов и дисков в существующий кластер.

Requirements
------------

---

Role Variables
--------------

Инициализация переменных в vars/main.yml

 minio_server_release: ""
  Можно указать конкретный релиз. Если оставить пустым - скачается последняя версия minio server 
  Релиз указывается в виде RELEASE.2019-06-27T21-13-50Z (то что идет после 'minio.', список релизов смотри здесь https://dl.min.io/server/minio/release/linux-amd64/archive)
  Кластер тестировался на версии RELEASE.2022-06-20T23-13-45Z

 mount_point: /mnt
 pattern_dir: 'miniod\d+'
  Эти две	переменные указывают на имена точек монтирования дисков для данных s3 хранилища
  Диски должны быть смонтированы по одинаковым путям с отличием в числовое значение в конце имени директории монтирования, при этом числа должны идти подряд, т.к. при создании пула необходимо указывать массив дисков в формате {1...3}, т.е. в данном случае диски должны быть примонтированны по следующим путям:
  /mnt/miniod1
  /mnt/miniod2
  /mnt/miniod3

 minio_access_key: ""
 minio_secret_key: ""
  Ключи для доступа по API можно задать вручную

 minio_root_user: ""
 minio_root_password: ""
  Логин пароль администратора можно задать вручную. Если оставить пустыми будут значения по умолчанию (minioadmin:minioadmin)

 minio_lb_url: "http://minio:9000"
  url балансировщика нагрузки

 minio_server_env_extra: ""
  Переменная для дополнительных параметров env файла minio

 minio_server_cluster_nodes:
  Список пулов кластера MinIO. Пул задается в следующем формате:
  'http://minio0{1...2}:9000/mnt/miniod{1...3}' - в данном случае пул имеет 2 сервера по 3 диска в каждом
  Минимальное общее количество дисков пула - 4!
  При расширении кластера следующие пулы должны иметь суммарно кратное первому пулу количество дисков, в данном примере следующий пул должен иметь или 6, или 12, или 18 и т.д. дисков, например
	minio_server_cluster_nodes:
	  - 'http://minio0{1...2}:9000/mnt/miniod{1...3}'
	  - 'http://minio0{3...5}:9000/mnt/miniod{1...2}'

	minio_server_cluster_nodes:
	  - 'http://minio0{1...2}:9000/mnt/miniod{1...3}'
	  - 'http://minio03:9000/mnt/miniod{1...18}'

	minio_server_cluster_nodes:
	  - 'http://minio0{1...2}:9000/mnt/miniod{1...3}'
	  - 'http://minio0{3...8}:9000/mnt/miniod1'
	  - 'http://minio0{9...11}:9000/mnt/miniod{1...2}'

---

   !!! При расширении кластера в inventory файл необходимо включить существующие ноды кластера, а в переменную minio_server_cluster_nodes: записать все существующие и добавляемые пулы !!!
   !!! При этом внести изменения в backend балансировщика после успешного расширения кластера !!!

---

Инициализация переменных в defaults/main.yml

  Все значения заданны по умолчанию, изменений при раскатке не требуется

Dependencies
------------

---

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - name: Install MinIO
    - hosts: minio
      roles:
         - minio-cluster

License
-------

BSD

Author Information
------------------

Роль написана на основе роли с github https://github.com/atosatto/ansible-minio.git
