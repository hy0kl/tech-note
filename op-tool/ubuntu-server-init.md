# 标准初始化

```
apt update
apt upgrade
```

## 一定得安装

```
apt install autoconf libtool cmake unzip pkg-config pkgconf libsqlite3-dev sqlite3 silversearcher-ag htop vim ctags tree
apt install openssl libssl-dev
apt install git dos2unix mercurial libpcre3-dev libxml2-dev libxslt1-dev libreadline-dev
```

## 选择性安装

```
apt install libjpeg-dev libbz2-dev libfreetype6-dev libtiff5-dev libwebp-dev
```

# SS

```
apt install libsodium-dev
```

# 音乐相关

```
sudo apt-get install ffmpeg libfftw3-dev libsndfile1-dev timidity
```

## midi 转 mp3

```
timidity m.mid -Ow -o - | ffmpeg -i - -acodec libmp3lame -ab 64k m.mp3
```

# gui

```
libao-dev libsamplerate0-dev libcurses-ocaml-dev libgtk2.0-dev
```


# To add the PPA if you don't already have it:

```
sudo add-apt-repository ppa:mscore-ubuntu/mscore-stable
```

# To install or upgrade MuseScore:

```
sudo apt update
sudo apt install musescore
```

