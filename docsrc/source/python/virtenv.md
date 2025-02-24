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


# venv

fedora: install pip
`sudo dnf install python3-pip`

## install
```
python3 -m venv project_venv
source project_venv/bin/activate
python -m pip install requests
deactivate
```

## delete
`rm -rv project_venv`

## update
`python -m pip install --update requests`



# Pip

## SSL Zertifikatswarnung 
https://jhooq.com/pip-install-connection-error/

```
WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1000)'))': /packages/68/1b/e0a87d256e40e8c888847551b20a017a6b98139178505dc7ffb96f04e954/dnspython-2.7.0-py3-none-any.whl.metadata
```

`pip install --trusted-host pypi.org --trusted-host files.pythonhosted.org <packet>`

alternativ

$HOME/.config/pip/pip.conf
```
[global]
trusted-host = pypi.python.org
               pypi.org
               files.pythonhosted.org
```

