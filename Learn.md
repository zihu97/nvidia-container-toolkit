         -------------------------------------------------------------------------------------------------------------------------
        ｜         cli             ｜       low-level runtime        ｜      low-level runtime hook      ｜       k8s plugin       ｜
 ---------------------------------------------------------------------------------------------------------------------------------
| nvdia ｜  nvidia-container-cli   ｜     nvidia-container-runtime   ｜   nvidia-container-runtime-hook  ｜    k8s-device-plugin   ｜
 ---------------------------------------------------------------------------------------------------------------------------------
| hw    ｜   ascend-docker-cli     ｜                /               ｜       ascend-docker-runtime      ｜   ascend-device-plugin ｜
 ---------------------------------------------------------------------------------------------------------------------------------

nvidia-container-cli是存放在单独的仓库libnvidia-container
ascend-docker-cli是存放在ascend-docker-runtime/cli

nvidia-container-runtime是runc的wrapper,将docker的daemon.json中的runtime配置为nvidia-container-runtime即可使用

nvidia-container-runtime-hook和ascend-docker-runtime都是hook插件，将config.json中的"hooks":"prestart"配置即可解析args调用cli配置
可通过nvidia-ctk configure，hw run --install来自动修改

-----------------------cmd-----------------------
OCI规范见：
https://github.com/opencontainers/runtime-spec
最常用的low-level runtimes（满足OCI规范）：
https://github.com/opencontainers/runc
high-level runtimes：
containerd/docker等

oci-add-hooks/nvdia-container-runtime-hook 其实都是runc的wrapper

[nvdia-container-runtime/main].go

https://github.com/opencontainers/runtime-spec/blob/main/config.md
Open Container Initiative (OCI) 的运行时规范，典型的config.json文件：
{
    "ociVersion": "1.0.2",
    "process": {
        "terminal": true,
        "user": {
            "uid": 0,
            "gid": 0
        },
        "args": [
            "sh"
        ],
        "env": [
            "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
        ],
        "cwd": "/",
        "capabilities": {
            "bounding": [
                "CAP_AUDIT_WRITE",
                "CAP_KILL",
                "CAP_NET_BIND_SERVICE"
            ]
        },
        "rlimits": [
            {
                "type": "RLIMIT_NOFILE",
                "hard": 1024,
                "soft": 1024
            }
        ]
    },
    "root": {
        "path": "rootfs",
        "readonly": true
    },
    "hostname": "my-container",
    "mounts": [
        {
            "destination": "/proc",
            "type": "proc",
            "source": "proc"
        },
        {
            "destination": "/dev",
            "type": "tmpfs",
            "source": "tmpfs",
            "options": [
                "nosuid",
                "strictatime",
                "mode=755",
                "size=65536k"
            ]
        }
    ],
    "linux": {
        "namespaces": [
            {
                "type": "pid"
            },
            {
                "type": "network"
            },
            {
                "type": "ipc"
            },
            {
                "type": "uts"
            },
            {
                "type": "mount"
            }
        ],
        "resources": {
            "memory": {
                "limit": 536870912
            },
            "cpu": {
                "shares": 1024
            }
        },
        "devices": [
            {
                "path": "/dev/null",
                "type": "c",
                "major": 1,
                "minor": 3,
                "fileMode": 438,
                "uid": 0,
                "gid": 0
            }
        ]
    }
}
