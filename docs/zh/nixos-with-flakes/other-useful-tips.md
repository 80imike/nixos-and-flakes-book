
## 使用 Git 管理 NixOS 配置 {#git-manage-nixos-config}

NixOS 的配置文件是纯文本，因此跟普通的 dotfiles 一样可以使用 Git 管理。

此外 Nix Flakes 配置也不一定需要放在 `/etc/nixos` 目录下，可以放在任意目录下，只要在部署时指定正确的路径即可。

> 我们在前面第 3 小节的代码注释中有说明过，可以通过 `sudo nixos-rebuild switch --flake .#xxx` 的 `--flake` 参数指定 Flakes 配置的文件夹路径，并通过 `#` 后面的值来指定使用的 outputs 名称。

比如我的使用方式是将 Nix Flakes 配置放在 `~/nixos-config` 目录下，然后在 `/etc/nixos` 目录下创建一个软链接：

```shell
sudo mv /etc/nixos /etc/nixos.bak  # 备份原来的配置
sudo ln -s ~/nixos-config/ /etc/nixos
```

然后就可以在 `~/nixos-config` 目录下使用 Git 管理配置了，配置使用普通的用户级别权限即可，不要求 owner 为 root.

另一种方法是直接删除掉 `/etc/nixos`，并在每次部署时指定配置文件路径：

```shell
sudo mv /etc/nixos /etc/nixos.bak  # 备份原来的配置
cd ~/nixos-config

# 通过 --flake .#nixos-test 参数，指定使用当前文件夹的 flake.nix，使用的 nixosConfiguraitons 名称为 nixos-test
sudo nixos-rebuild switch --flake .#nixos-test
```

两种方式都可以，看个人喜好。搞定之后，系统的回滚也变得非常简单，只需要切换到上一个 commit 即可：

```shell
cd ~/nixos-config
# 回滚到上一个 commit
git checkout HEAD^1
# 部署
sudo nixos-rebuild switch --flake .#nixos-test
```

Git 的更多操作这里就不介绍了，总之一般情况下的回滚都能直接通过 Git 完成，只在系统完全崩溃的情况下，才需要通过重启进入 grub，从上一个历史版本启动系统。

## 查看与清理历史数据 {#view-and-delete-history}

如前所述，NixOS 的每次部署都会生成一个新的版本，所有版本都会被添加到系统启动项中，除了重启电脑外，我们也可以通过如下命令查询当前可用的所有历史版本：

```shell
nix profile history --profile /nix/var/nix/profiles/system
```

以及清理历史版本释放存储空间的命令：

```shell
# 清理 7 天之前的所有历史版本
sudo nix profile wipe-history --profile /nix/var/nix/profiles/system  --older-than 7d
# 清理历史版本并不会删除数据，还需要手动 gc 下
sudo nix store gc --debug
```

以及查看系统层面安装的所有软件包（这个貌似只能用 `nix-env`）：

```shell
nix-env -qa
```