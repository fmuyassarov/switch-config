# switch-config

![components](network.drawio.svg)

## Inventory file yaratish

Bu yerda biz control qilmoqchi bolgan switch yoki routerlarni IP larini yozib chiqamiz. Undan keyin pastda boshqa faylda asosiy show run comandasini yozamiz, va oxiri run qilganimizda, bizni control qilmochi bolgan switch yoki routerlarinimizni IP larini inventory file dan topasan deb.

```ini
[switches]
; Switch/routerlarni IP larini yozib chiqasiz. Har biriga prefix
; qilib nom berishiz mumkin, masalan switch1, switch2 ha hokzazo
switch1 192.168.111.101
switch2 192.168.111.102

; switch larga cherez SSH kirsh uchun kerak bolgan variables
[switches:vars]
ansible_network_os=ios
ansible_connection=network_cli
; O'zgartirishiz kerak
ansible_user=ansible
ansible_password=cisco123
```

## Ansible Playbook ni yaratish. Asosiy ish mana shu yerda.

Bu yerda endi switch yoki routerlarni IP larini emas, balkim u larni qisqa ismini aytyapmiz. Yani, `hosts: switches` degani man pastdagi task larni switched gruppasiga kiradigan switch yoki routerlar da run qilmoqchi man. Ansible endi inventory file ga boradida, `[switches]` degan gruppani pastidagi barcha IP larda tasklarni run qilib chiqadi bitta ma bitta.

File nomi `show_run.yaml` misol uchun.
```yaml
---
   - name: show run commandasi, xoxlagan nom berishiz mumkin
     # inventory dagi gruppani nomi
     hosts: switches
     gather_facts: no
     # cherez network orqali ulanasan degani
     connection: network_cli

     tasks:
      - name: show run comandasi va nihoyat :)
      # Cisco ni operatsionkasi IOS bolgani uchun, ios_command
      # degan module ni ishlatamiz, cisco ni asosiy comandlari uchun doim shu.
        ios_command:
          commands:
          - show run
        # chiqgan natijani o'zgaruvchiga soxranit qilamiz, orgaruvchimizni nomi natija
        register: natija

       - name: show run ni natijasini (natija) save qilish file ga. 
        # Har bir switch uchun, alohida file yaratiladi o'szini nomi bilan
        # yani, natija switch1.txt, switch2.txt
         copy:
           content: "{{ natija.stdout | replace('\\n', '\n') }}"
           dest: "{{ inventory_hostname }}.txt"
```

## Run qilish

Admin laptopdan run qilasiz. Run qilishdan oldin admin laptop da Ansible ustanovka qilingan bo'lishi kerak 

Run qilish

```shell
$ ansible-playbook -i inventory.ini  show_run.yaml
```