# Conda

## Install miniconda

https://docs.conda.io/projects/conda/en/latest/user-guide/install/linux.html

```
mkdir -p ~/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
rm -rf ~/miniconda3/miniconda.sh
~/miniconda3/bin/conda init bash
```

## Update conda

```
conda update conda.
```

## create env

Spezifische python version:

```
conda create --name <my-env> python=3.12
```

Spezifische python version und packages
```
conda create -n myenv python=3.9 scipy=0.17.3 astroid babel
```

