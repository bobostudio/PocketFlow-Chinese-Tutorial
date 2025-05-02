# 环境安装与初始化配置

## 笔者开发环境

- Windows 10 WSL Ubuntu 22.04.5 LTS
- Python 3.10.12
- VSCode 1.99.3
- VSCode Extension: Python

## 安装 Python

```bash
sudo apt update
sudo apt upgrade
sudo apt install python3 python3.10-venv
```

## 新建虚拟环境

```bash
python3 -m venv p3 # 新建自己拟环境的名字
```

## 激活虚拟环境

```bash
source p3/bin/activate
```


## 安装 PocketFlow

```bash
pip install pocketflow openai PyYAML
```


## 教程使用的大模型

- DeepSeek


> 温馨提示：教程请保证在激活的虚拟环境 p3 下使用