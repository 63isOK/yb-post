# docker install

follow the office-doc,install docker-ce.
uninstall docker before install new docker.

## build a docker image for cpp dev

docker run -i -t --name=ubuntu  ubuntu:16.04

- install sudo

```shell
    apt update
    apt install sudo -y
```

- add user `yb`

```shell
    useradd -d /home/yb -m yb
    passwd yb // set password of yb:1
    usermod -aG sudo yb
    su yb
    sudo apt install git zsh -y
```

- build vim

```shell
    sudo apt install libncurses5-dev libgnome2-dev libgnomeui-dev \
      libgtk2.0-dev libatk1.0-dev libbonoboui2-dev \
      libcairo2-dev libx11-dev libxpm-dev libxt-dev python-dev \
      python3-dev ruby-dev lua5.1 liblua5.1-dev libperl-dev git -y

    sudo apt remove vim vim-runtime gvim
    sudo apt remove vim-tiny vim-common vim-gui-common vim-nox

    git clone https://github.com/vim/vim.git

    cd vim
    ./configure --with-features=huge \
                --enable-multibyte \
          --enable-rubyinterp=yes \
          --enable-pythoninterp=yes \
          --with-python-config-dir=/usr/lib/python2.7/config \
          # pay attention here check directory correct
          --enable-python3interp=yes \
          --with-python3-config-dir=/usr/lib/python3.5/config \
          --enable-perlinterp=yes \
          --enable-luainterp=yes \
                --enable-gui=gtk2 \
                --enable-cscope \
         --prefix=/usr/local

    make VIMRUNTIMEDIR=/usr/local/share/vim/vim81

    sudo make install

    sudo update-alternatives \
      --install /usr/bin/editor editor /usr/local/bin/vim 1
    sudo update-alternatives --set editor /usr/local/bin/vim
    sudo update-alternatives --install /usr/bin/vi vi /usr/local/bin/vim 1
    sudo update-alternatives --set vi /usr/local/bin/vim


```

- start

    docker start xxx  
    docker exec -u=yb -w=/home/yb -t -i ubuntu /usr/bin/zsh
