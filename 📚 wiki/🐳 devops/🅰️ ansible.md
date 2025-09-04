**Control node** - система, на которой установлен ansible. Команды `ansible` и `ansible-inventory` выполняются на ней.
**Inventory** - список нод, которыми ansible будет управлять.
**Managed node** - система, которой ansible управляет.
**Playbook** - скрипт управления.

Нужно для:
- убрать повторы и упростить работу
- управлять конфигурацией системы
- плавающие обновления
- нет *агентов* на целевых узлах - все управление через один хост
- ансибл ничего не будет делать, если система уже в состоянии, которое описано в настройках

SSH ключ основного хоста должен быть добавлен в *authorized keys* все хостов в inventory.

Структура
```
|- ansible
|--- inventory
|----- hosts.yml
|--- roles
|----- role_name
|------- tasks
|--------- main.yml
|--------- other.yml
|------- vars
|--------- main.yml
```

Пример `ansible/inventory/hosts.yml`
```yaml
windows:
  hosts:
    10.159.2.72:
      ansible_user: ptadmin
      ansible_password: c#V#LLOIeumQ[5ENYQ7#i~o6=!
    second_windows:
	  ansible_host: 10.159.2.71
      ansible_user: ptadmin
      ansible_password: Cu-F*c)V7u;Kz*Ds3Ads)]sT.C
```

Пример `tasks`
```yaml
- name: Windows | Create directory
  ansible.windows.win_file:
    path: "{{ install_directory }}"
    state: directory
- name: Windows | Upload package to target host
  ansible.windows.win_copy:
    src: "{{ agent_src_path }}"
    dest: "{{ install_directory }}"
- name: Windows | Stop service (ignore error if not exists or run)
  ansible.windows.win_service:
    name: "{{ win_service.name }}"
    state: stopped
  ignore_errors: true
```

Пример `vars`
```yaml
ansible_connection: winrm
ansible_port: 5985
_slash: \
install_directory: C:\Program Files\Standoff\Agent
win_service:
  binary: agent.exe
  name: AgentService
  description: AgentService
  display_name: AgentService
nexus_host: 10.125.255.245
```

Запуск
```bash
ansible-playbook -i ansible/inventory ansible/deploy_on_windows.yml --extra-vars "playbook_hostname=10.159.2.72 agent_src_path=/mnt/c/PT/standoff_agent/build/windows_agent.develop.664cb27e.zip agent_flags='--log_level INFO'" -vvv
```