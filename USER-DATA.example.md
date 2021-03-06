# USER-DATA EXAMPLE
<!---
vim: et sr sw=2 ts=2 smartindent:
-->

_... Ensure this container runs on CoreOS when expected i.e. before other custom containers_

```systemd
#cloud-config
coreos:
  units:
    - name: "coreos_aws_hostcfg.service"
      enable: true
      content: |
        [Unit]
        Description=Provide utility scripts, services for aws host
        After=docker.service docker.socket network-online.target
        Requires=docker.service docker.socket network-online.target

        [Service]
        Type=oneshot
        RemainAfterExit=yes
        TimeoutSec=60
        Environment="_C=coreos_aws_hostcfg"
        Environment="_DI=opsgang/coreos_aws_hostcfg:stable"
        EnvironmentFile=-/etc/custom/docker_image_versions
        ExecStartPre=-/usr/bin/docker kill ${_C}
        ExecStartPre=-/usr/bin/docker rm -f ${_C}
        ExecStartPre=/usr/bin/docker pull ${_DI}
        ExecStart=/bin/bash -c " \
        docker run                                   \
          --name ${_C}                               \
          --privileged                               \
          -v /home/core:/home/core                   \
          -v /etc/custom:/etc/custom                 \
          -v /etc/coreos:/etc/coreos                 \
          ${_DI} /bin/bash /assets/scripts/configure.sh"
        ExecStop=-/usr/bin/docker kill ${_C}
        ExecStopPost=-/usr/bin/docker rm -f ${_C}

        [Install]
        WantedBy=multi-user.target

    - name: "docker_network_apps.service"
      enable: true
      command: "start"
      content: |
        [Unit]
        Description=create a docker bridge network for all app containers
        After=docker.service docker.socket network-online.target
        Requires=docker.service docker.socket network-online.target

        [Service]
        Type=oneshot
        ExecStart=/bin/bash -c "                               \
          /usr/bin/docker network inspect apps >/dev/null 2>&1 \
          || /usr/bin/docker network create -d bridge apps     \
        "
        RemainAfterExit=yes
        TimeoutSec=60
        User=root

        [Install]
        WantedBy=multi-user.target

    - name: "instance_info.service"
      enable: true
      content: |
        [Unit]
        Description=Get instance info from ec2 metadata and tag api
        After=coreos_aws_hostcfg.service
        Requires=coreos_aws_hostcfg.service

        [Service]
        Type=oneshot
        RemainAfterExit=yes
        TimeoutSec=60
        Environment="_C=instance_info"
        Environment="_DI=opsgang/aws_env:stable"
        EnvironmentFile=-/etc/custom/docker_image_versions
        ExecStartPre=-/usr/bin/docker kill ${_C}
        ExecStartPre=-/usr/bin/docker rm -f ${_C}
        ExecStartPre=/usr/bin/docker pull ${_DI}
        ExecStart=/bin/bash -c " \
        docker run                         \
          --name ${_C}                     \
          -v /home/core/bin:/home/core/bin \
          -v /etc/custom:/etc/custom       \
          ${_DI} /bin/bash /home/core/bin/instance_info"
        ExecStop=-/usr/bin/docker kill ${_C}
        ExecStopPost=-/usr/bin/docker rm -f ${_C}

        [Install]
        WantedBy=multi-user.target

    - name: "credstash_to_fs.service"
      enable: true
      command: "start"
      content: |
        [Unit]
        Description=Get instance info from ec2 metadata and tag api
        After=instance_info.service
        Requires=instance_info.service

        [Service]
        Type=oneshot
        RemainAfterExit=yes
        TimeoutSec=60
        Environment="_C=credstash_to_fs"
        Environment="_DI=opsgang/aws_env:stable"
        EnvironmentFile=-/etc/custom/docker_image_versions
        ExecStartPre=-/usr/bin/docker kill ${_C}
        ExecStartPre=-/usr/bin/docker rm -f ${_C}
        ExecStartPre=/usr/bin/docker pull ${_DI}
        ExecStart=/bin/bash -c " \
        docker run                         \
          --name ${_C}                     \
          -v /home/core/bin:/home/core/bin \
          -v /etc/custom:/etc/custom       \
          ${_DI} /bin/bash /home/core/bin/credstash_to_fs"
        ExecStop=-/usr/bin/docker kill ${_C}
        ExecStopPost=-/usr/bin/docker rm -f ${_C}

        [Install]
        WantedBy=multi-user.target

```

