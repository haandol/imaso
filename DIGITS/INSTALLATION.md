# DIGITS 설치가이드

AWS EC2 의 GPU 인스턴스(g2.2xlarge) 상에서 DIGITS 를 설치하기 위한 문서입니다.


# 인스턴스 설정

```bash
sudo apt-get update
sudo apt-get dist-upgrade
sudo apt-get install language-pack-ko
```

먼저 인스턴스의 기본 데이터를 업데이트하고 언어팩을 설치한다.


# CUDA 설치

기본적으로 필요한 라이브러리들 설치

```bash
sudo apt-get install -y gcc g++ gfortran build-essential git wget linux-image-generic libopenblas-dev python-dev python-pip python-nose python-numpy python-scipy
```

CUDA를 다운로드한다.

```bash
sudo wget http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1404/x86_64/cuda-repo-ubuntu1404_7.0-28_amd64.deb
sudo dpkg -i cuda-repo-ubuntu1404_7.0-28_amd64.deb
sudo apt-get update
sudo apt-get install cuda
```

라이브러리 패스와 CUDA 실행위치를 환경변수에 등록해준다.

```bash
echo -e "\nexport PATH=/usr/local/cuda/bin:$PATH" >> .bashrc
echo -e "\nexport LD_LIBRARY_PATH=/usr/local/cuda/lib64" >> .bashrc
```


# cuDNN 설치

cuDNN 라이브러리는 nVidia 의 CUDA registered developer program 에 가입후 하루정도 지나야 다운로드가 가능하다.

```bash
gzip -d cudnn-7.0-linux-x64-v3.0-rc.tgz
tar xf cudnn-7.0-linux-x64-v3.0-rc.tgz

# copy the library files into CUDA's include and lib folders
sudo cp cuda/include/* /usr/local/cuda-7.0/include
sudo cp cuda/lib64/* /usr/local/cuda-7.0/lib64
```

# Caffe 설치

필요한 라이브러리들을 설치한다.

```bash
sudo apt-get install libprotobuf-dev libleveldb-dev \
    libsnappy-dev libopencv-dev libboost-all-dev libhdf5-serial-dev \
    libgflags-dev libgoogle-glog-dev liblmdb-dev protobuf-compiler \
    libatlas-base-dev
```

Caffe 를 클로닝받고 필요한 Python 모듈들을 설치한다.

```bash
git clone https://github.com/NVIDIA/caffe.git

cd ~/caffe/python
for req in $(cat requirements.txt); do sudo pip install $req; done

cd ~/caffe
cp Makefile.config.example Makefile.config

echo -e "\nexport CAFFE_HOME=/home/ubuntu/caffe" >> ~/.bashrc
source ~/.bashrc
```

컴파일을 하고 잘 설치되었는지 테스트해본다.

```bash
make all --jobs=4
make py
make runtest
```

# DIGITS 설치

git을 통해 프로젝트를 클로닝 받는다.

```bash
cd ~
git clone https://github.com/NVIDIA/DIGITS.git digits
cd digits
```

필요한 파이썬 라이브러리들을 설치한다.

```bash
sudo apt-get install graphviz gunicorn
for req in $(cat requirements.txt); do sudo pip install $req; done
```

설치가 완료되면 폴더로 이동해 ***개발서버***를 실행한다.

```bash
cd digits

# start the server
./digits-devserver
```

## 문제해결

실행시 몇가지 문제가 발생할 수 있다. 자주 만나게 되는 알려진 문제해결방법을 공유한다.


```bash
"libdc1394 error: Failed to initialize libdc1394"

# no big deal - either ignore or treat symptomatically
sudo ln /dev/null /dev/raw1394
```


```bash
"Gtk-WARNING **: Locale not supported by C library."

# not sure how serious this is - but it is easy to resolve
sudo apt-get install language-pack-en-base
sudo dpkg-reconfigure locales

# check what locales are available and then ...
locale -a
# ... set LC_ALL to it
echo -e "\nexport LC_ALL=\"en_US.utf8\"" >> ~/.bashrc
```


```bash
"Gdk-CRITICAL **: gdk_cursor_new_for_display: assertion 'GDK_IS_DISPLAY (display)' failed"

# connect with ssh flags -X
ssh -X -i ec2.pem -l ubuntu digits-server-dns
```


```bash
"Couldn't import dot_parser, loading of dot files will not be possible."

# reinstall pyparsing:
sudo pip uninstall pyparsing
sudo pip install pyparsing==1.5.7
sudo pip install pydot
```
