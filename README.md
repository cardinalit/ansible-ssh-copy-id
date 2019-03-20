# ansible-ssh-copy-id

`Ansible cli-playbook` для копирования **ssh-ключей** пользователя с текущей машины, откуда запускается команда, и ключа с личного компьютера. 

## Зависимости

Ansible, для подключение по паролю через ssh, использует консольную утилиту `sshpass`. Она будет необходима для корректной работы `cli-playbook`. 

```sh
$> sudo apt-get install sshpass
```

## Подготовка 

1. Клонируем репозиторий  
   ```sh
   $> git clone https://github.com/cardinalit/ansible-ssh-copy-id.git
   ```
2. Переходим в папку
   ```sh
   $> cd ansible-ssh-copy-id/
   ```
3. Редактируем файл `hosts` и настраиваем глобальные переменные для всех хостов в группе `[all:vars]`
   ```sh
   $> vim ./hosts

   1 [all:vars]
   2 ansible_user=ansible-task
   3 ansible_ssh_pass="{{ vault_all_ansible_pass }}"
   4 ansible_become=true
   5 ansible_become_pass="{{ vault_all_ansible_become_pass }}"
   6 ansible_default_remote_user=admin
   7
   8 [my-host-group]
   9 localhost
   ```
4. Далее необходимо создать / отредактировать существующий `ansible-vault`, который хранит чувствительные данные, такие как:  
   - **ssh пароль**
   - **пароль от sudo**  
   
   4.1 Создаем свой `ansible-vault`
      ```sh
      $> ansible-vault create ./group_vars/all

      New Vault password:
      Confirm New Vault password:
      ```
      Далее Вы увидите, откроется Ваш консольный редактор по умолчанию, и будет ожидать завершения ввода данных и сохранения файла. После этого, `ansible-vault` сохранит и зашифрует все данные с использованием Вашего пароля в директорию, которую Вы указали при запуске команды Выше.
      
      Формат записи данных сл.:  
      `var_name: value` 
      
      **Пример:** 
      ```yml
      vault_all_ansible_pass: your_password
      vault_all_ansible_become_pass: your_password
      ```
      > Эти переменные будут использоваться, как переменные во всем `cli-playbook`, поэтому важно давать им *максимально читабельные и интуитивно-понятные имена*.  
      
      Как Вы могли заметить ранее, `var_name` уже использовались в файле инвентаризации `hosts`, когда мы его редактировали.  
   
   4.2 Редактируем уже имеющийся в примере `ansible-vault`
      > **Пароль, используемый в примере:** `test`
      ```sh
      $> ansible-vault edit ./group_vars/all

      Vault password:
      ```
      Далее откроется редактор со сл. содержимым: 
      ```yml
      vault_all_ansible_pass: your_password
      vault_all_ansible_become_pass: your_password
      ```
      , где необходимо заменить `<your_password>` на Ваши реальные данные и сохранить. 
      
      Чтобы посмотреть данные, которые находятся в `ansible-vault`, используется команда: 
      ```sh
      $> ansible-vault view ./group_vars/all

      Vault password: 

      vault_all_ansible_pass: your_password
      vault_all_ansible_become_pass: your_password
      
      $>
      ```
5. Запуск 
   ```sh
   $> cd ./cli-playbook/
   $> ./ans-ssh-copy-id --ask-vault-pass

   Vault password: 

   ВАЖНО!

   Выполните на своем (домашнем) компьютере сл. команду `cat ~/.ssh/id_rsa.pub`
   Вывод команды скопируйте и вставьте сюда:
   ```
   После вставки Вашего публичного ключа, запустятся таски на выполнение для всех машин, известных в файле инвентаризации `hosts`. 