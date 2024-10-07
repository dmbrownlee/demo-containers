ARG debian_version=bookworm
FROM debian:${debian_version}

ARG username=user
ARG uid=1000
ARG gid=1000
#ARG addonpkgs="bash-completion tmux vim-nox vim-ale vim-airline vim-airline-themes vim-youcompleteme"
ARG addonpkgs="bash-completion jq python3-paramiko tmux vim-ale vim-airline vim-common vim-fugitive vim-nox vim-youcompleteme"
ARG helm_version=3.16.1
ARG arch=amd64


RUN /usr/bin/apt update -y \
  && /usr/bin/apt install -y bind9-dnsutils curl git gpg inetutils-ping lsb-release man openssh-client python3 python3-pip python3-venv python3-bcrypt software-properties-common sshpass sudo ${addonpkgs} unzip \

  && /usr/bin/curl -fsSL https://get.opentofu.org/opentofu.gpg | /usr/bin/tee /etc/apt/keyrings/opentofu.gpg >/dev/null \
  && /usr/bin/curl -fsSL https://packages.opentofu.org/opentofu/tofu/gpgkey | gpg --no-tty --batch --dearmor -o /etc/apt/keyrings/opentofu-repo.gpg >/dev/null \
  && /usr/bin/chmod a+r /etc/apt/keyrings/opentofu.gpg /etc/apt/keyrings/opentofu-repo.gpg \
  && /bin/echo -e "deb [signed-by=/etc/apt/keyrings/opentofu.gpg,/etc/apt/keyrings/opentofu-repo.gpg] https://packages.opentofu.org/opentofu/tofu/any/ any main\ndeb-src [signed-by=/etc/apt/keyrings/opentofu.gpg,/etc/apt/keyrings/opentofu-repo.gpg] https://packages.opentofu.org/opentofu/tofu/any/ any main\n" | /usr/bin/tee /etc/apt/sources.list.d/opentofu.list > /dev/null \
  && /usr/bin/chmod a+r /etc/apt/sources.list.d/opentofu.list \
  && /usr/bin/apt update -y \
  && /usr/bin/apt install -y tofu \
  && ln -s /usr/bin/tofu /usr/bin/terraform \
  && /usr/bin/curl -o /root/helm.tar.gz https://get.helm.sh/helm-v${helm_version}-linux-amd64.tar.gz \
  && /usr/bin/tar xvf /root/helm.tar.gz -C /root \
  && /usr/bin/cp /root/linux-amd64/helm /usr/local/bin/helm \
  && /usr/bin/rm -rf /root/helm.tar.gz /root/linux-amd64 \
  && /usr/sbin/groupadd -g ${gid} ${username} \
  && /usr/sbin/useradd -m -d /home/${username} -g ${username} -G sudo -s /bin/bash -u ${uid} ${username} \
  && /bin/echo "${username} ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/99${username}

USER ${username}
WORKDIR /home/${username}
RUN /bin/echo -e '\nif [ -f ~/.venv/ansible/bin/activate ]; then\n  . ~/.venv/ansible/bin/activate\nfi' | /usr/bin/tee -a ~/.bashrc \
  && /bin/echo -e '\neval $(/usr/bin/ssh-agent -s)' | /usr/bin/tee -a ~/.bashrc \
  && /usr/bin/tofu -install-autocomplete \
  && /usr/bin/git clone https://github.com/dmbrownlee/demo-proxmox-terraform.git \
  && /usr/bin/mkdir .venv \
  && /usr/bin/python3 -m venv .venv/ansible \
  && /bin/bash -c 'source ~/.venv/ansible/bin/activate && /usr/bin/env pip3 install ansible' \
  && /bin/bash -c '~/.venv/ansible/bin/ansible-galaxy collection install kubernetes.core' \
  && /bin/bash -c '~/.venv/ansible/bin/ansible-galaxy collection install cloud.terraform'

ENTRYPOINT ["/bin/bash"]
