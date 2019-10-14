# oh my zsh 的安装

```shell
sudo apt install git zsh curl wget -y

sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
sudo chsh -s $(which zsh)
git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

echo 'plugins=(git zsh-autosuggestions zsh-syntax-highlighting)' # 到.zshrc

# oh-my-zsh 也支持svn git go golang 等

```
