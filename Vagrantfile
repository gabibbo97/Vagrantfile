# Settings
BOX = "debian/stretch64"
CREATE_RUN_ON_ALL_SCRIPT = true
NETWORK_SUBNET = "10.10.10"
NODES_COUNT = 3
NODES_PREFIX = "node"
PROVISION_ANSIBLE_CREATE_INVENTORY = true
PROVISION_ANSIBLE_PLAYBOOK = ""
PROVISION_ANSIBLE_START_AT_TASK = ""
PROVISION_SCRIPT = <<SCRIPT
SCRIPT
SSH_ADD_USER_KEYS = true
VM_CPUS = 1
VM_MEMORY = 2000

# Logic
USER_HOME = Dir.home()
SCRIPT_DIR = Dir.getwd

if CREATE_RUN_ON_ALL_SCRIPT
  RUN_ON_ALL_SCRIPT = "#{SCRIPT_DIR}/runOnAll.sh"
  if not File.file?(RUN_ON_ALL_SCRIPT)
    $runOnAllScript = "#!/usr/bin/env bash\n"
    (1..NODES_COUNT).each() do |i|
      $runOnAllScript << "ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null vagrant@#{NETWORK_SUBNET}.#{9 + i} -- \"$@\"\n"
    end
    File.open(RUN_ON_ALL_SCRIPT, 'w') { |file| file.write($runOnAllScript) }
    File.chmod(0755, RUN_ON_ALL_SCRIPT)
  end
end

if PROVISION_ANSIBLE_CREATE_INVENTORY
  ANSIBLE_INV_FILE = "#{SCRIPT_DIR}/inventory"
  if not File.file?(ANSIBLE_INV_FILE)
    $ansibleInvFile = ""
    (1..NODES_COUNT).each() do |i|
      $ansibleInvFile << "#{NODES_PREFIX}-#{i} ansible_host=#{NETWORK_SUBNET}.#{9 + i} ansible_user=vagrant ansible_ssh_extra_args='-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null'\n"
    end
    File.open(ANSIBLE_INV_FILE, 'w') { |file| file.write($ansibleInvFile) }
  end
end

if SSH_ADD_USER_KEYS
  $ssh_user_key_script =  "mkdir -p ~/.ssh;"
  $ssh_user_key_script << "sudo mkdir -p ~/.ssh;"
  Dir.foreach("#{USER_HOME}/.ssh") do |file|
    if File.extname(file) == '.pub'
      $ssh_key = File.read("#{USER_HOME}/.ssh/#{file}")
      $ssh_user_key_script << "echo '#{$ssh_key}' | tee -a ~/.ssh/authorized_keys > /dev/null;"
      $ssh_user_key_script << "echo '#{$ssh_key}' | sudo tee -a ~/.ssh/authorized_keys > /dev/null;"
    end
  end
  $ssh_user_key_script << "sudo chmod -R 0600 ~/.ssh/authorized_keys;"
  $ssh_user_key_script << "chmod -R 0600 ~/.ssh/authorized_keys;"
end

Vagrant.configure("2") do |config|

  config.vm.box = BOX

  config.vm.synced_folder ".", "/vagrant", disabled: true

  if SSH_ADD_USER_KEYS
    config.vm.provision "shell", privileged: false, inline: $ssh_user_key_script
  end

  if PROVISION_SCRIPT.length > 0
    config.vm.provision "shell", inline: PROVISION_SCRIPT
  end

  (1..NODES_COUNT).each() do |i|
    config.vm.define "#{NODES_PREFIX}-#{i}" do |subconfig|
      subconfig.vm.hostname = "#{NODES_PREFIX}-#{i}"
      subconfig.vm.network "private_network", ip: "#{NETWORK_SUBNET}.#{9 + i}"

      subconfig.vm.post_up_message = <<-MESSAGE
        ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null vagrant@#{NETWORK_SUBNET}.#{9 + i}
      MESSAGE

      if PROVISION_ANSIBLE_PLAYBOOK.length > 0 && i == NODES_COUNT
        subconfig.vm.provision "ansible" do |ansible|
          ansible.playbook = PROVISION_ANSIBLE_PLAYBOOK
          ansible.limit = "all"
          ansible.start_at_task = PROVISION_ANSIBLE_START_AT_TASK
        end
      end
    end
  end

  config.vm.provider :libvirt do |libvirt|
    libvirt.cpus = VM_CPUS
    libvirt.memory = VM_MEMORY
    libvirt.cpu_mode = "host-passthrough"
  end

end
